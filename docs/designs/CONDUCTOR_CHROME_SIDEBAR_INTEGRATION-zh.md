# Chrome Sidebar + Conductor：我们需要的

## 我们要构建什么

目前，当 Claude 在 Conductor 工作区中工作时 — 编辑文件、运行测试、浏览你的应用 — 你只能从 Conductor 的聊天窗口观看。如果 Claude 在对你的网站进行 QA，你看到工具调用滚动而过，但你看不到实际上的浏览器。

我们构建了一个 Chrome sidebar 来解决这个问题。当你运行 `$B connect` 时，Chrome 会打开一个侧边面板，实时显示 Claude 正在做的一切。你可以在 sidebar 中输入消息，Claud 会执行 — "点击注册按钮"、"去设置页面"、"总结一下你看到的内容。"

问题在于：sidebar 目前运行自己独立的 Claude 实例。它看不到主 Conductor 会话在做什么，主会话也看不到 sidebar 在做什么。它们是两个互不通信的独立 Agent。

修复方案很简单：让 sidebar 成为 Conductor 会话的**窗口**，而不是一个独立的东西。

## 我们需要 Conductor 做什么（3 件事）

### 1. 让我们监控 Agent 在做什么

我们需要一种方式来订阅活动会话的事件。比如一个 SSE 流或 WebSocket，在事件发生时发送给我们：

- "Claude 正在编辑 `src/App.tsx`"
- "Claude 正在运行 `npm test`"
- "Claude 说：我会修复 CSS 问题..."

sidebar 已经知道如何渲染这些事件 — 工具调用显示为紧凑徽章，文本显示为聊天气泡。我们只需要一条从 Conductor 会话到扩展的管道。

### 2. 让我们向会话发送消息

当用户在 Chrome sidebar 中输入"点击另一个按钮"时，这条消息应该出现在 Conductor 会话中，就像用户在工作区聊天中输入了它一样。Agent 在下一次轮次中拾取并执行。

这就是魔力时刻：用户正在观看 Chrome，看到有问题，在 sidebar 中输入修正，Claude 响应 — 用户无需切换窗口。

### 3. 让我们从目录创建工作区

当 `$B connect` 启动时，它会创建一个 git worktree 用于文件隔离。我们希望将该 worktree 注册为 Conductor 工作区，这样用户就可以在 Conductor 的文件树中看到 sidebar Agent 的文件变更。这还为多个浏览器会话（各自拥有独立工作区）奠定了基础。

## 为什么这很重要

今天，`/qa` 和 `/design-review` 感觉像一个黑盒子。Claude 说"我发现了 3 个问题"，但你看不到它在看什么。当 sidebar 连接到 Conductor 后：

- **你实时观看 Claude 测试你的应用** — 每次点击、每次导航、每张截图都出现在 Chrome 中，而你在一旁观看
- **你可以中断** — "不，测试移动端视图"或"跳过那个页面" — 无需切换窗口
- **一个 Agent，两个视角** — 编辑代码的同一个 Claude 也在控制浏览器。无上下文重复，无过时状态

## 已完成的部分（gstack 侧）

我们这一侧的所有工作都已完成并交付：

- Chrome 扩展，运行 `$B connect` 时自动加载
- 侧边面板自动打开（用户零配置）
- 流式事件渲染（工具调用、文本、结果）
- 带消息队列的聊天输入
- 带状态横幅的重连逻辑
- 带持久聊天历史的会话管理
- Agent 生命周期（启动、停止、终止、超时检测）

我们这一侧唯一的改变：将数据源从"本地 `claude -p` 子进程"切换为"Conductor 会话流"。扩展代码保持不变。

**预估工作量：** Conductor 工程师 2-3 天，gstack 集成 1 天。
