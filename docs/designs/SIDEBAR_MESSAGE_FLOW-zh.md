# Sidebar 消息流

GStack Browser sidebar 实际工作方式。在修改
sidepanel.js、background.js、content.js、server.ts sidebar 端点
或 sidebar-agent.ts 之前阅读此文。

## 组件

```
┌─────────────────┐     ┌──────────────┐     ┌─────────────┐     ┌────────────────┐
│  sidepanel.js   │────▶│ background.js│────▶│  server.ts   │────▶│sidebar-agent.ts│
│  (Chrome 面板)   │     │ (svc worker) │     │  (Bun HTTP)  │     │  (Bun 进程)    │
└─────────────────┘     └──────────────┘     └─────────────┘     └────────────────┘
        ▲                                           │                      │
        │           轮询 /sidebar-chat              │    轮询队列文件        │
        └───────────────────────────────────────────┘                      │
                                                     ◀──────────────────────┘
                                                     POST /sidebar-agent/event
```

## 启动时间线

```
T+0ms     CLI 运行 `$B connect`
            ├── 服务器在端口 34567 启动
            ├── 将状态写入 .gstack/browse.json（pid、port、token）
            ├── 启动带扩展的 headed Chromium
            └── 清除 sidebar-agent-queue.jsonl

T+500ms   sidebar-agent.ts 由 CLI 启动
            ├── 从 .gstack/browse.json 读取认证 token
            ├── 如果缺失则创建队列文件
            ├── 设置 lastLine = 当前行数
            └── 每 200ms 开始轮询

T+1-3s    Chromium 中加载扩展
            ├── background.js：每 1s 健康轮询（快速启动）
            │     └── GET /health → 获取认证 token
            ├── content.js：在欢迎页面注入
            │     └── 不触发 gstack-extension-ready（等待 sidebar）
            └── 侧面板：可能通过 chrome.sidePanel.open() 自动打开

T+2-10s   侧面板连接
            ├── tryConnect() → 向 background 请求端口/token
            ├── 回退：直接 GET /health 获取 token
            ├── updateConnection(url, token)
            │     ├── 开始聊天轮询（1s 间隔）
            │     ├── 开始标签页轮询（2s 间隔）
            │     ├── 连接 SSE 活动流
            │     └── 发送 { type: 'sidebarOpened' } 到 background
            └── background 中继到 content 脚本→ 隐藏欢迎箭头

T+10s+    就绪，可以接收消息
```

## 消息流：用户输入 → Claude 响应

```
1. 用户在 sidebar 中输入"go to hn"，按 Enter

2. sidepanel.js sendMessage()
   ├── 立即渲染用户气泡（乐观）
   ├── 立即渲染思考点
   ├── 切换到快速轮询（300ms）
   └── chrome.runtime.sendMessage({ type: 'sidebar-command', message, tabId })

3. background.js
   ├── 获取活动 Chrome 标签页 URL
   └── POST /sidebar-command { message, activeTabUrl }
       带 Authorization: Bearer ${authToken}

4. server.ts /sidebar-command 处理程序
   ├── validateAuth(req)
   ├── syncActiveTabByUrl(extensionUrl) — 将 Playwright 标签页与 Chrome 标签页同步
   ├── pickSidebarModel(message) — 操作用 sonnet，分析用 opus
   ├── 将用户消息添加到聊天缓冲区
   ├── 构建系统 prompt + 参数
   └── 将 JSON 追加到 ~/.gstack/sidebar-agent-queue.jsonl

5. sidebar-agent.ts poll()（200ms 内）
   ├── 从队列文件读取新行
   ├── 解析 JSON 条目
   ├── 检查 processingTabs — 如果标签页已有 agent 在运行则跳过
   └── askClaude(entry) — 即发即忘

6. sidebar-agent.ts askClaude()
   ├── spawn('claude', ['-p', prompt, '--model', model, ...])
   ├── 逐行流式传输 stdout（stream-json 格式）
   ├── 对于每个事件：POST /sidebar-agent/event { type, tool, text, tabId }
   └── 关闭时：POST /sidebar-agent/event { type: 'agent_done' }

7. server.ts processAgentEvent()
   ├── 将条目添加到聊天缓冲区（内存 + 磁盘）
   ├── agent_done 时：将标签页状态设置为 'idle'
   └── agent_done 时：处理该标签页的下一个排队消息

8. sidepanel.js pollChat()（快速轮询期间每 300ms）
   ├── GET /sidebar-chat?after=${chatLineCount}&tabId=${tabId}
   ├── 渲染新条目（text、tool_use、agent_done）
   └── agent idle 时：移除思考点，停止快速轮询
```

## 箭头提示隐藏流（4 步信号链）

欢迎页面显示一个指向右侧的箭头，直到 sidebar 打开。

```
1. sidepanel.js updateConnection()
   └── chrome.runtime.sendMessage({ type: 'sidebarOpened' })

2. background.js
   └── chrome.tabs.sendMessage(activeTabId, { type: 'sidebarOpened' })

3. content.js onMessage 处理程序
   └── document.dispatchEvent(new CustomEvent('gstack-extension-ready'))

4. welcome.html 脚本
   └── addEventListener('gstack-extension-ready', () => arrow.classList.add('hidden'))
```

箭头不会在扩展加载时隐藏。仅在 sidebar 连接时隐藏。

## Auth Token 流

```
服务器启动 → AUTH_TOKEN = crypto.randomUUID()
    │
    ├── GET /health（无需认证）→ 返回 { token: AUTH_TOKEN }
    │
    ├── background.js checkHealth() → authToken = data.token
    │     └── 每次健康轮询时刷新（修复重启时 token 过时）
    │
    ├── sidepanel.js tryConnect() → serverToken 从 background 或 /health 获取
    │     └── 用于聊天轮询：Authorization: Bearer ${serverToken}
    │
    └── sidebar-agent.ts refreshToken() → 从 .gstack/browse.json 读取
          └── 用于事件中继：Authorization: Bearer ${authToken}
```

如果服务器重启，所有三个组件在 10 秒内获取新 token
（background 健康轮询间隔）。

## 模型路由

`server.ts` 中的 `pickSidebarModel(message)` 对消息进行分类：

| 模式 | 模型 | 原因 |
|---------|-------|-----|
| "click @e24"、"go to hn"、"screenshot" | sonnet | 确定性工具调用，不需要思考 |
| "这个页面说什么？"、"总结" | opus | 需要理解 |
| "查找 bug"、"检查损坏的链接" | opus | 分析任务 |
| "导航到 X 并填写表单" | sonnet | 面向操作，无分析词 |

分析词（`what`、`why`、`how`、`summarize`、`describe`、`analyze`、`read X and Y`）
始终覆盖动作动词并强制使用 opus。

## 已知故障模式

| 故障 | 症状 | 根本原因 | 修复 |
|---------|---------|------------|-----|
| 认证 token 过时 | 输入中显示"Unauthorized" | 服务器重启，background 持有旧 token | background.js 每次健康轮询刷新 token |
| 标签页 ID 不匹配 | 消息已发送，无可见响应 | 服务器分配 tabId 1，sidebar 轮询 tabId 0 | switchChatTab 在切换期间保留乐观 UI |
| Sidebar agent 未运行 | 消息永远排队 | Agent 进程启动失败或崩溃 | 检查 `ps aux | grep sidebar-agent` |
| Agent token 过时 | Agent 运行但 sidebar 无事件 | sidebar-agent 持有 .gstack/browse.json 中的旧 token | Agent 在每次事件 POST 前重新读取 token |
| 队列文件缺失 | spawnClaude 失败 | 服务器启动和 agent 启动之间的竞态 | 双方都在缺失时创建文件 |
| 乐观 UI 被清除 | 用户气泡 + 思考点消失 | switchChatTab 用欢迎屏幕替换 DOM | 设置 lastOptimisticMsg 时保留 DOM |

## 每个标签页的并发性

每个浏览器标签页可以同时运行自己的 agent：

- 服务器：`tabAgents: Map<number, TabAgentState>`，每个标签页队列（最多 5 个）
- sidebar-agent：`processingTabs: Set<number>` 防止重复派生
- 同一标签页的两条消息：顺序排队，按顺序处理
- 不同标签页的两条消息：并发运行

## 文件位置

| 组件 | 文件 | 运行于 |
|-----------|------|---------|
| Sidebar UI | `extension/sidepanel.js` | Chrome 侧面板 |
| Service worker | `extension/background.js` | Chrome 后台 |
| Content script | `extension/content.js` | 页面上下文 |
| 欢迎页面 | `browse/src/welcome.html` | 页面上下文 |
| HTTP 服务器 | `browse/src/server.ts` | Bun（编译二进制） |
| Agent 进程 | `browse/src/sidebar-agent.ts` | Bun（未编译，可以派生） |
| CLI 入口 | `browse/src/cli.ts` | Bun（编译二进制） |
| 队列文件 | `~/.gstack/sidebar-agent-queue.jsonl` | 文件系统 |
| 状态文件 | `.gstack/browse.json` | 文件系统 |
| 聊天日志 | `~/.gstack/sessions/<id>/chat.jsonl` | 文件系统 |
