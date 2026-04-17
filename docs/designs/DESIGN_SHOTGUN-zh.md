# 设计：Design Shotgun — Browser-to-Agent 反馈循环

生成日期：2026-03-27
分支：garrytan/agent-design-tools
状态：动态文档 — 随着 bug 的发现和修复而更新

## 这个功能做什么

Design Shotgun 生成多个 AI 设计稿，在用户的真实浏览器中并排打开为一个比较面板（comparison board），并收集结构化反馈（选择最喜欢的、评分替代品类型、留言、请求重新生成）。反馈流回编码 agent，agent 据此操作它：要么用已批准的方案继续，要么生成新的方案并重新加载比较面板。

用户无需离开浏览器标签页。agent 不再问冗余的问题。比较面板就是反馈机制本身。

## 核心问题：两个世界必须对话

```
  ┌─────────────────────┐          ┌──────────────────────┐
  │   用户的浏览器        │          │   编码 AGENT          │
  │   (真实 Chrome)       │          │   (Claude Code /     │
  │                     │          │    Conductor)         │
  │  比较面板             │          │                      │
  │  带按钮：              │          │  需要知道：             │
  │  - Submit            │   ???    │  - 选了什么           │
  │  - Regenerate        │ ──────── │  - 星级评分           │
  │  - More Like This    │          │  - 评论               │
  │  - Remix             │          │  - 请求重新生成？       │
  └─────────────────────┘          └──────────────────────┘
```

"???" 是难点。用户在 Chrome 中点击按钮。跑在终端里的 agent 需要知道这件事。这是两个完全独立的进程，没有共享内存、没有共享事件总线、没有 WebSocket 连接。

## 架构：链接如何工作

```
  用户的浏览器                    $D serve (Bun HTTP)              AGENT
  ═══════════════                   ═══════════════════              ═════
        │                                   │                           │
        │  GET /                            │                           │
        │ ◄─────── 返回面板 HTML ───────────│                           │
        │    (带 __GSTACK_SERVER_URL        │                           │
        │     注入到 <head>)                │                           │
        │                                   │                           │
        │  [用户评分、选择、评论]             │                           │
        │                                   │                           │
        │  POST /api/feedback               │                           │
        │ ────── {preferred:"A",...} ──────►│                           │
        │                                   │                           │
        │  ◄── {received:true} ────────────│                           │
        │                                   │── 写入 feedback.json ────►│
        │  [输入已禁用，显示                  │   (或 feedback-pending    │
        │   "Return to agent"]              │    .json 表示重新生成)     │
        │                                   │                           │
        │                                   │                  [agent 每
        │                                   │                   5秒轮询，
        │                                   │                   读文件]
```

### 三个文件

| 文件 | 何时写入 | 意味着 | Agent 操作 |
|------|---------|-------|----------|
| `feedback.json` | 用户点击 Submit | 最终选择，完成 | 读取并继续 |
| `feedback-pending.json` | 用户点击 Regenerate/More Like This | 想要新方案 | 读取、删除它、生成新方案、重新加载面板 |
| `feedback.json` (第2轮+) | 用户重新生成后点击 Submit | 迭代后的最终选择 | 读取并继续 |

### 状态机

```
   $D serve 启动
        │
        ▼
   ┌──────────┐
   │ SERVING  │◄──────────────────────────────────────┐
   │          │                                        │
   │ 面板在线， │  POST /api/feedback                    │
   │ 等待反馈   │  {regenerated: true}                   │
   │          │──────────────────►┌──────────────┐     │
   │          │                   │ REGENERATING │     │
   │          │                   │              │     │
   └────┬─────┘                   │ Agent 有     │     │
        │                         │ 10 分钟来     │     │
        │  POST /api/feedback     │ POST 新       │     │
        │  {regenerated: false}   │ 面板 HTML     │     │
        │                         └──────┬───────┘     │
        ▼                                │             │
   ┌──────────┐                POST /api/reload        │
   │  DONE    │                {html: "/new/board"}    │
   │          │                          │             │
   │ exit 0   │                          ▼             │
   └──────────┘                   ┌──────────────┐     │
                                  │  RELOADING   │─────┘
                                  │              │
                                  │ 面板自动刷新   │
                                  │ (同一标签页)   │
                                  └──────────────┘
```

### 端口发现

Agent 在后台运行 `$D serve` 并从 stderr 读取端口号：

```
SERVE_STARTED: port=54321 html=/path/to/board.html
SERVE_BROWSER_OPENED: url=http://127.0.0.1:54321
```

Agent 从 stderr 解析 `port=XXXXX`。此端口后续用于 POST `/api/reload`（用户请求重新生成时）。如果 agent 丢失了端口号，就无法重新加载面板。

### 为什么用 127.0.0.1 而不是 localhost

在某些系统上 `localhost` 可能解析为 IPv6 `::1`，而 Bun.serve() 只监听 IPv4。更重要的是，`localhost` 会把开发者工作过的所有域名的开发 cookie 都带上。在一台有很多活跃 session 的机器上，这会超出 Bun 的默认 header 大小限制（HTTP 431 错误）。`127.0.0.1` 避免了两类问题。

## 每个边界情况和陷阱

### 1. 僵尸表单问题（Zombie Form Problem）

**现象：** 用户提交反馈，POST 成功，服务器退出。但 HTML 页面还开着。它看起来是可交互的。用户可能会编辑反馈并再次点击 Submit。什么都不会发生，因为服务器已经不在了。

**修复：** 成功 POST 后，面板 JS 会：
- 禁用所有输入（按钮、单选、文本域、星级评分）
- 完全隐藏 Regenerate 栏
- 将 Submit 按钮替换为："Feedback received! Return to your coding agent."
- 显示："Want to make more changes? Run `/design-shotgun` again."
- 页面变为已提交内容的只读记录

**实现于：** `compare.ts:showPostSubmitState()`（第 484 行）

### 2. 死亡服务器问题（Dead Server Problem）

**现象：** 服务器超时（默认 10 分钟）或在用户还开着面板时崩溃。用户点击 Submit。fetch() 静默失败。

**修复：** `postFeedback()` 函数有 `.catch()` 处理。网络失败时：
- 显示红色错误横幅："Connection lost"
- 在可复制的 `<pre>` 块中展示收集的反馈 JSON
- 用户可以直接复制粘贴到编码 agent

**实现于：** `compare.ts:showPostFailure()`（第 546 行）

### 3. 过期的重新生成旋转器（Stale Regeneration Spinner）

**现象：** 用户点击 Regenerate。面板显示旋转器并每 2 秒轮询 `/api/progress`。Agent 崩溃或生成新方案耗时过长。旋转器无限旋转。

**修复：** Progress 轮询有一个硬性 5 分钟超时（150 次轮询 x 2 秒间隔）。5 分钟后：
- 旋转器替换为："Something went wrong."
- 显示："Run `/design-shotgun` again in your coding agent."
- 轮询停止。页面变为信息性质。

**实现于：** `compare.ts:startProgressPolling()`（第 511 行）

### 4. file:// URL 问题（原始 BUG）

**现象：** 技能模板最初使用 `$B goto file:///path/to/board.html`。但 `browse/src/url-validation.ts:71` 出于安全原因阻止 `file://` URL。退路的 `open file://...` 打开用户的 macOS 浏览器，而 `$B eval` 轮询的是 Playwright 的无头浏览器（不同进程，从未加载这个页面）。Agent 无限轮询空 DOM。

**修复：** `$D serve` 通过 HTTP 提供服务。面板永远不要用 `file://`。`$D compare` 的 `--serve` 标志将面板生成和 HTTP 服务合并在一个命令中。

**证据：** 见 `.context/attachments/image-v2.png`——一个真实用户遇到了这个完全一样的 bug。Agent 正确诊断了：(1) `$B goto` 拒绝 `file://` URL，(2) 即使有 browse daemon 也没有轮询循环。

### 5. 双击竞态（Double-Click Race）

**现象：** 用户快速点击 Submit 两次。两个 POST 请求到达服务器。第一个将状态设为 "done" 并安排 100ms 后 exit(0)。第二个在该 100ms 窗口内到达。

**当前状态：** 未完全防护。`handleFeedback()` 函数在处理前不检查状态是否已经是 "done"。第二个 POST 会成功并写入第二个 `feedback.json`（无害，数据相同）。exit 仍然在 100ms 后触发。

**风险：** 低。面板在第一次成功 POST 响应后禁用所有输入，所以第二次点击需要在约 1ms 内到达。而且两次写入包含相同的反馈数据。

**潜在修复：** 在 `handleFeedback()` 顶部添加 `if (state === 'done') return Response.json({error: 'already submitted'}, {status: 409})`。

### 6. 端口协调问题（Port Coordination Problem）

**现象：** Agent 后台运行 `$D serve` 后从 stderr 解析 `port=54321`。Agent 后续需要在重新生成时 POST `/api/reload`。如果 agent 丢失上下文（对话被压缩、上下文窗口填满），它可能不记得端口号了。

**当前状态：** 端口只打印到 stderr 一次。Agent 必须记住它。没有端口文件写入磁盘。

**潜在修复：** 启动时在面板 HTML 旁边写入 `serve.pid` 或 `serve.port` 文件。Agent 随时可读取：
```bash
cat "$_DESIGN_DIR/serve.port"  # → 54321
```

### 7. 反馈文件清理问题（Feedback File Cleanup Problem）

**现象：** 重新生成轮次的 `feedback-pending.json` 留在磁盘上。如果 agent 在读它之前崩溃，下一个 `$D serve` 会话会发现一个过期的文件。

**当前状态：** 解析器模板中的轮询循环说要在读取后删除 `feedback-pending.json`。但这依赖于 agent 完美地遵循指令。过期文件可能混淆新的会话。

**潜在修复：** `$D serve` 可以在启动时检查并删除过期的反馈文件。或者：用时间戳命名文件（`feedback-pending-1711555200.json`）。

### 8. 顺序生成规则

**现象：** 底层的 OpenAI GPT Image API 对并发的图片生成请求进行速率限制。当 3 个 `$D generate` 调用并行运行时，1 个成功而 2 个被中止。

**修复：** 技能模板必须明确说："ONE AT A TIME 生成 mockup。不要并行化 `$D generate` 调用。"这是 prompt 级别的指令，不是代码级别的锁。设计二进制文件不强制顺序执行。

**风险：** Agent 被训练为并行化独立工作。没有明确指令的话，它们会尝试同时运行 3 个生成。浪费 API 调用和金钱。

### 9. AskUserQuestion 冗余

**现象：** 用户通过面板提交反馈后（包含首选方案、评分、评论都在 JSON 中），agent 又问他们："你最喜欢哪个方案？"这很烦人。面板存在的意义就是避免这个问题。

**修复：** 技能模板必须说："不要使用 AskUserQuestion 询问用户的偏好。读取 `feedback.json`，里面包含了他们的选择。只在确认理解是否正确时使用 AskUserQuestion，而不是重新问。"

### 10. CORS 问题

**现象：** 如果面板 HTML 引用了外部资源（字体、CDN 图片），浏览器会发送带 `Origin: http://127.0.0.1:PORT` 的请求。大多数 CDN 允许这个，但有些可能阻止。

**当前状态：** 服务器未设置 CORS 头。面板 HTML 是自包含的（图片 base64 编码、样式内联），实践中这还没成为过问题。

**风险：** 对当前设计来说很低。如果面板加载外部资源才会有意义。

### 11. 大载荷问题（Large Payload Problem）

**现象：** POST 到 `/api/feedback` 的 body 没有大小限制。如果面板发送多 MB 的载荷，`req.json()` 会把所有内容解析到内存中。

**当前状态：** 实践中反馈 JSON 约 500 字节到 2KB。风险是理论上的而非实际的。面板 JS 构建的是固定形状的 JSON 对象。

### 12. fs.writeFileSync 错误

**现象：** `serve.ts:138` 中的 `feedback.json` 写入使用 `fs.writeFileSync()` 且无 try/catch。如果磁盘满了或目录只读，会抛出异常并使服务器崩溃。用户看到永远旋转的加载器（服务器死了，但面板不知道）。

**风险：** 实践中很低（面板 HTML 刚被写入同一目录，证明它是可写的）。但有 try/catch 并在失败时返回 500 会更干净。

## 完整流程（逐步）

### Happy Path：用户一次就选中

```
1. Agent 运行：$D compare --images "A.png,B.png,C.png" --output board.html --serve &
2. $D serve 在随机端口（如 54321）启动 Bun.serve()
3. $D serve 在用户浏览器中打开 http://127.0.0.1:54321
4. $D serve 打印到 stderr：SERVE_STARTED: port=54321 html=/path/board.html
5. $D serve 写入带注入 __GSTACK_SERVER_URL 的面板 HTML
6. 用户看到并排 3 个方案的比较面板
7. 用户选择方案 B，评分 A: 3/5, B: 5/5, C: 2/5
8. 用户在整体反馈中写 "B 的间距更好，就用它"
9. 用户点击 Submit
10. 面板 JS POST 到 http://127.0.0.1:54321/api/feedback
    Body: {"preferred":"B","ratings":{"A":3,"B":5,"C":2},"overall":"B 的间距更好","regenerated":false}
11. 服务器将 feedback.json 写入磁盘（与 board.html 相邻）
12. 服务器将反馈 JSON 打印到 stdout
13. 服务器响应 {received:true, action:"submitted"}
14. 面板禁用所有输入，显示 "Return to your coding agent"
15. 服务器在 100ms 后以代码 0 退出
16. Agent 的轮询循环找到 feedback.json
17. Agent 读取它、向用户摘要、继续
```

### 重新生成路径：用户想要不同的方案

```
1-6.  同上面 Happy Path
7.  用户点击 "Totally different" chiclet
8.  用户点击 Regenerate
9.  面板 JS POST 到 /api/feedback
    Body: {"regenerated":true,"regenerateAction":"different","preferred":"","ratings":{},...}
10. 服务器将 feedback-pending.json 写入磁盘
11. 服务器状态 → "regenerating"
12. 服务器响应 {received:true, action:"regenerate"}
13. 面板显示旋转器："Generating new designs..."
14. 面板开始每 2 秒轮询 GET /api/progress

    同时，在 agent 中：
15. Agent 的轮询循环找到 feedback-pending.json
16. Agent 读取它、删除它
17. Agent 运行：$D variants --brief "totally different direction" --count 3
    （ONE AT A TIME，不要并行）
18. Agent 运行：$D compare --images "new-A.png,new-B.png,new-C.png" --output board-v2.html
19. Agent POST：curl -X POST http://127.0.0.1:54321/api/reload -d '{"html":"/path/board-v2.html"}'
20. 服务器将 htmlContent 切换到新面板
21. 服务器状态 → "serving"（从 reloading 转来）
22. 面板的下次 /api/progress 轮询返回 {"status":"serving"}
23. 面板自动刷新：window.location.reload()
24. 用户看到 3 个新方案的比较面板
25. 用户选中一个、点击 Submit → 走 Happy Path 的第 10 步开始
```

### "More Like This" 路径

```
同重新生成，区别在于：
- regenerateAction 为 "more_like_B"（引用方案）
- Agent 用 $D iterate --image B.png --brief "more like this, keep the spacing"
  代替 $D variants
```

### 降级路径：$D serve 失败

```
1. Agent 尝试 $D compare --serve，失败（二进制文件缺失、端口错误等）
2. Agent 降级为：open file:///path/board.html
3. Agent 用 AskUserQuestion："我已打开设计面板。你最喜欢哪个方案？有什么反馈？"
4. 用户通过文本回复
5. Agent 使用文本反馈继续（无结构化 JSON）
```

## 实现这些的文件

| 文件 | 角色 |
|------|------|
| `design/src/serve.ts` | HTTP 服务器、状态机、文件写入、浏览器启动 |
| `design/src/compare.ts` | 面板 HTML 生成、评分/选择/重新生成 JS、POST 逻辑、提交后生命周期 |
| `design/src/cli.ts` | CLI 入口点，连接 `serve` 和 `compare --serve` 命令 |
| `design/src/commands.ts` | 命令注册表，定义 `serve` 和 `compare` 及其参数 |
| `scripts/resolvers/design.ts` | `generateDesignShotgunLoop()` — 模板解析器，输出轮询循环和重新加载指令 |
| `design-shotgun/SKILL.md.tmpl` | 技能模板，编排完整流程：上下文收集、方案生成、`{{DESIGN_SHOTGUN_LOOP}}`、反馈确认 |
| `design/test/serve.test.ts` | HTTP 端点和状态转换的单元测试 |
| `design/test/feedback-roundtrip.test.ts` | E2E 测试：浏览器点击 → JS fetch → HTTP POST → 磁盘上的文件 |
| `browse/test/compare-board.test.ts` | 比较面板 UI 的 DOM 级测试 |

## 还可能出什么问题

### 已知风险（按可能性排序）

1. **Agent 不遵守顺序生成规则** — 大多数 LLM 想要并行化。二元文件中没有强制机制时，这只能是可能被打断的 prompt 级别指令。

2. **Agent 丢失端口号** — 上下文压缩会丢弃 stderr 输出。Agent 无法重新加载面板。缓解方案：将端口写入文件。

3. **过期反馈文件** — 崩溃会话遗留的 `feedback-pending.json` 混淆下一次运行。缓解方案：启动时清理。

4. **fs.writeFileSync 崩溃** — 反馈文件写入没有 try/catch。磁盘满时服务器静默死亡。用户看到无限旋转器。

5. **Progress 轮询漂移** — 5 分钟内 `setInterval(fn, 2000)`。实践中 JavaScript 定时器精度够用。但如果浏览器标签页被放入后台，Chrome 可能将间隔节流到每分钟一次。

### 哪些部分工作得很好

1. **双通道反馈** — 前台用 stdout，后台用文件。两者总是激活。Agent 可以用任意一个可行通道。

2. **自包含 HTML** — 面板内联所有 CSS、JS 和 base64 编码图片。无外部依赖。离线可用。

3. **同一标签页重新生成** — 用户不切标签页。面板通过 `/api/progress` 轮询 + `window.location.reload()` 自动刷新。不会标签页爆炸。

4. **优雅降级** — POST 失败展示可复制 JSON。Progress 超时展示清晰的错误消息。没有静默失败。

5. **提交后生命周期** — 面板提交后变为只读。没有僵尸表单。清晰的"下一步做什么"提示。

## 测试覆盖

### 已测试的内容

| 流程 | 测试 | 文件 |
|------|------|------|
| Submit → feedback.json 写入磁盘 | 浏览器点击 → 文件 | `feedback-roundtrip.test.ts` |
| 提交后 UI 锁定 | 输入禁用、成功提示 | `feedback-roundtrip.test.ts` |
| Regenerate → feedback-pending.json | chiclet + regen 点击 → 文件 | `feedback-roundtrip.test.ts` |
| "More like this" → 具体操作 | more_like_B 在 JSON 中 | `feedback-roundtrip.test.ts` |
| 重新生成后显示旋转器 | DOM 显示加载文本 | `feedback-roundtrip.test.ts` |
| 完整重新生成 → 重新加载 → 提交 | 两轮往返 | `feedback-roundtrip.test.ts` |
| 服务器在随机端口启动 | 端口 0 绑定 | `serve.test.ts` |
| HTML 注入服务器 URL | __GSTACK_SERVER_URL 检查 | `serve.test.ts` |
| 无效 JSON 被拒绝 | 400 响应 | `serve.test.ts` |
| HTML 文件验证 | 缺失时 exit 1 | `serve.test.ts` |
| 超时行为 | 超时后 exit 1 | `serve.test.ts` |
| 面板 DOM 结构 | 单选框、星标、chiclet | `compare-board.test.ts` |

### 未测试的内容

| 差距 | 风险 | 优先级 |
|-----|------|--------|
| 双击提交竞态 | 低 — 首次响应后输入禁用 | P3 |
| Progress 轮询超时（150 次迭代） | 中 — 在测试中等待 5 分钟太长 | P2 |
| 重新生成期间服务器崩溃 | 中 — 用户看到无限旋转器 | P2 |
| POST 期间网络超时 | 低 — localhost 很快 | P3 |
| 放入后台的 Chrome 标签页节流间隔 | 中 — 可能需要将 5 分钟超时扩展到 30+ 分钟 | P2 |
| 大反馈载荷 | 低 — 面板构建固定形状 JSON | P3 |
| 并发会话（两个面板、一个服务器） | 低 — 每个 $D serve 有自己的端口 | P3 |
| 前一个会话的过期反馈文件 | 中 — 可能混淆新的轮询循环 | P2 |

## 潜在改进

### 短期（当前分支）

1. **将端口写入文件** — `serve.ts` 启动时将 `serve.port` 写入磁盘。Agent 随时可读取。5 行代码。
2. **启动时清理过期文件** — `serve.ts` 启动前删除 `feedback*.json`。3 行代码。
3. **防护双击** — 在 `handleFeedback()` 顶部检查 `state === 'done'`。2 行代码。
4. **文件写入的 try/catch** — 将 `fs.writeFileSync` 包裹在 try/catch 中，失败时返回 500。5 行代码。

### 中期（后续）

5. **WebSocket 替代轮询** — 用 WebSocket 连接替代 `setInterval` + `GET /api/progress`。面板在新 HTML 就绪时瞬间收到通知。消除轮询漂移和后台标签页节流。serve.ts 约 50 行 + compare.ts 约 20 行。

6. **Agent 的端口文件** — 将 `{"port": 54321, "pid": 12345, "html": "/path/board.html"}` 写入 `$_DESIGN_DIR/serve.json`。Agent 读这个而不解析 stderr。使系统对上下文丢失更健壮。

7. **反馈 schema 验证** — 写入前根据 JSON schema 验证 POST body。尽早捕获格式错误的反馈，而不是在下游混淆 agent。

### 长期（设计方向）

8. **持久化设计服务器** — 不每次启动 `$D serve`，而是运行长期设计守护进程（类似 browse daemon）。多个面板共享一个服务器。消除冷启动。但增加了守护进程生命周期管理的复杂性。

9. **实时协作** — 两个 agent（或一个 agent + 一个人）同时工作在同一个面板上。服务器通过 WebSocket 广播状态变更。需要反馈的冲突解决机制。
