# Chrome vs Chromium：为什么使用 Playwright 捆绑的 Chromium

## 最初愿景

构建 `$B connect` 时，计划是连接到用户的**真实 Chrome 浏览器** — 那个带有 cookie、会话、扩展和已打开标签页的浏览器。不再需要导入 cookie。设计需要：

1. `chromium.connectOverCDP(wsUrl)` 通过 CDP 连接到运行中的 Chrome
2. 优雅退出 Chrome，用 `--remote-debugging-port=9222` 重新启动
3. 访问用户真实的浏览上下文

这就是为什么存在 `chrome-launcher.ts`（361 行代码用于浏览器二进制发现、CDP 端口探测和运行时检测），以及为什么这个方法叫 `connectCDP()`。

## 实际情况

通过 Playwright 的 `channel: 'chrome'` 启动真实 Chrome 时，它静默阻止了 `--load-extension`。扩展无法加载。我们需要扩展来提供侧方面板（活动流、refs、聊天）。

实现回退到了使用 Playwright 捆绑 Chromium 的 `chromium.launchPersistentContext()` — 它可以通过 `--load-extension` 和 `--disable-extensions-except` 可靠地加载扩展。但命名保留了：`connectCDP()`、`connectionMode: 'cdp'`、`BROWSE_CDP_URL`、`chrome-launcher.ts`。

最初的愿景（访问用户真实浏览器状态）从未实现。我们每次都启动一个全新的浏览器 — 功能上与 Playwright 的 Chromium 相同，但带着 361 行死代码和误导性的命名。

## 发现（2026-03-22）

在一次 `/office-hours` 设计会话中，我们追踪了架构并发现：

1. `connectCDP()` 没有使用 CDP — 它调用的是 `launchPersistentContext()`
2. `connectionMode: 'cdp'` 有误导性 — 它只是"有头模式"
3. `chrome-launcher.ts` 是死代码 — 它唯一的引用在不可达的 `attemptReconnect()` 方法中
4. `preExistingTabIds` 是为保护我们从未连接过的真实 Chrome 标签页而设计的
5. `$B handoff`（无头 → 有头）使用了不同的 API（`launch()` + `newContext()`），无法加载扩展，创建了两种不同的"有头"体验

## 修复

### 重命名
- `connectCDP()` → `launchHeaded()`
- `connectionMode: 'cdp'` → `connectionMode: 'headed'`
- `BROWSE_CDP_URL` → `BROWSE_HEADED`

### 删除
- `chrome-launcher.ts`（361 行）
- `attemptReconnect()`（死方法）
- `preExistingTabIds`（死概念）
- `reconnecting` 字段（死状态）
- `cdp-connect.test.ts`（被删除代码的测试）

### 统一
- `$B handoff` 现在使用 `launchPersistentContext()` + 扩展加载（与 `$B connect` 相同）
- 一种有头模式，而非两种
- Handoff 自动附带扩展 + 侧方面板

### 门控
- sidebar 聊天通过 `--chat` 标志控制
- `$B connect`（默认）：仅活动流 + refs
- `$B connect --chat`：+ 实验性独立聊天 Agent

## 架构（修复后）

```
浏览器状态：
  无头模式（默认） ←→ 有头模式（$B connect 或 $B handoff）
      Playwright            Playwright（相同引擎）
      launch()              launchPersistentContext()
      不可见                可见 + 扩展 + 侧方面板

Sidebar（正交附加，仅在有头模式下）：
  Activity tab    — 始终开启，显示实时浏览命令
  Refs tab        — 始终开启，显示 @ref 覆盖
  Chat tab        — 通过 --chat 选择，实验性独立 Agent

数据桥（sidebar → 工作区）：
  Sidebar 写入 .context/sidebar-inbox/*.json
  工作区通过 $B inbox 读取
```

## 为什么不用真实 Chrome？

当通过 Playwright 启动时，真实 Chrome 会阻止 `--load-extension`。这是 Chrome 的安全功能 — 基于 Chromium 的浏览器限制了通过命令行参数加载的扩展，以防止恶意扩展注入。

Playwright 捆绑的 Chromium 没有这个限制，因为它是为测试和自动化设计的。`ignoreDefaultArgs` 选项让我们可以绕过 Playwright 自身的扩展阻止标志。

如果我们将来想要访问用户真实的 cookie/会话，路径是：
1. Cookie 导入（已通过 `$B cookie-import` 实现）
2. Conductor 会话注入（未来 — sidebar 向工作区 Agent 发送消息）

不要重新连接真实 Chrome。
