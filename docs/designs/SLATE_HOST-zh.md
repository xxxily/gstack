# Slate Host 集成——研究与设计文档

**日期：** 2026-04-02
**分支：** garrytan/slate-agent-support
**状态：** 研究完成，受阻于 host config 重构
**替代：** 无

## 什么是 Slate

Slate 是 Random Labs 的专有编码 agent CLI。
安装方式：`npm i -g @randomlabs/slate` 或 `brew install anthropic/tap/slate`。
许可证：专有。85MB 编译后的 Bun 二进制文件（arm64/x64，darwin/linux/windows）。
npm 包：`@randomlabs/slate@1.0.25`（精简的 8.8KB 启动器 + 平台特定的可选依赖）。

多模型：动态选择 Claude Sonnet/Opus/Haiku，以及其他模型。
为"swarm 编排"而构建，支持扩展的多小时会话。

## Slate 是一个 OpenCode 分叉

**已通过 85MB Mach-O arm64 二进制文件字符串分析确认：**

- 内部名称：`name: "opencode"`（二进制文件中的字面字符串）
- 所有 `OPENCODE_*` 环境变量与 `SLATE_*` 等效变量同时存在
- 共享 OpenCode 的工具/skill 架构、LSP 集成、终端管理
- 自有品牌、API 端点（`api.randomlabs.ai`、`agent-worker-prod.randomlabs.workers.dev`）和配置路径

这对集成很重要：OpenCode 约定大部分适用，但 Slate 在其上添加了自己的路径和环境变量。

## Skill 发现（来自二进制文件确认）

Slate 扫描 ALL 四个目录族来查找 skill。二进制文件中的错误信息确认：

```
"failed .slate directory scan for skills"
"failed .claude directory scan for skills"
"failed .agents directory scan for skills"
"failed .opencode directory scan for skills"
```

**发现路径（按 Slate 文档的优先级排序）：**

1. `.slate/skills/<name>/SKILL.md` — 项目级，最高优先级
2. `~/.slate/skills/<name>/SKILL.md` — 全局
3. `.opencode/skills/`、`.agents/skills/` — 兼容性回退
4. `.claude/skills/` — Claude Code 兼容性回退（最低优先级）
5. 通过 `slate.json` 自定义路径

**Glob 模式：** `**/SKILL.md` 和 `{skill,skills}/**/SKILL.md`

**命令：** 相同目录结构，但位于 `commands/` 子目录下：
`/.slate/commands/`、`/.claude/commands/`、`/.agents/commands/`、`/.opencode/commands/`

**Skill frontmatter：** YAML 格式，包含 `name` 和 `description` 字段（根据 Slate 文档）。
两个字段均未记录长度限制。

## 项目指令

Slate 读取 `CLAUDE.md` 和 `AGENTS.md` 作为项目指令。
二进制文件中确认了两个字面字符串。现有 gstack 项目不需要任何更改……CLAUDE.md 可直接使用。

## 配置

**配置文件：** `slate.json` / `slate.jsonc`（NOT opencode.json）

**配置选项（来自 Slate 文档）：**
- `privacy`（布尔值）—— 禁用遥测/日志
- 权限：每个工具 `allow`、`ask`、`deny`（`read`、`edit`、`bash`、`grep`、`webfetch`、`websearch`、`*`）
- 模型槽位：`models.main`、`models.subagent`、`models.search`、`models.reasoning`
- MCP 服务器：本地或远程，带有自定义命令和头部
- 自定义命令：`/commands` 带有模板

安装脚本不应创建 `slate.json`。用户自行配置权限。

## CLI 标志（Headless 模式）

```
--stream-json / --output-format stream-json  — JSONL 输出，"兼容 Anthropic Claude Code SDK"
--dangerously-skip-permissions               — 绕过所有权限检查（CI/自动化）
--input-format stream-json                   — 程序化输入
-q                                           — 非交互模式
-w <dir>                                     — 工作目录
--output-format text                         — 纯文本输出（默认）
```

**Stream-JSON 格式：** Slate 文档声称"兼容 Anthropic Claude Code SDK"。
尚未经验证。鉴于 OpenCode 传承，可能匹配 Claude Code 的
NDJSON 事件 schema（type: "assistant"、type: "tool_result"、type: "result"）。

**需要验证：** 使用有效 credits 运行 `slate -q "hello" --stream-json` 并
捕获实际 JSONL 事件，再构建会话运行器解析器。

## 环境变量（来自二进制文件字符串）

### Slate 特定的
```
SLATE_API_KEY                              — API 密钥
SLATE_AGENT                                — agent 选择
SLATE_AUTO_SHARE                           — 自动共享设置
SLATE_CLIENT                               — 客户端标识符
SLATE_CONFIG                               — 配置覆盖
SLATE_CONFIG_CONTENT                       — 内联配置
SLATE_CONFIG_DIR                           — 配置目录
SLATE_DANGEROUSLY_SKIP_PERMISSIONS         — 绕过权限
SLATE_DIR                                  — 数据目录覆盖
SLATE_DISABLE_AUTOUPDATE                   — 禁用自动更新
SLATE_DISABLE_CLAUDE_CODE                  — 完全禁用 Claude Code 集成
SLATE_DISABLE_CLAUDE_CODE_PROMPT           — 禁用 Claude Code prompt 加载
SLATE_DISABLE_CLAUDE_CODE_SKILLS           — 禁用 .claude/skills/ 加载
SLATE_DISABLE_DEFAULT_PLUGINS              — 禁用默认插件
SLATE_DISABLE_FILETIME_CHECK               — 禁用文件时间检查
SLATE_DISABLE_LSP_DOWNLOAD                 — 禁用 LSP 自动下载
SLATE_DISABLE_MODELS_FETCH                 — 禁用模型配置获取
SLATE_DISABLE_PROJECT_CONFIG               — 禁用项目级配置
SLATE_DISABLE_PRUNE                        — 禁用会话修剪
SLATE_DISABLE_TERMINAL_TITLE               — 禁用终端标题更新
SLATE_ENABLE_EXA                           — 启用 Exa 搜索
SLATE_ENABLE_EXPERIMENTAL_MODELS           — 启用实验性模型
SLATE_EXPERIMENTAL                         — 启用实验性功能
SLATE_EXPERIMENTAL_BASH_DEFAULT_TIMEOUT_MS — bash 超时覆盖
SLATE_EXPERIMENTAL_DISABLE_COPY_ON_SELECT  — 禁用选择时复制
SLATE_EXPERIMENTAL_DISABLE_FILEWATCHER     — 禁用文件监视器
SLATE_EXPERIMENTAL_EXA                     — Exa 搜索（替代标志）
SLATE_EXPERIMENTAL_FILEWATCHER             — 启用文件监视器
SLATE_EXPERIMENTAL_ICON_DISCOVERY          — 图标发现
SLATE_EXPERIMENTAL_LSP_TOOL               — LSP 工具
SLATE_EXPERIMENTAL_LSP_TY                 — LSP 类型检查
SLATE_EXPERIMENTAL_MARKDOWN               — markdown 模式
SLATE_EXPERIMENTAL_OUTPUT_TOKEN_MAX       — 输出 token 限制
SLATE_EXPERIMENTAL_OXFMT                  — oxfmt 集成
SLATE_EXPERIMENTAL_PLAN_MODE              — plan 模式
SLATE_FAKE_VCS                            — 用于测试的假 VCS
SLATE_GIT_BASH_PATH                       — git bash 路径（Windows）
SLATE_MODELS_URL                          — 模型配置 URL
SLATE_PERMISSION                          — 权限覆盖
SLATE_SERVER_PASSWORD                     — 服务器认证
SLATE_SERVER_USERNAME                     — 服务器认证
SLATE_TELEMETRY_DISABLED                  — 禁用遥测
SLATE_TEST_HOME                           — 测试主目录
SLATE_TOKEN_DIR                           — token 存储目录
```

### OpenCode 遗留（仍然有效）
```
OPENCODE_DISABLE_LSP_DOWNLOAD
OPENCODE_EXPERIMENTAL_DISABLE_FILEWATCHER
OPENCODE_EXPERIMENTAL_FILEWATCHER
OPENCODE_EXPERIMENTAL_ICON_DISCOVERY
OPENCODE_EXPERIMENTAL_LSP_TY
OPENCODE_EXPERIMENTAL_OXFMT
OPENCODE_FAKE_VCS
OPENCODE_GIT_BASH_PATH
OPENCODE_LIBC
OPENCODE_TERMINAL
```

### gstack 集成的关键环境变量

**`SLATE_DISABLE_CLAUDE_CODE_SKILLS`** — 设置后，禁用 `.claude/skills/` 加载。
这使得发布到 `.slate/skills/` 变得关键，而不仅仅是优化。
如果没有本地的 `.slate/` 发布，设置此标志时 gstack skill 会消失。

**`SLATE_TEST_HOME`** — 对 E2E 测试有用。可以将 Slate 的主目录
重定向到隔离的临时目录，类似于 Codex 测试使用临时 HOME。

**`SLATE_DANGEROUSLY_SKIP_PERMISSIONS`** — 无头 E2E 测试所需。

## 模型引用（来自二进制文件）

```
anthropic/claude-sonnet-4.6
anthropic/claude-opus-4
anthropic/claude-haiku-4
anthropic/slate              — Slate 自己的模型路由
openai/gpt-5.3-codex
google/nano-banana
randomlabs/fast-default-alpha
```

## API 端点（来自二进制文件）

```
https://api.randomlabs.ai                          — 主 API
https://api.randomlabs.ai/exaproxy                 — Exa 搜索代理
https://agent-worker-prod.randomlabs.workers.dev   — 生产 worker
https://agent-worker-dev.randomlabs.workers.dev    — 开发 worker
https://dashboard.randomlabs.ai                    — 仪表板
https://docs.randomlabs.ai                         — 文档
https://randomlabs.ai/config.json                  — 远程配置
```

Brew tap：`anthropic/tap/slate`（注意：位于 Anthropic 的 tap 下，而非 Random Labs）

## npm 包结构

```
@randomlabs/slate（8.8 kB，精简启动器）
├── bin/slate           — Node.js 启动器（在 node_modules 中查找平台二进制文件）
├── bin/slate1          — Bun 启动器（相同逻辑，import.meta.filename）
├── postinstall.mjs     — 验证平台二进制文件是否存在，必要时创建符号链接
└── package.json        — 声明所有平台的 optionalDependencies

平台包（每个 85MB）：
├── @randomlabs/slate-darwin-arm64
├── @randomlabs/slate-darwin-x64
├── @randomlabs/slate-linux-arm64
├── @randomlabs/slate-linux-x64
├── @randomlabs/slate-linux-x64-musl
├── @randomlabs/slate-linux-arm64-musl
├── @randomlabs/slate-linux-x64-baseline
├── @randomlabs/slate-linux-x64-baseline-musl
├── @randomlabs/slate-darwin-x64-baseline
├── @randomlabs/slate-windows-x64
└── @randomlabs/slate-windows-x64-baseline
```

二进制覆盖：`SLATE_BIN_PATH` 环境变量跳过所有发现，直接运行指定的二进制文件。

## 目前哪些已经可用

gstack skill 已经通过 `.claude/skills/` 回退路径在 Slate 中工作。
基本功能不需要任何更改。为 Claude Code 安装 gstack 并同时使用 Slate 的用户会发现他们的 skill 在两个 agent 中都可用。

## 一等支持能带来什么

1. **可靠性** — `.slate/skills/` 是 Slate 的最高优先级路径。不受
   `SLATE_DISABLE_CLAUDE_CODE_SKILLS` 影响。
2. **优化的 frontmatter** — 剥离 Slate 不使用的 Claude 特定字段（allowed-tools、hooks、version）。
   只保留 `name` 和 `description`。
3. **安装脚本** — 自动检测 `slate` 二进制文件，将 skill 安装到 `~/.slate/skills/`。
4. **E2E 测试** — 验证 skill 在 Slate 直接调用时能正常工作。

## 受阻于：Host Config 重构

Codex 的外部意见 review 指出，将 Slate 作为第 4 个 host（在 Claude、Codex、Factory 之后）添加是"路径别名的 host 爆炸"。当前架构存在：

- `type Host = 'claude' | 'codex' | 'factory'` 中硬编码的 host 名称
- `transformFrontmatter()` 中每个 host 的分支，逻辑几乎重复
- `EXTERNAL_HOST_CONFIG` 中每个 host 的配置，模式相似
- 安装脚本中每个 host 的函数（`create_codex_runtime_root`、`link_codex_skill_dirs`）
- host 名称在 `bin/gstack-platform-detect`、`bin/gstack-uninstall`、`bin/dev-setup` 中重复

添加 Slate 意味着再次复制所有这些模式。将 host 重构为
数据驱动（配置对象而非 if/else 分支）将使 Slate 集成
变得简单，并使未来的 host（任何新的 OpenCode 分叉、任何新 agent）零成本。

### 计划中缺少的内容（由 Codex 识别）

- `lib/worktree.ts` 只复制 `.agents/`，不复制 `.slate/` — worktree 中的 E2E 测试将
  没有 Slate skill
- `bin/gstack-uninstall` 不知道 `.slate/`
- `bin/dev-setup` 没有为贡献者开发模式连接 `.slate/`
- `bin/gstack-platform-detect` 没有检测 Slate
- E2E 测试应设置 `SLATE_DISABLE_CLAUDE_CODE_SKILLS=1` 以证明 `.slate/` 路径
  确实有效（而不仅仅回退到 `.claude/`）

## Session Runner 设计（后续）

当 JSONL 格式验证后，session runner 应该：

- 启动：`slate -q "<prompt>" --stream-json --dangerously-skip-permissions -w <dir>`
- 解析：兼容 Claude Code SDK 的 NDJSON（假设，需要验证）
- Skill：安装到测试 fixture 的 `.slate/skills/`（而非 `.claude/skills/`）
- 认证：使用 `SLATE_API_KEY` 或现有的 `~/.slate/` 凭据
- 隔离：使用 `SLATE_TEST_HOME` 进行主目录隔离
- 超时：默认 300s（与 Codex 相同）

```typescript
export interface SlateResult {
  output: string;
  toolCalls: string[];
  tokens: number;
  exitCode: number;
  durationMs: number;
  sessionId: string | null;
  rawLines: string[];
  stderr: string;
}
```

## 文档引用

- Slate 文档：https://docs.randomlabs.ai
- 快速入门：https://docs.randomlabs.ai/en/getting-started/quickstart
- Skills：https://docs.randomlabs.ai/en/using-slate/skills
- 配置：https://docs.randomlabs.ai/en/using-slate/configuration
- 快捷键：https://docs.randomlabs.ai/en/using-slate/hotkey_reference
