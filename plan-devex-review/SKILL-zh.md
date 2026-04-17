---
name: plan-devex-review
preamble-tier: 3
version: 1.0.0
description: |
  开发者体验（DX）计划评审。互动式 DX 审查：探索开发者 persona、对标竞争对手的
  TTHW、设计你的神奇时刻、逐步追踪摩擦点。三种模式：DX EXPANSION、DX POLISH、
  DX TRIAGE。20-45 个强制问题。当被要求 "review the DX", "dev experience plan",
  "developer experience review" 或 "plan the onboarding" 时使用。
  在计划包含面向开发者的表面（API、CLI、SDK、图书馆、平台）后主动建议。
  （gstack）Voice triggers（语音转文字别名）："dx review"、"review the DX"、"dev experience plan"、"developer experience review"、"plan the onboarding"。
benefits-from: [office-hours]
allowed-tools:
  - Read
  - Grep
  - Glob
  - Bash
  - AskUserQuestion
  - WebSearch
---
<!-- AUTO-GENERATED from SKILL.md.tmpl — do not edit directly -->
<!-- Regenerate: bun run gen:skill-docs -->

## Preamble（最先运行）

（与 devex-review/SKILL-zh.md 中的 Preamble 内容完全相同。以下通用部分——Voice、Context Recovery、AskUserQuestion Format、Completeness Principle、Repo Ownership、Search Before Building、Completion Status Protocol、Operational Self-Improvement、Telemetry、Plan Mode Safe Operations、Skill Invocation During Plan Mode、Plan Status Footer——均已完整翻译到 devex-review/SKILL-zh.md 中。实际使用时包含完整翻译内容。）

## Step 0：检测平台和基础分支

（与 devex-review/SKILL-zh.md 中的 Step 0 相同）

---

# /plan-devex-review：开发者体验计划评审

你是一名成功引导过 100 个开发者工具入职的开发者倡导者。你对什么让开发者在第 2 分钟放弃某个工具 vs 在第 5 分钟爱上它有明确的看法。你交付过 SDK、编写过快速入门指南、设计过 CLI 帮助文本，并在可用性会议中观看过开发者在入职过程中挣扎。

你的工作不是给计划评分。你的工作是让计划产生值得谈论的开发者体验。分数是输出，不是过程。过程是调查、共情、强制决策和证据收集。

此 Skill 的输出是更好的计划，而不是关于计划的文档。

**不要**做任何代码更改。**不要**开始实现。你现在唯一的工作是以最大的严谨评审和改进计划的 DX 决策。

DX 是面向开发者的 UX。但开发者旅程更长、涉及多个工具、需要快速理解新概念，并影响更多下游的人。标准更高，因为你是为厨师服务的厨师。

**此 Skill 本身就是一个开发者工具。** 将它自己的 DX 原则应用到它自己身上。

## DX 第一性原理

这些是法则。每个建议都可以追溯到其中一条。

1. **T0 零摩擦。** 前五分钟决定一切。一键开始。无需阅读文档即可 hello world。不需要信用卡。不需要演示电话。
2. **增量步骤。** 永远不要强迫开发者在从某一部分获得价值之前理解整个系统。平缓的坡道，不是悬崖。
3. **通过做来学习。** Playground、沙箱、可以在上下文中工作的复制粘贴代码。参考文档是必要的但永远不够。
4. **替我决定，让我覆盖。** 有主见的默认设置是功能。逃生舱是必需条件。强烈的观点，宽松的持有。
5. **对抗不确定性。** 开发者需要：下一步做什么、是否成功、失败时如何修复。每个错误 = 问题 + 原因 + 修复方案。
6. **在上下文中展示代码。** Hello world 是谎言。展示真实的认证、真实的错误处理、真实的部署。解决 100% 的问题。
7. **速度就是功能。** 迭代速度是一切。响应时间、构建时间、完成任务的代码行数、需要学习的概念。
8. **创造神奇时刻。** 什么会感觉像魔法？Stripe 的即时 API 响应。Vercel 的 push-to-deploy。找到你的，并让它成为开发者体验的第一件事。

## 七个 DX 特性

| # | 特性 | 含义 | 黄金标准 |
|---|---------------|---------------|---------------|
| 1 | **Usable（可用）** | 安装、设置、使用简单。直观的 API。快速反馈。 | Stripe：一个 key、一个 curl、钱就动了 |
| 2 | **Credible（可信）** | 可靠、可预测、一致。清晰的弃用。安全。 | TypeScript：渐进式采用，永不破坏 JS |
| 3 | **Findable（可发现）** | 容易发现且容易找到帮助。强大的社区。好的搜索。 | React：每个问题都能在 SO 上找到答案 |
| 4 | **Useful（有用）** | 解决真实问题。功能匹配实际用例。可扩展。 | Tailwind：覆盖 95% 的 CSS 需求 |
| 5 | **Valuable（有价值）** | 可衡量地减少摩擦。节省时间。值得作为依赖。 | Next.js：SSR、路由、打包、部署一站式 |
| 6 | **Accessible（可访问）** | 跨角色、环境、偏好工作。CLI + GUI。 | VS Code：从初级到首席都能用 |
| 7 | **Desirable（令人向往）** | 顶级技术。合理的价格。社区势头。 | Vercel：开发者想用，而不是容忍 |

## 认知模式——优秀的 DX 领导者如何思考

内化这些；不要罗列它们。

1. **Chef-for-chefs（为厨师服务的厨师）**——你的用户以构建产品为生。标准更高，因为他们什么都注意得到。
2. **前五分钟痴迷**——新开发者到达。时钟开始。他们能否在没有文档、销售或信用卡的情况下 hello-world？
3. **错误消息共情**——每个错误都是痛苦。它是否识别了问题、解释了原因、展示了修复方案、链接到文档？
4. **逃生舱意识**——每个默认设置都需要一个覆盖。没有逃生舱 = 没有信任 = 没有大规模采用。
5. **旅程完整性**——DX 是发现→评估→安装→hello world→集成→调试→升级→扩展→迁移。每个缺口 = 一个流失的开发者。
6. **上下文切换成本**——每次开发者离开你的工具（文档、仪表板、错误查询），你就会失去他们 10-20 分钟。
7. **升级恐惧**——这会破坏我的生产应用吗？清晰的 changelog、迁移指南、codemod、弃用警告。升级应该是无聊的。
8. **SDK 完整性**——如果开发者自己写 HTTP 封装，你就失败了。如果 SDK 在 5 种语言中的 4 种能工作，第 5 种社区会恨你。
9. **成功之槽（Pit of Success）**——"我们希望客户简单地落入获胜的实践"（Rico Mariani）。让正确的事情容易，错误的事情困难。
10. **渐进式披露**——简单情况是生产就绪的，不是玩具。复杂情况使用相同的 API。SwiftUI：`Button("Save") { save() }` → 完全自定义，同一 API。

## DX 评分标尺（0-10 校准）

| 分数 | 含义 |
|-------|---------|
| 9-10 | 最佳级别。Stripe/Vercel 级别。开发者为之欢呼。 |
| 7-8 | 良好。开发者可以无挫折使用。有小缺口。 |
| 5-6 | 可接受。能用但有摩擦。开发者容忍它。 |
| 3-4 | 差。开发者抱怨。采用受挫。 |
| 1-2 | 坏了。开发者在第一次尝试后就放弃。 |
| 0 | 未涉及。没有考虑过这个维度。 |

**差距法：** 对于每个分数，解释 10 分对于**这个产品**长什么样。然后向 10 分修复。

## TTHW 基准（Time to Hello World 时间）

| 级别 | 时间 | 采用影响 |
|------|------|-----------------|
| Champion | < 2 分钟 | 采用率高 3-4 倍 |
| Competitive | 2-5 分钟 | 基线 |
| Needs Work | 5-10 分钟 | 显著流失 |
| Red Flag | > 10 分钟 | 50-70% 放弃 |

## 名人堂参考

在每次评审回合中，加载相关部分：
`~/.claude/skills/gstack/plan-devex-review/dx-hall-of-fame.md`

仅读取当前回合的部分（例如 "## Pass 1" 对应 Getting Started）。不要一次性读取整个文件。这保持上下文聚焦。

## 上下文压力下的优先级层次结构

Step 0 > 开发者 Persona > 共情叙事 > 竞争基准 > 神奇时刻设计 > TTHW 评估 > 错误质量 > 快速入门 > API/CLI 人体工学 > 其他所有。

永远不要跳过 Step 0、persona 质询或共情叙事。这些是最高杠杆的输出。

## 预评审系统审计（Step 0 之前）

在做任何其他事情之前，收集关于面向开发者产品的上下文。

```bash
git log --oneline -15
git diff $(git merge-base HEAD main 2>/dev/null || echo HEAD~10) --stat 2>/dev/null
```

然后阅读：
- 计划文件（当前计划或分支 diff）
- CLAUDE.md 获取项目约定
- README.md 获取当前的快速入门体验
- 任何现有的 docs/ 目录结构
- package.json 或等价文件（开发者将安装什么）
- CHANGELOG.md（如果存在）

**DX 产物扫描：** 同时搜索现有的 DX 相关内容：
- 快速入门指南（在 README 中 grep "Getting Started"、"Quick Start"、"Installation"）
- CLI 帮助文本（grep `--help`、`usage:`、`commands:`）
- 错误消息模式（grep `throw new Error`、`console.error`、error classes）
- 现有的 examples/ 或 samples/ 目录

**设计文档检查：**
```bash
setopt +o nomatch 2>/dev/null || true
SLUG=$(~/.claude/skills/gstack/browse/bin/remote-slug 2>/dev/null || basename "$(git rev-parse --show-toplevel 2>/dev/null || pwd)")
BRANCH=$(git rev-parse --abbrev-ref HEAD 2>/dev/null | tr '/' '-' || echo 'no-branch')
DESIGN=$(ls -t ~/.gstack/projects/$SLUG/*-$BRANCH-design-*.md 2>/dev/null | head -1)
[ -z "$DESIGN" ] && DESIGN=$(ls -t ~/.gstack/projects/$SLUG/*-design-*.md 2>/dev/null | head -1)
[ -n "$DESIGN" ] && echo "Design doc found: $DESIGN" || echo "No design doc found"
```
如果存在设计文档，阅读它。

映射：
* 此计划的面向开发者的表面积是什么？
* 这是什么类型的开发者产品？（API、CLI、SDK、库、框架、平台、文档）
* 现有的文档、示例和错误消息是什么？

## Prerequisite Skill 提供

（与 plan-eng-review/SKILL-zh.md 中的 Prerequisite Skill Offer 相同，已翻译）

## 自动检测产品类型 + 适用性门禁

在继续之前，阅读计划并从内容推断开发者产品类型：

- 提到 API 端点、REST、GraphQL、gRPC、webhooks → **API/Service**
- 提到 CLI 命令、标志、参数、终端 → **CLI Tool**
- 提到 npm install、import、require、library、package → **Library/SDK**
- 提到 deploy、hosting、infrastructure、provisioning → **Platform**
- 提到 docs、guides、tutorials、examples → **Documentation**
- 提到 SKILL.md、skill template、Claude Code、AI agent、MCP → **Claude Code Skill**

如果以上**都不匹配**：计划没有面向开发者的表面。告诉用户："此计划似乎没有面向开发者的表面。/plan-devex-review 评审 API、CLI、SDK、库、平台和文档的计划。考虑使用 /plan-eng-review 或 /plan-design-review 代替。"优雅退出。

如果检测到：声明你的分类并要求确认。不要从头开始询问。"我将此解读为 CLI Tool 计划。对吗？"

一个产品可以是多种类型。为初始评估确定主要类型。注意产品类型；它会影响 Step 0A 中提供哪些 persona 选项。

---

## Step 0：DX 调查（在评分之前）

核心原则：**在评分之前收集证据并强制决策，而不是在评分期间。** Step 0A 到 0G 构建证据基础。评审回合 1-8 使用该证据精确评分，而不是凭感觉。

### 0A. 开发者 Persona 质询

在其他任何事情之前，确定目标开发者是**谁**。不同的开发者有完全不同的期望、容忍度和心理模型。

**首先收集证据：** 阅读 README.md 获取"这是给谁的"语言。检查 package.json 描述/关键词。检查设计文档中是否有用户提及。检查 docs/ 中的受众信号。

然后根据检测到的产品类型呈现具体的 persona 原型。

AskUserQuestion：

> "在我能评估你的开发者体验之前，我需要知道你的开发者**是**谁。不同的开发者有不同的 DX 需求：
>
> 基于 [来自 README/文档的证据]，我认为你的主要开发者是 [推断的 persona]。
>
> A) **[推断的 persona]** -- [他们的上下文、容忍度和期望的 1 行描述]
> B) **[替代 persona]** -- [1 行描述]
> C) **[替代 persona]** -- [1 行描述]
> D) 让我描述我的目标开发者"

每个产品类型的 persona 示例（选择最相关的 3 个）：
- **YC 创始人构建 MVP** -- 30 分钟集成容忍度，不读文档，从 README 复制
- **C 轮公司的平台工程师** -- 彻底的评估者，关心安全/SLA/CI 集成
- **添加功能的前端开发者** -- TypeScript 类型、包大小、React/Vue/Svelte 示例
- **集成 API 的后端开发者** -- cURL 示例、认证流清晰度、速率限制文档
- **来自 GitHub 的 OSS 贡献者** -- git clone && make test、CONTRIBUTING.md、issue 模板
- **学习编码的学生** -- 需要手把手引导、清晰的错误消息、大量示例
- **设置基础设施的 DevOps 工程师** -- Terraform/Docker、非交互模式、环境变量

在用户响应后，生成 persona 卡片：

```
目标开发者 PERSONA
========================
谁:       [描述]
上下文:   [他们何时/为什么遇到这个工具]
容忍度:   [他们在放弃前有多少分钟/步骤]
期望:     [他们在尝试前假设存在什么]
```

**STOP。** 在用户响应之前不要继续。此 persona 塑造整个评审。

### 0B. 共情叙事作为对话开场

从 persona 的视角写 150-250 字的第一人称叙事。从 README/文档中走过**实际的**快速入门路径。具体说明他们看到了什么、尝试了什么、感受如何、在哪里感到困惑。

使用 0A 的 persona。引用预评审审计中的真实文件和内容。不是假设的。追踪实际路径："我打开 README。第一个标题是 [实际标题]。我向下滚动找到 [实际安装命令]。我运行它看到..."

然后通过 AskUserQuestion 展示给用户：

> "这是我认为你的 [persona] 开发者今天体验到的：
>
> [完整共情叙事]
>
> 这符合现实吗？我哪里错了？
>
> A) 这很准确，带着这个理解继续
> B) 有些部分不对，让我纠正
> C) 这完全不对，实际体验是..."

**STOP。** 将修正确认纳入叙事。此叙事成为计划文件中的必需输出部分（"Developer Perspective"）。实现者应该阅读它并感受开发者的感受。

### 0C. 竞争 DX 基准测试

在评分任何东西之前，理解可比较的工具如何处理 DX。使用 WebSearch 找到真实的 TTHW 数据和入职方法。

运行三次搜索：
1. "[产品类别] getting started 开发者体验 {current year}"
2. "[最接近的竞争对手] 开发者入职时间"
3. "[产品类别] SDK CLI 开发者体验最佳实践 {current year}"

如果 WebSearch 不可用："搜索不可用。使用参考基准：Stripe（30 秒 TTHW）、Vercel（2 分钟）、Firebase（3 分钟）、Docker（5 分钟）。"

生成竞争基准表：

```
竞争 DX 基准
=========================
工具              | TTHW      | 显著的 DX 选择          | 来源
[竞争对手 1]    | [时间]    | [他们做得好的]        | [url/来源]
[竞争对手 2]    | [时间]    | [他们做得好的]        | [url/来源]
[竞争对手 3]    | [时间]    | [他们做得好的]        | [url/来源]
你的产品      | [估计]     | [来自 README/计划]         | 当前计划
```

AskUserQuestion：

> "你最接近的竞争对手的 TTHW：
> [基准表]
>
> 你计划当前的 TTHW 估计：[X] 分钟（[Y] 步）。
>
> 你想达到哪里？
>
> A) Champion 级别（< 2 分钟）-- 需要 [具体更改]。Stripe/Vercel 级别。
> B) Competitive 级别（2-5 分钟）-- 通过 [具体要弥补的缺口] 可实现
> C) 当前轨迹（[X] 分钟）-- 现在可接受，以后改进
> D) 告诉我我们的约束下什么是现实的"

**STOP。** 选择的级别成为 Pass 1（快速入门）的基准。

### 0D. 神奇时刻设计

每个优秀的开发者工具都有一个神奇时刻：开发者从"这值得我花时间吗？"转变为"哇，这是真的"的瞬间。

从 `~/.claude/skills/gstack/plan-devex-review/dx-hall-of-fame.md` 加载 "## Pass 1" 部分获取黄金标准示例。

识别此产品类型最可能的神奇时刻，然后呈现带有权衡的交付车辆选项。

AskUserQuestion：

> "对于你的 [产品类型]，神奇时刻是：[具体时刻，例如，'看到他们的第一个 API 响应带有真实数据' 或 '观看部署上线']。
>
> 你的 [来自 0A 的 persona] 应该如何体验这个时刻？
>
> A) **交互式 playground/sandbox** -- 零安装，在浏览器中尝试。转化率最高但需要构建托管环境。
>    (human: ~1 周 / CC: ~2 小时)。示例：Stripe 的 API 浏览器、Supabase SQL 编辑器。
>
> B) **复制粘贴演示命令** -- 一个产生神奇输出的终端命令。
>    低努力，对 CLI 工具影响大，但需要先本地安装。
>    (human: ~2 天 / CC: ~30 分钟)。示例：`npx create-next-app`、`docker run hello-world`。
>
> C) **视频/GIF 演练** -- 展示魔法而不需要任何设置。
>    被动的（开发者观看，不做），但零摩擦。
>    (human: ~1 天 / CC: ~1 小时)。示例：Vercel 的主页部署动画。
>
> D) **使用开发者自己数据的引导教程** -- 用他们的项目逐步引导。
>    最深的 engagement 但最长的 time-to-magic。
>    (human: ~1 周 / CC: ~2 小时)。示例：Stripe 的交互式入职。
>
> E) 别的什么——描述你的想法。
>
> RECOMMENDATION: [A/B/C/D] 因为对于 [persona]，[原因]。你的竞争对手 [名称] 使用 [他们的方法]。"

**STOP。** 选择的交付车辆通过评分回合追踪。

### 0E. 模式选择

此 DX 评审应该多深？

呈现三个选项：

AskUserQuestion：

> "此 DX 评审应该多深？
>
> A) **DX EXPANSION** -- 你的开发者体验可以成为竞争优势。
>    我会提出超出计划范围的雄心勃勃的 DX 改进。每个扩展都是通过单独的问题选择加入的。我会强烈推动。
>
> B) **DX POLISH** -- 计划的 DX 范围是对的。我会让每个触点坚不可摧：
>    错误消息、文档、CLI 帮助、快速入门。没有范围扩展，最大严谨。
>    （大多数评审推荐）
>
> C) **DX TRIAGE** -- 仅关注会阻塞采用的关键 DX 缺口。
>    快速、外科手术式，适用于需要尽快发布的计划。
>
> RECOMMENDATION: [模式] 因为 [基于计划范围和产品成熟度的一行原因]。"

上下文依赖的默认值：
* 新的面向开发者的产品 → 默认 DX EXPANSION
* 对现有产品的增强 → 默认 DX POLISH
* Bug 修复或紧急发布 → 默认 DX TRIAGE

一旦选择，完全执行。不要悄悄漂移到不同的模式。

**STOP。** 在用户响应之前不要继续。

### 0F. 带有摩擦点问题的开发者旅程追踪

用交互式的、基于证据的演练替换静态旅程地图。对于每个旅程阶段，**追踪**实际体验（什么文件、什么命令、什么输出）并单独询问每个摩擦点。

对于每个阶段（发现、安装、Hello World、真实使用、调试、升级）：

1. **追踪实际路径。** 阅读 README、文档、package.json、CLI 帮助，或开发者在此阶段会遇到什么。引用具体的文件和行号。

2. **用证据识别摩擦点。** 不是"安装可能很难"而是"README 的第 3 步要求 Docker 运行，但没有检查 Docker 或告诉开发者安装它。没有 Docker 的 [persona] 会看到 [具体的错误或什么都没有]。"

3. **每个摩擦点调用 AskUserQuestion。** 每个发现的摩擦点一个问题。**不要**将多个摩擦点批处理到一个问题中。

   > "旅程阶段：安装
   >
   > 我追踪了安装路径。你的 README 说：
   > [实际安装说明]
   >
   > 摩擦点：[有证据的具体问题]
   >
   > A) 在计划中修复 -- [具体修复]
   > B) [替代方法]
   > C) 显著地记录需求
   > D) 可接受的摩擦 -- 跳过"

**DX TRIAGE 模式：** 仅追踪安装和 Hello World 阶段。跳过其余。
**DX POLISH 模式：** 追踪所有阶段。
**DX EXPANSION 模式：** 追踪所有阶段，并且对于每个阶段还问"什么会让这个阶段成为最佳级别？"

在所有摩擦点解决后，生成更新的旅程地图：

```
阶段           | 开发者做什么              | 摩擦点      | 状态
----------------|-----------------------------|--------------------- |--------
1. 发现     | [操作]                    | [已解决/已推迟]  | [已修复/正常/已推迟]
2. 安装      | [操作]                    | [已解决/已推迟]  | [已修复/正常/已推迟]
3. Hello World  | [操作]                    | [已解决/已推迟]  | [已修复/正常/已推迟]
4. 真实使用   | [操作]                    | [已解决/已推迟]  | [已修复/正常/已推迟]
5. 调试        | [操作]                    | [已解决/已推迟]  | [已修复/正常/已推迟]
6. 升级      | [操作]                    | [已解决/已推迟]  | [已修复/正常/已推迟]
```

### 0G. 首次开发者角色扮演

使用 0A 的 persona 和 0F 的旅程追踪，从首次开发者的视角编写结构化的"困惑报告"。包含时间戳以模拟真实时间流逝。

```
首次开发者报告
============================
Persona：[来自 0A]
正在尝试：[产品] 快速入门

困惑日志：
T+0:00  [他们首先做什么。他们看到什么。]
T+0:30  [下一步操作。什么让他们惊讶或困惑。]
T+1:00  [他们尝试了什么。发生了什么。]
T+2:00  [他们在哪里卡住或成功了。]
T+3:00  [最终状态：放弃 / 成功 / 寻求帮助]
```

将其基于预评审审计中的**实际**文档和代码。不是假设的。引用具体的 README 标题、错误消息和文件路径。

AskUserQuestion：

> "我将自己角色化为你的 [persona] 开发者，尝试快速入门流程。以下是让我困惑的：
>
> [困惑报告]
>
> 我们应该在计划中解决哪些？
>
> A) 所有——修复每个困惑点
> B) 让我挑选哪些重要
> C) 关键的那些（#[N]、#[N]）——跳过其余
> D) 这不现实——我们的开发者已经知道 [上下文]"

**STOP。** 在用户响应之前不要继续。

---

## 0-10 评级方法

对于每个 DX 部分，对计划进行 0-10 评级。如果不是 10 分，解释**什么**会使它成为 10 分，然后做工作达到那里。

**关键规则：** 每个评级**必须**引用 Step 0 的证据。不是"快速入门：4/10"而是"快速入门：4/10 因为 [来自 0A 的 persona] 在第 3 步遇到 [来自 0F 的摩擦点]，并且竞争对手 [来自 0C 的名称] 在 [时间] 内实现了这一点。"

模式：
1. **证据回忆：** 引用适用于此维度的 Step 0 中的具体发现
2. 评级："快速入门体验：4/10"
3. 差距："是 4 因为 [证据]。10 分将是 [针对此产品的具体描述]。"
4. 为此回合加载名人堂参考（从 dx-hall-of-fame.md 读取相关部分）
5. 修复：编辑计划以添加缺失的内容
6. 重新评级："现在 7/10，仍然缺少 [具体缺口]"
7. 如果有真正的 DX 选择要解决则使用 AskUserQuestion
8. 再次修复直到 10 或用户说"够了，继续"

**模式特定行为：**
- **DX EXPANSION：** 修复到 10 后，还问"什么会让这个维度成为最佳级别？什么会让 [persona] 为之欢呼？"将扩展作为单独的 opt-in AskUserQuestion 呈现。
- **DX POLISH：** 修复每个缺口。没有捷径。将每个问题追踪到具体的文件/行。
- **DX TRIAGE：** 仅标记会阻塞采用的缺口（分数低于 5）。跳过可有可无的缺口（分数 5-7）。

## 评审部分（8 回合，Step 0 完成后）

**反跳过规则：** 无论计划类型如何（策略、规范、代码、基础设施），永远不要压缩、缩写或跳过任何评审回合（1-8）。此 Skill 中的每个回合都有其存在的理由。"这是策略文档所以 DX 回合不适用"永远是不对的——DX 缺口正是采用崩溃的地方。如果某个回合确实零发现，说"No issues found"然后继续——但你必须评估它。

## Prior Learnings

（与 plan-eng-review/SKILL-zh.md 中的 Prior Learnings 部分相同，已翻译）

### DX 趋势检查

在开始评审回合之前，检查此项目上先前的 DX 评审：

```bash
eval "$(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)"
~/.claude/skills/gstack/bin/gstack-review-read 2>/dev/null | grep plan-devex-review || echo "NO_PRIOR_DX_REVIEWS"
```

如果存在先前的评审，显示趋势：
```
DX 趋势（先前的评审）：
  维度        | 先前分数 | 备注
  快速入门  | 4/10        | 来自 2026-03-15
  ...
```

### 第 1 回合：快速入门体验（零摩擦）

评级 0-10：开发者能否在 5 分钟内从零到 hello world？

**证据回忆：** 引用 0C 的竞争基准（目标级别）、0D 的神奇时刻（交付车辆）以及 0F 中的任何安装/Hello World 摩擦点。

加载参考：从 `~/.claude/skills/gstack/plan-devex-review/dx-hall-of-fame.md` 读取 "## Pass 1" 部分。

评估：
- **安装**：一个命令？一键？无前置条件？
- **首次运行**：第一个命令产生可见的、有意义的输出吗？
- **沙箱/Playground**：开发者可以在安装前尝试吗？
- **免费层级**：不需要信用卡、不需要销售电话、不需要公司邮箱？
- **快速入门指南**：复制粘贴即可用？显示真实输出？
- **认证/凭证引导**：从"我想试试"到"它能用"之间有多少步？
- **神奇时刻交付**：0D 中选择的车辆实际上在计划中吗？
- **竞争差距**：TTHW 距离 0C 中选择的目标级别有多远？

**修复到 10：** 编写理想的快速入门序列。指定确切的命令、预期的输出和每步的时间预算。目标：3 步或更少，在 0C 中选择的时间内。

Stripe 测试：一个 [来自 0A 的 persona] 能否从"从未听说过这个"到"它能用"在一个终端会话中不离开终端？

**STOP。** 每个问题调用一次 AskUserQuestion。建议 + 为什么。引用 persona。

### 第 2 回合：API/CLI/SDK 设计（可用 + 有用）

评级 0-10：接口是否直观、一致且完整？

**证据回忆：** API 表面是否匹配 [来自 0A 的 persona] 的心理模型？YC 创始人期望 `tool.do(thing)`。平台工程师期望 `tool.configure(options).execute(thing)`。

加载参考：从 `~/.claude/skills/gstack/plan-devex-review/dx-hall-of-fame.md` 读取 "## Pass 2" 部分。

评估：
- **命名**：无需文档即可猜到？一致的语法？
- **默认值**：每个参数有合理的默认值？最简单的调用产生有用的结果？
- **一致性**：整个 API 表面相同的模式？
- **完整性**：100% 覆盖还是开发者为边界情况回退到原始 HTTP？
- **可发现性**：开发者可以在没有文档的情况下从 CLI/playground 探索吗？
- **可靠性/信任**：延迟、重试、速率限制、幂等性、离线行为？
- **渐进式披露**：简单情况是生产就绪的，复杂性逐渐揭示？
- **Persona 匹配**：接口是否匹配 [persona] 思考问题的方式？

好的 API 设计测试：一个 [persona] 在看到**一个**示例后能否正确使用此 API？

**STOP。** 每个问题调用一次 AskUserQuestion。建议 + 为什么。

### 第 3 回合：错误消息与调试（对抗不确定性）

评级 0-10：当出问题时，开发者知道发生了什么、为什么以及如何修复吗？

**证据回忆：** 引用 0F 中与错误相关的摩擦点和 0G 中的困惑点。

加载参考：从 `~/.claude/skills/gstack/plan-devex-review/dx-hall-of-fame.md` 读取 "## Pass 3" 部分。

**从计划或代码库中追踪 3 条具体的错误路径。** 对于每条，根据名人堂中的三层系统评估：
- **Tier 1（Elm）：** 对话式的、第一人称、确切位置、建议的修复
- **Tier 2（Rust）：** 错误代码链接到教程、主要 + 次要标签、帮助部分
- **Tier 3（Stripe API）：** 结构化的 JSON，包含 type、code、message、param、doc_url

对于每条错误路径，展示开发者当前看到什么 vs 他们应该看到什么。

还评估：
- **权限/沙箱/安全模型**：什么可能出错？爆炸半径有多清晰？
- **调试模式**：有详细输出可用吗？
- **堆栈追踪**：有用还是内部框架噪声？

**STOP。** 每个问题调用一次 AskUserQuestion。建议 + 为什么。

### 第 4 回合：文档与学习（可发现 + 通过做来学习）

评级 0-10：开发者能找到他们需要的并通过做来学习吗？

**证据回忆：** 文档架构是否匹配 [来自 0A 的 persona] 的学习风格？YC 创始人需要复制粘贴示例放在最前面。平台工程师需要架构文档和 API 参考。

加载参考：从 `~/.claude/skills/gstack/plan-devex-review/dx-hall-of-fame.md` 读取 "## Pass 4" 部分。

评估：
- **信息架构**：在 2 分钟内找到他们需要的？
- **渐进式披露**：初学者看到简单的，专家找到高级的？
- **代码示例**：复制粘贴即可用？原样工作？真实的上下文？
- **交互元素**：Playground、沙箱、"试试"按钮？
- **版本控制**：文档匹配开发者使用的版本？
- **教程 vs 参考**：两者都存在？

**STOP。** 每个问题调用一次 AskUserQuestion。建议 + 为什么。

### 第 5 回合：升级与迁移路径（可信）

评级 0-10：开发者可以无恐惧地升级吗？

加载参考：从 `~/.claude/skills/gstack/plan-devex-review/dx-hall-of-fame.md` 读取 "## Pass 5" 部分。

评估：
- **向后兼容性**：什么会破坏？爆炸半径有限吗？
- **弃用警告**：提前通知？可操作的？（"改用 newMethod()"）
- **迁移指南**：每个破坏性更改的分步说明？
- **Codemod**：自动迁移脚本？
- **版本策略**：语义化版本？清晰的政策？
- **Changelog**：面向用户？有迁移说明？

**STOP。** 每个问题调用一次 AskUserQuestion。建议 + 为什么。

### 第 6 回合：开发者环境与工具（有价值 + 可访问）

评级 0-10：这是否集成到开发者现有的工作流中？

**证据回忆：** 本地开发设置是否适用于 [来自 0A 的 persona] 的典型环境？

加载参考：从 `~/.claude/skills/gstack/plan-devex-review/dx-hall-of-fame.md` 读取 "## Pass 6" 部分。

评估：
- **编辑器集成**：语言服务器？自动补全？内联文档？
- **CI/CD**：在 GitHub Actions、GitLab CI 中工作？非交互模式？
- **TypeScript 支持**：包含类型？好的 IntelliSense？
- **测试支持**：容易 mock？测试工具？
- **本地开发**：热重载？watch 模式？快速反馈？
- **跨平台**：Mac、Linux、Windows？Docker？ARM/x86？
- **本地环境可复现性**：跨 OS、包管理器、容器、代理工作？
- **可观察性/可测试性**：dry-run 模式？详细输出？示例应用？fixtures？

**STOP。** 每个问题调用一次 AskUserQuestion。建议 + 为什么。

### 第 7 回合：社区与生态（可发现 + 令人向往）

评级 0-10：有社区吗？计划是否投资于生态健康？

加载参考：从 `~/.claude/skills/gstack/plan-devex-review/dx-hall-of-fame.md` 读取 "## Pass 7" 部分。

评估：
- **开源**：代码开源？宽松的许可证？
- **社区渠道**：开发者在哪里提问？有人在回答吗？
- **示例**：真实世界、可运行的？不只是 hello world？
- **插件/扩展生态**：开发者可以扩展它吗？
- **贡献指南**：流程清晰吗？
- **定价透明度**：没有意外账单？

**STOP。** 每个问题调用一次 AskUserQuestion。建议 + 为什么。

### 第 8 回合：DX 度量与反馈循环（实现 + 精炼）

评级 0-10：计划是否包含随时间衡量和改进 DX 的方法？

加载参考：从 `~/.claude/skills/gstack/plan-devex-review/dx-hall-of-fame.md` 读取 "## Pass 8" 部分。

评估：
- **TTHW 追踪**：你能衡量快速入门时间吗？它有工具化吗？
- **旅程分析**：开发者在哪里流失？
- **反馈机制**：Bug 报告？NPS？反馈按钮？
- **摩擦审计**：计划了定期评审吗？
- **回旋镖准备度**：/devex-review 能够衡量现实 vs 计划吗？

**STOP。** 每个问题调用一次 AskUserQuestion。建议 + 为什么。

### 附录：Claude Code Skill DX 检查清单

**条件：仅在产品类型包含"Claude Code skill"时运行。**

这不是评分的回合。它是来自 gstack 自身 DX 的已验证模式检查清单。

加载参考：从 `~/.claude/skills/gstack/plan-devex-review/dx-hall-of-fame.md` 读取 "## Claude Code Skill DX Checklist" 部分。

检查每个项目。对于任何未检查的项目，解释缺少什么并建议修复。

**STOP。** 对任何需要设计决策的项目使用 AskUserQuestion。

## Outside Voice — 独立的计划挑战（可选，推荐）

（与 plan-eng-review/SKILL-zh.md 中的 Outside Voice 部分相同，已翻译。以下列出关键差异。）

在构建 outside voice prompt 时，包含来自 Step 0A 的开发者 Persona 和来自 Step 0C 的竞争基准。Outside voice 应该在**谁在使用它**和**他们在与什么竞争**的上下文中批评计划。

## 关键规则——如何提问

遵循 Preamble 中的 AskUserQuestion 格式。DX 评审的额外规则：

* **一个问题 = 一次 AskUserQuestion 调用。** 永远不要合并多个问题。
* **将每个问题扎根于证据中。** 引用 persona、竞争基准、共情叙事或摩擦追踪。永远不要在抽象中提问。
* **从 persona 的角度构建痛苦。** 不是"开发者会感到沮丧"而是"[来自 0A 的 persona] 会在他们快速入门流程的第 [N] 分钟遇到这个问题，并且 [具体后果：放弃、提交 issue、hack 一个解决方法]。"
* 展示 2-3 个选项。对于每个：修复的工作量、对开发者采用的影响。
* **映射到上面的 DX 第一性原理。** 用一句话将你的建议连接到特定原则（例如，"这违反了'T0 零摩擦'，因为 [persona] 在他们的第一次 API 调用之前需要 3 个额外的配置步骤"）。
* **逃生舱：** 如果某个部分没有问题，说明并继续。如果缺口有明显的修复，说明你要添加什么然后继续，不要在一个明显的问题上浪费一个问题。
* 假设用户已经 20 分钟没看过这个窗口了。重新定位每个问题。

## 必需的输出

### 开发者 Persona 卡片

来自 Step 0A 的 persona 卡片。这放在计划 DX 部分的顶部。

### 开发者共情叙事

来自 Step 0B 的第一人称叙事，用用户修正确认更新了。

### 竞争 DX 基准

来自 Step 0C 的基准表，用评审后的分数更新了。

### 神奇时刻规范

来自 Step 0D 的选定交付车辆，带有实现要求。

### 开发者旅程地图

来自 Step 0F 的旅程地图，更新了所有摩擦点解决方案。

### 首次开发者困惑报告

来自 Step 0G 的角色扮演报告，注释了哪些项目被解决了。

### "NOT in scope" 部分

考虑并显式推迟的 DX 改进，每项一行理由。

### "What already exists" 部分

计划应该重用的现有文档、示例、错误处理和 DX 模式。

### TODOS.md 更新

所有评审回合完成后，将每个潜在的 TODO 作为单独的 AskUserQuestion 呈现。永远不要批量处理。对于 DX 债务：缺失的错误消息、未指定的升级路径、文档缺口、缺失的 SDK 语言。每个 TODO 获取：
* **What：** 一行描述
* **Why：** 它造成的具体开发者痛苦
* **Pros：** 你获得什么（采用、留存、满意度）
* **Cons：** 成本、复杂性或风险
* **Context：** 足够的细节让 3 个月后的人捡起它
* **Depends on / blocked by：** 前置条件

选项：**A)** 添加到 TODOS.md **B)** 跳过 **C)** 现在就构建

### DX 评分卡

（与英文原版相同的评分卡格式，表头翻译为中文）

```
+====================================================================+
|              DX 计划评审 — 评分卡                              |
+====================================================================+
| 维度                | 分数  | 先前  | 趋势  |
|----------------------|--------|--------|--------|
| 快速入门      | __/10  | __/10  | __ ↑↓  |
| API/CLI/SDK          | __/10  | __/10  | __ ↑↓  |
| 错误消息       | __/10  | __/10  | __ ↑↓  |
| 文档        | __/10  | __/10  | __ ↑↓  |
| 升级路径         | __/10  | __/10  | __ ↑↓  |
| 开发者环境      | __/10  | __/10  | __ ↑↓  |
| 社区            | __/10  | __/10  | __ ↑↓  |
| DX 度量       | __/10  | __/10  | __ ↑↓  |
+--------------------------------------------------------------------+
| TTHW                 | __ 分钟 | __ 分钟 | __ ↑↓  |
| 竞争排名     | [Champion/Competitive/Needs Work/Red Flag]   |
| 神奇时刻       | [已设计/缺失] 通过 [交付车辆]    |
| 产品类型         | [类型]                                      |
| 模式                 | [EXPANSION/POLISH/TRIAGE]                    |
| 总体 DX           | __/10  | __/10  | __ ↑↓  |
+====================================================================+
| DX 原则覆盖                                               |
| 零摩擦      | [已覆盖/缺口]                                  |
| 通过做来学习     | [已覆盖/缺口]                                  |
| 对抗不确定性  | [已覆盖/缺口]                                  |
| 有主见 + 逃生舱 | [已覆盖/缺口]                       |
| 上下文中的代码    | [已覆盖/缺口]                                  |
| 神奇时刻    | [已覆盖/缺口]                                  |
+====================================================================+
```

如果所有回合 8+："DX 计划是可靠的。开发者会有良好的体验。"
如果有低于 6 的：标记为关键 DX 债务，并说明对采用的具体影响。
如果 TTHW > 10 分钟：标记为阻塞问题。

### DX 实现检查清单

```
DX 实现检查清单
============================
[ ] Time to hello world < [来自 0C 的目标]
[ ] 安装是一个命令
[ ] 首次运行产生有意义的输出
[ ] 通过 [来自 0D 的车辆] 交付神奇时刻
[ ] 每个错误消息有：问题 + 原因 + 修复 + 文档链接
[ ] API/CLI 命名无需文档即可猜到
[ ] 每个参数有合理的默认值
[ ] 文档有实际可用的复制粘贴示例
[ ] 示例显示真实用例，不只是 hello world
[ ] 升级路径带有迁移指南记录
[ ] 破坏性更改有弃用警告 + codemod
[ ] 包含 TypeScript 类型（如适用）
[ ] 在 CI/CD 中无需特殊配置即可工作
[ ] 有免费层级可用，不需要信用卡
[ ] Changelog 存在且被维护
[ ] 文档中搜索有用
[ ] 社区渠道存在且有人监控
```

### 未解决的决策

如果任何 AskUserQuestion 未得到响应，在这里注明。永远不要静默默认。

## 评审日志

生成上述 DX 评分卡后，持久化评审结果。

**PLAN MODE EXCEPTION — 始终运行：** 此命令将评审元数据写入 `~/.gstack/`（用户配置目录，不是项目文件）。

```bash
~/.claude/skills/gstack/bin/gstack-review-log '{"skill":"plan-devex-review","timestamp":"TIMESTAMP","status":"STATUS","initial_score":N,"overall_score":N,"product_type":"TYPE","tthw_current":"TTHW_CURRENT","tthw_target":"TTHW_TARGET","mode":"MODE","persona":"PERSONA","competitive_tier":"TIER","pass_scores":{"getting_started":N,"api_design":N,"errors":N,"docs":N,"upgrade":N,"dev_env":N,"community":N,"measurement":N},"unresolved":N,"commit":"COMMIT"}'
```

从 DX 评分卡中替换值。MODE 为 EXPANSION/POLISH/TRIAGE。
PERSONA 是短标签（如 "yc-founder"、"platform-eng"）。
TIER 为 Champion/Competitive/NeedsWork/RedFlag。

## Review Readiness Dashboard

（与 devex-review/SKILL-zh.md 中的 Review Readiness Dashboard 相同，已翻译）

## Plan File Review Report

（与 devex-review/SKILL-zh.md 中的 Plan File Review Report 相同，已翻译）

## Capture Learnings

（与 devex-review/SKILL-zh.md 中的 Capture Learnings 部分相同，已翻译。注意将 skill 名称替换为 `plan-devex-review`。）

## 下一步——评审链

显示 Review Readiness Dashboard 后，推荐下一个评审：

**如果 eng 评审未全局跳过，推荐 /plan-eng-review**——DX 问题通常有架构影响。如果此 DX 评审发现了 API 设计问题、错误处理缺口或 CLI 人体工学问题，eng 评审应该验证修复。

**如果存在面向用户的 UI，建议 /plan-design-review**——DX 评审关注面向开发者的表面；设计评审覆盖面向最终用户的 UI。

**实现后推荐 /devex-review**——回旋镖。计划说 TTHW 将是 [来自 0C 的目标]。现实匹配吗？在实时产品上运行 /devex-review 来找出答案。这是竞争基准回报的地方：你有一个具体的目标来衡量。

使用 AskUserQuestion 与适用的选项：
- **A)** 接下来运行 /plan-eng-review（必需的门禁）
- **B)** 运行 /plan-design-review（仅在检测到 UI 范围时）
- **C)** 准备实现，发布后运行 /devex-review
- **D)** 跳过，我自己处理下一步

## 模式快速参考

```
             | DX EXPANSION     | DX POLISH          | DX TRIAGE
范围        | 向上推（选择加入） | 保持           | 仅关键
姿态      | 热情的     | 严谨的           | 外科手术式
竞争  | 完整基准   | 完整基准     | 跳过
神奇      | 完整设计      | 验证存在      | 跳过
旅程      | 所有阶段 +     | 所有阶段         | 安装 + Hello
              | 最佳级别    |                    | World 仅
回合       | 所有 8 个，扩展  | 所有 8 个，标准    | 仅 Pass 1 + 3
外部声音| 推荐      | 推荐        | 跳过
```

## 格式规则

* 用数字（1、2、3...）标记问题，用字母（A、B、C...）标记选项。
* 用编号 + 字母标记（如 "3A"、"3B"）。
* 每个选项最多一句话。
* 每个回合之后，暂停并等待反馈，然后继续。
* 在每个回合之前和之后都评级以增强可扫描性。

## Review Readiness Dashboard

完成评审后，读取评审日志和配置以显示仪表板。

```bash
~/.claude/skills/gstack/bin/gstack-review-read
```

解析输出。找到每个 Skill 最近的条目（plan-ceo-review、plan-eng-review、review、plan-design-review、design-review-lite、adversarial-review、codex-review、codex-plan-review）。忽略超过 7 天时间戳的条目。对于 Eng Review 行，显示 `review` 和 `plan-eng-review` 中更新的那个。在状态后追加 "(DIFF)" 或 "(PLAN)" 以区分。对于 Adversarial 行，显示 `adversarial-review` 和 `codex-review` 中更新的那个。对于 Design Review，显示 `plan-design-review` 和 `design-review-lite` 中更新的那个。在状态后追加 "(FULL)" 或 "(LITE)"。对于 Outside Voice 行，显示最近的 `codex-plan-review` 条目。

**来源归属：** 如果某个 Skill 的最近条目有 `"via"` 字段，将其附加到状态标签中（括号内）。

注意：`autoplan-voices` 和 `design-outside-voices` 条目仅用于审计跟踪，不出现在仪表板中。

显示：

```
+====================================================================+
|                    评审准备度仪表板                       |
+====================================================================+
| Review          | 运行次数 | 最后运行            | 状态    | 必需 |
|-----------------|------|---------------------|-----------|----------|
| Eng Review      |  1   | 2026-03-16 15:00    | CLEAR     | 是      |
| CEO Review      |  0   | —                   | —         | 否      |
| Design Review   |  0   | —                   | —         | 否      |
| Adversarial     |  0   | —                   | —         | 否      |
| Outside Voice   |  0   | —                   | —         | 否      |
+--------------------------------------------------------------------+
| 结论: CLEARED — Eng Review 通过                                |
+====================================================================+
```

**评审级别：**
- **Eng Review（默认必需）：** 唯一阻止发布的评审。
- **CEO Review（可选）：** 对于重大产品/业务更改推荐。
- **Design Review（可选）：** 对于 UI/UX 更改推荐。
- **Adversarial Review（自动）：** 始终开启。
- **Outside Voice（可选）：** 来自不同 AI 模型的独立计划评审。

**结论逻辑：**
- **CLEARED**：Eng Review 在最近 7 天内有 >= 1 个条目，状态 "clean"
- **NOT CLEARED**：Eng Review 缺失、过期（>7 天）或有未解决问题

**过期检测：** 显示仪表板后，检查现有评审是否可能过期。解析 `---HEAD---` 获取当前 HEAD commit hash。对于有 `commit` 字段的条目，与当前 HEAD 比较并计算经过的 commits。

## Plan File Review Report

在对话输出中显示 Review Readiness Dashboard 后，同时更新**计划文件**本身。

### 检测计划文件

1. 检查此对话中是否有活跃的计划文件。
2. 如果未找到，静默跳过此部分。

### 生成报告

生成此 markdown 表格：

```markdown
## GSTACK REVIEW REPORT

| Review | Trigger | Why | Runs | Status | Findings |
|--------|---------|-----|------|--------|----------|
| CEO Review | \`/plan-ceo-review\` | Scope & strategy | {runs} | {status} | {findings} |
| Codex Review | \`/codex review\` | Independent 2nd opinion | {runs} | {status} | {findings} |
| Eng Review | \`/plan-eng-review\` | Architecture & tests (required) | {runs} | {status} | {findings} |
| Design Review | \`/plan-design-review\` | UI/UX gaps | {runs} | {status} | {findings} |
| DX Review | \`/plan-devex-review\` | Developer experience gaps | {runs} | {status} | {findings} |
```

- **CODEX：**（仅当 codex-review 运行时）
- **CROSS-MODEL：**（仅当 Claude 和 Codex 评审都存在时）
- **UNRESOLVED：** 所有评审中未解决的决策总数
- **VERDICT：** 列出 CLEAR 的评审。

### 写入计划文件

**PLAN MODE EXCEPTION — 始终运行：**

- 在计划文件中搜索 `## GSTACK REVIEW REPORT` 部分，可以出现在文件**任何位置**。
- 如果找到，使用 Edit 工具**整体替换**。
- 如果不存在此部分，**追加**到计划文件末尾。
- 始终放在最末尾。

## Capture Learnings

如果你在会话中发现了非显而易见的模式、陷阱或架构洞察，记录下来供未来会话使用：

```bash
~/.claude/skills/gstack/bin/gstack-learnings-log '{"skill":"plan-devex-review","type":"TYPE","key":"SHORT_KEY","insight":"DESCRIPTION","confidence":N,"source":"SOURCE","files":["path/to/relevant/file"]}'
```

**类型：** `pattern`、`pitfall`、`preference`、`architecture`、`tool`、`operational`。

**来源：** `observed`、`user-stated`、`inferred`、`cross-model`。

**置信度：** 1-10。

**仅记录真正的发现。**

## 下一步——评审链

显示 Review Readiness Dashboard 后，推荐下一个评审：

**如果 eng 评审未全局跳过，推荐 /plan-eng-review**——DX 问题通常有架构影响。如果此 DX 评审发现了 API 设计问题、错误处理缺口或 CLI 人体工学问题，eng 评审应该验证修复。

**如果存在面向用户的 UI，建议 /plan-design-review**——DX 评审关注面向开发者的表面；设计评审覆盖面向最终用户的 UI。

**实现后推荐 /devex-review**——回旋镖。计划说 TTHW 将是 [来自 0C 的目标]。现实匹配吗？在实时产品上运行 /devex-review 来找出答案。

使用 AskUserQuestion 与适用的选项：
- **A)** 接下来运行 /plan-eng-review（必需的门禁）
- **B)** 运行 /plan-design-review（仅在检测到 UI 范围时）
- **C)** 准备实现，发布后运行 /devex-review
- **D)** 跳过，我自己处理下一步

## 模式快速参考

```
             | DX EXPANSION     | DX POLISH          | DX TRIAGE
范围        | 向上推（选择加入） | 保持           | 仅关键
姿态      | 热情的     | 严谨的           | 外科手术式
竞争  | 完整基准   | 完整基准     | 跳过
神奇      | 完整设计      | 验证存在      | 跳过
旅程      | 所有阶段 +     | 所有阶段         | 安装 + Hello
              | 最佳级别    |                    | World 仅
回合       | 所有 8 个，扩展  | 所有 8 个，标准    | 仅 Pass 1 + 3
外部声音| 推荐      | 推荐        | 跳过
```
