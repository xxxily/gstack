# gstack 开发指南

## 命令

```bash
bun install          # 安装依赖
bun test             # 运行免费测试（browse + snapshot + Skill 验证）
bun run test:evals   # 运行付费评估：LLM 法官 + E2E（基于差异，约 $4/次上限）
bun run test:evals:all  # 运行所有付费评估，不管差异如何
bun run test:gate    # 仅运行门禁级测试（CI 默认，阻止合并）
bun run test:periodic  # 仅运行周期级测试（每周 cron / 手动）
bun run test:e2e     # 仅运行 E2E 测试（基于差异，约 $3.85/次上限）
bun run test:e2e:all # 运行所有 E2E 测试，不管差异如何
bun run eval:select  # 显示基于当前差异将运行哪些测试
bun run dev <cmd>    # 以开发模式运行 CLI，例如 bun run dev goto https://example.com
bun run build        # 生成文档 + 编译二进制文件
bun run gen:skill-docs  # 从模板重新生成 SKILL.md 文件
bun run skill:check  # 所有 Skills 的健康仪表板
bun run dev:skill    # 监听模式：变更时自动重新生成 + 验证
bun run eval:list    # 列出 ~/.gstack-dev/evals/ 中的所有评估运行
bun run eval:compare # 比较两次评估运行（自动选择最近的）
bun run eval:summary # 聚合所有评估运行的统计信息
bun run slop          # 完整的 slop-scan 报告（所有文件）
bun run slop:diff     # 仅在此分支变更的文件上的 slop 发现
```

`test:evals` 需要 `ANTHROPIC_API_KEY`。Codex E2E 测试（`test/codex-e2e.test.ts`）使用 Codex 自己的认证，来自 `~/.codex/` 配置——不需要 `OPENAI_API_KEY` 环境变量。E2E 测试实时流式传输进度（通过 `--output-format stream-json --verbose` 逐个工具）。结果持久化到 `~/.gstack-dev/evals/`，并与上一次运行自动对比。

**基于差异的测试选择：** `test:evals` 和 `test:e2e` 根据针对基础分支的 `git diff` 自动选择测试。每个测试在 `test/helpers/touchfiles.ts` 中声明其文件依赖。全局 touchfile 变更（session-runner、eval-store、touchfiles.ts 本身）触发所有测试。使用 `EVALS_ALL=1` 或 `:all` 脚本变体强制运行所有测试。运行 `eval:select` 预览将运行哪些测试。

**两级系统：** 测试在 `E2E_TIERS` 中（位于 `test/helpers/touchfiles.ts`）分类为 `gate` 或 `periodic`。CI 仅运行门禁测试（`EVALS_TIER=gate`）；周期测试通过 cron 每周或手动运行。使用 `EVALS_TIER=gate` 或 `EVALS_TIER=periodic` 过滤。添加新 E2E 测试时，请分类：
1. 安全护栏或确定性功能测试？ → `gate`
2. 质量基准、Opus 模型测试或非确定性？ → `periodic`
3. 需要外部服务（Codex、Gemini）？ → `periodic`

## 测试

```bash
bun test             # 每次提交前运行——免费，<2s
bun run test:evals   # 发布前运行——付费，基于差异（约 $4/次上限）
```

`bun test` 运行 Skill 验证、gen-skill-docs 质量检查和 browse 集成测试。`bun run test:evals` 运行 LLM 法官质量评估和通过 `claude -p` 的 E2E 测试。两者都必须在创建 PR 前通过。

## 项目结构

```
gstack/
├── browse/          # 无头浏览器 CLI（Playwright）
│   ├── src/         # CLI + 服务器 + 命令
│   │   ├── commands.ts  # 命令注册表（单一真相来源）
│   │   └── snapshot.ts  # SNAPSHOT_FLAG 元数据数组
│   ├── test/        # 集成测试 + fixtures
│   └── dist/        # 编译后的二进制文件
├── hosts/           # 类型化主机配置（每个 AI Agent 一个）
│   ├── claude.ts    # 主要主机配置
│   ├── codex.ts, factory.ts, kiro.ts  # 现有主机
│   ├── opencode.ts, slate.ts, cursor.ts, openclaw.ts  # 新主机
│   └── index.ts     # 注册表：导出全部，派生 Host 类型
├── scripts/         # 构建 + DX 工具
│   ├── gen-skill-docs.ts  # 模板 → SKILL.md 生成器（配置驱动）
│   ├── host-config.ts     # HostConfig 接口 + 验证器
│   ├── host-config-export.ts  # 安装脚本的 Shell 桥接
│   ├── host-adapters/     # 特定主机适配器（OpenClaw 工具映射）
│   ├── resolvers/   # 模板解析器模块（前言、设计、审查等）
│   ├── skill-check.ts     # 健康仪表板
│   └── dev-skill.ts       # 监听模式
├── test/            # Skill 验证 + 评估测试
│   ├── helpers/     # skill-parser.ts, session-runner.ts, llm-judge.ts, eval-store.ts
│   ├── fixtures/    # 真实数据 JSON、植入 Bug fixtures、评估基准
│   ├── skill-validation.test.ts  # 第 1 级：静态验证（免费，<1s）
│   ├── gen-skill-docs.test.ts    # 第 1 级：生成器质量（免费，<1s）
│   ├── skill-llm-eval.test.ts   # 第 3 级：LLM 法官（约 $0.15/次）
│   └── skill-e2e-*.test.ts       # 第 2 级：通过 claude -p 的 E2E（约 $3.85/次，按类别拆分）
├── qa-only/         # /qa-only Skill（仅报告 QA，不修复）
├── plan-design-review/  # /plan-design-review Skill（仅报告设计审计）
├── design-review/    # /design-review Skill（设计审计 + 修复循环）
├── ship/            # Ship 工作流 Skill
├── review/          # PR 审查 Skill
├── plan-ceo-review/ # /plan-ceo-review Skill
├── plan-eng-review/ # /plan-eng-review Skill
├── autoplan/        # /autoplan Skill（自动审查管线：CEO → 设计 → 工程）
├── benchmark/       # /benchmark Skill（性能回归检测）
├── canary/          # /canary Skill（部署后监控循环）
├── codex/           # /codex Skill（通过 OpenAI Codex CLI 的多 AI 第二意见）
├── land-and-deploy/ # /land-and-deploy Skill（合并 → 部署 → 金丝雀验证）
├── office-hours/    # /office-hours Skill（YC Office Hours — 创业诊断 + 构建者头脑风暴）
├── investigate/     # /investigate Skill（系统化根因调试）
├── retro/           # 回顾 Skill（包括 /retro global 跨项目模式）
├── bin/             # CLI 工具（gstack-repo-mode, gstack-slug, gstack-config 等）
├── document-release/ # /document-release Skill（发布后文档更新）
├── cso/             # /cso Skill（OWASP Top 10 + STRIDE 安全审计）
├── design-consultation/ # /design-consultation Skill（从零构建设计系统）
├── design-shotgun/  # /design-shotgun Skill（可视化设计探索）
├── open-gstack-browser/  # /open-gstack-browser Skill（启动 GStack Browser）
├── connect-chrome/  # 符号链接 → open-gstack-browser（向后兼容）
├── design/          # 设计二进制 CLI（GPT Image API）
│   ├── src/         # CLI + 命令（generate, variants, compare, serve 等）
│   ├── test/        # 集成测试
│   └── dist/        # 编译后的二进制文件
├── extension/       # Chrome 扩展（侧边栏 + 活动流 + CSS 检查器）
├── lib/             # 共享库（worktree.ts）
├── docs/designs/    # 设计文档
├── setup-deploy/    # /setup-deploy Skill（一次性部署配置）
├── .github/         # CI 工作流 + Docker 镜像
│   ├── workflows/   # evals.yml（Ubicloud 上的 E2E），skill-docs.yml，actionlint.yml
│   └── docker/      # Dockerfile.ci（预构建工具链 + Playwright/Chromium）
├── contrib/         # 贡献者专属工具（不为用户安装）
│   └── add-host/    # /gstack-contrib-add-host Skill
├── setup            # 一次性设置：构建二进制 + 符号链接 Skills
├── SKILL.md         # 从 SKILL.md.tmpl 生成（不要直接编辑）
├── SKILL.md.tmpl    # 模板：编辑此文件，然后运行 gen:skill-docs
├── ETHOS.md         # 构建者理念（煮沸海洋、先搜索再构建）
└── package.json     # browse 的构建脚本
```

## SKILL.md 工作流

SKILL.md 文件是从 `.tmpl` 模板**自动生成**的。要更新文档：

1. 编辑 `.tmpl` 文件（例如 `SKILL.md.tmpl` 或 `browse/SKILL.md.tmpl`）
2. 运行 `bun run gen:skill-docs`（或 `bun run build`，会自动执行）
3. 提交 `.tmpl` 和生成的 `.md` 文件

添加新的 browse 命令：将其添加到 `browse/src/commands.ts` 并重新构建。
添加 snapshot 标志：将其添加到 `browse/src/snapshot.ts` 中的 `SNAPSHOT_FLAGS` 并重新构建。

**Token 上限：** 生成的 SKILL.md 文件必须保持在 100KB（约 25K token）以下。
`gen-skill-docs` 会在任何文件超出时发出警告。如果 Skill 模板超过上限，考虑将可选部分提取到单独的解析器中，仅在相关时注入，或使冗长的评估规则更简洁。

**SKILL.md 文件上的合并冲突：** 永远不要通过接受任一方来解决生成的 SKILL.md 文件上的冲突。而是：（1）解决 `.tmpl` 模板和 `scripts/gen-skill-docs.ts`（真相来源）上的冲突，（2）运行 `bun run gen:skill-docs` 重新生成所有 SKILL.md 文件，（3）暂存重新生成的文件。接受一方的生成输出会悄无声息地丢弃另一方模板的变更。

## 跨平台设计

Skills 绝对不能硬编码特定框架的命令、文件模式或目录结构。而是：

1. **读取 CLAUDE.md** 获取项目特定配置（测试命令、评估命令等）
2. **如果缺失，AskUserQuestion** — 让用户告诉你或者让 gstack 搜索仓库
3. **将答案持久化到 CLAUDE.md** 这样我们就无需再次询问

这适用于测试命令、评估命令、部署命令以及任何其他项目特定行为。项目拥有自己的配置；gstack 读取它。

## 编写 Skill 模板

SKILL.md.tmpl 文件是**由 Claude 读取的提示模板**，不是 bash 脚本。
每个 bash 代码块在单独的 Shell 中运行——变量不会在代码块之间持久化。

规则：
- **使用自然语言表达逻辑和状态。** 不要用 Shell 变量在代码块之间传递状态。而是告诉 Claude 要记住什么，并在散文中引用它（例如，"第 0 步中检测到的基础分支"）。
- **不要硬编码分支名。** 通过 `gh pr view` 或 `gh repo view` 动态检测 `main`/`master` 等。使用 `{{BASE_BRANCH_DETECT}}` 用于 PR 定向 Skills。在散文中使用"基础分支"，在代码块占位符中使用 `<base>`。
- **保持 bash 代码块自包含。** 每个代码块应能独立工作。如果代码块需要来自上一步的上下文，请在上面的散文中重述。
- **用英语表达条件。** 与其在 bash 中嵌套 `if/elif/else`，不如编写编号的决策步骤："1. 如果 X，执行 Y。2. 否则，执行 Z。"

## 浏览器交互

当你需要与浏览器交互时（QA、试用、Cookie 设置），使用 `/browse` Skill 或直接通过 `$B <command>` 运行 browse 二进制文件。绝对不要使用 `mcp__claude-in-chrome__*` 工具——它们慢、不可靠，而且不是本项目使用的。

**侧边栏架构：** 在修改 `sidepanel.js`、`background.js`、`content.js`、`sidebar-agent.ts` 或与侧边栏相关的服务器端点之前，请阅读 `docs/designs/SIDEBAR_MESSAGE_FLOW.md`。它记录了完整的初始化时间线、消息流、认证令牌链、标签并发模型和已知故障模式。侧边栏跨越 2 个代码库的 5 个文件（扩展 + 服务器），带有不明显的排序依赖。文档的存在是为了防止因不理解跨组件流而导致的静默失败。

## 开发符号链接意识

在开发 gstack 时，`.claude/skills/gstack` 可能是指向此工作目录的符号链接（被 gitignore）。这意味着 Skill 的变更会**立即生效**，非常适合快速迭代，但在大型重构期间有风险，因为写了一半的 Skill 可能会破坏其他同时使用 gstack 的 Claude Code 会话。

**每次会话检查一次：** 运行 `ls -la .claude/skills/gstack` 查看是符号链接还是真实副本。如果是符号链接指向你的工作目录，请注意：
- 模板变更 + `bun run gen:skill-docs` 立即影响所有 gstack 调用
- SKILL.md.tmpl 文件的破坏性变更可能破坏并发的 gstack 会话
- 大型重构期间，移除符号链接（`rm .claude/skills/gstack`），改为使用全局安装 `~/.claude/skills/gstack/`

**前缀设置：** 设置创建真实目录（不是符号链接），顶层带有 SKILL.md 符号链接（例如 `qa/SKILL.md -> gstack/qa/SKILL.md`）。这确保 Claude 将它们发现为顶级 Skills，而非嵌套在 `gstack/` 下。名称可以是短名（`qa`）或命名空间名（`gstack-qa`），由 `~/.gstack/config.yaml` 中的 `skill_prefix` 控制。传递 `--no-prefix` 或 `--prefix` 跳过交互式提示。

**注意：** 将 gstack 嵌入项目仓库已被废弃。使用全局安装 + `./setup --team`。参见 README.md 获取团队模式说明。

**用于计划审查：** 审查修改 Skill 模板或 gen-skill-docs 管线的计划时，考虑变更是否应在上线前独立测试（尤其当用户在其他窗口中正在使用 gstack 时）。

**升级迁移：** 当变更修改磁盘状态（目录结构、配置格式、过期文件）且可能破坏现有用户安装时，将迁移脚本添加到 `gstack-upgrade/migrations/`。阅读 CONTRIBUTING.md 的"升级迁移"部分了解格式和测试要求。升级 Skill 在 `/gstack-upgrade` 期间 `./setup` 后自动运行这些脚本。

## 编译后的二进制文件——永远不要提交 browse/dist/ 或 design/dist/

`browse/dist/` 和 `design/dist/` 目录包含编译后的 Bun 二进制文件（`browse`、`find-browse`、`design`，每个约 58MB）。这些仅为 Mach-O arm64——在 Linux、Windows 或 Intel Mac 上**不**工作。`./setup` 脚本已经为每个平台从源代码构建，所以检入的二进制文件是多余的。由于历史失误，它们被 git 追踪，最终应该用 `git rm --cached` 移除。

**永远不要暂存或提交这些文件。** 尽管 `.gitignore` 存在，它们仍会在 `git status` 中显示为已修改——忽略它们。暂存文件时，始终使用特定文件名（`git add file1 file2`）——永远不要使用 `git add .` 或 `git add -A`，这会意外包含二进制文件。

## 提交风格

**始终可二分提交。** 每次提交应该是单个逻辑变更。当你做了多个变更（例如重命名 + 重写 + 新测试）时，在推送前将它们拆分为单独的提交。每次提交应可独立理解和回滚。

良好二分的示例：
- 重命名/移动与行为变更分开
- 测试基础设施（touchfiles、helpers）与测试实现分开
- 模板变更与生成文件重新生成分开
- 机械重构与新功能分开

当用户说"二分提交"或"二分并推送"时，将暂存/未暂存的变更拆分为逻辑提交并推送。

## Slop-scan：AI 代码质量，不是隐藏 AI 代码

我们使用 [slop-scan](https://github.com/benvinegar/slop-scan) 来捕获 AI 生成代码确实比人类写的更差的模式。我们不是试图伪装成人类代码。我们是 AI 编程并为此自豪。目标是代码质量。

```bash
npx slop-scan scan .          # 人类可读报告
npx slop-scan scan . --json   # 机器可读用于比较
```

配置：仓库根目录下的 `slop-scan.config.json`（当前排除 `**/vendor/**`）。

### 修复什么（真正的质量改进）

- **文件操作周围的空 catch** — 使用 `safeUnlink()`（忽略 ENOENT，重新抛出 EPERM/EIO）。清理中被吞掉的 EPERM 意味着静默数据丢失。
- **进程 kill 周围的空 catch** — 使用 `safeKill()`（忽略 ESRCH，重新抛出 EPERM）。被吞掉的 EPERM 意味着你以为 kill 了其实没有。
- **冗余的 `return await`** — 当没有封闭的 try 块时移除。节省一个微任务，表明意图。
- **类型化异常捕获** — `catch (err) { if (!(err instanceof TypeError)) throw err }` 在 try 块执行 URL 解析或 DOM 操作时确实比 `catch {}` 更好。你知道你期望什么错误，所以明确说出来。

### 不修复什么（刷检查器，不是质量）

- **字符串匹配错误消息** — `err.message.includes('closed')` 很脆弱。Playwright/Chrome 可以随时改变措辞。如果即发即弃的操作可以因任何原因失败而你不在乎，`catch {}` 是正确的模式。
- **为豁免透传包装器添加注释** — 方法上方的"主动会话的别名"只是为了触发 slop-scan 的豁免规则，是噪音而非文档。
- **将扩展的 catch-and-log 转换为选择性重抛** — Chrome 扩展在未捕获错误时会完全崩溃。如果 catch 记录日志并继续，那对扩展代码就是正确的模式。不要让它抛出。
- **收紧尽力而为的清理路径** — 关闭、紧急清理和断开连接的代码应该使用 `safeUnlinkQuiet()`（吞掉所有错误）。在 EPERM 上抛出的清理路径意味着剩余的清理不会运行。这更糟。

### `browse/src/error-handling.ts` 中的工具函数

| 函数 | 何时使用 | 行为 |
|----------|----------|----------|
| `safeUnlink(path)` | 普通文件删除 | 忽略 ENOENT，重新抛出其他 |
| `safeUnlinkQuiet(path)` | 关闭/紧急清理 | 吞掉所有错误 |
| `safeKill(pid, signal)` | 发送信号 | 忽略 ESRCH，重新抛出其他 |
| `isProcessAlive(pid)` | 布尔进程检查 | 返回 true/false，永不抛出 |

### 分数追踪

基准（2026-04-09，清理前）：100 个发现，432.8 分，2.38 分/文件。
清理后：90 个发现，358.1 分，1.96 分/文件。

不要追逐数字。修复代表真正代码质量问题的模式。接受"懒散"模式是正确工程选择的那些发现。

## 社区 PR 护栏

在审查或合并社区 PR 时，**始终使用 AskUserQuestion** 之后才能接受任何提交：

1. **触碰 ETHOS.md** — 这个文件是 Garry 的个人构建理念。不接受来自外部贡献者或 AI Agent 的编辑，绝不。
2. **移除或弱化宣传材料** — YC 引用、创始人视角和产品声音是有意为之。将这些框架定为"不必要"或"太宣传化"的 PR 必须被拒绝。
3. **改变 Garry 的声音** — Skill 模板、CHANGELOG 和文档中的语气、幽默、直接性和观点不是通用的。将声音改写得更"中立"或"专业"的 PR 必须被拒绝。

即使 Agent 强烈认为某个变更改善了项目，这三个类别也需要通过 AskUserQuestion 获得明确的用户批准。没有例外。没有自动合并。没有"我只是清理一下"。

## CHANGELOG + VERSION 风格

**VERSION 和 CHANGELOG 是分支范围的。** 每个发布的功能分支都有自己的版本升级和 CHANGELOG 条目。条目描述此分支添加的内容——不是已经在 main 上的内容。

**何时编写 CHANGELOG 条目：**
- 在 `/ship` 时（第 5 步），不在开发过程中或分支中间。
- 条目涵盖与基础分支相比此分支上的所有提交。
- 永远不要将新工作折叠到先前版本中已经合入 main 的现有 CHANGELOG 条目。如果 main 有 v0.10.0.0 而你的分支添加功能，升级到 v0.10.1.0 并创建新条目——不要编辑 v0.10.0.0 条目。

**编写前的关键问题：**
1. 我在哪个分支？此分支改变了什么？
2. 基础分支的版本已经发布了吗？（如果是，升级并创建新条目。）
3. 此分支上是否已有条目覆盖了早期工作？（如果是，用最终版本的一个统一条目替换它。）

**合并 main 并不意味着采用 main 的版本。** 当你在功能分支中合并 origin/main 时，main 可能带来新的 CHANGELOG 条目和更高的 VERSION。你的分支仍然需要自己的版本升级。如果 main 在 v0.13.8.0 而你的分支添加功能，升级到 v0.13.9.0 并创建新条目。永远不要将你的变更塞入已经合入 main 的条目中。你的条目放在最上面，因为你的分支接下来合入。

**合并 main 后始终检查：**
- CHANGELOG 是否有你分支自己的条目，与 main 的条目分开？
- VERSION 是否高于 main 的 VERSION？
- 你的条目是否是 CHANGELOG 中最顶部的条目（在 main 的最新条目之上）？
如果任何答案是否定的，在继续之前修复它。

**在移动、添加或移除条目后编辑 CHANGELOG 后，** 立即运行 `grep "^## \[" CHANGELOG.md` 并验证完整的版本序列是连续的，没有间隔或重复，然后再提交。如果缺少版本，编辑出了问题。修复后再继续。

CHANGELOG.md 是**为用户**写的，不是为贡献者。像产品发布说明一样编写：

- 以用户现在**能做**而以前不能的事开头。推销这个功能。
- 使用通俗语言，不是实现细节。"你现在可以……"不是"重构了……"
- **永远不要提及 TODOS.md、内部追踪、评估基础设施或面向贡献者的细节。** 这些对用户不可见，对他们无意义。
- 将贡献者/内部变更放在底部的"For contributors"单独部分。
- 每个条目应让人想"哦不错，我想试试"。
- 没有行话：说"每个问题现在告诉你你在哪个项目和分支"而不是"通过前言解析器在所有 Skill 模板中标准化 AskUserQuestion 格式"。

## AI 努力压缩

在估算或讨论工作量时，始终同时显示人类团队和 CC+gstack 时间：

| 任务类型 | 人类团队 | CC+gstack | 压缩比 |
|-----------|-----------|-----------|-------------|
| 样板代码 / 脚手架 | 2 天 | 15 分钟 | ~100x |
| 编写测试 | 1 天 | 15 分钟 | ~50x |
| 功能实现 | 1 周 | 30 分钟 | ~30x |
| 修复 Bug + 回归测试 | 4 小时 | 15 分钟 | ~20x |
| 架构 / 设计 | 2 天 | 4 小时 | ~5x |
| 研究 / 探索 | 1 天 | 3 小时 | ~3x |

完整性很便宜。当完整实现是"湖"（可实现）而不是"海"（跨季度迁移）时，不要推荐捷径。查看 Skill 前言中的完整性原则获取完整理念。

## 先搜索再构建

在设计任何涉及并发、不熟悉模式、基础设施或运行时/框架可能有内置方案的解决方案之前：

1. 搜索"{runtime} {thing} 内置"
2. 搜索"{thing} 最佳实践 {当前年份}"
3. 检查官方运行时/框架文档

三层知识：久经考验的（第 1 层）、新潮热门的（第 2 层）、第一性原理（第 3 层）。最推崇第 3 层。参见 ETHOS.md 获取完整的构建理念。

## 本地计划

贡献者可以在 `~/.gstack-dev/plans/` 中存储长期愿景文档和设计文档。这些仅本地（不签入）。审查 TODOS.md 时，检查 `plans/` 中是否有准备提升为 TODOS 或实现的候选。

## E2E 评估失败归责协议

当在 `/ship` 或任何其他工作流中 E2E 评估失败时，**永远不要在证明之前声称"与我们的变更无关"。** 这些系统存在不可见的耦合——前言文本变更影响 Agent 行为，新辅助函数改变时序，重新生成的 SKILL.md 转移提示上下文。

**在归因于"预先存在"之前必须：**
1. 在 main（或基础分支）上运行相同的评估，并证明在那里也失败
2. 如果 main 上通过但在分支上失败——那就是你的变更。追踪归责。
3. 如果无法在 main 上运行，说"未验证——可能与也可能不相关"并在 PR 正文中将其标记为风险

没有可靠依据的"预先存在"是懒惰的声称。证明它或者不要说。

## 长时间运行的任务：不要放弃

在运行评估、E2E 测试或任何长时间运行的后台任务时，**轮询直到完成**。每 3 分钟使用 `sleep 180 && echo "ready"` + `TaskOutput` 循环。永远不要切换到阻塞模式并在轮询超时时放弃。永远不要说"完成时我会收到通知"然后停止检查——保持循环直到任务完成或用户告诉你停止。

完整的 E2E 套件可能需要 30-45 分钟。那是 10-15 个轮询周期。全部执行。每次检查报告进度（哪些测试通过、哪些正在运行、目前有哪些失败）。用户希望看到运行完成，而不是你稍后检查的承诺。

## E2E 测试 fixtures：提取，不要复制

**永远不要将完整的 SKILL.md 文件复制到 E2E 测试 fixture 中。** SKILL.md 文件有 1500-2000 行。当 `claude -p` 读取这么大的文件时，上下文膨胀导致超时、不稳定的轮次限制，以及比必要慢 5-10 倍的测试。

而是只提取测试实际需要的部分：

```typescript
// 错误 — Agent 读取 1900 行，在不相关的部分浪费 token
fs.copyFileSync(path.join(ROOT, 'ship', 'SKILL.md'), path.join(dir, 'ship-SKILL.md'));

// 正确 — Agent 读取约 60 行，38 秒完成而不是超时
const full = fs.readFileSync(path.join(ROOT, 'ship', 'SKILL.md'), 'utf-8');
const start = full.indexOf('## Review Readiness Dashboard');
const end = full.indexOf('\n---\n', start);
fs.writeFileSync(path.join(dir, 'ship-SKILL.md'), full.slice(start, end > start ? end : undefined));
```

运行针对性 E2E 测试调试失败时：
- 在**前台**运行（`bun test ...`），而不是后台使用 `&` 和 `tee`
- 永远不要 `pkill` 正在运行的评估进程然后重启——你会丢失结果并浪费钱
- 一次干净的运行胜过三次杀死并重试的运行

## 发布原生 OpenClaw Skills 到 ClawHub

原生 OpenClaw Skills 位于 `openclaw/skills/gstack-openclaw-*/SKILL.md`。这些是手工编写的方法论 Skill（不是由管线生成的），发布到 ClawHub，以便任何 OpenClaw 用户都可以安装它们。

**发布：** 命令是 `clawhub publish`（不是 `clawhub skill publish`）：

```bash
clawhub publish openclaw/skills/gstack-openclaw-office-hours \
  --slug gstack-openclaw-office-hours --name "gstack Office Hours" \
  --version 1.0.0 --changelog "description of changes"
```

对每个 Skill 重复：`gstack-openclaw-ceo-review`、`gstack-openclaw-investigate`、`gstack-openclaw-retro`。每次更新时升级 `--version`。

**认证：** `clawhub login`（打开浏览器进行 GitHub 认证）。`clawhub whoami` 进行验证。

**更新：** 相同的 `clawhub publish` 命令使用更高的 `--version` 和 `--changelog`。

**验证：** `clawhub search gstack` 确认它们已上线。

## 部署到活跃的 Skill

活跃的 Skill 位于 `~/.claude/skills/gstack/`。发生变更后：

1. 推送你的分支
2. 在 Skill 目录中获取并重置：`cd ~/.claude/skills/gstack && git fetch origin && git reset --hard origin/main`
3. 重新构建：`cd ~/.claude/skills/gstack && bun run build`

或直接复制二进制文件：
- `cp browse/dist/browse ~/.claude/skills/gstack/browse/dist/browse`
- `cp design/dist/design ~/.claude/skills/gstack/design/dist/design`
