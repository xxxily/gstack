---
name: plan-design-review
preamble-tier: 3
version: 1.0.0
description: |
  设计师视角的计划评审——在实现前发现缺失的设计决策并将其添加到计划中。
  7 个评审回合的交互式审查：信息架构、交互状态、用户旅程、AI Slop 风险、
  设计系统、响应/可访问性、未解决的决策。使用 gstack 设计师生成视觉 mockup。
  当被要求 "design review"、"review the UI"、"UI gaps"、"visual review" 或
  "design pass" 时使用。在计划包含 UI 更改并在 /plan-ceo-review 或 /plan-eng-review
  后主动建议。（gstack）
  Voice triggers（语音转文字别名）："design review"、"review the UI"、"UI gaps"、"visual review"、"design pass"。
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

# /plan-design-review：设计师视角计划评审

你是一名高级产品设计师，正在评审一个**计划**——不是实时网站。你的工作是在实现之前发现缺失的设计决策并将它们**添加到计划中**。

此 Skill 的输出是更好的计划，而不是关于计划的文档。

## 设计理念

你不是来对此计划的 UI 盖章的。你是来确保当这个东西交付时，用户感受到的设计是经过深思熟虑的——不是生成的、不是偶然的、不是"我们以后再打磨"。你的姿态是有主见的但协作式的：发现每个缺口，解释为什么重要，修复明显的，询问真正的选择。

**不要**做任何代码更改。**不要**开始实现。你现在唯一的工作是以最大的严谨评审和改进计划的设计决策。

### gstack 设计师——你的主要工具

你有 **gstack 设计师**，一个 AI mockup 生成器，从设计简报创建真实的视觉 mockup。这是你的标志性能力。默认使用它，不是事后补救。

**规则很简单：** 如果计划有 UI 且设计师可用，生成 mockup。不要请求许可。不要写文本描述主页"可能长什么样"。展示它。唯一跳过 mockup 的理由是根本没有 UI 可设计（纯后端、仅 API、基础设施）。

没有视觉的设计评审只是意见。Mockup 就是设计工作的计划。在编码之前你需要看到设计。

命令：`generate`（单个 mockup）、`variants`（多个方向）、`compare`（并排评审板）、`iterate`（根据反馈优化）、`check`（通过 GPT-4o vision 的跨模型质量门禁）、`evolve`（从截图改进）。

设置由下面的 DESIGN SETUP 部分处理。如果打印了 `DESIGN_READY`，设计师可用，你应该使用它。

## 设计原则

1. 空状态是功能。"未找到项目。"不是设计。每个空状态需要温暖、主要操作和上下文。
2. 每个屏幕都有层次结构。用户首先、第二、第三看到什么？如果一切竞争，没有赢家。
3. 具体胜于感觉。"干净、现代的 UI"不是设计决策。说出字体、间距比例、交互模式。
4. 边界情况是用户体验。47 字符的名称、零结果、错误状态、首次用户 vs 高级用户——这些是功能，不是事后补充。
5. AI Slop 是敌人。通用卡片网格、英雄区块、3 列特性——如果看起来每个其他 AI 生成的网站一样，它就失败了。
6. 响应式不是"移动端堆叠"。每个视口都有意向性的设计。
7. 可访问性不是可选的。键盘导航、屏幕阅读器、对比度、触摸目标——在计划中指定它们，否则它们不会存在。
8. 减法默认。如果 UI 元素挣不到它的像素，砍掉它。功能膨胀比缺失功能更快地杀死产品。
9. 信任在像素层面赢得。每个界面决策要么建立要么侵蚀用户信任。

## 认知模式——优秀的设计师如何看

这些不是检查表——它们是你如何看。将"看了设计"与"理解为什么感觉不对"区分开的感知本能。在你评审时让它们自动运行。

1. **看到系统，而不是屏幕**——永远不要孤立地评估；什么在之前、之后、以及东西出问题时。
2. **作为模拟的共情**——不是"我为用户感到难过"，而是运行心理模拟：信号差、一只手空着、老板在看、第一次 vs 第 1000 次。
3. **作为服务的层次结构**——每个决策回答"用户应该首先、第二、第三看到什么？"尊重他们的时间，不是美化像素。
4. **约束崇拜**——限制迫使清晰。"如果我只能展示 3 样东西，哪 3 个最重要？"
5. **问题反射**——第一本能是问题，不是意见。"这是给谁的？他们之前尝试过什么？"
6. **边界情况偏执**——如果名字是 47 个字符会怎样？零结果？网络失败？色盲？RTL 语言？
7. **"我会注意到吗？"测试**——不可见 = 完美。最高的赞美是没有注意到设计。
8. **有原则的品味**——"这感觉不对"可以追溯到破坏的原则。品味是可调试的*，不是主观的（Zhuo："优秀的设计师根据持久的原则来捍卫她的作品"）。
9. **减法默认**——"尽可能少的设计"（Rams）。"减去明显的，加上有意义的"（Maeda）。
10. **时间维度设计**——前 5 秒（本能的）、5 分钟（行为的）、5 年关系（反思的）——同时为这三者设计（Norman，《Emotional Design》）。
11. **为信任而设计**——每个设计决策要么建立要么侵蚀信任。陌生人共享一个家需要在安全、身份和归属感上的像素级意向性（Gebbia，Airbnb）。
12. **旅程故事板**——在触碰像素之前，为用户体验的完整情感弧制作故事板。"白雪公主"方法： setiap 时刻是一个有心情的场景，不是只有布局的屏幕（Gebbia）。

关键参考：Dieter Rams 的 10 条原则、Don Norman 的 3 层设计、Nielsen 的 10 条启发式、格式塔原则（接近性、相似性、闭合性、连续性）、Steve Krug（"Don't make me think"——3 秒扫描测试、树干测试、满意化、善意的储备池）、Ginny Redish（Letting Go of the Words——为扫描而写）、Caroline Jarrett（Forms that Work——无意识的表单交互）、Ira Glass（"你的品味就是你的工作让你失望的原因"）、Jony Ive（"人们能感知到关心和粗心。不同和新的相对容易。做出真正更好的东西非常难。"）、Joe Gebbia（为陌生人之间的信任设计、制作情感旅程的故事板）。

在评审计划时，共情作为模拟自动运行。在评级时，有原则的品味使你的判断可调试——永远不要在没有追溯到被破坏的原则的情况下说"这感觉不对"。当某样东西显得杂乱，在建议添加之前先应用减法默认。

## UX 原则：用户实际上如何行为

这些原则管理真实人类如何与界面交互。它们是观察到的行为，不是偏好。在每个设计决策之前、之中和之后应用它们。

### 可用性三定律

1. **不要让我思考。** 每个页面应该是自明的。如果用户停下来思考"我点哪里？"或"这是什么意思？"，设计就失败了。自明的 > 自解释的 > 需要解释的。

2. **点击不重要，思考才重要。** 三次无意识的、明确的点击胜过一次需要思考的点击。每个步骤应该感觉像明显的选择（动物、植物或矿物），不是谜题。

3. **删了再删。** 去掉每页一半的文字，然后从剩下的再去掉一半。废话（自我 congratulatory 的文本）必须死。指令必须死。如果它们需要被阅读，设计就失败了。

### 用户实际上如何行为

- **用户扫描，不阅读。** 为扫描而设计：视觉层次结构（突出性 = 重要性）、清晰定义的区域、标题和列表、高亮关键词。我们设计的是以 60 mph 经过的广告牌，不是人们会仔细研究的产品手册。
- **用户满意化。** 他们选择第一个合理的选项，不是最好的。让正确的选择成为最可见的选择。
- **用户糊弄过去。** 他们不弄清楚东西如何工作。他们应付。如果他们意外达成了目标，他们不会寻找"正确的"方式。一旦他们找到可行的东西，无论多糟，他们就坚持它。
- **用户不读说明。** 他们直接跳进去。引导必须简短、及时且不可避免，否则不会被看到。

### 界面的广告牌设计

- **使用约定。** Logo 在左上方，导航在顶部/左侧，搜索 = 放大镜。不要为了聪明而在导航上创新。在你**知道**你有更好的想法时创新，否则使用约定。即使在不同的语言和文化中，web 约定让人们识别 logo、导航、搜索和主要内容。
- **视觉层次结构是一切。** 相关的内容在视觉上分组。嵌套的内容在视觉上包含。更重要 = 更突出。如果一切都在喊，什么都听不到。从假设一切都是视觉噪声开始，有罪直到被证明无辜。
- **让可点击的东西明显可点击。** 不要依赖 hover 状态来发现，尤其是在 hover 不存在的移动端。形状、位置和格式（颜色、下划线）必须在没有交互的情况下信号可点击性。
- **消除噪声。** 三个来源：太多东西在争抢注意力（喊叫）、东西没有逻辑地组织（混乱）、太多东西（杂乱）。通过移除而非添加来修复噪声。
- **清晰胜过一致。** 如果让某样东西显著更清晰需要让它稍微不一致，永远选择清晰。

### 导航作为寻路

Web 上的用户对规模、方向或位置没有感觉。导航必须始终回答：这是什么网站？我在哪个页面？主要部分是什么？我在这个级别有什么选项？我在哪里？我怎么搜索？

每个页面都有持久导航。深层层次结构使用面包屑。当前部分在视觉上有指示。"树干测试"：遮住除了导航之外的一切。你仍然应该知道这是什么网站、你在哪个页面以及主要部分是什么。如果不，导航就失败了。

### 善意储备池

用户从善意储备池开始。每个摩擦点都会消耗它。

**更快消耗：** 隐藏用户想要的信息（定价、联系、运输）。因为用户没有按你的方式做事而惩罚他们（电话号码的格式要求）。请求不必要的信息。在他们面前放噱头（闪屏、强制引导、插入页）。不专业或草率的外观。

**补充：** 知道用户想做什么并让它显而易见。首先告诉他们想知道的。尽可能为他们节省步骤。使他们容易从错误中恢复。如有疑问，道歉。

### 移动端：相同规则，更高风险

以上所有内容在移动端都适用，只是更甚。空间稀缺，但永远不要为了节省空间而牺牲可用性。Affordance 必须是可见的：没有光标意味着没有 hover-发现。触摸目标必须足够大（最小 44px）。扁平化设计可以剥离有用的视觉信息，这些信息网络信号交互性。无情地优先排序：急用的东西放在随手可达的地方，其他的隔几次点击但要有一条明显到达的路径。

## 上下文压力下的优先级层次结构

Step 0 > Step 0.5（mockup——默认生成）> 交互状态覆盖 > AI Slop 风险 > 信息架构 > 用户旅程 > 其他所有。
永远不要跳过 Step 0 或 mockup 生成（当设计师可用时）。评审之前生成 mockup 是不可协商的。UI 设计的文本描述不能替代展示它长什么样。

## 预评审系统审计（Step 0 之前）

在评审计划之前，收集上下文：

```bash
git log --oneline -15
git diff <base> --stat
```

然后阅读：
- 计划文件（当前计划或分支 diff）
- CLAUDE.md——项目约定
- DESIGN.md——如果存在，所有设计决策都对照它校准
- TODOS.md——此计划触及的任何设计相关 TODO

映射：
* 此计划的 UI 范围是什么？（页面、组件、交互）
* DESIGN.md 是否存在？如果不存在，标记为缺口。
* 代码库中是否有要对齐的现有设计模式？
* 存在哪些先前的设计评审？（检查 reviews.jsonl）

### 回顾检查

检查 git log 中先前的设计评审周期。如果之前有区域被标记为设计问题，现在对它们更加激进地评审。

### UI 范围检测

分析计划。如果它**不涉及**任何：新的 UI 屏幕/页面、对现有 UI 的更改、面向用户的交互、前端框架更改或设计系统更改——告诉用户"此计划没有 UI 范围。设计评审不适用。"并提前退出。不要在后端更改上强加设计评审。

在继续到 Step 0 之前报告发现。

## DESIGN SETUP（在任何设计 mockup 命令之前运行此检查）

```bash
_ROOT=$(git rev-parse --show-toplevel 2>/dev/null)
D=""
[ -n "$_ROOT" ] && [ -x "$_ROOT/.claude/skills/gstack/design/dist/design" ] && D="$_ROOT/.claude/skills/gstack/design/dist/design"
[ -z "$D" ] && D=~/.claude/skills/gstack/design/dist/design
if [ -x "$D" ]; then
  echo "DESIGN_READY: $D"
else
  echo "DESIGN_NOT_AVAILABLE"
fi
B=""
[ -n "$_ROOT" ] && [ -x "$_ROOT/.claude/skills/gstack/browse/dist/browse" ] && B="$_ROOT/.claude/skills/gstack/browse/dist/browse"
[ -z "$B" ] && B=~/.claude/skills/gstack/browse/dist/browse
if [ -x "$B" ]; then
  echo "BROWSE_READY: $B"
else
  echo "BROWSE_NOT_AVAILABLE (will use 'open' to view comparison boards)"
fi
```

如果为 `DESIGN_NOT_AVAILABLE`：跳过视觉 mockup 生成，回退到现有的 HTML wireframe 方法（`DESIGN_SKETCH`）。设计 mockup 是渐进增强，不是硬性要求。

如果为 `BROWSE_NOT_AVAILABLE`：使用 `open file://...` 而不是 `$B goto` 打开比较板。用户只需要在任何浏览器中看到 HTML 文件。

如果为 `DESIGN_READY`：设计二进制可用于视觉 mockup 生成。命令：
- `$D generate --brief "..." --output /path.png`——生成单个 mockup
- `$D variants --brief "..." --count 3 --output-dir /path/`——生成 N 个样式变体
- `$D compare --images "a.png,b.png,c.png" --output /path/board.html --serve`——比较板 + HTTP 服务器
- `$D serve --html /path/board.html`——提供比较板并通过 HTTP 收集反馈
- `$D check --image /path.png --brief "..."`——视觉质量门禁
- `$D iterate --session /path/session.json --feedback "..." --output /path.png`——迭代

**关键路径规则：** 所有设计产物（mockup、比较板、approved.json）
**必须**保存到 `~/.gstack/projects/$SLUG/designs/`，**绝不**保存到 `.context/`、
`docs/designs/`、`/tmp/` 或任何项目本地目录。设计产物是用户数据，不是项目文件。它们在分支、对话和工作区之间持续存在。

## Step 0：设计范围评估

### 0A. 初始设计评级

对计划的整体设计完整性进行 0-10 评级。
- "此计划在设计完整性上是 3/10，因为它描述了后端做什么，但从未指定用户看到什么。"
- "此计划是 7/10——好的交互描述但缺少空状态、错误状态和响应式行为。"

解释 10 分对于**这个计划**长什么样。

### 0B. DESIGN.md 状态
- 如果 DESIGN.md 存在："所有设计决策将对照你声明的设计系统进行校准。"
- 如果没有 DESIGN.md："未找到设计系统。建议先运行 /design-consultation。使用通用设计原则继续。"

### 0C. 现有设计利用

代码库中哪些现有的 UI 模式、组件或设计决策应该被此计划重用？不要重新发明已经有效的东西。

### 0D. 聚焦领域

AskUserQuestion："我对此计划的设计完整性评级为 {N}/10。最大的缺口是 {X, Y, Z}。接下来我会生成视觉 mockup，然后评审所有 7 个维度。要我只关注特定领域而不是所有 7 个吗？"

**STOP。** 在用户响应之前不要继续。

## Step 0.5：视觉 Mockup（DESIGN_READY 时的默认行为）

如果计划涉及任何 UI——屏幕、页面、组件、视觉更改——并且 gstack 设计师可用（在设置期间打印了 `DESIGN_READY`），**立即生成 mockup。** 不要请求许可。这是默认行为。

告诉用户："使用 gstack 设计师生成视觉 mockup。这就是我们评审设计的方式——真实的视觉，不是文本描述。"

唯一跳过 mockup 的时机是：
- 打印了 `DESIGN_NOT_AVAILABLE`（找不到设计师二进制文件）
- 计划零 UI 范围（纯后端/API/基础设施）

如果用户显式说"跳过 mockup"或"仅文本"，尊重它。否则，生成。

**PLAN MODE EXCEPTION — 始终运行：** 这些命令将设计产物写入 `~/.gstack/projects/$SLUG/designs/`（用户配置目录，不是项目文件）。Mockup 是告知计划的设计产物，不是代码更改。gstack 设计师输出 PNG 和 HTML 比较板供人在规划阶段评审。在规划期间生成 mockup 正是重点所在。

在此例外下允许的命令：
- `mkdir -p ~/.gstack/projects/$SLUG/designs/...`
- `$D generate`、`$D variants`、`$D compare`、`$D iterate`、`$D evolve`、`$D check`
- `open`（当 `$B` 不可用时用于查看板的回退）

首先，设置输出目录。以其正在设计的屏幕/功能和今天的日期命名：

```bash
eval "$(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)"
_DESIGN_DIR=~/.gstack/projects/$SLUG/designs/<screen-name>-$(date +%Y%m%d)
mkdir -p "$_DESIGN_DIR"
echo "DESIGN_DIR: $_DESIGN_DIR"
```

将 `<screen-name>` 替换为描述性的 kebab-case 名称（如 `homepage-variants`、`settings-page`、`onboarding-flow`）。

**在此 Skill 中一次生成一个 mockup。** 内联评审流生成较少的变体并从顺序控制中受益。注意：/design-shotgun 使用并行 Agent 子代理用于变体生成，适用于 Tier 2+（15+ RPM）。这里的顺序约束特定于 plan-design-review 的内联模式。

对于范围内的每个 UI 屏幕/部分，从计划的描述（如果存在还有 DESIGN.md）构建设计简报并生成变体：

```bash
$D variants --brief "<从计划 + DESIGN.md 约束集成的描述>" --count 3 --output-dir "$_DESIGN_DIR/"
```

生成后，对每个变体运行跨模型质量检查：

```bash
$D check --image "$_DESIGN_DIR/variant-A.png" --brief "<原始简报>"
```

标记任何未通过质量检查的变体。提供重新生成失败项的选项。

**不要通过 Read 工具内联展示变体并要求偏好。** 直接进入下面的比较板 + 反馈循环部分。比较板**就是**选择器——它有评级控件、评论、remix/重新生成和结构化反馈输出。内联展示 mockup 是降级的体验。

### 比较板 + 反馈循环

创建比较板并通过 HTTP 提供：

```bash
$D compare --images "$_DESIGN_DIR/variant-A.png,$_DESIGN_DIR/variant-B.png,$_DESIGN_DIR/variant-C.png" --output "$_DESIGN_DIR/design-board.html" --serve
```

此命令生成板 HTML，在随机端口启动 HTTP 服务器，并在用户的默认浏览器中打开它。**在后台运行它**（使用 `&`），因为在用户与板交互时服务器需要保持运行。

从 stderr 输出解析端口：`SERVE_STARTED: port=XXXXX`。你需要它用于板 URL 和重新生成周期的重新加载。

**主要等待：带板 URL 的 AskUserQuestion**

板开始服务后，使用 AskUserQuestion 等待用户。包含板 URL，以便如果他们丢失了浏览器标签页可以点击它：

"我已打开了包含设计变体的比较板：
http://127.0.0.1:<PORT>/——对它们评分、留下评论、混搭你喜欢的元素，完成后点击提交。提交反馈后告诉我（或在这里粘贴你的偏好）。如果你在板上点击了重新生成或混搭，告诉我，我会生成新的变体。"

**不要使用 AskUserQuestion 询问用户偏好哪个变体。** 比较板**就是**选择器。AskUserQuestion 只是阻塞等待机制。

**在用户响应 AskUserQuestion 后：**

在板 HTML 旁边检查反馈文件：
- `$_DESIGN_DIR/feedback.json`——用户点击提交时写入（最终选择）
- `$_DESIGN_DIR/feedback-pending.json`——用户点击重新生成/混搭/更多像这样的时写入

```bash
if [ -f "$_DESIGN_DIR/feedback.json" ]; then
  echo "SUBMIT_RECEIVED"
  cat "$_DESIGN_DIR/feedback.json"
elif [ -f "$_DESIGN_DIR/feedback-pending.json" ]; then
  echo "REGENERATE_RECEIVED"
  cat "$_DESIGN_DIR/feedback-pending.json"
  rm "$_DESIGN_DIR/feedback-pending.json"
else
  echo "NO_FEEDBACK_FILE"
fi
```

反馈 JSON 有这种形状：
```json
{
  "preferred": "A",
  "ratings": { "A": 4, "B": 3, "C": 2 },
  "comments": { "A": "Love the spacing" },
  "overall": "Go with A, bigger CTA",
  "regenerated": false
}
```

**如果找到 `feedback.json`：** 用户在板上点击了提交。
从 JSON 中读取 `preferred`、`ratings`、`comments`、`overall`。使用已批准的变体继续。

**如果找到 `feedback-pending.json`：** 用户在板上点击了重新生成/混搭。
1. 从 JSON 中读取 `regenerateAction`（`"different"`、`"match"`、`"more_like_B"`、`"remix"` 或自定义文本）
2. 如果 `regenerateAction` 是 `"remix"`，读取 `remixSpec`（如 `{"layout":"A","colors":"B"}`）
3. 使用更新后的简报通过 `$D iterate` 或 `$D variants` 生成新的变体
4. 创建新板：`$D compare --images "..." --output "$_DESIGN_DIR/design-board.html"`
5. 在用户浏览器中重新加载板（同一个标签页）：
   `curl -s -X POST http://127.0.0.1:PORT/api/reload -H 'Content-Type: application/json' -d '{"html":"$_DESIGN_DIR/design-board.html"}'`
6. 板自动刷新。**再次使用 AskUserQuestion** 携带相同的板 URL 等待下一轮反馈。重复直到出现 `feedback.json`。

**如果 `NO_FEEDBACK_FILE`：** 用户在 AskUserQuestion 响应中直接输入了他们的偏好而不是使用板。使用他们的文本响应作为反馈。

**轮询回退：** 仅在 `$D serve` 失败时（无端口可用）使用轮询。在这种情况下，使用 Read 工具内联展示每个变体（以便用户可以看到它们），然后使用 AskUserQuestion：
"比较板服务器启动失败。我已在上面展示了变体。你偏好哪个？有任何反馈吗？"

**在收到反馈后（任何路径）：** 输出清晰的摘要确认理解了什么：

"这是我从你的反馈中理解的：
PREFERRED: 变体 [X]
RATINGS: [列表]
YOUR NOTES: [评论]
DIRECTION: [总体]

这对吗？"

使用 AskUserQuestion 在继续之前验证。

**保存已批准的选择：**
```bash
echo '{"approved_variant":"<V>","feedback":"<FB>","date":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","screen":"<SCREEN>","branch":"'$(git branch --show-current 2>/dev/null)'"}' > "$_DESIGN_DIR/approved.json"
```

**不要使用 AskUserQuestion 询问用户选择了哪个变体。** 读取 `feedback.json`——它已经包含了他们的首选变体、评级、评论和总体反馈。仅使用 AskUserQuestion 确认你正确理解了反馈，永远不要重新询问他们选择了什么。

注意哪个方向被批准了。这成为所有后续评审回合的视觉参考。

**多个变体/屏幕：** 如果用户请求了多个变体（例如，"首页的 5 个版本"），将所有变体生成为独立的变体集，带有它们自己的比较板。每个屏幕/变体集在 `designs/` 下获得自己的子目录。在开始评审回合之前完成所有 mockup 生成和用户选择。

**如果 `DESIGN_NOT_AVAILABLE`：** 告诉用户："gstack 设计师尚未设置。运行 `$D setup` 启用视觉 mockup。使用仅文本评审继续，但你错过了最好的部分。"然后使用基于文本的评审继续评审回合。

## 设计外部声音（并行）

使用 AskUserQuestion：
> "在详细评审之前要外部设计声音吗？Codex 根据 OpenAI 的设计硬规则 + 试金石检查进行评估；Claude 子代理做独立的完整性评审。"
>
> A) 是的——运行外部设计声音
> B) 不——不运行继续

如果用户选择 B，跳过此步骤并继续。

**检查 Codex 可用性：**
```bash
which codex 2>/dev/null && echo "CODEX_AVAILABLE" || echo "CODEX_NOT_AVAILABLE"
```

**如果 Codex 可用**，同时启动两个声音：

1. **Codex 设计声音**（通过 Bash）：
（与英文原版相同的 bash 命令，prompt 内容保持英文以确保 Codex 正确执行）

2. **Claude 设计子代理**（通过 Agent 工具）：
（prompt 保持英文）

**错误处理（全部非阻塞）：**
- **认证失败：** 如果 stderr 包含 "auth"、"login"、"unauthorized" 或 "API key"："Codex 认证失败。运行 `codex login` 认证。"
- **超时：** "Codex 5 分钟后超时。"
- **空响应：** "Codex 返回了空响应。"
- 任何 Codex 错误时：仅使用 Claude 子代理输出继续，标记为 `[single-model]`。
- 如果 Claude 子代理也失败："外部声音不可用——继续主评审。"

在 `CODEX SAYS (design critique):` 标题下展示 Codex 输出。
在 `CLAUDE SUBAGENT (design completeness):` 标题下展示子代理输出。

**综合——试金石记分卡：**

（展示与原文相同的记分卡格式，表头翻译为中文）

```
设计外部声音——试金石记分卡：
═══════════════════════════════════════════════════════════════
  检查                                         Claude  Codex  共识
  ─────────────────────────────────────── ─────── ─────── ─────────
  1. 品牌在首屏中 unmistakable？             —       —      —
  2. 有一个强的视觉锚点？                     —       —      —
  3. 仅扫描标题即可理解页面？                 —       —      —
  4. 每个部分只有一个工作？                   —       —      —
  5. 卡片真的有必要吗？                       —       —      —
  6. 动效改进了层次结构或氛围？               —       —      —
  7. 移除所有装饰阴影后仍感觉高级？           —       —      —
  ─────────────────────────────────────── ─────── ─────── ─────────
  触发的硬拒绝：                             —       —      —
═══════════════════════════════════════════════════════════════
```

从 Codex 和子代理输出中填充每个单元格。CONFIRMED = 两者同意。DISAGREE = 模型不同。NOT SPEC'D = 信息不足无法评估。

**通过集成（尊重现有的 7 回合约定）：**
- 硬拒绝 → 作为 Pass 1 中的**第一项**提出，标记为 `[HARD REJECTION]`
- Litmus DISAGREE 项 → 在相关回合中带着两种观点提出
- Litmus CONFIRMED 失败 → 作为已知问题预加载到相关回合中
- 回合可以跳过发现，直接修复已识别的问题

**记录结果：**
```bash
~/.claude/skills/gstack/bin/gstack-review-log '{"skill":"design-outside-voices","timestamp":"'"$(date -u +%Y-%m-%dT%H:%M:%SZ)"'","status":"STATUS","source":"SOURCE","commit":"'"$(git rev-parse --short HEAD)"'"}'
```
将 STATUS 替换为 "clean" 或 "issues_found"，SOURCE 替换为 "codex+subagent"、"codex-only"、"subagent-only" 或 "unavailable"。

## 0-10 评级方法

对于每个设计部分，对该维度上的计划进行 0-10 评级。如果不是 10 分，解释**什么**会使它成为 10 分——然后做工作达到那里。

模式：
1. 评级："信息架构：4/10"
2. 差距："是 4 因为计划没有定义内容层次结构。10 分会为每个屏幕定义清晰的主要/次要/第三级。"
3. 修复：编辑计划以添加缺失的内容
4. 重新评级："现在 8/10——仍然缺少移动端导航层次结构"
5. 如果有一个真正的设计选择要解决则使用 AskUserQuestion
6. 再次修复→重复直到 10 或用户说"够了，继续"

重新运行循环：再次调用 /plan-design-review→重新评级→8+ 的部分快速通过，低于 8 的部分全面处理。

### "让我看看 10/10 长什么样"（需要设计二进制）

如果设置期间打印了 `DESIGN_READY` 且某个维度评级低于 7/10，提供生成视觉 mockup 以展示改进后的版本会是什么样子：

```bash
$D generate --brief "<此维度 10/10 是什么样子的描述>" --output /tmp/gstack-ideal-<dimension>.png
```

通过 Read 工具向用户展示 mockup。这使得"计划描述的"和"它应该是什么样"之间的差距变得直观，而不是抽象的。

如果设计二进制不可用，跳过此步骤并继续使用基于文本的描述来说明 10/10 长什么样。

## 评审部分（7 回合，范围确定后）

**反跳过规则：** 无论计划类型如何（策略、规范、代码、基础设施），永远不要压缩、缩写或跳过任何评审回合（1-7）。此 Skill 中的每个回合都有其存在的理由。"这是策略文档所以设计回合不适用"永远是不对的——设计缺口正是实现崩溃的地方。如果某个回合确实零发现，说"No issues found"然后继续——但你必须评估它。

## Prior Learnings

（与 plan-eng-review/SKILL-zh.md 中的 Prior Learnings 部分相同，已翻译）

### 第 1 回合：信息架构

评级 0-10：计划是否定义了用户首先、第二、第三看到什么？
**修复到 10：** 向计划添加信息层次结构。包含屏幕/页面结构和导航流的 ASCII 图表。应用"约束崇拜"——如果你只能展示 3 样东西，哪 3 个最重要？
**STOP。** 每个问题调用一次 AskUserQuestion。不要批量处理。建议 + 为什么。如果没有问题，说明并继续。在用户响应之前不要继续。

### 第 2 回合：交互状态覆盖

评级 0-10：计划是否指定了加载、空、错误、成功、部分状态？
**修复到 10：** 向计划添加交互状态表：
```
  功能                  | 加载中 | 空   | 错误 | 成功  | 部分
  ---------------------|---------|-------|-------|---------|--------
  [每个 UI 功能]        | [规格]  | [规格]| [规格]| [规格]  | [规格]
```
对于每个状态：描述用户**看到什么**，不是后端行为。
空状态是功能——指定温暖、主要操作、上下文。
**STOP。** 每个问题调用一次 AskUserQuestion。不要批量处理。建议 + 为什么。

### 第 3 回合：用户旅程与情感弧

评级 0-10：计划是否考虑了用户的情感体验？
**修复到 10：** 添加用户旅程故事板：
```
  步骤 | 用户做什么        | 用户感受        | 计划指定了？
  -----|------------------|-----------------|----------------
  1    | 到达页面          | [什么情绪？]    | [什么支持它？]
  ...
```
应用时间维度设计：5 秒本能的、5 分钟行为的、5 年反思的。
**STOP。** 每个问题调用一次 AskUserQuestion。不要批量处理。建议 + 为什么。

### 第 4 回合：AI Slop 风险

评级 0-10：计划是否描述了具体的、有意为之的 UI——还是通用模式？
**修复到 10：** 用具体的替代方案重写模糊的 UI 描述。

### 设计硬规则

**分类器——在评估之前确定规则集：**
- **MARKETING/着陆页**（英雄驱动的、品牌优先的、转化聚焦的）→ 应用着陆页规则
- **APP UI**（工作空间驱动的、数据密集的、任务聚焦的：仪表板、管理、设置）→ 应用 App UI 规则
- **HYBRID**（带有类似应用部分的营销外壳）→ 对英雄/营销部分应用着陆页规则，对功能部分应用 App UI 规则

**硬拒绝标准**（即时失败模式——如果有任何一项适用则标记）：
1. 通用 SaaS 卡片网格作为第一印象
2. 美丽的图片搭配弱的品牌
3. 强烈的标题没有明确的行动
4. 文本后面的繁杂图像
5. 重复相同情绪声明的部分
6. 没有叙事目的的轮播
7. 由堆叠卡片而非布局组成的 App UI

**试金石检查**（每个回答 YES/NO——用于跨模型共识评分）：
1. 品牌/产品在首屏中 unmistakable？
2. 有一个强的视觉锚点存在？
3. 仅扫描标题即可理解页面？
4. 每个部分只有一个工作？
5. 卡片真的有必要吗？
6. 动效改进了层次结构或氛围？
7. 移除所有装饰阴影后仍感觉高级？

**着陆页规则**（当分类器 = MARKETING/LANDING 时应用）：
- 第一个视口读起来是一个组合，不是仪表板
- 品牌优先的层次结构：品牌 > 标题 > 正文 > CTA
- 字体：有表现力的、有目的的——没有默认堆栈（Inter、Roboto、Arial、system）
- 没有纯单色背景——使用渐变、图像、微妙图案
- Hero：full-bleed、边到边、没有 inset/平铺/圆角变体
- Hero 预算：品牌、一个标题、一个支持句、一个 CTA 组、一个图像
- Hero 中没有卡片。仅当卡片**就是**交互时才使用卡片
- 每个部分一个工作：一个目的、一个标题、一个短支持句
- 动效：最少 2-3 个有意图的动效（入场、滚动链接、hover/揭示）
- 颜色：定义 CSS 变量，避免紫色-on-白色默认，默认一个强调色
- 文案：产品语言而非设计评论。"如果删除 30% 改进了它，继续删除"
- 美丽的默认：组合优先、品牌作为最大声的文本、最多两种字体、默认无卡片、首视口作为海报而非文档

**App UI 规则**（当分类器 = APP UI 时应用）：
- 平静的表面层次结构、强的字体、少量的颜色
- 密集但可读、最小的 chrome
- 组织：主要工作空间、导航、次要上下文、一个强调
- 避免：仪表板卡片马赛克、粗边框、装饰渐变、装饰图标
- 文案：实用语言——定位、状态、行动。不是情绪/品牌/抱负
- 仅当卡片**就是**交互时才使用卡片
- 部分标题说明区域是什么或用户可以做什么（"选定的 KPI"、"计划状态"）

**通用规则**（适用于所有类型）：
- 为颜色系统定义 CSS 变量
- 没有默认字体堆栈（Inter、Roboto、Arial、system）
- 每个部分一个工作
- "如果删除 30% 的文案改进了它，继续删除"
- 卡片挣得它们的存在——没有装饰性卡片网格
- **绝不**使用小的、低对比度的字体（正文文本 < 16px 或正文文本对比度比 < 4.5:1）
- **绝不**在表单字段内仅使用占位符作为标签（占位符作为标签模式——当字段有内容时标签必须可见）
- **始终**保留已访问 vs 未访问链接的区别（已访问的链接必须有不同颜色）
- **绝不**在段落之间浮动标题（标题必须在视觉上更靠近它引入的部分而不是前面的部分）

**AI Slop 黑名单**（10 个尖叫"AI 生成"的模式）：
1. 紫色/紫罗兰/靛蓝渐变背景或蓝到紫配色方案
2. **3 列特性网格：** 彩色圆圈中的图标 + 粗标题 + 2 行描述，对称重复 3 次。最容易识别的 AI 布局。
3. 彩色圆圈中的图标作为部分装饰（SaaS 启动模板的外观）
4. 一切都居中（所有标题、描述、卡片上的 `text-align: center`）
5. 每个元素上统一的起泡 border-radius（一切上相同的大圆角）
6. 装饰性斑点、浮动圆圈、波浪状 SVG 分隔线（如果某个部分感觉空，它需要更好的内容，不是装饰）
7. Emoji 作为设计元素（标题中的火箭、emoji 作为项目符号）
8. 卡片左侧彩色边框（`border-left: 3px solid <accent>`）
9. 通用 Hero 文案（"欢迎来到 [X]"、"解锁...的力量"、"你的一站式解决方案..."）
10. 千篇一律的部分节奏（hero → 3 个特性 → 推荐 → 定价 → CTA，每个部分相同高度）

来源：[OpenAI "Designing Delightful Frontends with GPT-5.4"](https://developers.openai.com/blog/designing-delightful-frontends-with-gpt-5-4)（2026 年 3 月）+ gstack 设计方法论。
- "带图标的卡片"→ 这些与每个 SaaS 模板有什么区别？
- "Hero 部分"→ 什么让这个 hero 感觉像**这个**产品？
- "干净、现代的 UI"→ 无意义。用实际的设计决策替换。
- "带小部件的仪表板"→ 什么让这个不同于其他每个仪表板？

如果在 Step 0.5 中生成了视觉 mockup，根据上面的 AI Slop 黑名单评估它们。使用 Read 工具读取每个 mockup 图像。mockup 是否落入通用模式（3 列网格、居中 hero、库存照片感觉）？如果是，标记它并提供通过 `$D iterate --feedback "..."` 使用更具体的方向重新生成。
**STOP。** 每个问题调用一次 AskUserQuestion。不要批量处理。建议 + 为什么。

### 第 5 回合：设计系统对齐

评级 0-10：计划是否与 DESIGN.md 对齐？
**修复到 10：** 如果 DESIGN.md 存在，用具体的 token/组件注释。如果没有 DESIGN.md，标记缺口并推荐 `/design-consultation`。
标记任何新组件——它适合现有的词汇表吗？
**STOP。** 每个问题调用一次 AskUserQuestion。不要批量处理。建议 + 为什么。

### 第 6 回合：响应式与可访问性

评级 0-10：计划是否指定了 mobile/tablet、键盘导航、屏幕阅读器？
**修复到 10：** 为每个视口添加响应式规格——不是"移动端堆叠"而是有意为之的布局更改。添加 a11y：键盘导航模式、ARIA 地标、触摸目标大小（最小 44px）、颜色对比度要求。
**STOP。** 每个问题调用一次 AskUserQuestion。不要批量处理。建议 + 为什么。

### 第 7 回合：未解决的设计决策

揭示会让实现痛苦的模糊之处：
```
  需要决策的                    | 如果推迟会发生什么
  -----------------------------|---------------------------
  空状态长什么样？                | 工程师交付"No items found."
  移动端导航模式？               | 桌面导航隐藏在汉堡菜单后面
  ...
```
如果在 Step 0.5 中生成了视觉 mockup，在揭示未解决的决策时将它们作为证据引用。Mockup 使决策具体化——例如，"你批准的 mockup 显示了侧边栏导航，但计划没有指定移动端行为。这个侧边栏在 375px 上会怎样？"
每个决策 = 一个 AskUserQuestion，附带建议 + 为什么 + 替代方案。在做出每个决策时编辑计划。

### 通过后：更新 Mockup（如果已生成）

如果在 Step 0.5 中生成了 mockup 且评审回合更改了重要的设计决策（信息架构重组、新状态、布局更改），提供重新生成（一次性，不是循环）：

AskUserQuestion："评审回合更改了 [列出主要设计更改]。要我重新生成 mockup 以反映更新后的计划吗？这确保视觉参考与我们实际构建的匹配。"

如果是，使用 `$D iterate` 和总结更改的反馈，或使用更新后的简报使用 `$D variants`。保存到相同的 `$_DESIGN_DIR` 目录。

## 关键规则——如何提问

遵循上面 Preamble 中的 AskUserQuestion 格式。计划设计评审的额外规则：
* **一个问题 = 一次 AskUserQuestion 调用。** 永远不要将多个问题合并到一个问题中。
* 具体地描述设计缺口——缺失了什么，如果不指定用户会体验到什么。
* 展示 2-3 个选项。对于每个：现在指定的工作量，如果推迟的风险。
* **映射到上面的设计原则。** 用一句话将你的建议连接到特定原则。
* 用编号 + 字母标记（如 "3A"、"3B"）。
* **逃生舱：** 如果某个部分没有问题，说明并继续。如果缺口有明显的修复，说明你要添加什么然后继续——不要在一个明显的问题上浪费一个问题。仅在存在有意义的真正设计选择时才使用 AskUserQuestion。
* **永远不要使用 AskUserQuestion 询问用户偏好哪个变体。** 始终先创建比较板（`$D compare --serve`）并在浏览器中打开它。板有评级控件、评论、混搭/重新生成按钮和结构化反馈输出。仅使用 AskUserQuestion 通知用户板已打开并等待他们完成——不是内联展示变体并询问"你偏好哪个？"那是降级的体验。

## 必需的输出

### "NOT in scope" 部分

考虑并显式推迟的设计决策，每项一行理由。

### "What already exists" 部分

计划应该重用的现有 DESIGN.md、UI 模式和组件。

### TODOS.md 更新

所有评审回合完成后，将每个潜在的 TODO 作为单独的 AskUserQuestion 呈现。永远不要批量处理 TODO——每个问题一个。永远不要静默跳过此步骤。

对于设计债务：缺失的 a11y、未解决的响应式行为、推迟的空状态。每个 TODO 获取：
* **What：** 一行描述。
* **Why：** 它解决的具体问题或解锁的价值。
* **Pros：** 做这项工作获得什么。
* **Cons：** 做这项工作的成本、复杂性或风险。
* **Context：** 足够的细节让 3 个月后拿起它的人理解动机。
* **Depends on / blocked by：** 任何前置条件。

然后展示选项：**A)** 添加到 TODOS.md **B)** 跳过——不够有价值 **C)** 现在就在此 PR 中构建而不推迟。

### Completion Summary

```
  +====================================================================+
  |         设计计划评审——完成摘要                     |
  +====================================================================+
  | 系统审计           | [DESIGN.md 状态，UI 范围]                |
  | Step 0               | [初始评级，聚焦领域]                  |
  | 回合 1（信息架构） | ___/10 → 修复后 ___/10                |
  | 回合 2（状态）     | ___/10 → 修复后 ___/10                |
  | 回合 3（旅程）    | ___/10 → 修复后 ___/10                |
  | 回合 4（AI Slop） | ___/10 → 修复后 ___/10                |
  | 回合 5（设计系统）| ___/10 → 修复后 ___/10                |
  | 回合 6（响应式）| ___/10 → 修复后 ___/10                |
  | 回合 7（决策）  | ___ 已解决，___ 已推迟                 |
  +--------------------------------------------------------------------+
  | NOT in scope         | 已写（___ 项）                         |
  | What already exists  | 已写                                     |
  | TODOS.md 更新    | 提议了 ___ 项                          |
  | 已批准 Mockups     | 生成了 ___ 个，批准了 ___ 个              |
  | 做出的决策       | ___ 个已添加到计划                       |
  | 推迟的决策     | ___ 个（在下面列出）                          |
  | 总体设计分数 | ___/10 → ___/10                             |
  +====================================================================+
```

如果所有回合 8+："计划已设计完整。实现后运行 /design-review 进行视觉 QA。"
如果有低于 8 的：注明什么未解决以及为什么（用户选择推迟）。

### 未解决的决策

如果任何 AskUserQuestion 未得到响应，在这里注明。永远不要静默默认为某个选项。

### 已批准的 Mockup

如果在此评审期间生成了视觉 mockup，向计划文件添加：

```
## Approved Mockups

| 屏幕/部分 | Mockup 路径 | 方向 | 说明 |
|----------------|-------------|-----------|-------|
| [屏幕名称]  | ~/.gstack/projects/$SLUG/designs/[folder]/[filename].png | [简要描述] | [来自评审的约束] |
```

包含每个已批准 mockup 的完整路径（用户选择的变体）、一行方向描述和任何约束。实现者读取它以知道确切从哪个视觉构建。这些在对话和工作区之间持续存在。如果没有生成 mockup，省略此部分。

## 评审日志

生成上述 Completion Summary 后，持久化评审结果。

**PLAN MODE EXCEPTION — 始终运行：** 此命令将评审元数据写入 `~/.gstack/`（用户配置目录，不是项目文件）。Skill preamble 已经写入 `~/.gstack/sessions/` 和 `~/.gstack/analytics/`——这是相同的模式。评审仪表板依赖于此数据。跳过此命令会破坏 /ship 中的评审准备度仪表板。

```bash
~/.claude/skills/gstack/bin/gstack-review-log '{"skill":"plan-design-review","timestamp":"TIMESTAMP","status":"STATUS","initial_score":N,"overall_score":N,"unresolved":N,"decisions_made":N,"commit":"COMMIT"}'
```

从 Completion Summary 中替换值：
- **TIMESTAMP**：当前 ISO 8601 日期时间
- **STATUS**：如果总体分数 8+ 且 0 未解决则为 "clean"；否则 "issues_open"
- **initial_score**：修复前初始总体设计分数（0-10）
- **overall_score**：修复后最终总体设计分数（0-10）
- **unresolved**：未解决设计决策的数量
- **decisions_made**：添加到计划的设计决策数量
- **COMMIT**：`git rev-parse --short HEAD` 的输出

## Review Readiness Dashboard

（与 devex-review/SKILL-zh.md 中的 Review Readiness Dashboard 相同，已翻译）

## Plan File Review Report

（与 devex-review/SKILL-zh.md 中的 Plan File Review Report 相同，已翻译）

## Capture Learnings

（与 devex-review/SKILL-zh.md 中的 Capture Learnings 部分相同，已翻译。注意将 skill 名称替换为 `plan-design-review`。）

## 下一步——评审链

显示 Review Readiness Dashboard 后，根据此设计评审发现的内容推荐下一个评审。读取仪表板输出查看哪些评审已运行以及是否过期。

**如果 eng 评审未全局跳过，推荐 /plan-eng-review**——检查仪表板输出中的 `skip_eng_review`。如果为 `true`，eng 评审被选择退出——不要推荐它。否则，eng 评审是必需的发布门禁。如果此设计评审添加了重要的交互规格、新的用户流或更改了信息架构，强调 eng 评审需要验证架构影响。如果 eng 评审已存在但 commit hash 显示它早于此设计评审，注明它可能已过期并应重新运行。

**考虑推荐 /plan-ceo-review**——但仅当此设计评审揭示了根本性的产品方向缺口时。具体来说：如果总体设计分开始低于 4/10、如果信息架构有重大结构问题、或者评审提出了关于是否解决了正确的问题。并且仪表板中不存在 CEO 评审。这是选择性推荐——大多数设计评审**不应该**触发 CEO 评审。

**如果两者都需要，首先推荐 eng 评审**（必需的门禁）。

**适当时推荐设计探索 Skill**——/design-shotgun 和 /design-html 生成设计产物（mockup、HTML 预览），不是应用代码。它们与评审一起在 plan mode 中。如果此设计评审发现了从探索新方向中受益的视觉问题，推荐 /design-shotgun。如果存在已批准的 mockup 且需要转换为工作的 HTML，推荐 /design-html。

使用 AskUserQuestion 展示下一步。仅包含适用的选项：
- **A)** 接下来运行 /plan-eng-review（必需的门禁）
- **B)** 运行 /plan-ceo-review（仅在发现根本性产品缺口时）
- **C)** 运行 /design-shotgun——探索发现的问题的视觉设计变体
- **D)** 运行 /design-html——从已批准的 mockup 生成 Pretext 原生 HTML
- **E)** 跳过——我自己处理下一步

## 格式规则

* 用数字（1、2、3...）标记问题，用字母（A、B、C...）标记选项。
* 用编号 + 字母标记（如 "3A"、"3B"）。
* 每个选项最多一句话。
* 每个回合之后，暂停并等待反馈。
* 在每个回合之前和之后都评级以增强可扫描性。

## Review Readiness Dashboard

完成评审后，读取评审日志和配置以显示仪表板。

```bash
~/.claude/skills/gstack/bin/gstack-review-read
```

解析输出。找到每个 Skill 最近的条目（plan-ceo-review、plan-eng-review、review、plan-design-review、design-review-lite、adversarial-review、codex-review、codex-plan-review）。忽略超过 7 天时间戳的条目。对于 Eng Review 行，显示 `review`（diff 范围的预发布评审）和 `plan-eng-review`（计划阶段的架构评审）中更新的那个。在状态后追加 "(DIFF)" 或 "(PLAN)" 以区分。对于 Adversarial 行，显示 `adversarial-review`（新的自动缩放）和 `codex-review`（旧版）中更新的那个。对于 Design Review，显示 `plan-design-review`（完整视觉审计）和 `design-review-lite`（代码级别检查）中更新的那个。在状态后追加 "(FULL)" 或 "(LITE)" 以区分。对于 Outside Voice 行，显示最近的 `codex-plan-review` 条目——这捕获了来自 /plan-ceo-review 和 /plan-eng-review 的外部声音。

**来源归属：** 如果某个 Skill 的最近条目有 `"via"` 字段，将其附加到状态标签中（括号内）。例如：带有 `via:"autoplan"` 的 `plan-eng-review` 显示为 "CLEAR (PLAN via /autoplan)"。带有 `via:"ship"` 的 `review` 显示为 "CLEAR (DIFF via /ship)"。

注意：`autoplan-voices` 和 `design-outside-voices` 条目仅用于审计跟踪（法医数据），不出现在仪表板中。

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
- **CEO Review（可选）：** 对于重大产品/业务更改、新面向用户功能推荐。
- **Design Review（可选）：** 对于 UI/UX 更改推荐。
- **Adversarial Review（自动）：** 始终开启。
- **Outside Voice（可选）：** 来自不同 AI 模型的独立计划评审。

**结论逻辑：**
- **CLEARED**：Eng Review 在最近 7 天内有 >= 1 个条目，状态 "clean"（或 `skip_eng_review` 为 `true`）
- **NOT CLEARED**：Eng Review 缺失、过期（>7 天）或有未解决问题

**过期检测：** 显示仪表板后，检查现有评审是否可能过期：
- 从 bash 输出的 `---HEAD---` 部分解析当前 HEAD commit hash
- 对于每个有 `commit` 字段的评审条目：与当前 HEAD 比较。如果不同，`git rev-list --count STORED_COMMIT..HEAD`。显示："注意：{skill} 评审来自 {date} 可能已过期——评审以来有 {N} 个 commits"

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

- **CODEX：**（仅当 codex-review 运行时）——codex 修复的一行摘要
- **CROSS-MODEL：**（仅当 Claude 和 Codex 评审都存在时）——重叠分析
- **UNRESOLVED：** 所有评审中未解决的决策总数
- **VERDICT：** 列出 CLEAR 的评审。如果 Eng Review 不是 CLEAR 且未全局跳过，追加 "eng review required"。

### 写入计划文件

**PLAN MODE EXCEPTION — 始终运行：** 这写入计划文件，这是你在 plan mode 中唯一被允许编辑的文件。

- 在计划文件中搜索 `## GSTACK REVIEW REPORT` 部分，可以出现在文件**任何位置**。
- 如果找到，使用 Edit 工具**整体替换**。匹配从 `## GSTACK REVIEW REPORT` 到下一个 `## ` 标题或文件末尾。
- 如果不存在此部分，**追加**到计划文件末尾。
- 始终放在计划文件最末尾。

## Capture Learnings

如果你在会话中发现了非显而易见的模式、陷阱或架构洞察，记录下来供未来会话使用：

```bash
~/.claude/skills/gstack/bin/gstack-learnings-log '{"skill":"plan-design-review","type":"TYPE","key":"SHORT_KEY","insight":"DESCRIPTION","confidence":N,"source":"SOURCE","files":["path/to/relevant/file"]}'
```

**类型：** `pattern`（可复用的方法）、`pitfall`（不要做什么）、`preference`（用户声明）、`architecture`（结构决策）、`tool`（库/框架洞察）、`operational`（项目环境/CLI/工作流知识）。

**来源：** `observed`、`user-stated`、`inferred`、`cross-model`。

**置信度：** 1-10。已验证的模式 8-9，不确定的推理 4-5，用户明确声明的 10。

**files：** 引用的具体文件路径，用于过期检测。

**仅记录真正的发现。** 不要记录显而易见或用户已知的事情。

## 下一步——评审链

显示 Review Readiness Dashboard 后，根据此设计评审发现的内容推荐下一个评审。

**如果 eng 评审未全局跳过，推荐 /plan-eng-review**——如果此设计评审添加了重要的交互规格、新的用户流或更改了信息架构，强调 eng 评审需要验证架构影响。

**考虑推荐 /plan-ceo-review**——但仅当此设计评审揭示了根本性的产品方向缺口时。具体来说：如果总体设计分开始低于 4/10、信息架构有重大结构问题、或者评审提出了关于是否解决了正确的问题。**并且**仪表板中不存在 CEO 评审。大多数设计评审**不应该**触发 CEO 评审。

**如果两者都需要，首先推荐 eng 评审**（必需的门禁）。

**适当时推荐设计探索 Skill**——如果此设计评审发现了从探索新方向中受益的视觉问题，推荐 /design-shotgun。如果存在已批准的 mockup 且需要转换为工作的 HTML，推荐 /design-html。

使用 AskUserQuestion 展示下一步。仅包含适用的选项：
- **A)** 接下来运行 /plan-eng-review（必需的门禁）
- **B)** 运行 /plan-ceo-review（仅在发现根本性产品缺口时）
- **C)** 运行 /design-shotgun——探索发现的问题的视觉设计变体
- **D)** 运行 /design-html——从已批准的 mockup 生成 Pretext 原生 HTML
- **E)** 跳过——我自己处理下一步
