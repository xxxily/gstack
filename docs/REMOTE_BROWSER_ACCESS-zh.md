# 远程浏览器访问 — 如何与 GStack Browser 配对

GStack Browser 服务器可以与任何能发起 HTTP 请求的 AI 代理共享。代理获得对真实 Chromium 浏览器的受限访问权限：导航页面、读取内容、点击元素、填写表单、截图。每个代理拥有自己的标签页。

本文档面向远程代理的参考手册。快速入门指令由 `$B pair-agent` 生成，内嵌了实际凭证。

## 架构

```
你的机器                               远程代理
─────────────                         ────────────
GStack Browser 服务器                 任意 AI 代理
  ├── Chromium（Playwright）          （OpenClaw、Hermes、Codex 等）
  ├── HTTP API，localhost:PORT            │
  ├── ngrok 隧道（可选）                  │
  │     https://xxx.ngrok.dev ─────────────┘
  └── Token 注册表
        ├── Root token（仅本地）
        ├── Setup key（5 分钟过期，一次性）
        └── Session token（24 小时，受限作用域）
```

## 连接流程

1. **用户运行** `$B pair-agent`（或 Claude Code 中的 `/pair-agent`）
2. **服务器创建** 一次性 setup key（5 分钟内过期）
3. **用户复制** 指令块到另一个代理的聊天窗口
4. **远程代理运行** `POST /connect` 携带 setup key
5. **服务器返回** 受限 session token（默认 24 小时）
6. **远程代理创建** 自己的标签页，通过 `POST /command` 携带 `newtab`
7. **远程代理浏览** 使用 `POST /command` 携带 session token + tabId

## API 参考

### 认证

除 `/connect` 和 `/health` 之外的所有端点都需要 Bearer token：

```
Authorization: Bearer gsk_sess_...
```

### 端点

#### POST /connect
用 setup key 换取 session token。无需认证。速率限制为每分钟 3 次。

```json
请求:  {"setup_key": "gsk_setup_..."}
响应: {"token": "gsk_sess_...", "expires": "ISO8601", "scopes": ["read","write"], "agent": "agent-name"}
```

#### POST /command
发送浏览器命令。需要 Bearer 认证。

```json
请求:  {"command": "goto", "args": ["https://example.com"], "tabId": 1}
响应: （命令的纯文本结果）
```

#### GET /health
服务器状态。无需认证。返回状态、标签页数、模式、运行时间。

### 命令

#### 导航
| 命令 | 参数 | 描述 |
|---------|------|-------------|
| `goto` | `["URL"]` | 导航到指定 URL |
| `back` | `[]` | 后退 |
| `forward` | `[]` | 前进 |
| `reload` | `[]` | 重新加载页面 |

#### 读取内容
| 命令 | 参数 | 描述 |
|---------|------|-------------|
| `snapshot` | `["-i"]` | 带 @ref 标签的交互式快照（最实用） |
| `text` | `[]` | 完整页面文本 |
| `html` | `["selector?"]` | 指定元素或完整页面的 HTML |
| `links` | `[]` | 页面上所有链接 |
| `screenshot` | `["/tmp/s.png"]` | 截图 |
| `url` | `[]` | 当前 URL |

#### 交互
| 命令 | 参数 | 描述 |
|---------|------|-------------|
| `click` | `["@e3"]` | 点击元素（使用 snapshot 返回的 @ref） |
| `fill` | `["@e5", "text"]` | 填写表单字段 |
| `select` | `["@e7", "option"]` | 选择下拉菜单值 |
| `type` | `["text"]` | 输入文本（键盘） |
| `press` | `["Enter"]` | 按键 |
| `scroll` | `["down"]` | 滚动页面 |

#### 标签页
| 命令 | 参数 | 描述 |
|---------|------|-------------|
| `newtab` | `["URL?"]` | 创建新标签页（写入操作前必须执行） |
| `tabs` | `[]` | 列出所有标签页 |
| `closetab` | `["id?"]` | 关闭标签页 |

## Snapshot → @ref 模式

这是最强大的浏览模式。与其手写 CSS 选择器：

1. 运行 `snapshot -i` 获取带标注元素的交互式快照
2. 快照返回如下文本：
   ```
   [Page Title]
   @e1 [link] "Home"
   @e2 [button] "Sign In"
   @e3 [input] "Search..."
   ```
3. 在命令中直接使用 `@e` 引用：`click @e2`、`fill @e3 "search query"`

这就是快照系统的工作方式，比猜测 CSS 选择器可靠得多。始终先执行 `snapshot -i`，再使用 ref。

## 作用域

| 作用域 | 允许的操作 |
|-------|-----------|
| `read` | snapshot、text、html、links、screenshot、url、tabs、console 等 |
| `write` | goto、click、fill、scroll、newtab、closetab 等 |
| `admin` | eval、js、cookies、storage、cookie-import、useragent 等 |
| `meta` | tab、diff、frame、responsive、watch |

默认 token 获得 `read` + `write`。Admin 需要在配对时使用 `--admin` 标志。

## 标签页隔离

每个代理只拥有它创建的标签页。规则：
- **读取：** 任何代理可以读取任何标签页（snapshot、text、screenshot）
- **写入：** 只有标签页创建者可以写入（click、fill、goto 等）
- **无主标签页：** 预先存在的标签页对写入仅限 root 访问
- **第一步：** 尝试交互前始终先执行 `newtab`

## 错误码

| 状态码 | 含义 | 解决方法 |
|------|---------|------------|
| 401 | Token 无效、过期或已吊销 | 请用户重新运行 /pair-agent |
| 403 | 命令不在作用域内或标签页不属于你 | 使用 newtab 或请求 --admin |
| 429 | 超过速率限制（>10 请求/秒） | 等待 Retry-After 头指定的时间 |

## 安全模型

- Setup key 5 分钟内过期且只能使用一次
- Session token 默认 24 小时过期（可配置）
- Root token 绝不会出现在指令块或连接字符串中
- Admin 作用域（JS 执行、Cookie 访问）默认被拒绝
- Token 可以即时吊销：`$B tunnel revoke agent-name`
- 所有代理活动都有带归因（clientId）的日志记录

## 同机器快捷方式

如果两个代理在同一台机器上，跳过复制粘贴步骤：

```bash
$B pair-agent --local openclaw    # 写入 ~/.openclaw/skills/gstack/browse-remote.json
$B pair-agent --local codex       # 写入 ~/.codex/skills/gstack/browse-remote.json
$B pair-agent --local cursor      # 写入 ~/.cursor/skills/gstack/browse-remote.json
```

不需要隧道。直接使用 localhost。

## ngrok 隧道设置

对于不同机器上的远程代理：

1. 在 [ngrok.com](https://ngrok.com) 注册（免费版本可用）
2. 从控制台复制你的 auth token
3. 保存：`echo 'NGROK_AUTHTOKEN=your_token' > ~/.gstack/ngrok.env`
4. 可选，申请一个稳定域名：`echo 'NGROK_DOMAIN=your-name.ngrok-free.dev' >> ~/.gstack/ngrok.env`
5. 带隧道启动：`BROWSE_TUNNEL=1 $B restart`
6. 运行 `$B pair-agent` — 它会自动使用隧道 URL
