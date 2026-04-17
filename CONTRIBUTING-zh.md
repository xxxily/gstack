# 为 gstack 做贡献

感谢你想让 gstack 变得更好。无论你是在修复 Skill 提示中的拼写错误，还是在构建全新的工作流，本指南都能帮你快速上手。

## 快速开始

gstack Skills 是 Claude Code 从 `skills/` 目录发现的 Markdown 文件。通常它们位于 `~/.claude/skills/gstack/`（你的全局安装）。但在开发 gstack 本身时，你希望 Claude Code 使用你**工作树中**的 Skills——这样编辑可以立即生效，无需复制或部署任何东西。

这就是开发模式的作用。它将你的仓库符号链接到本地 `.claude/skills/` 目录中，Claude Code 直接从你的检出中读取 Skills。

```bash
git clone <repo> && cd gstack
bun install                    # 安装依赖
bin/dev-setup                  # 激活开发模式
```

现在编辑任何 `SKILL.md`，在 Claude Code 中调用它（例如 `/review`），即可实时看到变更。开发完成后：

```bash
bin/dev-teardown               # 停用 — 回到全局安装
```

## 运营自我改进

gstack 自动从失败中学习。在每个 Skill 会话结束时，Agent 会反思出了什么问题（CLI 错误、错误方法、项目特点）并将运营学习记录到 `~/.gstack/projects/{slug}/learnings.jsonl`。未来会话自动展示这些学习成果，所以 gstack 会随着时间在你的代码库上变得更聪明。

无需设置。学习成果自动记录。通过 `/learn` 查看。

### 贡献者工作流

1. **正常使用 gstack** — 运营学习成果自动捕获
2. **检查你的学习成果：** `/learn` 或 `ls ~/.gstack/projects/*/learnings.jsonl`
3. **Fork 并克隆 gstack**（如果还没有的话）
4. **将你的 fork 符号链接到你遇到 Bug 的项目中：**
   ```bash
   # 在你的核心项目中（即 gstack 让你困扰的那个项目）
   ln -sfn /path/to/your/gstack-fork .claude/skills/gstack
   cd .claude/skills/gstack && bun install && bun run build && ./setup
   ```
   设置会为每个 Skill 创建带有 SKILL.md 符号链接的目录（`qa/SKILL.md -> gstack/qa/SKILL.md`）
   并询问你的前缀偏好。传递 `--no-prefix` 跳过提示并使用短名称。
5. **修复问题** — 你的变更在此项目中立即生效
6. **通过实际使用 gstack 测试** — 去做让你困扰的事，验证已修复
7. **从你的 fork 打开 PR**

这是最好的贡献方式：在做真实工作的同时修复 gstack，在真正感受到痛点的项目中。

### 会话感知

当你同时打开 3+ 个 gstack 会话时，每个问题都会告诉你属于哪个项目、哪个分支、以及发生了什么。所有 Skills 的格式都是一致的。

## 在 gstack 仓库内部开发 gstack

当你编辑 gstack Skills 并想通过在同一仓库中实际使用 gstack 来测试它们时，`bin/dev-setup` 会帮你完成。它创建 `.claude/skills/` 符号链接（被 gitignore），指回你的工作树，这样 Claude Code 使用你的本地编辑而不是全局安装。

```
gstack/                          <- 你的工作树
├── .claude/skills/              <- 由 dev-setup 创建（gitignore）
│   ├── gstack -> ../../         <- 符号链接回仓库根目录
│   ├── review/                  <- 真实目录（短名称，默认）
│   │   └── SKILL.md -> gstack/review/SKILL.md
│   ├── ship/                    <- 或 gstack-review/、gstack-ship/（如果使用了 --prefix）
│   │   └── SKILL.md -> gstack/ship/SKILL.md
│   └── ...                      <- 每个 Skill 一个目录
├── review/
│   └── SKILL.md                 <- 编辑这个，用 /review 测试
├── ship/
│   └── SKILL.md
├── browse/
│   ├── src/                     <- TypeScript 源代码
│   └── dist/                    <- 编译后的二进制文件（gitignore）
└── ...
```

设置创建真实目录（不是符号链接），内部带有 SKILL.md 符号链接。这确保 Claude 将它们发现为顶级 Skills，而非嵌套在 `gstack/` 下。名称取决于你的前缀设置（`~/.gstack/config.yaml`）。短名称（`/review`、`/ship`）是默认的。如果你偏好命名空间名称（`/gstack-review`、`/gstack-ship`），运行 `./setup --prefix`。

## 日常工作流

```bash
# 1. 进入开发模式
bin/dev-setup

# 2. 编辑 Skill
vim review/SKILL.md

# 3. 在 Claude Code 中测试 — 变更即刻生效
#    > /review

# 4. 编辑 browse 源代码？重新构建二进制文件
bun run build

# 5. 今天完成了？拆除
bin/dev-teardown
```

## 测试与评估

### 设置

```bash
# 1. 复制 .env.example 并添加你的 API Key
cp .env.example .env
# 编辑 .env → 设置 ANTHROPIC_API_KEY=sk-ant-...

# 2. 安装依赖（如果还没有安装）
bun install
```

Bun 自动加载 `.env` — 无需额外配置。Conductor 工作区的 `.env` 会自动从主工作区继承（参见下方的"Conductor 工作区"）。

### 测试级别

| 级别 | 命令 | 成本 | 测试内容 |
|------|---------|------|---------------|
| 1 — 静态 | `bun test` | 免费 | 命令验证、snapshot 标志、SKILL.md 正确性、TODOS-format.md 引用、可观测性单元测试 |
| 2 — E2E | `bun run test:e2e` | ~$3.85 | 通过 `claude -p` 子进程的完整 Skill 执行 |
| 3 — LLM 评估 | `bun run test:evals` | ~$0.15 独立运行 | LLM 法官对生成的 SKILL.md 文档评分 |
| 2+3 | `bun run test:evals` | ~$4 合计 | E2E + LLM 法官（同时运行两者） |

```bash
bun test                     # 仅第 1 级（每次提交运行，<5s）
bun run test:e2e             # 仅第 2 级：E2E（需要 EVALS=1，不能在 Claude Code 内部运行）
bun run test:evals           # 第 2 + 3 级合计（约 $4/次）
```

### 第 1 级：静态验证（免费）

通过 `bun test` 自动运行。不需要 API Key。

- **Skill 解析测试**（`test/skill-parser.test.ts`） — 从 SKILL.md bash 代码块中提取每个 `$B` 命令并针对 `browse/src/commands.ts` 中的命令注册表进行验证。捕获拼写错误、已移除的命令和无效的 snapshot 标志。
- **Skill 验证测试**（`test/skill-validation.test.ts`） — 验证 SKILL.md 文件仅引用真实的命令和标志，且命令描述满足质量阈值。
- **生成器测试**（`test/gen-skill-docs.test.ts`） — 测试模板系统：验证占位符正确解析，输出包含标志的值提示（例如 `-d <N>` 而不仅仅是 `-d`），关键命令的描述经过丰富（例如 `is` 列出有效状态，`press` 列出键示例）。

### 第 2 级：通过 `claude -p` 的 E2E（约 $3.85/次）

将 `claude -p` 作为子进程生成，参数为 `--output-format stream-json --verbose`，流式传输 NDJSON 获取实时进度，并扫描浏览错误。这是最接近"这个 Skill 是否真的端到端工作"的测试。

```bash
# 必须在普通终端中运行 — 不能嵌套在 Claude Code 或 Conductor 内部
EVALS=1 bun test test/skill-e2e-*.test.ts
```

- 由 `EVALS=1` 环境变量门控（防止意外产生昂贵的运行）
- 如果在 Claude Code 内部运行则自动跳过（`claude -p` 无法嵌套）
- API 连接预检查 — 在 ConnectionRefused 上快速失败，避免浪费预算
- 实时进度输出到 stderr：`[Ns] turn T tool #C: Name(...)`
- 保存完整的 NDJSON 转录和失败 JSON 用于调试
- 测试位于 `test/skill-e2e-*.test.ts`（按类别拆分），运行器逻辑在 `test/helpers/session-runner.ts`

### E2E 可观测性

E2E 测试运行时，会在 `~/.gstack-dev/` 中生成机器可读的产物：

| 产物 | 路径 | 用途 |
|----------|------|---------|
| 心跳 | `e2e-live.json` | 当前测试状态（每次工具调用更新） |
| 部分结果 | `evals/_partial-e2e.json` | 已完成的测试（在 kill 后仍存活） |
| 进度日志 | `e2e-runs/{runId}/progress.log` | 仅追加的文本日志 |
| NDJSON 转录 | `e2e-runs/{runId}/{test}.ndjson` | 每次测试的原始 `claude -p` 输出 |
| 失败 JSON | `e2e-runs/{runId}/{test}-failure.json` | 失败时的诊断数据 |

**实时仪表板：** 在第二个终端中运行 `bun run eval:watch` 查看实时仪表板，显示已完成的测试、当前正在运行的测试和成本。使用 `--tail` 同时显示 progress.log 的最后 10 行。

**评估历史工具：**

```bash
bun run eval:list            # 列出所有评估运行（每次运行的轮次、持续时间、成本）
bun run eval:compare         # 比较两次运行 — 显示每次测试的增量 + Takeaway 注释
bun run eval:summary         # 聚合统计 + 每次运行的效率平均值
```

**评估比较注释：** `eval:compare` 生成自然语言的 Takeaway 部分，解释运行之间的变化——标记回归、注意改进、指出效率提升（更少的轮次、更快、更便宜），并生成总体摘要。这由 `eval-store.ts` 中的 `generateCommentary()` 驱动。

产物永远不会被清理——它们累积在 `~/.gstack-dev/` 中用于事后调试和趋势分析。

### 第 3 级：LLM 法官（约 $0.15/次）

使用 Claude Sonnet 对生成的 SKILL.md 文档在三个维度上评分：

- **清晰度** — AI Agent 能否在没有歧义的情况下理解说明？
- **完整性** — 是否记录了所有命令、标志和用法模式？
- **可操作性** — Agent 能否仅使用文档中的信息执行任务？

每个维度得分 1-5。阈值：每个维度必须 **≥ 4**。还有一个回归测试，将生成的文档与 `origin/main` 中手动维护的基准进行比较——生成的必须得分相等或更高。

```bash
# 需要 .env 中的 ANTHROPIC_API_KEY — 包含在 bun run test:evals 中
```

- 使用 `claude-sonnet-4-6` 保持评分稳定
- 测试位于 `test/skill-llm-eval.test.ts`
- 直接调用 Anthropic API（不是 `claude -p`），所以可以在任何地方工作，包括 Claude Code 内部

### CI

GitHub Action（`.github/workflows/skill-docs.yml`）在每次推送和 PR 时运行 `bun run gen:skill-docs --dry-run`。如果生成的 SKILL.md 文件与已提交的不同，CI 失败。这在合并前捕获过时文档。

测试直接针对浏览二进制文件运行——它们不需要开发模式。

## 编辑 SKILL.md 文件

SKILL.md 文件是从 `.tmpl` 模板**自动生成**的。不要直接编辑 `.md`——你的变更会在下次构建时被覆盖。

```bash
# 1. 编辑模板
vim SKILL.md.tmpl              # 或 browse/SKILL.md.tmpl

# 2. 为所有主机重新生成
bun run gen:skill-docs --host all

# 3. 检查健康状态（报告所有主机）
bun run skill:check

# 或使用监听模式——保存时自动重新生成
bun run dev:skill
```

有关模板编写最佳实践（自然语言而非 Shell 习惯用法、动态分支检测、`{{BASE_BRANCH_DETECT}}` 用法），请参见 CLAUDE.md 的"编写 Skill 模板"部分。

添加浏览命令时，将其添加到 `browse/src/commands.ts`。添加 snapshot 标志时，将其添加到 `browse/src/snapshot.ts` 中的 `SNAPSHOT_FLAGS`。然后重新构建。

## 多主机开发

gstack 从一组 `.tmpl` 模板为 8 个主机生成 SKILL.md 文件。每个主机是 `hosts/*.ts` 中的类型化配置。生成器读取这些配置以生成适合主机的输出（不同的 frontmatter、路径、工具名称）。

**支持的主机：** Claude（主要）、Codex、Factory、Kiro、OpenCode、Slate、Cursor、OpenClaw。

### 为所有主机生成

```bash
# 为特定主机生成
bun run gen:skill-docs                    # Claude（默认）
bun run gen:skill-docs --host codex       # Codex
bun run gen:skill-docs --host opencode    # OpenCode
bun run gen:skill-docs --host all         # 全部 8 个主机

# 或使用 build，这会完成所有主机 + 编译二进制
bun run build
```

### 主机之间的差异

每个主机配置（`hosts/*.ts`）控制：

| 方面 | 示例（Claude vs Codex） |
|--------|---------------------------|
| 输出目录 | `{skill}/SKILL.md` vs `.agents/skills/gstack-{skill}/SKILL.md` |
| Frontmatter | 完整（name, description, hooks, version）vs 最小（name + description） |
| 路径 | `~/.claude/skills/gstack` vs `$GSTACK_ROOT` |
| 工具名称 | "use the Bash tool" vs 相同（Factory 重写为"run this command"） |
| Hook Skills | `hooks:` frontmatter vs 内联安全建议散文 |
| 抑制的部分 | 无 vs Codex 自我调用部分被剥离 |

参见 `scripts/host-config.ts` 获取完整的 `HostConfig` 接口。

### 测试主机输出

```bash
# 运行所有静态测试（包括所有主机的参数化冒烟测试）
bun test

# 检查所有主机的新鲜度
bun run gen:skill-docs --host all --dry-run

# 健康仪表板覆盖所有主机
bun run skill:check
```

### 添加新主机

参见 [docs/ADDING_A_HOST.md](docs/ADDING_A_HOST.md) 获取完整指南。简而言之：

1. 创建 `hosts/myhost.ts`（从 `hosts/opencode.ts` 复制）
2. 添加到 `hosts/index.ts`
3. 添加 `.myhost/` 到 `.gitignore`
4. 运行 `bun run gen:skill-docs --host myhost`
5. 运行 `bun test`（参数化测试自动覆盖）

零生成器、设置或工具代码变更需要。

### 添加新 Skill

添加新 Skill 模板时，所有主机会自动获得它：
1. 创建 `{skill}/SKILL.md.tmpl`
2. 运行 `bun run gen:skill-docs --host all`
3. 动态模板发现会拾取它，无需更新静态列表
4. 提交 `{skill}/SKILL.md`，外部主机输出在设置时生成并被 gitignore

## Conductor 工作区

如果你使用 [Conductor](https://conductor.build) 并行运行多个 Claude Code 会话，`conductor.json` 会自动设置工作区生命周期：

| 钩子 | 脚本 | 作用 |
|------|--------|-------------|
| `setup` | `bin/dev-setup` | 从主工作区复制 `.env`、安装依赖、符号链接 Skills |
| `archive` | `bin/dev-teardown` | 移除 Skill 符号链接、清理 `.claude/` 目录 |

当 Conductor 创建新工作区时，`bin/dev-setup` 自动运行。它检测主工作区（通过 `git worktree list`），复制你的 `.env` 以便 API Key 延续，并设置开发模式——无需手动步骤。

**首次设置：** 将 `ANTHROPIC_API_KEY` 放在主仓库的 `.env` 中（参见 `.env.example`）。每个 Conductor 工作区自动继承它。

## 注意事项

- **SKILL.md 文件是自动生成的。** 编辑 `.tmpl` 模板，而不是 `.md`。运行 `bun run gen:skill-docs` 重新生成。
- **TODOS.md 是统一的待办列表。** 按 Skill/组件组织，带有 P0-P4 优先级。`/ship` 自动检测已完成的项目。所有计划/审查/回顾 Skills 读取它作为上下文。
- **Browse 源代码变更需要重新构建。** 如果你修改了 `browse/src/*.ts`，运行 `bun run build`。
- **开发模式覆盖你的全局安装。** 项目本地 Skills 优先于 `~/.claude/skills/gstack`。`bin/dev-teardown` 恢复全局版本。
- **Conductor 工作区是独立的。** 每个工作区是自己的 git worktree。`bin/dev-setup` 通过 `conductor.json` 自动运行。
- **`.env` 跨工作区传播。** 在主仓库设置一次，所有 Conductor 工作区都会获得。
- **`.claude/skills/` 被 gitignore。** 符号链接永远不会被提交。

## 在真实项目中测试变更

**这是推荐的 gstack 开发方式。** 将你的 gstack 检出符号链接到你实际使用它的项目中，这样你在做真实工作时变更即刻生效。

### 第 1 步：符号链接你的检出

```bash
# 在你的核心项目中（不是 gstack 仓库）
ln -sfn /path/to/your/gstack-checkout .claude/skills/gstack
```

### 第 2 步：运行设置以创建每个 Skill 的符号链接

仅有 `gstack` 符号链接不够。Claude Code 通过单独的顶级目录（`qa/SKILL.md`、`ship/SKILL.md` 等）发现 Skills，而不是通过 `gstack/` 目录本身。运行 `./setup` 创建它们：

```bash
cd .claude/skills/gstack && bun install && bun run build && ./setup
```

设置会询问你想要短名称（`/qa`）还是命名空间名称（`/gstack-qa`）。你的选择保存到 `~/.gstack/config.yaml` 并在未来运行中记住。跳过提示时，传递 `--no-prefix`（短名称）或 `--prefix`（命名空间）。

### 第 3 步：开发

编辑模板，运行 `bun run gen:skill-docs`，下一次 `/review` 或 `/qa` 调用会立即拾取它。无需重启。

### 回到稳定的全局安装

移除项目本地的符号链接。Claude Code 回退到 `~/.claude/skills/gstack/`：

```bash
rm .claude/skills/gstack
```

每个 Skill 目录（`qa/`、`ship/` 等）包含指向 `gstack/...` 的 SKILL.md 符号链接，所以它们会自动解析到全局安装。

### 切换前缀模式

如果你用一个前缀设置安装了 gstack 并想切换：

```bash
cd .claude/skills/gstack && ./setup --no-prefix   # 切换到 /qa, /ship
cd .claude/skills/gstack && ./setup --prefix       # 切换到 /gstack-qa, /gstack-ship
```

设置自动清理旧符号链接。无需手动清理。

### 替代方案：将你的全局安装指向一个分支

如果你不想要每个项目的符号链接，可以切换全局安装：

```bash
cd ~/.claude/skills/gstack
git fetch origin
git checkout origin/<branch>
bun install && bun run build && ./setup
```

这影响所有项目。要恢复：`git checkout main && git pull && bun run build && ./setup`。

## 社区 PR 分级（Wave 流程）

当社区 PR 积累时，将它们分批处理为主题 Wave：

1. **分类** — 按主题分组（安全、功能、基础设施、文档）
2. **去重** — 如果两个 PR 修复同一件事，选择变更行数更少的那个。关闭另一个并附带指向获胜者的注释。
3. **Collector 分支** — 创建 `pr-wave-N`，合并干净的 PR，解决脏 PR 的冲突，通过 `bun test && bun run build` 验证
4. **附带上下文关闭** — 每个关闭的 PR 都获得一条评论，解释原因以及什么（如果有）替代了它。贡献者做了真实工作；用清晰的沟通尊重他们。
5. **作为一个 PR 发布** — 单个 PR 到 main，所有归属保留在合并提交中。包含合并了什么和关闭了什么的摘要表。

参见 [PR #205](../../pull/205)（v0.8.3）作为第一个 Wave 的示例。

## 升级迁移

当发布变更了磁盘状态（目录结构、配置格式、过期文件）且 `./setup` 本身无法修复时，添加迁移脚本以便现有用户获得干净的升级。

### 何时添加迁移

- 改变了 Skill 目录的创建方式（符号链接 vs 真实目录）
- 重命名或移动了 `~/.gstack/config.yaml` 中的配置键
- 需要删除先前版本的孤儿文件
- 改变了 `~/.gstack/` 状态文件的格式

不要为以下情况添加迁移：新功能（用户自动获得）、新 Skills（设置会发现它们）或仅代码变更（无磁盘状态）。

### 如何添加

1. 创建 `gstack-upgrade/migrations/v{VERSION}.sh`，其中 `{VERSION}` 匹配需要修复的发布的 VERSION 文件。
2. 使其可执行：`chmod +x gstack-upgrade/migrations/v{VERSION}.sh`
3. 脚本必须是**幂等的**（安全地多次运行）且**非致命的**（失败会被记录但不会阻止升级）。
4. 在顶部包含注释块，解释改变了什么、为什么需要迁移以及影响哪些用户。

示例：

```bash
#!/usr/bin/env bash
# 迁移：v0.15.2.0 — 修复 Skill 目录结构
# 影响：在 v0.15.2.0 之前使用 --no-prefix 安装的用户
set -euo pipefail
SCRIPT_DIR="$(cd "$(dirname "$0")/../.." && pwd)"
"$SCRIPT_DIR/bin/gstack-relink" 2>/dev/null || true
```

### 如何运行

在 `/gstack-upgrade` 期间，`./setup` 完成后（第 4.75 步），升级 Skill 扫描 `gstack-upgrade/migrations/` 并运行每个版本比用户旧版本新的 `v*.sh` 脚本。脚本按版本顺序运行。失败会被记录但永远不会阻止升级。

### 测试迁移

迁移作为 `bun test`（第 1 级，免费）的一部分进行测试。测试套件验证 `gstack-upgrade/migrations/` 中的所有迁移脚本是可执行的且无语法错误解析。

## 发布你的变更

当你对 Skill 编辑感到满意时：

```bash
/ship
```

这会运行测试、审查差异、分级 Greptile 评论（带 2 级升级）、管理 TODOS.md、升级版本并打开 PR。参见 `ship/SKILL.md` 获取完整工作流。
