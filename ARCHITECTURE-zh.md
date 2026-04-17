# 架构

本文档解释 gstack **为什么**以这样的方式构建。关于设置和命令，参见 CLAUDE.md。关于贡献，参见 CONTRIBUTING.md。

## 核心理念

gstack 为 Claude Code 提供一个持久浏览器和一组有观点的工作流 Skill。浏览器是最难的部分——其他一切都是 Markdown。

关键洞察：与浏览器交互的 AI Agent 需要**亚秒级延迟**和**持久状态**。如果每条命令都冷启动浏览器，每个工具调用要等 3-5 秒。如果浏览器在命令之间挂了，你会丢失 Cookie、标签和登录会话。所以 gstack 运行一个长期存活的 Chromium 守护进程，CLI 通过 localhost HTTP 与之通信。

```
Claude Code                     gstack
─────────                      ──────
                               ┌──────────────────────┐
  Tool call: $B snapshot -i    │  CLI（编译后的二进制文件）│
  ─────────────────────────→   │  • 读取状态文件   │
                               │  • POST /command      │
                               │    到 localhost:PORT   │
                               └──────────┬───────────┘
                                          │ HTTP
                               ┌──────────▼───────────┐
                               │  Server (Bun.serve)   │
                               │  • 分派命令  │
                               │  • 与 Chromium 通信   │
                               │  • 返回纯文本  │
                               └──────────┬───────────┘
                                          │ CDP
                               ┌──────────▼───────────┐
                               │  Chromium（无头）   │
                               │  • 持久标签     │
                               │  • Cookie 跨命令保留  │
                               │  • 30 分钟空闲超时  │
                               └───────────────────────┘
```

首次调用启动一切（约 3 秒）。此后每次调用：约 100-200ms。

## 为什么选择 Bun

Node.js 可以工作。但 Bun 在这里更好，原因有三：

1. **编译二进制文件。** `bun build --compile` 生成单个约 58MB 的可执行文件。运行时不需要 `node_modules`，不需要 `npx`，不需要 PATH 配置。二进制文件直接运行。这很重要，因为 gstack 安装到 `~/.claude/skills/`，用户不期望在那里管理 Node.js 项目。

2. **原生 SQLite。** Cookie 解密直接读取 Chromium 的 SQLite Cookie 数据库。Bun 内置 `new Database()`——不需要 `better-sqlite3`，不需要原生附加组件编译，不需要 gyp。少一个在不同机器上会坏的东西。

3. **原生 TypeScript。** 服务器在开发期间以 `bun run server.ts` 运行。不需要编译步骤，不需要 `ts-node`，不需要调试 source map。编译后的二进制文件用于部署；源文件用于开发。

4. **内置 HTTP 服务器。** `Bun.serve()` 快速、简单，不需要 Express 或 Fastify。服务器总共处理约 10 个路由。使用框架反而是负担。

瓶颈始终是 Chromium，而不是 CLI 或服务器。Bun 的启动速度（编译后的二进制文件约 1ms vs Node 约 100ms）不错，但不是我们选择它的原因。编译后的二进制文件和原生 SQLite 才是。

## 守护进程模型

### 为什么不在每条命令时启动浏览器？

Playwright 可以在约 2-3 秒内启动 Chromium。对于单次截屏没问题。但对于有 20+ 条命令的 QA 会话，就是 40+ 秒的浏览器启动开销。更糟的是：命令之间丢失所有状态。Cookie、localStorage、登录会话、打开的标签——全没了。

守护进程模型意味着：

- **持久状态。** 登录一次，保持登录。打开一个标签，它保持打开。localStorage 跨命令持久化。
- **亚秒级命令。** 首次调用后，每条命令只是一个 HTTP POST。约 100-200ms 往返，包括 Chromium 的工作。
- **自动生命周期。** 服务器首次使用时自动启动，空闲 30 分钟后自动关闭。不需要进程管理。

### 状态文件

服务器写入 `.gstack/browse.json`（通过 tmp + rename 原子写入，权限 0o600）：

```json
{ "pid": 12345, "port": 34567, "token": "uuid-v4", "startedAt": "...", "binaryVersion": "abc123" }
```

CLI 读取这个文件来找到服务器。如果文件缺失或服务器未通过 HTTP 健康检查，CLI 会生成新服务器。在 Windows 上，Bun 二进制文件中基于 PID 的进程检测不可靠，所以健康检查（GET /health）是所有平台上的主要存活信号。

### 端口选择

10000-60000 之间的随机端口（碰撞时最多重试 5 次）。这意味着 10 个 Conductor 工作区可以各自运行自己的浏览守护进程，零配置、零端口冲突。旧方法（扫描 9400-9409）在多工作区设置中频繁出问题。

### 版本自动重启

构建将 `git rev-parse HEAD` 写入 `browse/dist/.version`。每次 CLI 调用时，如果二进制文件的版本与运行中服务器的 `binaryVersion` 不匹配，CLI 会杀死旧服务器并启动新的。这完全防止了"过时二进制文件"类 Bug——重新构建二进制文件，下一条命令自动使用新版本。

## 安全模型

### 仅 localhost

HTTP 服务器绑定到 `localhost`，不是 `0.0.0.0`。无法从网络访问。

### Bearer Token 认证

每个服务器会话生成一个随机 UUID 令牌，以 0o600 权限（仅所有者可读）写入状态文件。每个 HTTP 请求必须包含 `Authorization: Bearer <token>`。如果令牌不匹配，服务器返回 401。

这防止同一机器上的其他进程与你的浏览服务器通信。Cookie 选择器 UI（`/cookie-picker`）和健康检查（`/health`）例外——它们仅限 localhost 且不执行命令。

### Cookie 安全

Cookie 是 gstack 处理的最敏感数据。设计如下：

1. **Keychain 访问需要用户批准。** 每个浏览器的首次 Cookie 导入触发 macOS Keychain 对话框。用户必须点击"允许"或"始终允许"。gstack 从不静默访问凭据。

2. **解密在进程中进行。** Cookie 值在内存中解密（PBKDF2 + AES-128-CBC），加载到 Playwright 上下文中，绝不以明文写入磁盘。Cookie 选择器 UI 从不显示 Cookie 值——仅显示域名和数量。

3. **数据库是只读的。** gstack 将 Chromium Cookie DB 复制到临时文件（避免与运行中的浏览器发生 SQLite 锁冲突）并以只读方式打开。它从不修改你真实浏览器的 Cookie 数据库。

4. **密钥缓存是基于会话的。** Keychain 密码 + 派生的 AES 密钥在服务器生命周期内缓存在内存中。当服务器关闭（空闲超时或显式停止）时，缓存消失。

5. **日志中不包含 Cookie 值。** 控制台、网络和对话框日志从不包含 Cookie 值。`cookies` 命令输出 Cookie 元数据（域名、名称、过期时间），但值被截断。

### Shell 注入防护

浏览器注册表（Comet、Chrome、Arc、Brave、Edge）是硬编码的。数据库路径从已知常量构建，从不来自用户输入。Keychain 访问使用 `Bun.spawn()` 和显式参数数组，不使用 Shell 字符串插值。

## Ref 系统

Refs（`@e1`、`@e2`、`@c1`）是 Agent 在不编写 CSS 选择器或 XPath 的情况下定位页面元素的方式。

### 工作原理

```
1. Agent 运行：$B snapshot -i
2. 服务器调用 Playwright 的 page.accessibility.snapshot()
3. 解析器遍历 ARIA 树，分配顺序 refs：@e1, @e2, @e3...
4. 对每个 ref，构建 Playwright Locator：getByRole(role, { name }).nth(index)
5. 将 Map<string, RefEntry> 存储在 BrowserManager 实例上（role + name + Locator）
6. 返回带注解的树作为纯文本

后来：
7. Agent 运行：$B click @e3
8. 服务器解析 @e3 → Locator → locator.click()
```

### 为什么用 Locators，不用 DOM 变更

显而易见的方法是向 DOM 注入 `data-ref="@e1"` 属性。这会在以下情况崩溃：

- **CSP（内容安全策略）。** 许多生产站点阻止脚本修改 DOM。
- **React/Vue/Svelte 水合。** 框架协调可以剥离注入的属性。
- **Shadow DOM。** 无法从外部进入 Shadow Root 内部。

Playwright Locators 在 DOM 之外工作。它们使用可访问性树（Chromium 在内部维护）和 `getByRole()` 查询。无需 DOM 修改、无需 CSP 问题、无需框架冲突。

### Ref 生命周期

Refs 在导航时清除（主帧的 `framenavigated` 事件）。这是正确的——导航后所有 Locators 都过时了。Agent 必须再次运行 `snapshot` 获取新的 refs。这是设计使然：过期的 ref 应该明确失败，而不是点击错误的元素。

### Ref 过期检测

SPA 可以在不触发 `framenavigated` 的情况下修改 DOM（例如 React Router 转换、标签切换、模态框打开）。这使 refs 过期，尽管页面 URL 没有改变。为了捕获这一点，`resolveRef()` 在使用任何 ref 之前执行异步 `count()` 检查：

```
resolveRef(@e3) → entry = refMap.get("e3")
                → count = await entry.locator.count()
                → 如果 count === 0：抛出 "Ref @e3 is stale — element no longer exists. Run 'snapshot' to get fresh refs."
                → 如果 count > 0：返回 { locator }
```

这会快速失败（约 5ms 开销），而不是等待 Playwright 的 30 秒操作超时在缺失元素上过期。`RefEntry` 在 Locator 旁边存储 `role` 和 `name` 元数据，以便错误消息可以告诉 Agent 元素是什么。

### 光标交互式 refs（@c）

`-C` 标志查找可点击但不在 ARIA 树中的元素——带有 `cursor: pointer` 样式的元素、带有 `onclick` 属性的元素或自定义 `tabindex`。它们在单独的命名空间中获得 `@c1`、`@c2` refs。这捕获框架渲染为 `<div>` 但实际上是按钮的自定义组件。

## 日志架构

三个环形缓冲区（每个 50,000 条目，O(1) 推入）：

```
浏览器事件 → CircularBuffer（内存中）→ 异步刷新到 .gstack/*.log
```

控制台消息、网络请求和对话框事件各有自己的缓冲区。刷新每 1 秒执行一次——服务器仅追加自上次刷新以来的新条目。这意味着：

- HTTP 请求处理永远不会被磁盘 I/O 阻塞
- 日志在服务器崩溃后仍可存活（最多 1 秒数据丢失）
- 内存有界（50K 条目 × 3 个缓冲区）
- 磁盘文件仅追加，可被外部工具读取

`console`、`network` 和 `dialog` 命令从内存缓冲区读取，不是磁盘。磁盘文件用于事后调试。

## SKILL.md 模板系统

### 问题

SKILL.md 文件告诉 Claude 如何使用浏览命令。如果文档列出了不存在的标志，或遗漏了已添加的命令，Agent 会遇到错误。手工维护的文档总是与代码漂移。

### 解决方案

```
SKILL.md.tmpl          （人工编写的散文 + 占位符）
        ↓
gen-skill-docs.ts      （读取源代码元数据）
        ↓
SKILL.md               （已提交，自动生成部分）
```

模板包含需要人工判断的工作流、提示和示例。占位符在构建时从源代码填充：

| 占位符 | 来源 | 生成内容 |
|-------------|--------|-------------------|
| `{{COMMAND_REFERENCE}}` | `commands.ts` | 分类的命令表 |
| `{{SNAPSHOT_FLAGS}}` | `snapshot.ts` | 带示例的标志参考 |
| `{{PREAMBLE}}` | `gen-skill-docs.ts` | 启动块：更新检查、会话追踪、贡献者模式、AskUserQuestion 格式 |
| `{{BROWSE_SETUP}}` | `gen-skill-docs.ts` | 二进制发现 + 设置说明 |
| `{{BASE_BRANCH_DETECT}}` | `gen-skill-docs.ts` | 动态基础分支检测，用于 PR 定向 Skills（ship、review、qa、plan-ceo-review） |
| `{{QA_METHODOLOGY}}` | `gen-skill-docs.ts` | /qa 和 /qa-only 共享的 QA 方法论块 |
| `{{DESIGN_METHODOLOGY}}` | `gen-skill-docs.ts` | /plan-design-review 和 /design-review 共享的设计审计方法论 |
| `{{REVIEW_DASHBOARD}}` | `gen-skill-docs.ts` | /ship 预检的审查就绪仪表板 |
| `{{TEST_BOOTSTRAP}}` | `gen-skill-docs.ts` | 测试框架检测、引导、CI/CD 设置，用于 /qa、/ship、/design-review |
| `{{CODEX_PLAN_REVIEW}}` | `gen-skill-docs.ts` | 可选的跨模型计划审查（Codex 或 Claude 子Agent回退），用于 /plan-ceo-review 和 /plan-eng-review |
| `{{DESIGN_SETUP}}` | `resolvers/design.ts` | $D 设计二进制的发现模式，镜像 `{{BROWSE_SETUP}}` |
| `{{DESIGN_SHOTGUN_LOOP}}` | `resolvers/design.ts` | /design-shotgun、/plan-design-review、/design-consultation 共享的比较板反馈循环 |
| `{{UX_PRINCIPLES}}` | `resolvers/design.ts` | 用户行为基础（扫描、满足、善意储备、主干测试），用于 /design-html、/design-shotgun、/design-review、/plan-design-review |

这在结构上是健全的——如果命令存在于代码中，它会出现在文档中。如果不存在，它就不可能出现。

### 前言

每个 Skill 以 `{{PREAMBLE}}` 块开始，在 Skill 自己的逻辑之前运行。它在一个 bash 命令中处理五件事：

1. **更新检查** — 调用 `gstack-update-check`，报告是否有可用升级。
2. **会话追踪** — 触碰 `~/.gstack/sessions/$PPID` 并统计活跃会话（过去 2 小时内修改的文件）。当 3+ 会话运行时，所有 Skills 进入"ELI16 模式"——每个问题都为用户重新建立上下文，因为他们在多窗口之间切换。
3. **运营自我改进** — 在每个 Skill 会话结束时，Agent 反思失败（CLI 错误、错误方法、项目特点）并将运营学习记录到项目的 JSONL 文件中，供未来会话使用。
4. **AskUserQuestion 格式** — 通用格式：上下文、问题、`RECOMMENDATION: Choose X because ___`、字母选项。所有 Skills 一致。
5. **先搜索再构建** — 在构建基础设施或不熟悉的模式之前，先搜索。三层知识：久经考验的（第 1 层）、新潮热门的（第 2 层）、第一性原理（第 3 层）。当第一性原理推理揭示传统智慧是错误的时，Agent 命名"顿悟时刻"并记录它。参见 `ETHOS.md` 获取完整的构建理念。

### 为什么提交而不是运行时生成？

三个原因：

1. **Claude 在 Skill 加载时读取 SKILL.md。** 用户调用 `/browse` 时没有构建步骤。文件必须已存在且正确。
2. **CI 可以验证新鲜度。** `gen:skill-docs --dry-run` + `git diff --exit-code` 在合并前捕获过时文档。
3. **Git blame 可用。** 你可以看到命令何时添加以及在哪个提交中。

### 模板测试级别

| 级别 | 内容 | 成本 | 速度 |
|------|------|------|-------|
| 1 — 静态验证 | 解析 SKILL.md 中的每个 `$B` 命令，针对注册表验证 | 免费 | <2s |
| 2 — 通过 `claude -p` 的 E2E | 生成真实 Claude 会话，运行每个 Skill，检查错误 | ~$3.85 | ~20min |
| 3 — LLM 法官 | Sonnet 对文档的清晰度/完整性/可操作性评分 | ~$0.15 | ~30s |

第 1 级在每次 `bun test` 运行时运行。第 2+3 级由 `EVALS=1` 门控。思路是：免费捕获 95% 的问题，仅将 LLM 用于判断调用。

## 命令分派

命令按副作用分类：

- **READ**（text、html、links、console、cookies 等）：无变更。可安全重试。返回页面状态。
- **WRITE**（goto、click、fill、press 等）：变更页面状态。非幂等。
- **META**（snapshot、screenshot、tabs、chain 等）：不适合读/写的服务器级操作。

这不仅仅是组织性的。服务器用它来分派：

```typescript
if (READ_COMMANDS.has(cmd))  → handleReadCommand(cmd, args, bm)
if (WRITE_COMMANDS.has(cmd)) → handleWriteCommand(cmd, args, bm)
if (META_COMMANDS.has(cmd))  → handleMetaCommand(cmd, args, bm, shutdown)
```

`help` 命令返回全部三个集合，以便 Agent 可以自发现可用命令。

## 错误哲学

错误是给 AI Agent 看的，不是给人类看的。每条错误消息必须可操作：

- "Element not found" → "Element not found or not interactable. Run `snapshot -i` to see available elements."
- "Selector matched multiple elements" → "Selector matched multiple elements. Use @refs from `snapshot` instead."
- 超时 → "Navigation timed out after 30s. The page may be slow or the URL may be wrong."

Playwright 的原生错误通过 `wrapError()` 重写，以剥离内部堆栈跟踪并添加指导。Agent 应该能够读取错误并在无人干预的情况下知道下一步做什么。

### 崩溃恢复

服务器不会尝试自我修复。如果 Chromium 崩溃（`browser.on('disconnected')`），服务器立即退出。CLI 在下一条命令中检测到已死的服务器并自动重启。这比尝试重新连接到半死的浏览器进程更简单、更可靠。

## E2E 测试基础设施

### 会话运行器（`test/helpers/session-runner.ts`）

E2E 测试将 `claude -p` 生成为完全独立的子进程——不是通过 Agent SDK，因为它无法嵌套在 Claude Code 会话中。运行器：

1. 将提示写入临时文件（避免 Shell 转义问题）
2. 生成 `sh -c 'cat prompt | claude -p --output-format stream-json --verbose'`
3. 从 stdout 流式传输 NDJSON 以获取实时进度
4. 与可配置的超时竞争
5. 将完整的 NDJSON 转录解析为结构化结果

`parseNDJSON()` 函数是纯函数——无 I/O、无副作用——使其可独立测试。

### 可观测性数据流

（数据流图见英文原文）

**所有权拆分：** session-runner 拥有心跳（当前测试状态），eval-store 拥有部分结果（已完成测试状态）。监视器读取两者。两个组件对彼此一无所知——它们仅通过文件系统共享数据。

**一切都是非致命的：** 所有可观测性 I/O 都包裹在 try/catch 中。写入失败永远不会导致测试失败。测试本身是真相来源；可观测性是尽力而为的。

**机器可读的诊断：** 每个测试结果包含 `exit_reason`（success、timeout、error_max_turns、error_api、exit_code_N）、`timeout_at_turn` 和 `last_tool_call`。这使得支持 `jq` 查询。

### 评估持久化（`test/helpers/eval-store.ts`）

`EvalCollector` 累积测试结果并以两种方式写入：

1. **增量：** `savePartial()` 在每次测试后写入 `_partial-e2e.json`（原子：写 `.tmp`，`fs.renameSync`）。在 kill 后仍存活。
2. **最终：** `finalize()` 写入带时间戳的评估文件（例如 `e2e-20260314-143022.json`）。部分文件永远不会被清理——它与最终文件并存以供可观测性。

`eval:compare` 比较两次评估运行。`eval:summary` 聚合 `~/.gstack-dev/evals/` 中所有运行的统计信息。

### 测试级别

第 1 级在每次 `bun test` 运行时运行。第 2+3 级由 `EVALS=1` 门控。思路是：免费捕获 95% 的问题，仅将 LLM 用于判断调用和集成测试。

## 这里故意不包含什么

- **无 WebSocket 流式传输。** HTTP 请求/响应更简单、可通过 curl 调试、且足够快。流式传输会增加复杂性，收益甚微。
- **无 MCP 协议。** MCP 为每个请求添加 JSON Schema 开销并需要持久连接。纯 HTTP + 纯文本输出更轻量，更容易调试。
- **无多用户支持。** 每个工作区一个服务器，一个用户。Token 认证是纵深防御，不是多租户。
- **无 Windows/Linux Cookie 解密。** macOS Keychain 是唯一支持的凭据存储。Linux（GNOME Keyring/kwallet）和 Windows（DPAPI）在架构上可行但未实现。
- **无 iframe 自动发现。** `$B frame` 支持跨框架交互（CSS 选择器、@ref、`--name`、`--url` 匹配），但 ref 系统在 `snapshot` 期间不自动爬取 iframe。必须先显式进入框架上下文。
