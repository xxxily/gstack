# gstack

> "我认为从去年十二月以来，我可能连一行代码都没敲过，这是一个极其巨大的变化。" —— [Andrej Karpathy](https://fortune.com/2026/03/21/andrej-karpathy-openai-cofounder-ai-agents-coding-state-of-psychosis-openclaw/)，No Priors 播客，2026 年 3 月

听到 Karpathy 这么说，我想知道是怎么做到的。一个人如何像一个 20 人团队那样交付？Peter Steinberger 构建了 [OpenClaw](https://github.com/openclaw/openclaw)—— 247K GitHub 星——基本上独自用 AI agent 完成。革命已经到来。一个拥有正确工具链的构建者可以比传统团队更快。

我是 [Garry Tan](https://x.com/garrytan)，[Y Combinator](https://www.ycombinator.com/) 的总裁兼 CEO。我与数千家初创公司合作过——Coinbase、Instacart、Rippling——当它们还只是一两个人在车库里的时候。在加入 YC 之前，我是 Palantir 最早的 eng/PM/designer 之一，共同创立了 Posterous（被 Twitter 收购），并构建了 Bookface，YC
的內部社交网络。

**gstack 就是我的答案。** 我做产品二十年了，现在我交付的代码比任何时候都多。过去 60 天：**600,000+ 行生产代码**（35% 测试），**每天 10,000-20,000 行**，兼职，同时全职运营 YC。以下是我最近 `/retro` 在 3 个项目上的数据：**一周内新增 140,751 行，362 个提交，净增约 115k 行 LOC**。

**2026 年——1,237 次提交且还在增加：**

![GitHub contributions 2026 — 1,237 contributions, 1-3 月大幅加速](docs/images/github-2026.png)

**2013 年——我在 YC 构建 Bookface 时（772 次提交）：**

![GitHub contributions 2013 — 在 YC 构建 Bookface 时 772 次提交](docs/images/github-2013.png)

同一个人。不同的时代。区别在于工具链。

**gstack 就是我的方法论。** 它把 Claude Code 变成一个虚拟工程团队——重新思考产品的 CEO、锁定架构的工程经理、捕捉 AI slop 的设计师、发现生产 bug 的评审员、打开真实浏览器做测试的 QA 主管、做 OWASP + STRIDE 审计的安全官，以及发布 PR 的发布工程师。23 个专家和 8 个强效工具，全部是斜杠命令、全部是 Markdown、全部免费、MIT 许可。

这是我的开源软件工厂。我每天都在用。我分享它，因为这些工具应该对所有人可用。

Fork 它。改进它。让它成为你的。如果你想吐槽免费开源软件——随你便，但我建议你先试试。

**适合谁：**
- **创始人和 CEO**——尤其是还想亲自交付的技术型创始人
- **Claude Code 首次用户**——结构化的角色而不是空白的 prompt
- **Tech lead 和 staff 工程师**——每个 PR 上的严格评审、QA 和发布自动化

## 快速开始

1. 安装 gstack（30 秒——见下面）
2. 运行 `/office-hours`——描述你在构建什么
3. 对任何功能想法运行 `/plan-ceo-review`
4. 对有变更的任何 branch 运行 `/review`
5. 在你的 staging URL 上运行 `/qa`
6. 到这里就行了。你会知道这是否适合你。

## 安装 —— 30 秒

**要求：** [Claude Code](https://docs.anthropic.com/en/docs/claude-code)、[Git](https://git-scm.com/)、[Bun](https://bun.sh/) v1.0+、[Node.js](https://nodejs.org/)（仅 Windows）

### 步骤 1：在你的机器上安装

打开 Claude Code 并粘贴这个。Claude 会完成剩下的。

> 安装 gstack：运行 **`git clone --single-branch --depth 1 https://github.com/garrytan/gstack.git ~/.claude/skills/gstack && cd ~/.claude/skills/gstack && ./setup`** 然后向 CLAUDE.md 添加一个 "gstack" 部分，说明对所有网页浏览使用 gstack 的 /browse 技能，永远不要使用 mcp__claude-in-chrome__* 工具，并列出可用的技能：/office-hours、/plan-ceo-review、/plan-eng-review、/plan-design-review、/design-consultation、/design-shotgun、/design-html、/review、/ship、/land-and-deploy、/canary、/benchmark、/browse、/connect-chrome、/qa、/qa-only、/design-review、/setup-browser-cookies、/setup-deploy、/retro、/investigate、/document-release、/codex、/cso、/autoplan、/plan-devex-review、/devex-review、/careful、/freeze、/guard、/unfreeze、/gstack-upgrade、/learn。然后询问用户是否也想将 gstack 添加到当前项目，以便团队成员也能获得。

### 步骤 2：Team 模式——共享 Repo 的自动更新（推荐）

每个开发者全局安装，更新自动发生：

```bash
cd ~/.claude/skills/gstack && ./setup --team
```

然后引导你的 Repo 让团队成员获得：

```bash
cd <your-repo>
~/.claude/skills/gstack/bin/gstack-team-init required  # 或：optional
git add .claude/ CLAUDE.md && git commit -m "require gstack for AI-assisted work"
```

你的 Repo 中没有 vendored 文件，没有版本漂移，没有手动升级。每次 Claude Code session 都以快速自动更新检查开始（节流到每小时一次、网络故障安全、完全静默）。

> **想贡献或需要完整历史？** 上面的命令使用 `--depth 1` 快速安装。如果你计划贡献或需要完整 git 历史，改为完整克隆：
> ```bash
> git clone https://github.com/garrytan/gstack.git ~/.claude/skills/gstack
> ```

### OpenClaw

OpenClaw 通过 ACP 生成 Claude Code session，所以当 Claude Code 安装了 gstack 时，所有 gstack 技能都能正常工作。将此粘贴到你的 OpenClaw agent：

> 安装 gstack：运行 `git clone --single-branch --depth 1 https://github.com/garrytan/gstack.git ~/.claude/skills/gstack && cd ~/.claude/skills/gstack && ./setup` 为 Claude Code 安装 gstack。然后在 AGENTS.md 中添加一个 "Coding Tasks" 部分，说明：在为编码工作生成 Claude Code session 时，告诉 session 使用 gstack 技能。包含这些示例——安全审计："Load gstack. Run /cso"，代码评审："Load gstack. Run /review"，QA 测试 URL："Load gstack. Run /qa https://..."，端到端构建一个功能："Load gstack. Run /autoplan, implement the plan, then run /ship"，构建前先规划："Load gstack. Run /office-hours then /autoplan. Save the plan, don't implement."

**安装后，只需自然地与你的 OpenClaw agent 对话：**

| 你说 | 会发生什么 |
|---------|-------------|
| "修复 README 中的错字" | 简单——Claude Code session，不需要 gstack |
| "在这个 repo 上做安全审计" | 生成 Claude Code 并带 `Run /cso` |
| "给我做一个通知功能" | 生成 Claude Code 并带 /autoplan → implement → /ship |
| "帮我规划 v2 API 重新设计" | 生成 Claude Code 并带 /office-hours → /autoplan，保存计划 |

参见 [docs/OPENCLAW.md](docs/OPENCLAW.md) 了解高级分派路由和 gstack-lite/gstack-full prompt 模板。

### 原生 OpenClaw 技能（通过 ClawHub）

四个方法学技能，可以直接在你的 OpenClaw agent 中工作，不需要 Claude Code session。从 ClawHub 安装：

```
clawhub install gstack-openclaw-office-hours gstack-openclaw-ceo-review gstack-openclaw-investigate gstack-openclaw-retro
```

| Skill | 它做什么 |
|-------|-------------|
| `gstack-openclaw-office-hours` | 产品质询，6 个强制问题 |
| `gstack-openclaw-ceo-review` | 战略挑战，4 个范围模式 |
| `gstack-openclaw-investigate` | 根因调试方法 |
| `gstack-openclaw-retro` | 每周工程回顾 |

这些是对话式技能。你的 OpenClaw agent 通过聊天直接运行它们。

### 其他 AI Agent

gstack 在 8 个 AI 编码 agent 上工作，不仅仅是 Claude。安装自动检测你安装了哪些 agent：

```bash
git clone --single-branch --depth 1 https://github.com/garrytan/gstack.git ~/gstack
cd ~/gstack && ./setup
```

或用 `./setup --host <name>` 指定特定 agent：

| Agent | 标志 | 安装到 |
|-------|------|-------------------|
| OpenAI Codex CLI | `--host codex` | `~/.codex/skills/gstack-*/` |
| OpenCode | `--host opencode` | `~/.config/opencode/skills/gstack-*/` |
| Cursor | `--host cursor` | `~/.cursor/skills/gstack-*/` |
| Factory Droid | `--host factory` | `~/.factory/skills/gstack-*/` |
| Slate | `--host slate` | `~/.slate/skills/gstack-*/` |
| Kiro | `--host kiro` | `~/.kiro/skills/gstack-*/` |

**想添加对另一个 agent 的支持？** 参见 [docs/ADDING_A_HOST.md](docs/ADDING_A_HOST.md)。
只需一个 TypeScript 配置文件，零代码更改。

## 看它工作

```
你：    我想做一个日历的每日简报应用。
你：    /office-hours
Claude: [询问痛点——具体例子，不是假设]

你：    多个 Google 日历、信息过期的活动、错误的位置。
        准备时间太长，结果也不够好...

Claude: 我要推翻你的框架。你说的"每日简报
        应用"。但你实际描述的是一个个人首席
        参谋 AI。
        [提取了 5 个你没意识到的能力]
        [挑战 4 个前提——你同意、不同意或调整]
        [生成 3 个实现方法并附工作量估算]
        推荐：明天交付最窄的切入点，从
        实际使用中学习。完整愿景是 3 个月的项目——从
        真正有效的每日简报开始。
        [写设计文档 → 自动输入下游技能]

你：    /plan-ceo-review
        [阅读设计文档，挑战范围，运行 10 节评审]

你：    /plan-eng-review
        [数据流的 ASCII 图、状态机、错误路径]
        [测试矩阵、失败模式、安全顾虑]

你：    批准计划。退出 plan mode。
        [在 11 个文件中写了 2,400 行。约 8 分钟。]

你：    /review
        [自动修复] 2 个问题。[询问] 竞态条件 → 你批准修复。

你：    /qa https://staging.myapp.com
        [打开真实浏览器，走查流程，发现并修复了一个 bug]

你：    /ship
        测试：42 → 51 (+9 新测试)。PR: github.com/you/app/pull/42
```

你说"每日简报应用。"Agent 说"你在构建首席参谋 AI"——因为它听了你的痛点，而不是你的功能请求。八个命令，端到端。那不是 copilot。那是一个团队。

## Sprint

gstack 是一个流程，不是工具的集合。技能按照 sprint 的顺序运行：

**思考 → 计划 → 构建 → 评审 → 测试 → 交付 → 反思**

每个技能都为下一个提供输入。`/office-hours` 写一份 `/plan-ceo-review` 读取的设计文档。`/plan-eng-review` 写一份 `/qa` 接手的测试计划。`/review` 捕获 `/ship` 确认已修复的 bug。没有遗漏，因为每一步都知道前一步是什么。

| Skill | 你的专员 | 他们做什么 |
|-------|----------------|--------------|
| `/office-hours` | **YC Office Hours** | 从这里开始。六个强制问题，在写代码之前重新定义你的产品。推翻你的框架，挑战前提，生成实现替代方案。设计文档输入所有下游技能。 |
| `/plan-ceo-review` | **CEO / 创始人** | 重新思考问题。找到藏在请求中的 10 星产品。四种模式：扩展、选择性扩展、保持范围、缩减。 |
| `/plan-eng-review` | **工程经理** | 锁定架构、数据流、图、边界情况和测试。迫使隐藏的假设浮出水面。 |
| `/plan-design-review` | **高级设计师** | 对每个设计维度评分 0-10，解释 10 分长什么样，然后编辑计划以达到目标检测 AI Slop。互动——每个设计选择一个 AskUserQuestion。 |
| `/plan-devex-review` | **Developer Experience 负责人** | 互动 DX 评审：探索开发者画像、与竞争对手的 TTHW 基准、设计你的神奇时刻、逐步追踪摩擦点。三种模式：DX EXPANSION、DX POLISH、DX TRIAGE。20-45 个强制问题。 |
| `/design-consultation` | **设计顾问** | 从零开始构建完整的设计系统。研究全景、提出创意风险、生成真实的产品 mockup。 |
| `/review` | **Staff 工程师** | 找到那些通过 CI 但在生产中爆炸的 bug。自动修复简单的问题。标记完整性差距。 |
| `/investigate` | **调试器** | 系统性根因调试。铁律：没有调查就没有修复。追踪数据流、测试假设、3 次失败修复后停止。 |
| `/design-review` | **会写代码的设计师** | 与 /plan-design-review 相同的审计，然后修复发现的问题。原子提交，修复前后截图。 |
| `/devex-review` | **DX 测试员** | 实时开发者体验审计。实际测试你的上手流程：导航文档、尝试入门流程、计时 TTHW、截图错误。与 `/plan-devex-review` 评分对比——回飞镖，显示你的计划是否匹配现实。 |
| `/design-shotgun` | **设计探索者** | "给我看选项。"生成 4-6 个 AI mockup 变体，在浏览器中打开比较板，收集你的反馈，然后迭代。品味记忆学习你喜欢什么。重复直到你喜欢上某个，然后交给 `/design-html`。 |
| `/design-html` | **设计工程师** | 将 mockup 转为真正能用的生产 HTML。先决条件：计算布局——文本在 resize 时重排、高度根据内容调整、布局是动态的。30KB，零依赖。检测 React/Svelte/Vue。按设计类型智能 API 路由（落地页 vs 仪表盘 vs 表单）。输出可以交付，不是 demo。 |
| `/qa` | **QA 主管** | 测试你的应用、发现 bug、原子提交修复它们、重新验证。自动为每个修复生成回归测试。 |
| `/qa-only` | **QA 报告员** | 与 /qa 相同的方法但仅报告。纯 bug 报告，不更改代码。 |
| `/pair-agent` | **多 Agent 协调器** | 与任何 AI agent 分享你的浏览器。一个命令、一次粘贴、已连接。适用于 OpenClaw、Hermes、Codex、Cursor 或任何能 curl 的东西。每个 agent 获得自己的标签页。自动启动有头模式以便你观看一切。自动启动 ngrok 隧道给远程 agent。作用域 token、标签页隔离、速率限制、活动归因。 |
| `/cso` | **首席安全官** | OWASP Top 10 + STRIDE 威胁模型。零噪音：17 个误报排除、8/10+ 置信度门槛、独立发现验证。每个发现包含具体的利用场景。 |
| `/ship` | **发布工程师** | 同步 main、运行测试、审计覆盖率、推送、打开 PR。如果你没有测试框架，它引导测试框架。 |
| `/land-and-deploy` | **发布工程师** | 合并 PR、等待 CI 和部署、验证生产健康。一个命令从"已批准"到"生产验证"。 |
| `/canary` | **SRE** | 部署后监控循环。监控控制台错误、性能回归和页面失败。 |
| `/benchmark` | **性能工程师** | 基线页面加载时间、Core Web Vitals 和资源大小。在每个 PR 上比较前后。 |
| `/document-release` | **技术撰写人** | 更新所有项目文档以匹配你刚发布的内容。自动捕获过时的 README。 |
| `/retro` | **工程经理** | 团队感知每周回顾。按人细分、交付记录、测试健康趋势、成长机会。`/retro global` 跨所有项目和 AI 工具（Claude Code、Codex、Gemini）运行。 |
| `/browse` | **QA 工程师** | 给 agent 眼睛。真实的 Chromium 浏览器、真实的点击、真实的截图。每个命令约 100ms。`/open-gstack-browser` 启动 GStack Browser，带侧边栏、反机器人隐身和自动模型路由。 |
| `/setup-browser-cookies` | **Session 管理器** | 从你的真实浏览器（Chrome、Arc、Brave、Edge）导入 cookies 到无头 session。测试认证页面。 |
| `/autoplan` | **评审流水线** | 一个命令，全面评审的计划。自动运行 CEO → 设计 → 工程评审，带编码决策原则。仅 surfaced 品味决策供你批准。 |
| `/learn` | **记忆** | 管理 gstack 跨 session 学到的东西。评审、搜索、修剪和导出项目特定的模式、陷阱和偏好。学习在 session 之间累积，gstack 对你的代码库越来越聪明。 |

### 我该用哪个评审？

| 构建目标... | 计划阶段（代码之前） | 实时审计（发布之后） |
|-----------------|--------------------------|----------------------------|
| **终端用户**（UI、web 应用、移动） | `/plan-design-review` | `/design-review` |
| **开发者**（API、CLI、SDK、文档） | `/plan-devex-review` | `/devex-review` |
| **架构**（数据流、性能、测试） | `/plan-eng-review` | `/review` |
| **以上全部** | `/autoplan`（自动运行 CEO → 设计 → 工程 → DX，自动检测适用项） | — |

### 强效工具

| Skill | 它做什么 |
|-------|-------------|
| `/codex` | **第二意见**——来自 OpenAI Codex CLI 的独立代码评审。三种模式：评审（通过/失败门槛）、敌对挑战、开放咨询。当 `/review` 和 `/codex` 都运行时进行跨模型分析。 |
| `/careful` | **安全护栏**——在破坏性命令前警告（rm -rf、DROP TABLE、force-push）。说"be careful"激活。可以覆盖任何警告。 |
| `/freeze` | **编辑锁定**——将文件编辑限制到一个目录。防止调试时意外更改范围外的内容。 |
| `/guard` | **完整安全**——`/careful` + `/freeze` 合为一体。生产工作的最大安全。 |
| `/unfreeze` | **解锁**——移除 `/freeze` 边界。 |
| `/open-gstack-browser` | **GStack Browser**——启动 GStack Browser，带侧边栏、反机器人隐身、自动模型路由（Sonnet 用于操作、Opus 用于分析）、一键 cookie 导入和 Claude Code 集成。清理页面、拍智能截图、编辑 CSS，并将信息传回终端。 |
| `/setup-deploy` | **部署配置器**——`/land-and-deploy` 的一次性设置。检测你的平台、生产 URL 和部署命令。 |
| `/gstack-upgrade` | **自更新**——升级 gstack 到最新版。检测全局 vs vendored 安装、同步两者、显示变更内容。 |

**[每个技能的深入解析和示例 →](docs/skills.md)**

## 并行 Sprint

gstack 用一个 sprint 就很强。十个同时跑起来就变得有趣了。

**设计是核心。** `/design-consultation` 从零开始构建你的设计系统，研究外面有什么、提出创意风险，并写 `DESIGN.md`。但真正的魔法是 shotgun-to-HTML 流水线。

**`/design-shotgun` 是你探索的方式。** 你描述你想要的。它用 GPT Image 生成 4-6 个 AI mockup 变体。然后在浏览器中打开比较板，所有变体并排。你挑喜欢的、留下反馈（"更多留白"、"更大胆的标题"、"去掉渐变"），它生成新的一轮。重复直到你爱上某个。几轮后品味记忆启动，所以它开始偏向你真正喜欢的东西。不再用语言描述你的愿景然后希望 AI 能懂。你看选项、挑好的、视觉化迭代。

**`/design-html` 让它变为现实。** 拿那个已批准的 mockup（来自 `/design-shotgun`、CEO 计划、设计评审或只是一个描述）并把它变成生产质量的 HTML/CSS。不是那种在一个视口宽度下看起还好但在别处全坏的 AI HTML。这使用了 Pretext 来计算文本布局：文本在 resize 时真的重排、高度根据内容调整、布局是动态的。30KB 额外开销，零依赖。它检测你的框架（React、Svelte、Vue）并输出正确的格式。智能 API 路由根据它是落地页、仪表盘、表单还是卡片布局选择不同的 Pretext 模式。输出是你会真正交付的东西，不是 demo。

**`/qa` 是巨大的解锁。** 它让我从 6 个并行 worker 到了 12 个。Claude Code 说*"我看到问题了"*然后真的修复它、生成回归测试、验证修复——这改变了我工作的方式。Agent 现在有眼睛了。

**智能评审路由。** 就像运转良好的初创公司一样：CEO 不需要看基础设施 bug 修复，设计评审对后端变更不需要。gstack 追踪哪些评审已运行、判断什么合适、然后做聪明的事。准备就绪仪表板告诉你在交付前的位置。

**测试一切。** `/ship` 如果你的项目没有测试框架就从头引导。每次 `/ship` 运行产生覆盖率审计。每次 `/qa` bug 修复生成一个回归测试。100% 测试覆盖率是目标——测试让 vibe coding 安全而不是 yolo coding。

**`/document-release` 是你从未有过的工程师。** 它阅读你项目中的每个文档文件、交叉引用 diff、并更新所有漂移的内容。README、ARCHITECTURE、CONTRIBUTING、CLAUDE.md、TODOS——全部自动保持最新。而且现在 `/ship` 自动调用它——文档不需要额外命令就能保持最新。

**真实浏览器模式。** `/open-gstack-browser` 启动 GStack Browser，一个 AI 控制的 Chromium，带反机器人隐身、自定义品牌和内置侧边栏扩展。Google 和 NYTimes 等站点没有验证码。菜单栏说"GStack Browser"而不是"Chrome for Testing。"你正常的 Chrome 保持不动。所有现有浏览命令不变。`$B disconnect` 返回无头。浏览器在窗口打开期间保持活跃...没有闲置超时在你工作时杀掉它。

**侧边栏 agent——你的 AI 浏览器助手。** 在 Chrome 侧面板中输入自然语言，子 Claude 实例执行它。"导航到设置页并截图。""用测试数据填充这个表单。""遍历这个列表中的每一项并提取价格。"侧边栏自动路由到正确的模型：Sonnet 用于快速操作（点击、导航、截图）和 Opus 用于阅读和分析。每个任务最多 5 分钟。侧边栏 agent 在隔离的 session 中运行，所以它不会干扰你主要的 Claude Code 窗口。一键 cookie 导入直接在侧边栏页脚。

**个人自动化。** 侧边栏 agent 不仅用于开发工作流。例如："浏览我孩子的学校家长门户，把所有其他家长的姓名、电话和照片添加到我的 Google 联系人。"两种获取认证的方式：（1）在有头浏览器中登录一次，你的 session 持久化，或（2）点击侧边栏页脚的"cookies"按钮从你真实 Chrome 导入 cookies。认证后，Claude 导航目录、提取数据、创建联系人。

**当 AI 卡住时的浏览器交接。** 遇到 CAPTCHA、认证墙或 MFA 提示？`$B handoff` 在完全相同的页面打开一个可见的 Chrome，所有 cookies 和标签页完好无损。解决问题、告诉 Claude 你完成了，`$B resume` 从它离开的地方继续。Agent 甚至在连续 3 次失败后自动建议这个。

**`/pair-agent` 是跨 agent 协调。** 你在用 Claude Code。你还运行了 OpenClaw。或 Hermes。或 Codex。你想让它们都看同一个网站。输入 `/pair-agent`、选你的 agent、打开一个 GStack Browser 窗口让你观看。技能打印一段指令。粘贴到另一个 agent 的聊天中。它交换一次性设置密钥获取 session token、创建自己的标签页、开始浏览。你在同一个浏览器中看到两个 agent 工作，各自在自己的标签页中、互不干扰。如果安装了 ngrok，隧道自动启动，这样另一个 agent 可以在完全不同的机器上。同机器 agent 获得零摩擦快捷方式，直接写凭证。这是 AI agent 首次能在不同供应商之间通过共享浏览器协调，具有真正的安全性：作用域 token、标签页隔离、速率限制、域限制、活动归因。

**多 AI 第二意见。** `/codex` 从 OpenAI Codex CLI 获取独立评审——完全不同的 AI 看同一个 diff。三种模式：带通过/失败门槛的代码评审、积极尝试破解你代码的敌对挑战、带 session 继续性的开放咨询。当 `/review`（Claude）和 `/codex`（OpenAI）都评审同一个 branch 时，你获得跨模型分析，显示哪些发现重叠、哪些是每个独有的。

**按需安全护栏。** 说"be careful"，`/careful` 在任何破坏性命令前警告——rm -rf、DROP TABLE、force-push、git reset --hard。`/freeze` 在调试时将编辑锁定到一个目录，防止 Claude 意外"修复"无关代码。`/guard` 激活两者。`/investigate` 自动冻结到被调查的模块。

**主动技能建议。** gstack 注意到你处于哪个阶段——头脑风暴、评审、调试、测试——并建议合适的技能。不喜欢？说"stop suggesting"，它会跨 session 记住。

## 10-15 个并行 Sprint

gstack 用一个 sprint 就很强大。十个同时运行就是变革性的。

[Conductor](https://conductor.build) 并行运行多个 Claude Code session——每个在自己隔离的工作区中。一个 session 在 `/office-hours` 研究新想法、另一个在 PR 上做 `/review`、第三个实现一个功能、第四个在 staging 上运行 `/qa`，还有六个在其他 branch 上。同时运行。我经常运行 10-15 个并行 sprint——这是目前的实际上限。

Sprint 结构让并行成为可能。没有流程，十个 agent 是十个混乱源。有流程——思考、计划、构建、评审、测试、交付——每个 agent 确切知道做什么、何时停止。你像 CEO 管理团队一样管理他们：检查重要的决策、让其余的自己跑。

### 语音输入（AquaVoice、Whisper 等）

gstack 技能有语音友好的触发短语。自然地表达你想要的——"run a security check"、"test the website"、"do an engineering review"——正确的技能就会激活。你不需要记住斜杠命令名或缩写。

## 卸载

### 选项 1：运行卸载脚本

如果 gstack 安装在你的机器上：

```bash
~/.claude/skills/gstack/bin/gstack-uninstall
```

这处理技能、符号链接、全局状态（`~/.gstack/`）、项目本地状态、浏览守护进程和临时文件。使用 `--keep-state` 保留配置和分析。使用 `--force` 跳过确认。

### 选项 2：手动移除（无本地 repo）

如果你没有 clone 的 repo（例如你通过 Claude Code 粘贴安装后来删除了 clone）：

```bash
# 1. 停止浏览守护进程
pkill -f "gstack.*browse" 2>/dev/null || true

# 2. 移除指向 gstack/ 的每个技能符号链接
find ~/.claude/skills -maxdepth 1 -type l 2>/dev/null | while read -r link; do
  case "$(readlink "$link" 2>/dev/null)" in gstack/*|*/gstack/*) rm -f "$link" ;; esac
done

# 3. 移除 gstack
rm -rf ~/.claude/skills/gstack

# 4. 移除全局状态
rm -rf ~/.gstack

# 5. 移除集成（跳过任何你没安装过的）
rm -rf ~/.codex/skills/gstack* 2>/dev/null
rm -rf ~/.factory/skills/gstack* 2>/dev/null
rm -rf ~/.kiro/skills/gstack* 2>/dev/null
rm -rf ~/.openclaw/skills/gstack* 2>/dev/null

# 6. 移除临时文件
rm -f /tmp/gstack-* 2>/dev/null

# 7. 每个项目清理（从每个项目根目录运行）
rm -rf .gstack .gstack-worktrees .claude/skills/gstack 2>/dev/null
rm -rf .agents/skills/gstack* .factory/skills/gstack* 2>/dev/null
```

### 清理 CLAUDE.md

卸载脚本不编辑 CLAUDE.md。在添加了 gstack 的每个项目中，移除 `## gstack` 和 `## Skill routing` 部分。

### Playwright

`~/Library/Caches/ms-playwright/`（macOS）保留原样，因为其他工具可能共享它。如果没有其他东西需要它，移除它。

---

免费、MIT 许可、开源。无高级套餐、无等待名单。

我开源了我构建软件的方式。你可以 fork 它并让它成为你的。

> **我们在招人。** 想每天交付 10K+ 行代码并帮助加固 gstack？
> 来 YC 工作 —— [ycombinator.com/software](https://ycombinator.com/software)
> 极具竞争力的薪资和股权。旧金山，Dogpatch 区。

## 文档

| 文档 | 涵盖内容 |
|-----|---------------|
| [技能深入解析](docs/skills.md) | 每个技能的哲学、示例和工作流（包括 Greptile 集成） |
| [构建者理念](ETHOS.md) | 构建者理念：煮沸湖泊、构建前搜索、三层知识 |
| [架构](ARCHITECTURE.md) | 设计决策和系统内部 |
| [浏览器参考](BROWSER.md) | `/browse` 完整命令参考 |
| [贡献指南](CONTRIBUTING.md) | 开发设置、测试、贡献者模式和开发模式 |
| [变更日志](CHANGELOG.md) | 每个版本的新内容 |

## 隐私与遥测

gstack 包含**选择加入**的使用遥测，以帮助改进项目。以下是具体发生的事：

- **默认关闭。** 除非你显式说 yes，什么都不会被发送。
- **首次运行时，** gstack 询问你想不想分享匿名使用数据。你可以说不。
- **（如果你选加入）发送什么：** 技能名称、持续时间、成功/失败、gstack 版本、操作系统。就这些。
- **绝不发送什么：** 代码、文件路径、repo 名称、branch 名称、prompt 或任何用户生成的内容。
- **随时更改：** `gstack-config set telemetry off` 立即禁用一切。

数据存储在 [Supabase](https://supabase.com)（开源 Firebase 替代品）。schema 在 [`supabase/migrations/`](supabase/migrations/) 中——你可以验证具体收集了什么。repo 中的 Supabase 发布 key 是一个公共 key（类似 Firebase API key）——行级安全策略拒绝所有直接访问。遥测通过验证的边缘函数流，强制 schema 检查、事件类型白名单和字段长度限制。

**本地分析始终可用。** 运行 `gstack-analytics` 查看你的个人使用仪表板，来自本地 JSONL 文件——不需要远程数据。

## 故障排除

**技能没出现？** `cd ~/.claude/skills/gstack && ./setup`

**`/browse` 失败？** `cd ~/.claude/skills/gstack && bun install && bun run build`

**安装过期了？** 运行 `/gstack-upgrade`——或在 `~/.gstack/config.yaml` 中设置 `auto_upgrade: true`

**想要更短的命令？** `cd ~/.claude/skills/gstack && ./setup --no-prefix`——从 `/gstack-qa` 切换到 `/qa`。你的选择会被记住，用于未来升级。

**想要命名空间命令？** `cd ~/.claude/skills/gstack && ./setup --prefix`——从 `/qa` 切换到 `/gstack-qa`。当你与其他技能包一起运行 gstack 时有用。

**Codex 说"Skipped loading skill(s) due to invalid SKILL.md"？*** 你的 Codex 技能描述过时了。修复：`cd ~/.codex/skills/gstack && git pull && ./setup --host codex`——或对于 repo 本地安装：`cd "$(readlink -f .agents/skills/gstack)" && git pull && ./setup --host codex`

**Windows 用户：** gstack 在 Windows 11 上通过 Git Bash 或 WSL 工作。需要 Node.js 加上 Bun——Bun 在 Windows 上有已知的 Playwright pipe transport bug（[bun#4253](https://github.com/oven-sh/bun/issues/4253)）。浏览服务器自动回退到 Node.js。确保 `bun` 和 `node` 都在你的 PATH 上。

**Claude 说看不到技能？** 确保你项目的 `CLAUDE.md` 有 gstack 部分。添加这个：

```
## gstack
Use /browse from gstack for all web browsing. Never use mcp__claude-in-chrome__* tools.
Available skills: /office-hours, /plan-ceo-review, /plan-eng-review, /plan-design-review,
/design-consultation, /design-shotgun, /design-html, /review, /ship, /land-and-deploy,
/canary, /benchmark, /browse, /open-gstack-browser, /qa, /qa-only, /design-review,
/setup-browser-cookies, /setup-deploy, /retro, /investigate, /document-release, /codex,
/cso, /autoplan, /pair-agent, /careful, /freeze, /guard, /unfreeze, /gstack-upgrade, /learn.
```

## 许可

MIT。永远免费。去构建什么东西吧。
