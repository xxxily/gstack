---
name: setup-browser-cookies
preamble-tier: 1
version: 1.0.0
description: |
  从你真实的 Chromium 浏览器导入 Cookie 到 headless browse session。
  打开交互式选择器 UI，你可以选择要导入哪些 Cookie 域名。
  在 QA 测试已认证页面之前使用。当用户要求 "import cookies"、"login to the site" 或 "authenticate the browser" 时使用。(gstack)
allowed-tools:
  - Bash
  - Read
  - AskUserQuestion
---
<!-- AUTO-GENERATED from SKILL.md.tmpl — do not edit directly -->
<!-- Regenerate: bun run gen:skill-docs -->

# 设置浏览器 Cookie

将已登录 session 从你真实的 Chromium 浏览器导入到 headless browse session。

## CDP 模式检查

首先，检查 browse 是否已连接到用户的真实浏览器：
```bash
$B status 2>/dev/null | grep -q "Mode: cdp" && echo "CDP_MODE=true" || echo "CDP_MODE=false"
```
如果 `CDP_MODE=true`：告诉用户"不需要 —— 你已经通过 CDP 连接到真实浏览器。你的 Cookie 和 session 已可用。" 然后停止。无需导入 Cookie。

## 工作原理

1. 找到 browse 二进制文件
2. 运行 `cookie-import-browser` 检测已安装的浏览器并打开选择器 UI
3. 用户在浏览器中选择要导入哪些 Cookie 域名
4. Cookie 被解密并加载到 Playwright session 中

## 步骤

### 1. 找到 browse 二进制文件

## SETUP（在任何 browse 命令之前运行此检查）

```bash
_ROOT=$(git rev-parse --show-toplevel 2>/dev/null)
B=""
[ -n "$_ROOT" ] && [ -x "$_ROOT/.claude/skills/gstack/browse/dist/browse" ] && B="$_ROOT/.claude/skills/gstack/browse/dist/browse"
[ -z "$B" ] && B=~/.claude/skills/gstack/browse/dist/browse
if [ -x "$B" ]; then
  echo "READY: $B"
else
  echo "NEEDS_SETUP"
fi
```

如果显示 `NEEDS_SETUP`：
1. 告诉用户："gstack browse 需要一次性构建（约 10 秒）。可以继续吗？" 然后 STOP 并等待。
2. 运行：`cd <SKILL_DIR> && ./setup`
3. 如果 `bun` 未安装：
   ```bash
   if ! command -v bun >/dev/null 2>&1; then
     BUN_VERSION="1.3.10"
     BUN_INSTALL_SHA="bab8acfb046aac8c72407bdcce903957665d655d7acaa3e11c7c4616beae68dd"
     tmpfile=$(mktemp)
     curl -fsSL "https://bun.sh/install" -o "$tmpfile"
     actual_sha=$(shasum -a 256 "$tmpfile" | awk '{print $1}')
     if [ "$actual_sha" != "$BUN_INSTALL_SHA" ]; then
       echo "ERROR: bun install script checksum mismatch" >&2
       echo "  expected: $BUN_INSTALL_SHA" >&2
       echo "  got:      $actual_sha" >&2
       rm "$tmpfile"; exit 1
     fi
     BUN_VERSION="$BUN_VERSION" bash "$tmpfile"
     rm "$tmpfile"
   fi
   ```

### 2. 打开 Cookie 选择器

```bash
$B cookie-import-browser
```

这会自动检测已安装的 Chromium 浏览器，并在你的默认浏览器中打开交互式选择器 UI，你可以在其中：
- 在已安装的浏览器之间切换
- 搜索域名
- 点击 "+" 导入某个域名的 Cookie
- 点击垃圾桶移除已导入的 Cookie

告诉用户：**"Cookie 选择器已打开 —— 在你的浏览器中选择要导入的域名，完成后告诉我。"**

### 3. 直接导入（替代方式）

如果用户直接指定了域名（例如 `/setup-browser-cookies github.com`），跳过 UI：

```bash
$B cookie-import-browser comet --domain github.com
```

如果指定了浏览器，将 `comet` 替换为相应的浏览器。

### 4. 验证

用户确认完成后：

```bash
$B cookies
```

向用户显示已导入 Cookie 的摘要（域名计数）。

## 注意事项

- 在 macOS 上，每个浏览器的首次导入可能会触发 Keychain 对话框 —— 点击"允许" / "始终允许"
- 在 Linux 上，`v11` Cookie 可能需要 `secret-tool`/libsecret 访问；`v10` Cookie 使用 Chromium 的标准回退密钥
- Cookie 选择器在与 browse 服务器相同的端口上提供服务（无需额外进程）
- UI 中只显示域名和 Cookie 计数 —— 不暴露 Cookie 值
- browse session 在命令之间持久化 Cookie，所以导入的 Cookie 立即生效
