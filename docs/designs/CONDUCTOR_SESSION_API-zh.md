# Conductor Session Streaming API 提案

## 问题

当 Claude 通过 CDP 控制你的真实浏览器时（gstack `$B connect`），你需要看两个窗口：**Conductor**（查看 Claude 的思考过程）和 **Chrome**（查看 Claude 的操作）。

gstack 的 Chrome 扩展 Side Panel 会显示 browse 活动——每条命令、结果和错误。但要实现*完整*的 session 镜像（包括 Claude 的思考、tool call、代码编辑），Side Panel 需要 Conductor 暴露 conversation stream。

## 这能实现什么

在 gstack Chrome 扩展 Side Panel 中添加一个 "Session" 标签页，展示：
- Claude 的思考/内容（为性能考虑做了截断）
- Tool call 名称 + 图标（Edit、Bash、Read 等）
- Turn 边界与成本估算
- 对话进行中的实时更新

用户可以在一个地方看到所有内容——浏览器中 Claude 的操作 + Side Panel 中 Claude 的思考——无需来回切换窗口。

## 提议的 API

### `GET http://127.0.0.1:{PORT}/workspace/{ID}/session/stream`

Server-Sent Events 端点，以 NDJSON 事件形式重新转发 Claude Code 的对话内容。

**事件类型**（复用 Claude Code 的 `--output-format stream-json` 格式）：

```
event: assistant
data: {"type":"assistant","content":"Let me check that page...","truncated":true}

event: tool_use
data: {"type":"tool_use","name":"Bash","input":"$B snapshot","truncated_input":true}

event: tool_result
data: {"type":"tool_result","name":"Bash","output":"[snapshot output...]","truncated_output":true}

event: turn_complete
data: {"type":"turn_complete","input_tokens":1234,"output_tokens":567,"cost_usd":0.02}
```

**内容截断：** 在 stream 中，tool 输入/输出限制为 500 字符。完整数据保留在 Conductor 的 UI 中。Side Panel 是一个概览视图，不是替代品。

### `GET http://127.0.0.1:{PORT}/api/workspaces`

发现端点，列出活跃的 workspace。

```json
{
  "workspaces": [
    {
      "id": "abc123",
      "name": "gstack",
      "branch": "garrytan/chrome-extension-ctrl",
      "directory": "/Users/garry/gstack",
      "pid": 12345,
      "active": true
    }
  ]
}
```

Chrome 扩展会通过将 browse server 的 git repo（来自 `/health` 响应）与 workspace 的目录或名称进行匹配，自动选择一个 workspace。

## 安全性

- **仅限 localhost。** 与 Claude Code 自身 debug 输出采用相同的信任模型。
- **无需认证。** 如果 Conductor 需要认证，可在 workspace 列表中包含一个 Bearer token，扩展在 SSE 请求中携带该 token。
- **内容截断** 是一项隐私功能——长代码输出、文件内容和敏感的 tool 结果永远不会离开 Conductor 的完整 UI。

## gstack 构建（扩展端）

已在 Side Panel "Session" 标签页中搭建好骨架（当前显示占位内容）。

当 Conductor 的 API 可用时：
1. Side Panel 通过端口探测或手动输入发现 Conductor
2. 获取 `/api/workspaces`，与 browse server 的 repo 进行匹配
3. 向 `/workspace/{id}/session/stream` 打开 `EventSource`
4. 渲染：assistant 消息、tool 名称 + 图标、turn 边界、成本
5. 优雅降级：显示 "Connect Conductor for full session view"

预估工作量：`sidepanel.js` 中约 ~200 行代码。

## Conductor 构建（服务端）

1. SSE 端点，按 workspace 重新转发 Claude Code 的 stream-json
2. `/api/workspaces` 发现端点，包含活跃 workspace 列表
3. 内容截断（tool 输入/输出限制为 500 字符）

预估工作量：约 ~100-200 行代码（前提是 Conductor 已在其内部捕获了 Claude Code 的 stream——它确实如此，用于自身 UI 渲染）。

## 设计决策

| 决策 | 选择 | 理由 |
|----------|--------|-----------|
| 传输层 | SSE（而非 WebSocket） | 单向通信、自动重连、更简单 |
| 格式 | Claude 的 stream-json | Conductor 已经在解析这个格式；无需新 schema |
| 发现方式 | HTTP 端点（而非文件） | Chrome 扩展无法读取文件系统 |
| 认证 | 无（localhost） | 与 browse server、CDP 端口、Claude Code 一致 |
| 截断 | 500 字符 | Side Panel 宽度约 300px；长内容无意义 |
