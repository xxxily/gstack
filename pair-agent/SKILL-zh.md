---
name: pair-agent
version: 0.1.0
description: |
  将远程 AI agent 与你的浏览器配对。一个命令生成设置密钥并
  打印其他 agent 可遵循的连接说明。适用于 OpenClaw、
  Hermes、Codex、Cursor 或任何能发起 HTTP 请求的 agent。远程 agent
  获得自己的标签页，带有受限访问权限（默认读写，按需 admin）。
  使用场景："pair agent"、"connect agent"、"share browser"、"remote browser"、
  "let another agent use my browser"或"give browser access"。(gstack)
  语音触发（语音到文本别名）："pair agent"、"connect agent"、"share my browser"、"remote browser access"。
allowed-tools:
  - Bash
  - Read
  - AskUserQuestion

---
<!-- AUTO-GENERATED from SKILL.md.tmpl — do not edit directly -->
<!-- Regenerate: bun run gen:skill-docs -->

（Preamble、Voice、Context Recovery、AskUserQuestion Format、Completeness Principle 等标准部分与其他 skill 相同，已省略）

# /pair-agent —— 与另一个 AI Agent 共享你的浏览器

你正坐在 Claude Code 中，浏览器在运行。你还打开了另一个 AI agent
（OpenClaw、Hermes、Codex、Cursor，随便什么）。你想让那个 agent
能够使用**你的**浏览器浏览网页。这个 skill 让它变成现实。

## 工作原理

你的 gstack 浏览器运行一个本地 HTTP 服务。此 skill 创建一个一次性设置密钥，
打印一段说明，你将这些说明粘贴到另一个 agent 中。
另一个 agent 用密钥交换会话令牌，创建自己的标签页，然后开始
浏览。每个 agent 都有自己的标签页。它们不会互相干扰对方的标签页。

设置密钥 5 分钟内过期，且只能使用一次。如果泄露，在有人滥用之前就已失效。会话令牌有效期 24 小时。

**同机器：** 如果另一个 agent 在同一机器上运行（如 OpenClaw 本地运行），
你可以跳过复制粘贴流程，直接将凭证写入
agent 的配置目录。

**远程：** 如果另一个 agent 在不同机器上，你需要 ngrok 隧道。
此 skill 会告诉你是否需要以及如何设置。

## 设置（在任何 browse 命令之前运行此检查）

（代码块保持不变）

如果 `NEEDS_SETUP`：
1. 告诉用户："gstack browse 需要一次性构建（约 10 秒）。可以继续吗？"然后**停止**并等待。
2. 运行：`cd <SKILL_DIR> && ./setup`
3. 如果 `bun` 未安装：（代码块保持不变）

## 步骤 1：检查前提条件

```bash
$B status 2>/dev/null
```

如果 browse 服务未运行，启动它：

```bash
$B goto about:blank
```

这确保在配对之前服务处于正常运行状态。

## 步骤 2：询问需求

使用 AskUserQuestion：

> 你想将哪个 agent 与浏览器配对？这将决定
> 说明格式和凭证写入位置。

选项：
- A) OpenClaw（本地或远程）
- B) Codex / OpenAI Agents（本地）
- C) Cursor（本地）
- D) 另一个 Claude Code 会话（本地或远程）
- E) 其他（通用 HTTP 说明——适用于 Hermes）

根据回答，设置 `TARGET_HOST`：
- A → `openclaw`
- B → `codex`
- C → `cursor`
- D → `claude`
- E → generic（无特定主机配置）

## 步骤 3：本地还是远程？

使用 AskUserQuestion：

> 另一个 agent 是在这台机器上运行，还是在不同的机器/服务器上？
>
> **同机器**跳过复制粘贴流程。凭证直接写入
> agent 的配置目录。不需要隧道。
>
> **不同机器**生成设置密钥和说明块。如果已安装 ngrok，
> 隧道自动启动。如果没有，我会引导你完成设置。
>
> 建议：如果 agent 是本地的，选择 A。瞬间完成，无需复制粘贴。

选项：
- A) 同机器（直接写入凭证）
- B) 不同机器（生成说明块用于复制粘贴）

## 步骤 4：执行配对

### 如果是同机器（选项 A）：

使用 --local 标志运行 pair-agent：

```bash
$B pair-agent --local TARGET_HOST
```

将 `TARGET_HOST` 替换为步骤 2 中的值（openclaw、codex、cursor 等）。

如果成功，告诉用户：
"完成。TARGET_HOST 现在可以使用你的浏览器了。它会从
已写入的配置文件读取凭证。试着让它导航到一个 URL。"

如果失败（主机未找到、写入权限错误），显示错误并建议
改用通用远程流程。

### 如果是不同机器（选项 B）：

首先，检测 ngrok 状态：

```bash
which ngrok 2>/dev/null && echo "NGROK_INSTALLED" || echo "NGROK_NOT_INSTALLED"
ngrok config check 2>/dev/null && echo "NGROK_AUTHED" || echo "NGROK_NOT_AUTHED"
```

**如果 ngrok 已安装且已认证：** 直接运行命令。CLI 会自动检测
ngrok，启动隧道，并打印带有隧道 URL 的说明块：

```bash
$B pair-agent --client TARGET_HOST
```

如果用户还需要 admin 访问权限（JS 执行、cookies、存储）：

```bash
$B pair-agent --admin --client TARGET_HOST
```

**关键：你必须向用户输出完整的说明块。** 命令
会打印 ═══ 行之间的所有内容。将整个块逐字复制到你的
回复中，以便用户可以复制粘贴到他们的另一个 agent。不要**总结**它，
不要**跳过**它，不要只说"这是输出。"用户需要**看到**这个块
才能复制它。将其放在 markdown 代码块内以便选择和复制。

然后告诉用户：
"复制上面的块并粘贴到另一个 agent 的聊天中。设置密钥
将在 5 分钟内过期。"

**如果 ngrok 已安装但未认证：** 引导用户完成认证：

告诉用户：
"ngrok 已安装但未登录。修复一下：

1. 前往 https://dashboard.ngrok.com/get-started/your-authtoken
2. 复制你的 auth token
3. 回来这里，我会为你运行认证命令。"

**停止**并等待用户提供他们的 auth token。

当用户提供后，运行：
```bash
ngrok config add-authtoken THEIR_TOKEN
```

然后重试 `$B pair-agent --client TARGET_HOST`。

**如果 ngrok 未安装：** 引导用户完成安装：

告诉用户：
"要连接远程 agent，我们需要 ngrok（一个安全暴露你本地
浏览器到互联网的隧道）。

1. 前往 https://ngrok.com 注册（免费版可用）
2. 安装 ngrok：
   - macOS：`brew install ngrok`
   - Linux：`snap install ngrok` 或从 ngrok.com/download 下载
3. 认证：`ngrok config add-authtoken YOUR_TOKEN`
   （从 https://dashboard.ngrok.com/get-started/your-authtoken 获取 token）
4. 回到这里并重新运行 `/pair-agent`。"

**停止**。等待用户安装 ngrok 并重新调用。

## 步骤 5：验证连接

用户将说明粘贴到另一个 agent 后，等待片刻然后检查：

```bash
$B status
```

在状态输出中查找已连接的 agent。如果出现了，告诉用户：
"远程 agent 已连接并拥有自己的标签页。如果打开了 GStack Browser，
你可以在侧面板中看到它的活动。"

## 远程 agent 能做什么

默认（读写）访问：
- 导航到 URL、点击元素、填写表单、截图
- 读取页面内容（文本、HTML、快照）
- 创建新标签页（每个 agent 有自己的）
- 不能执行任意 JavaScript、读取 cookies 或访问存储

admin 访问（--admin 标志）：
- 以上全部，加上 JS 执行、cookie 访问、存储访问
- 谨慎使用。仅用于你完全信任的 agent。

## 故障排除

**"Tab not owned by your agent"**——远程 agent 尝试与它
未创建的标签页交互。告诉它先运行 `newtab` 获取自己的标签页。

**"Domain not allowed"**——令牌有域名限制。重新配对，放宽
域名访问或取消域名限制。

**"Rate limit exceeded"**——agent 发送请求超过 10 次/秒。它应该
等待 Retry-After 头部并放慢速度。

**"Token expired"**——24 小时会话已过期。再次运行 `/pair-agent` 生成
新的设置密钥。

**Agent 无法连接服务器**——如果是远程，检查 ngrok 隧道是否在运行
（`$B status`）。如果是本地，检查 browse 服务是否运行。

## 平台特定说明

### OpenClaw / AlphaClaw

OpenClaw agent 使用 `exec` 工具而非 `Bash`。说明块使用
`exec curl` 语法，OpenClaw 原生理解。使用 `--local openclaw` 时，
凭证写入 `~/.openclaw/skills/gstack/browse-remote.json`。

### Codex

Codex agent 可以通过 `codex exec` 执行 shell 命令。说明块中的
curl 命令直接可用。使用 `--local codex` 时，凭证写入
`~/.codex/skills/gstack/browse-remote.json`。

### Cursor

Cursor 的 AI 可以运行终端命令。说明块原样可用。
使用 `--local cursor` 时，凭证写入
`~/.cursor/skills/gstack/browse-remote.json`。

## 撤销访问

断开特定 agent：

```bash
$B tunnel revoke AGENT_NAME
```

断开所有 agent 并轮换根令牌：

```bash
# 这会立即使所有作用域令牌失效
$B tunnel rotate
```
