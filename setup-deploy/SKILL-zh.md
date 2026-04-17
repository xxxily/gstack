---
name: setup-deploy
preamble-tier: 2
version: 1.0.0
description: |
  为 /land-and-deploy 配置部署设置。检测你的部署平台（Fly.io、Render、Vercel、Netlify、Heroku、GitHub Actions、自定义）、
  生产 URL、健康检查端点和部署状态命令。将配置写入 CLAUDE.md，以便将来所有部署都是自动的。
  当用户输入 "setup deploy"、"configure deployment"、"set up land-and-deploy"、
  "how do I deploy with gstack"、"add deploy config" 时使用。
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - AskUserQuestion
---
<!-- AUTO-GENERATED from SKILL.md.tmpl — do not edit directly -->
<!-- Regenerate: bun run gen:skill-docs -->

# /setup-deploy — 为 gstack 配置部署

你正在帮助用户配置他们的部署，以便 `/land-and-deploy` 可以自动工作。你的工作是检测部署平台、生产 URL、健康检查和部署状态命令 —— 然后将所有内容持久化到 CLAUDE.md。

这运行一次后，`/land-and-deploy` 会读取 CLAUDE.md 并完全跳过检测。

## 用户调用
当用户输入 `/setup-deploy` 时，运行此 skill。

## 指令

### 步骤 1: 检查现有配置

```bash
grep -A 20 "## Deploy Configuration" CLAUDE.md 2>/dev/null || echo "NO_CONFIG"
```

如果配置已存在，显示它并询问：

- **Context:** CLAUDE.md 中已存在部署配置。
- **RECOMMENDATION:** 如果你的设置发生了变化，选择 A 进行更新。
- A) 从头重新配置（覆盖现有配置）
- B) 编辑特定字段（显示当前配置，让我修改一项）
- C) 完成 —— 配置看起来正确

如果用户选择 C，停止。

### 步骤 2: 检测平台

运行平台检测：

```bash
# Platform config files
[ -f fly.toml ] && echo "PLATFORM:fly" && cat fly.toml
[ -f render.yaml ] && echo "PLATFORM:render" && cat render.yaml
[ -f vercel.json ] || [ -d .vercel ] && echo "PLATFORM:vercel"
[ -f netlify.toml ] && echo "PLATFORM:netlify" && cat netlify.toml
[ -f Procfile ] && echo "PLATFORM:heroku"
[ -f railway.json ] || [ -f railway.toml ] && echo "PLATFORM:railway"

# GitHub Actions deploy workflows
for f in $(find .github/workflows -maxdepth 1 \( -name '*.yml' -o -name '*.yaml' \) 2>/dev/null); do
  [ -f "$f" ] && grep -qiE "deploy|release|production|staging|cd" "$f" 2>/dev/null && echo "DEPLOY_WORKFLOW:$f"
done

# Project type
[ -f package.json ] && grep -q '"bin"' package.json 2>/dev/null && echo "PROJECT_TYPE:cli"
find . -maxdepth 1 -name '*.gemspec' 2>/dev/null | grep -q . && echo "PROJECT_TYPE:library"
```

### 步骤 3: 平台特定的设置

根据检测到的内容，引导用户完成平台特定的配置。

#### Fly.io

如果检测到 `fly.toml`：

1. 提取应用名称：`grep -m1 "^app" fly.toml | sed 's/app = "\(.*\)"/\1/'`
2. 检查 `fly` CLI 是否已安装：`which fly 2>/dev/null`
3. 如果已安装，验证：`fly status --app {app} 2>/dev/null`
4. 推断 URL：`https://{app}.fly.dev`
5. 设置部署状态命令：`fly status --app {app}`
6. 设置健康检查：`https://{app}.fly.dev`（如果应用有 `/health` 端点则使用该端点）

请用户确认生产 URL。一些 Fly 应用使用自定义域名。

#### Render

如果检测到 `render.yaml`：

1. 从 render.yaml 提取服务名称和类型
2. 检查 Render API key：`echo $RENDER_API_KEY | head -c 4`（不要暴露完整密钥）
3. 推断 URL：`https://{service-name}.onrender.com`
4. Render 在推送到连接的分支时自动部署 —— 无需部署工作流
5. 设置健康检查：推断的 URL

请用户确认。Render 使用从连接的 git 分支自动部署 —— 合并到 main 后，Render 会自动选取。`/land-and-deploy` 中的"部署等待"应轮询 Render URL，直到它响应新版本。

#### Vercel

如果检测到 vercel.json 或 .vercel：

1. 检查 `vercel` CLI：`which vercel 2>/dev/null`
2. 如果已安装：`vercel ls --prod 2>/dev/null | head -3`
3. Vercel 在推送时自动部署 —— PR 上预览，合并到 main 后生产部署
4. 设置健康检查：来自 vercel 项目设置的生产 URL

#### Netlify

如果检测到 netlify.toml：

1. 从 netlify.toml 提取站点信息
2. Netlify 在推送时自动部署
3. 设置健康检查：生产 URL

#### 仅 GitHub Actions

如果检测到部署工作流但没有平台配置：

1. 读取工作流文件以了解它做什么
2. 提取部署目标（如果提到）
3. 请用户提供生产 URL

#### 自定义 / 手动

如果什么都没检测到：

使用 AskUserQuestion 收集信息：

1. **如何触发部署？**
   - A) 推送到 main 时自动（Fly、Render、Vercel、Netlify 等）
   - B) 通过 GitHub Actions 工作流
   - C) 通过部署脚本或 CLI 命令（描述它）
   - D) 手动（SSH、仪表板等）
   - E) 此项目不部署（库、CLI、工具）

2. **生产 URL 是什么？**（自由文本 —— 应用运行的 URL）

3. **gstack 如何检查部署是否成功？**
   - A) 特定 URL 的 HTTP 健康检查（例如 /health、/api/status）
   - B) CLI 命令（例如 `fly status`、`kubectl rollout status`）
   - C) 检查 GitHub Actions 工作流状态
   - D) 没有自动化方式 —— 只需检查 URL 是否可加载

4. **有任何合并前或合并后的 hook 吗？**
   - 合并前要运行的命令（例如 `bun run build`）
   - 合并后但在部署验证之前要运行的命令

### 步骤 4: 写入配置

读取 CLAUDE.md（或创建它）。找到并替换 `## Deploy Configuration` 部分（如果存在），或者在末尾追加。

```markdown
## Deploy Configuration (configured by /setup-deploy)
- Platform: {platform}
- Production URL: {url}
- Deploy workflow: {workflow file or "auto-deploy on push"}
- Deploy status command: {command or "HTTP health check"}
- Merge method: {squash/merge/rebase}
- Project type: {web app / API / CLI / library}
- Post-deploy health check: {health check URL or command}

### Custom deploy hooks
- Pre-merge: {command or "none"}
- Deploy trigger: {command or "automatic on push to main"}
- Deploy status: {command or "poll production URL"}
- Health check: {URL or command}
```

### 步骤 5: 验证

写入后，验证配置是否有效：

1. 如果配置了健康检查 URL，尝试它：
```bash
curl -sf "{health-check-url}" -o /dev/null -w "%{http_code}" 2>/dev/null || echo "UNREACHABLE"
```

2. 如果配置了部署状态命令，尝试它：
```bash
{deploy-status-command} 2>/dev/null | head -5 || echo "COMMAND_FAILED"
```

报告结果。如果任何一项失败，记录下来但不要阻止 —— 即使健康检查暂时无法访问，配置仍然有用。

### 步骤 6: 摘要

```
DEPLOY CONFIGURATION — COMPLETE
════════════════════════════════
Platform:      {platform}
URL:           {url}
Health check:  {health check}
Status cmd:    {status command}
Merge method:  {merge method}

已保存到 CLAUDE.md。/land-and-deploy 将自动使用这些设置。

下一步：
- 运行 /land-and-deploy 合并并部署你当前的 PR
- 编辑 CLAUDE.md 中的 "## Deploy Configuration" 部分以更改设置
- 再次运行 /setup-deploy 以重新配置
```

## 重要规则

- **永远不要暴露密钥。** 不要打印完整的 API key、token 或密码。
- **与用户确认。** 始终显示检测到的配置并在写入前请求确认。
- **CLAUDE.md 是真实来源。** 所有配置都放在那里 —— 不在单独的 config 文件中。
- **幂等的。** 多次运行 /setup-deploy 会干净地覆盖之前的配置。
- **平台 CLI 是可选的。** 如果 `fly` 或 `vercel` CLI 未安装，回退到基于 URL 的健康检查。
