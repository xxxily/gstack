# /open-gstack-browser — 启动 GStack Browser

启动 GStack Browser —— AI 控制的 Chromium，内置侧边栏扩展、反机器人 stealth 和自定义品牌标识。你可以实时看到每个操作。

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

## 步骤 0: 飞行前清理

在连接之前，终止任何滞后的 browse 服务器并清理可能从崩溃中持久化的锁文件。

```bash
# 终止任何现有的 browse 服务器
if [ -f "$(git rev-parse --show-toplevel 2>/dev/null)/.gstack/browse.json" ]; then
  _OLD_PID=$(cat "$(git rev-parse --show-toplevel)/.gstack/browse.json" 2>/dev/null | grep -o '"pid":[0-9]*' | grep -o '[0-9]*')
  [ -n "$_OLD_PID" ] && kill "$_OLD_PID" 2>/dev/null || true
  sleep 1
  [ -n "$_OLD_PID" ] && kill -9 "$_OLD_PID" 2>/dev/null || true
  rm -f "$(git rev-parse --show-toplevel)/.gstack/browse.json"
fi
# 清理 Chromium profile 锁（可能在崩溃后仍然存在）
_PROFILE_DIR="$HOME/.gstack/chromium-profile"
for _LF in SingletonLock SingletonSocket SingletonCookie; do
  rm -f "$_PROFILE_DIR/$_LF" 2>/dev/null || true
done
echo "Pre-flight cleanup done"
```

## 步骤 1: 连接

```bash
$B connect
```

这将启动 headed 模式的 GStack Browser（重新品牌的 Chromium），带有：
- 你可以观察的可见窗口（不是你的常规 Chrome）
- 通过 `launchPersistentContext` 自动加载的 gstack 侧边栏扩展
- 反机器人 stealth 补丁（Google 和 NYTimes 等站点无 CAPTCHA）
- 自定义用户代理和 GStack Browser 品牌标识
- 用于聊天命令的侧边栏代理进程

`connect` 命令自动发现扩展。始终使用端口 **34567**。

连接后打印完整输出。确认输出中包含 `Mode: headed`。

## 步骤 2: 验证

```bash
$B status
```

确认 `Mode: headed`。读取端口：

```bash
cat "$(git rev-parse --show-toplevel 2>/dev/null)/.gstack/browse.json" 2>/dev/null | grep -o '"port":[0-9]*' | grep -o '[0-9]*'
```

端口应为 **34567**。同时找到扩展路径：

```bash
_EXT_PATH=""
_ROOT=$(git rev-parse --show-toplevel 2>/dev/null)
[ -n "$_ROOT" ] && [ -f "$_ROOT/.claude/skills/gstack/extension/manifest.json" ] && _EXT_PATH="$_ROOT/.claude/skills/gstack/extension"
[ -z "$_EXT_PATH" ] && [ -f "$HOME/.claude/skills/gstack/extension/manifest.json" ] && _EXT_PATH="$HOME/.claude/skills/gstack/extension"
echo "EXTENSION_PATH: ${_EXT_PATH:-NOT FOUND}"
```

## 步骤 3: 引导用户打开 Side Panel

使用 AskUserQuestion：

> Chrome 已启动并受 gstack 控制。你应该看到 Playwright 的 Chromium，页面顶部有一条金色闪烁线。
>
> Side Panel 扩展应已自动加载。打开方法：
> 1. 在工具栏查找**拼图图标**（扩展）
> 2. 点击**拼图** → 找到 **gstack browse** → 点击**图钉图标**
> 3. 点击工具栏中已固定的 **gstack 图标**
> 4. Side Panel 在右侧打开，显示实时活动动态
>
> **端口：** 34567

选项：
- A) 我可以看到 Side Panel —— 开始吧！
- B) 我可以看到 Chrome 但找不到扩展
- C) 出了问题

如果 B：

> 1. 在地址栏输入 `chrome://extensions`
> 2. 查找 **"gstack browse"**
> 3. 如果未列出，点击 **"Load unpacked"**，按 **Cmd+Shift+G**，粘贴路径：`{EXTENSION_PATH}`
> 4. 加载后固定并点击图标
>
> 如果 Side Panel 徽章保持灰色（已断开），点击 gstack 图标并手动输入端口 **34567**。

如果 C：
1. 运行 `$B status` 并显示输出
2. 如果不健康，重新运行步骤 0 清理 + 步骤 1 连接
3. 如果健康但浏览器不可见，尝试 `$B focus`
4. 询问用户看到了什么

## 步骤 4: 演示

```bash
$B goto https://news.ycombinator.com
```

等待 2 秒后：

```bash
$B snapshot -i
```

告诉用户："检查 Side Panel —— 你应该看到 `goto` 和 `snapshot` 命令出现在活动动态中。"

## 步骤 5: 侧边栏聊天

> Side Panel 还有**聊天标签页**。尝试输入"截图并描述此页面。"侧边栏代理（子 Claude 实例）在浏览器中执行请求 —— 命令会出现在活动动态中。
>
> 侧边栏代理可以导航页面、点击按钮、填写表单和读取内容。每个任务最多 5 分钟。在隔离 session 中运行。

## 步骤 6: 下一步

> **实时观察 Claude 工作：**
> - 运行任何 gstack skill（`/qa`、`/design-review`、`/benchmark`）
> - 无需导入 Cookie —— Playwright 浏览器共享自己的 session
>
> **直接控制浏览器：**
> - **侧边栏聊天** —— 在 Side Panel 中输入自然语言
> - **Browse 命令** —— `$B goto <url>`、`$B click <sel>`、`$B fill <sel> <val>`、`$B snapshot -i`
>
> **窗口管理：**
> - `$B focus` —— 将 Chrome 带到前台
> - `$B disconnect` —— 关闭 headed Chrome，返回 headless 模式

然后继续用户要求的任务。
