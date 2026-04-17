---
name: design-review
preamble-tier: 4
version: 2.0.0
description: |
  设计师的眼睛做 QA：发现视觉不一致、间距问题、层次结构问题、
  AI slop 模式和缓慢交互——然后修复它们。迭代修复源代码中的问题，
  原子提交每个修复并在修复前后用截图重新验证。对于计划阶段的设
  计评审（实现之前），使用 /plan-design-review。
  当被要求"审计设计"、"视觉 QA"、"检查看起来好不好"或"设计打磨"时使用。
  当用户提到视觉不一致或想要打磨在线站点的外观时主动建议。(gstack)
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - AskUserQuestion
  - WebSearch
---
<!-- AUTO-GENERATED from SKILL.md.tmpl — do not edit directly -->
<!-- Regenerate: bun run gen:skill-docs -->

> Preamble、Voice、Context Recovery、AskUserQuestion 格式、Completeness Principle、
> Repo Ownership、Search Before Building、Completion Status Protocol、Escalation、
> Operational Self-Improvement、Telemetry、Plan Mode Safe Operations、
> Skill Invocation During Plan Mode、Plan Status Footer 和 SETUP 部分与 office-hours
> 相同，请参阅 `../office-hours/SKILL-zh.md` 中对应的翻译内容。
---
name: design-review
preamble-tier: 2
version: 1.0.0
description: |
  设计审查（Design Audit）：对已实现的 HTML/CSS 进行逐页视觉审计。比较实现与设计系统、
  检查响应式行为、识别不一致，并自动修复机械问题。
  适用于："审查设计"、"这看起来对吗"、"视觉审计"或"像素级检查"。
  当用户实现设计并想验证它是否与 mockup 匹配时主动建议。（gstack）
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Agent
  - AskUserQuestion
---

---

## UX 原则：用户实际如何行为

这些原则支配真实人类如何与界面交互。它们是观察到的行为，不是偏好。在每个设计决策之前、期间和之后应用它们。

### 可用性三定律

1. **别让我思考。** 每个页面应该是自明的。如果用户停下来思考"我点哪里？"或"这是什么意思？"，设计就失败了。
   自明 > 自解释 > 需要说明。

2. **点击次数不重要，思考才重要。** 三次无脑、明确的选择
   胜过一次需要思考的点击。每一步应该感觉像一个显而易见的
   选择（动物、植物还是矿物），不是拼图。

3. **省略，再省略。** 去掉每页上一半的词，然后再去掉剩下的一半。废话（自我祝贺的文本）必须死。
   说明必须死。如果需要阅读，设计就失败了。

### 用户实际如何行为

- **用户扫描，不阅读。** 为扫描而设计：视觉层次结构
  （突出度 = 重要性）、清晰定义的区域、标题和项目符号列表、
  高亮关键词。我们在设计以 60 英里时速驶过的广告牌，不是
  人们会研读的产品手册。
- **用户满足即可。** 他们选第一个合理的选项，不是最好的。
  让正确的选择成为最显眼的选择。
- **用户摸索。** 他们不弄清楚东西怎么工作。他们即兴发挥。
  如果偶然达成了目标，他们不会去寻找"正确"的方式。
  一旦找到有效的东西，无论多差，他们都坚持用。
- **用户不读说明。** 他们直接上手。指南必须简短、
  及时、不可避免，否则不会被看到。

### 界面的广告牌设计

- **使用惯例。** Logo 左上、导航顶部/左侧、搜索 = 放大镜。
  不要在导航上为了聪明而创新。在**知道**你有更好的
  想法时才创新，否则使用惯例。即使跨语言和文化，
  网页惯例让人们能识别 logo、导航、搜索和主要内容。
- **视觉层次结构是一切。** 相关的事物在视觉上分组。嵌套
  的事物在视觉上被包含。越重要 = 越突出。如果所有东西
  都在喊，什么也听不到。从假设所有东西都是视觉噪音开始，
  有罪直到证明无辜。
- **让可点击的东西明显可点击。** 不要依赖 hover 状态来
  发现，尤其是在 hover 不存在的移动端。形状、位置、
  和格式（颜色、下划线）必须在无交互时传递可点击性。
- **消除噪音。** 三个来源：太多东西在抢注意力
  （大喊）、东西没有逻辑地组织（混乱）、东西太多
  （杂乱）。通过移除而非添加来修复噪音。
- **清晰度胜过一致性。** 如果让某物明显更清晰
  需要让它轻微不一致，每次都选择清晰度。

### 导航作为问路

网页上的用户没有尺度、方向或位置感。导航
必须始终回答：这是什么网站？我在哪一页？主要的
部分是什么？我在这个层级有哪些选项？我在哪？我怎么搜索？

每个页面持久的导航。深层层次结构的面包屑。
当前部分视觉标示。"树干测试"：遮住一切只留
导航。你仍然应该知道这是什么网站、你在哪一页、
主要部分是什么。如果不是，导航就失败了。

### 善意水库

用户开始时有一个善意储备。每个摩擦点都在消耗它。

**消耗更快：** 隐藏用户想要的信息（定价、联系、运输）。惩罚
用户没按你的方式做事（电话号码格式要求）。
请求不必要的信息。用华丽东西挡路（闪屏、
强制导览、插页）。不专业或邋遢的外观。

**补充：** 知道用户想做什么并让它明显。提前告诉他们
想知道的。尽可能帮他们省步骤。
让他们轻松从错误中恢复。有疑问时，道歉。

### 移动端：同样的规则，更高的赌注

以上所有内容在移动端适用，而且更要如此。空间稀缺，但永远
不要为了节省空间牺牲可用性。Affordance 必须**可见**：没有光标
意味着没有 hover 来发现。触摸目标必须足够大（最小 44px）。
扁平化设计可能会剥离发出交互信号的有效视觉信息。
无情地排优先级：紧急需要的东西放在随手可及处，其他
的几步之内，有明显路径到达。

## Phase 1-6: 设计审计基线

## Modes

### Full（默认）
系统审查从首页可访问的所有页面。访问 5-8 页。完整的清单评估、响应式截图、交互流程测试。生成带字母评分的完整设计审计报告。

### Quick (`--quick`)
仅首页 + 2 个关键页。第一印象 + 设计系统提取 + 简略清单。最快获得设计评分的路径。

### Deep (`--deep`)
全面审查：10-15 页、每个交互流程、详尽清单。适用于发布前审计或重大改版。

### Diff-aware（在无 URL 的 feature branch 上自动触发）
在 feature branch 上时，范围限制在 branch 更改影响的页面：
1. 分析 branch diff：`git diff main...HEAD --name-only`
2. 映射更改的文件到受影响的页面/路由
3. 检测常见本地端口上的运行应用（3000、4000、8080）
4. 仅审计受影响的页面、比较设计质量前后

### Regression（`--regression` 或找到之前的 `design-baseline.json`）
运行完整审计，然后加载之前的 `design-baseline.json`。比较：每类别评分差异、新发现、已解决的发现。在报告中输出回归表。

---

## Phase 1: First Impression

最独特像设计师的输出。在分析任何东西之前形成直觉反应。

1. 导航到目标 URL
2. 拍一张全屏桌面截图：`$B screenshot "$REPORT_DIR/screenshots/first-impression.png"`
3. 用以下结构化批评格式写**第一印象**：
   - "这个站点传达了**[什么]**。"（一眼看出什么——能力？玩味？困惑？）
   - "我注意到**[观察]**。"（什么引人注目，正面或负面——要具体）
   - "我的眼睛首先去的 3 个东西是：**[1]**、**[2]**、**[3]**。"（层次检查——这些是设计师打算的 3 个东西吗？如果不是，视觉层次在撒谎。）
   - "如果一个词形容这个：**[词]**。"（直觉判断）

**叙述模式：** 用第一人称写这一部分，就像你第一次扫描页面的用户。"我看这个页面... 我的眼睛去了 logo，然后一整面我完全跳过的文字墙，然后... 等一下，那是按钮吗？" 说出具体元素、它的位置、它的视觉重量。如果你不能具体命名，你不是在真正扫描，你在生成套话。

**页面区域测试：** 指向页面上每个清晰定义的区域。你能立即说出它的目的吗？（"我可以买的东西"、"今日优惠"、"怎么搜索。"）2 秒内说不出名字的区域定义不好。列出它们。

这是用户首先阅读的部分。要有主见。设计师不模棱两可——他们做出反应。

---

## Phase 2: 设计系统提取

提取站点实际使用的设计系统（不是 DESIGN.md 说的，而不是渲染出来的）：

```bash
# Fonts in use (capped at 500 elements to avoid timeout)
$B js "JSON.stringify([...new Set([...document.querySelectorAll('*')].slice(0,500).map(e => getComputedStyle(e).fontFamily))])"

# Color palette in use
$B js "JSON.stringify([...new Set([...document.querySelectorAll('*')].slice(0,500).flatMap(e => [getComputedStyle(e).color, getComputedStyle(e).backgroundColor]).filter(c => c !== 'rgba(0, 0, 0, 0)'))])"

# Heading hierarchy
$B js "JSON.stringify([...document.querySelectorAll('h1,h2,h3,h4,h5,h6')].map(h => ({tag:h.tagName, text:h.textContent.trim().slice(0,50), size:getComputedStyle(h).fontSize, weight:getComputedStyle(h).fontWeight})))"

# Touch target audit (find undersized interactive elements)
$B js "JSON.stringify([...document.querySelectorAll('a,button,input,[role=button]')].filter(e => {const r=e.getBoundingClientRect(); return r.width>0 && (r.width<44||r.height<44)}).map(e => ({tag:e.tagName, text:(e.textContent||'').trim().slice(0,30), w:Math.round(e.getBoundingClientRect().width), h:Math.round(e.getBoundingClientRect().height)})).slice(0,20))"

# Performance baseline
$B perf
```

将发现结构化为**推断的设计系统**：
- **Fonts：** 表带使用计数。如果超过 3 个不同字体家族则标记。
- **颜色：** 提取的调色板。如果超过 12 种唯一非灰色则标记。注明暖/冷/混合。
- **标题标尺：** h1-h6 大小。标记跳过的层级、非系统性的尺寸跳跃。
- **间距模式：** 采样 padding/margin 值。标记非标尺值。

提取后，提供：*"要我将此保存为你的 DESIGN.md 吗？我可以将这些观察锁定为你项目的设计系统基线。"*

---

## Phase 3: 逐页视觉审计

对范围内的每个页面：

```bash
$B goto <url>
$B snapshot -i -a -o "$REPORT_DIR/screenshots/{page}-annotated.png"
$B responsive "$REPORT_DIR/screenshots/{page}"
$B console --errors
$B perf
```

### 认证检测

在第一次导航后，检查 URL 是否变成了登录路径：
```bash
$B url
```
如果 URL 包含 `/login`、`/signin`、`/auth` 或 `/sso`：站点需要认证。AskUserQuestion："此站点需要认证。要从浏览器导入 cookies 吗？如果需要，先运行 `/setup-browser-cookies`。"

### Trunk Test（在每个页面上运行）

想象毫无背景地落到这个页面。你能立即回答：
1. 这什么网站？（网站 ID 可见且可识别）
2. 我在哪个页面？（页面名称醒目，匹配我点击的东西）
3. 主要部分有哪些？（主导航可见且清晰）
4. 我在这个层级有哪些选项？（局部导航或内容选择明显）
5. 我在整体中的位置？（"你在这里"指示器、面包屑）
6. 我怎么搜索？（搜索框不找就能找到）

评分：PASS（6 个清晰）/ PARTIAL（4-5 个清晰）/ FAIL（3 个或更少清晰）。
Trunk Test 的 FAIL 是高影响发现，无论视觉设计多抛光都是如此。

### 设计审计清单（10 个类别，约 80 项）

在每个页面应用这些。每个发现获得影响评级（高/中/打磨）和类别。

**1. 视觉层次与构图**（8 项）
- 清晰的焦点？每个视图一个主 CTA？
- 眼睛自然从左上到右下流动？
- 视觉噪音——抢注意力的竞争元素？
- 信息密度适合内容类型？
- Z-index 清晰——没有意外重叠？
- 首屏内容在 3 秒内传达目的？
- 眯眼测试：模糊时层次仍可见？
- 空白是有意为之，不是留剩的？

**2. Typography**（15 项）
- 字体数量 <=3（超过则标记）
- 标尺遵循比例（1.25 大三度或 1.333 完全四度）
- 行高：正文 1.5x，标题 1.15-1.25x
- 行宽：每行 45-75 字符（66 理想）
- 标题层次：不跳级（没有 h2 直接 h1→h3）
- 字重对比：>=2 个字重用于层次
- 无黑名单字体（Papyrus、Comic Sans、Lobster、Impact、Jokerman）
- 如果主字体是 Inter/Roboto/Open Sans/Poppins → 标记为可能通用
- 标题上使用 `text-wrap: balance` 或 `text-pretty`（通过 `$B css <标题> text-wrap` 检查）
- 使用弯引号，不是直引号
- 使用省略号字符（`…`）不是三个点（`...`）
- 数字列上使用 `font-variant-numeric: tabular-nums`
- 正文 >= 16px
- 说明/标签 >= 12px
- 小写文本无字间距

**3. 颜色与对比度**（10 项）
- 调色板连贯（<=12 种唯一非灰色）
- WCAG AA：正文 4.5:1，大字（18px+）3:1，UI 组件 3:1
- 语义颜色一致（成功=绿、错误=红、警告=黄/琥珀色）
- 不单独用颜色编码（始终添加标签、图标或样式）
- 暗色模式：surface 使用高度，不是明度反转
- 暗色模式：文字使用近白（~#E0E0E0），不是纯白
- 暗色模式主强调色降低饱和度 10-20%
- html 元素上有 `color-scheme: dark`（如果有暗色模式）
- 无仅红/绿组合（8% 男性有红绿色盲）
- 中调色板始终温暖或凉爽——不混合

**4. 间距与布局**（12 项）
- 所有断点处网格一致
- 间距使用标尺（4px 或 8px 基），不是随意值
- 对齐一致——没有漂出网格的
- 节奏：相关项更近，不同部分更远
- Border-radius 层次（不是所有元素统一大圆角）
- 内半径 = 外半径 - 间隙（嵌套元素）
- 移动端无水平滚动
- 设置了最大内容宽度（正文不铺满）
- 刘海屏设备使用 `env(safe-area-inset-*)`
- URL 反映状态（过滤器、标签、分页在 query 参数中）
- 使用 Flex/grid 布局（不是 JS 测量）
- 断点：mobile (375)、tablet (768)、desktop (1024)、wide (1440)

**5. 交互状态**（10 项）
- 所有交互元素有 hover 状态
- 存在 `focus-visible` 环（绝不要无替代的 `outline: none`）
- 按下状态有深度效果或颜色变化
- 禁用状态：降低透明度 + `cursor: not-allowed`
- 加载：骨架形状匹配真实内容布局
- 空状态：温暖消息 + 主操作 + 视觉（不仅是"暂无项目。"）
- 错误消息：具体 + 包含修复/下一步
- 成功：确认动画或颜色，自动消失
- 所有交互元素触摸目标 >= 44px
- 所有可点击元素有 `cursor: pointer`
- 无脑选择审计：每个决策点（按钮、链接、下拉、模态选择）都是无脑点击（很明显会发生什么）。如果点击需要思考这是否是正确选择，标记为 HIGH。

**6. 响应式设计**（8 项）
- 移动端布局有*设计*意义（不只是堆叠的桌面列）
- 移动端触摸目标足够（>= 44px）
- 任何视口无水平滚动
- 图片处理响应式（srcset、sizes 或 CSS containment）
- 移动端无需缩放即可阅读（正文 >= 16px）
- 导航适当折叠（汉堡菜单、底部导航等）
- 表单在移动端可用（正确的 input 类型、移动端无 autoFocus）
- 视口 meta 中无 `user-scalable=no` 或 `maximum-scale=1`

**7. 动效与动画**（6 项）
- Easing：进入 ease-out、退出 ease-in、移动 ease-in-out
- 持续时间：50-700ms 范围（除非页面切换否则无更慢）
- 目的：每个动画传达某些东西（状态变化、注意力、空间关系）
- 尊重 `prefers-reduced-motion`（检查：`$B js "matchMedia('(prefers-reduced-motion: reduce)').matches"`）
- 无 `transition: all`——明确列出属性
- 仅 `transform` 和 `opacity` 做动画（不是 width、height、top、left 等布局属性）

**8. 内容与微文案**（8 项）
- 空状态有温暖设计（消息 + 操作 + 插图/图标）
- 错误消息具体：发生了什么 + 为什么 + 下一步做什么
- 按钮标签具体（"Save API Key" 不是 "Continue" 或 "Submit"）
- 生产环境无占位/lorem ipsum 文本可见
- 处理了截断（`text-overflow: ellipsis`、`line-clamp` 或 `break-words`）
- 主动语态（"安装 CLI" 不是 "CLI 将被安装"）
- 加载状态以 `…` 结束（"保存中…" 不是 "Saving..."）
- 破坏性操作有确认模态或撤销窗口
- 废话检测：扫描以"欢迎来到..."开头或告诉用户站点多棒的简介段落。如果你能听到"blah blah blah"，那就是废话。标记删除。
- 说明检测：任何超过一句的可见说明。如果用户需要阅读说明，设计就失败了。标记说明 AND 它们弥补的交互。
- 废话词数统计：统计页面上所有可见词。将每个文本块分类为"有用内容" vs "废话"（欢迎段落、自我祝贺的文本、没人读的说明）。报告："这个页面有 X 个词。Y 个（Z%）是废话。"

**9. AI Slop 检测**（10 种反模式——黑名单）

测试：受尊敬工作室的人类设计师会发布这个吗？

- 紫色/紫罗兰/靛蓝渐变背景或蓝到紫的配色方案
- **3 列特征网格：** 彩色圆圈中的图标 + 粗体标题 + 2 行描述，对称重复 3 次。最可识别的 AI 布局。
- 彩色圆圈中的图标作为区域装饰（SaaS 启动模板外观）
- 全部居中（所有标题、描述、卡片 `text-align: center`）
- 所有元素统一大圆角
- 装饰 blob、浮动圆圈、波浪 SVG 分隔线（如果一个区域感觉空，它需要更好的内容，不是装饰）
- Emoji 作为设计元素（标题中的火箭、emoji 作为项目符号）
- 卡片左侧彩色边框（`border-left: 3px solid <accent>`）
- 通用英雄文案（"欢迎来到 [X]"、"释放...的力量"、"你的一站式解决方案..."）
- 模具化区域节奏（英雄 → 3 个特征 → 评价 → 定价 → CTA，每个区域同高）

**10. 作为设计的性能**（6 项）
- LCP < 2.0s（web 应用）、< 1.5s（信息站点）
- CLS < 0.1（加载期间无明显布局偏移）
- 骨架质量：形状匹配真实内容布局、闪烁动画
- 图片：`loading="lazy"`、设置 width/height 尺寸、WebP/AVIF 格式
- 字体：`font-display: swap`、preconnect 到 CDN 源
- 无明显字体交换闪烁（FOUT）——预加载关键字体

---

## Phase 4: 交互流程审评

走查 2-3 个关键用户流程并评估*感觉*，不仅是功能：

```bash
$B snapshot -i
$B click @e3           # 执行操作
$B snapshot -D          # 差异比较看什么变了
```

评估：
- **响应感觉：** 点击感觉响应迅速？有延迟或缺失加载状态？
- **转换质量：** 转换是有意的还是通用/缺失？
- **反馈清晰度：** 操作明确成功还是失败？反馈即时吗？
- **表单打磨：** 焦点状态可见？验证时机正确？错误靠近源？

**叙述模式：** 用第一人称叙述流程。"我点击'注册'... spinner 出现... 3 秒过去了... 还在转... 我开始紧张了。终于仪表盘加载了，但我在？导航没有高亮任何东西。" 说出具体元素、它的位置、它的视觉重量。如果你不能具体命名，你不是在真正体验流程，你在生成套话。

### Goodwill Reservoir（在流程中追踪）

在走查用户流程时，维护一个心理善意计量（从 70/100 开始）。
这些分数是启发式的，不是测量的。价值在于识别具体的
消耗和填充，不是最终数字。

扣分：
- 隐藏用户想要的信息（定价、联系、运输）：扣 15
- 格式惩罚（拒绝有效输入如电话号码中的横线）：扣 10
- 不必要的信息请求：扣 10
- 阻挡任务的插页、闪屏、强制导览：扣 15
- 邋遢或不专业的外观：扣 10
- 需要思考的模糊选择：每个扣 5

加分：
- 顶部用户任务明显突出：加 10
- upfront 表明成本和限制：加 5
- 节省步骤（直接链接、智能默认、自动填充）：每个加 5
- 优雅的错误恢复，有具体修复指示：加 10
- 出错时道歉：加 5

用视觉仪表盘报告最终善意分数：

```
Goodwill: 70 ████████████████████░░░░░░░░░░
  Step 1: 登录页面        70 → 75  (+5 明显的主操作)
  Step 2: 仪表盘          75 → 60  (-15 插页导览弹窗)
  Step 3: 设置            60 → 50  (-10 电话格式惩罚)
  Step 4: 计费            50 → 35  (-15 隐藏的定价信息)
  FINAL: 35/100 ⚠️ 关键 UX 债务
```

低于 30 = 关键 UX 债务。30-60 = 需要工作。高于 60 = 健康。
将最大的消耗和填充作为具体发现包含。

---

## Phase 5: 跨页一致性

比较页面之间的截图和观察：
- 导航栏在所有页面一致？
- 页脚一致？
- 组件复用 vs 一次性设计（同一个按钮在不同页面上样式不同？）
- 语气一致性（一个页面活泼而另一个企业化？）
- 间距节奏跨页连贯？

---

## Phase 6: 编译报告

### 输出位置

**本地：** `.gstack/design-reports/design-audit-{domain}-{YYYY-MM-DD}.md`

**项目范围：**
```bash
eval "$(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)" && mkdir -p ~/.gstack/projects/$SLUG
```
写入：`~/.gstack/projects/{slug}/{user}-{branch}-design-audit-{datetime}.md`

**基线：** 写入 `design-baseline.json` 用于回归模式：
```json
{
  "date": "YYYY-MM-DD",
  "url": "<target>",
  "designScore": "B",
  "aiSlopScore": "C",
  "categoryGrades": { "hierarchy": "A", "typography": "B", ... },
  "findings": [{ "id": "FINDING-001", "title": "...", "impact": "high", "category": "typography" }]
}
```

### 评分系统

**双重 headline 评分：**
- **Design Score: {A-F}**——所有 10 个类别的加权平均
- **AI Slop Score: {A-F}**——独立评级，带精炼判断

**每类别评级：**
- **A：** 有意为、抛光、令人愉悦。展示设计思考。
- **B：** 坚实的基本功，小的不一致。看起专业。
- **C：** 功能但通用。没有大问题，没有设计观点。
- **D：** 明显的问题。感觉未完成或粗心。
- **F：** 真正损害用户体验。需要重大返工。

**评级计算：** 每个类别从 A 开始。每个高影响发现降一个字母等级。每个中等影响发现降半个字母等级。打磨发现被记录但不影响评级。最低 F。

**Design Score 的类别权重：**
| 类别 | 权重 |
|----------|--------|
| 视觉层次 | 15% |
| Typography | 15% |
| 间距与布局 | 15% |
| 颜色与对比度 | 10% |
| 交互状态 | 10% |
| 响应式 | 10% |
| 内容质量 | 10% |
| AI Slop | 5% |
| 动效 | 5% |
| 性能感觉 | 5% |

AI Slop 在 Design Score 中占 5%，但也作为 headline 指标独立评级。

### 回归输出

当存在之前的 `design-baseline.json` 或使用 `--regression` 标志时：
- 加载基线评级
- 比较：每类别差异、新发现、已解决的发现
- 将回归表追加到报告

---

## 设计批评格式

使用结构化反馈，不是意见：
- "我注意到..."——观察（例如"我注意到主 CTA 和次要动作在竞争"）
- "我在想..."——问题（例如"我在想用户是否会理解这里的'处理'是什么意思"）
- "如果..."——建议（例如"如果把搜索移到更显眼的位置会怎样？"）
- "我认为...因为..."——有理据的意见（例如"我认为区域之间的间距太均匀了，因为没有创造层次"）

将所有内容与用户目标和产品目标联系起来。始终在提出问题的同时建议具体改进。

---

## 重要规则

1. **像设计师思考，不是 QA 工程师。** 你关心东西是否感觉对、看起有意为、尊重用户。你不**只**关心东西是否"能用。"
2. **截图是证据。** 每个发现至少需要一张截图。使用带注释截图（`snapshot -a`）高亮元素。
3. **要具体且可操作。** "把 X 改成 Y 因为 Z"——不是"间距感觉不太对。"
4. **永远不要阅读源代码。** 评估渲染后的站点，不是实现。（例外：可以从提取的观察中提供编写 DESIGN.md。）
5. **AI Slop 检测是你的超能力。** 大多数开发者无法评估他们的站点是否看起像 AI 生成的。你能。直接说出来。
6. **快速胜利很重要。** 始终包含"Quick Wins"部分——3-5 个高影响修复，每个少于 30 分钟。
7. **对棘手 UI 使用 `snapshot -C`。** 发现无障碍树遗漏的可点击 div。
8. **响应式是设计，不仅是"没坏。"** 移动端堆叠的桌面布局不是响应式设计——是偷懒。评估移动端布局是否有*设计*意义。
9. **增量文档。** 发现每个发现时就写入报告。不要批量处理。
10. **深度胜过广度。** 5-10 个有截图和具体建议的充分记录的发现 > 20 个模糊的观察。
11. **向用户展示截图。** 在每个 `$B screenshot`、`$B snapshot -a -o` 或 `$B responsive` 命令之后，对输出文件使用 Read tool 以便用户内联查看。对于 `responsive`（3 个文件），读取全部三个。这很关键——否则截图对用户是不可见的。

### 设计硬性规则

**分类器——在评估前确定规则集：**
- **MARKETING/LANDING PAGE**（英雄驱、品牌导向、转化导向）→ 应用 Landing Page 规则
- **APP UI**（工作区驱、数据密集、任务导向：仪表盘、管理、设置）→ 应用 App UI 规则
- **HYBRID**（营销壳带类应用区域）→ 对英雄/营销区域应用 Landing Page 规则，对功能区域应用 App UI 规则

**硬性拒绝标准**（即时失败模式——如果任何适用则标记）：
1. 通用 SaaS 卡片网格作为第一印象
2. 漂亮图片配弱品牌
3. 强标题无明确操作
4. 文字后面是繁忙的图像
5. 区域重复相同情绪陈述
6. 没有叙事目的的轮播
7. 由堆叠卡片而非布局组成的 App UI

**Litmus 检查**（每个回答 YES/NO——用于跨模型共识评分）：
1. 品牌/产品在第一屏 unmistakable？
2. 存在一个强视觉锚点？
3. 仅扫描标题就能理解页面？
4. 每个区域有一个工作？
5. 卡片真的有必要吗？
6. 动效改善了层次或氛围？
7. 去掉所有装饰阴影后设计是否仍有品质感？

**Landing page 规则**（当分类器 = MARKETING/LANDING 时应用）：
- 第一视口读作一个构图，不是仪表盘
- 品牌第一层次：品牌 > 标题 > 正文 > CTA
- Typography：有表现力、有目的——不是默认栈（Inter、Roboto、Arial、system）
- 无平面单色背景——使用渐变、图像、微妙样式
- 英雄：全出血、到边缘、无内嵌/平铺/圆角变体
- 英雄预算：品牌、一个标题、一补充句、一个 CTA 组、一张图
- 英雄中无卡片。仅当卡片**是**交互时才用卡片
- 每个区域一个工作：一个目的、一个标题、一句短补充句
- 动效：至少 2-3 个有意动效（入场、滚动关联、hover/揭示）
- 颜色：定义 CSS 变量、避免白色上的紫色默认、一个强调色默认
- 文案：产品语言而非设计评论。"如果删掉 30% 改善了就继续删"
- 美丽的默认值：构图第一、品牌是最响的文字、最多两种字体、默认无卡片、第一视口是海报不是文档

**App UI 规则**（当分类器 = APP UI 时应用）：
- 平静表面层次、强 typography、少颜色
- 密但可读、最小 chrome
- 组织：主工作区、导航、次要上下文、一个强调色
- 避免：仪表盘卡片马赛克、粗边框、装饰渐变、装饰图标
- 文案：实用语言——定向、状态、操作。不是情绪/品牌/抱负
- 仅当卡片**是**交互时才用卡片
- 区域标题说明区域是什么或用户能做什么（"选定的 KPI"、"计划状态"）

**通用规则**（适用于所有类型）：
- 为颜色系统定义 CSS 变量
- 无默认字体栈（Inter、Roboto、Arial、system）
- 每个区域一个工作
- "如果删掉 30% 的文案改善了就继续删"
- 卡片赚得存在——无装饰卡片网格
- 永远不要使用小号、低对比度文字（正文 < 16px 或正文对比度 < 4.5:1）
- 永远不要把标签仅放在表单字段内作为唯一标签（占位即标签模式——字段有内容时标签必须可见）
- 始终保留已访问与未访问链接的区别（已访问链接必须有不同颜色）
- 永远不要让标题漂浮在段落之间（标题必须在视觉上更靠近它引入的区域，而不是前一个区域）

**AI Slop 黑名单**（10 种尖叫"AI 生成"的模式）：
1. 紫色/紫罗兰/靛蓝渐变背景或蓝到紫的配色方案
2. **3 列特征网格：** 彩色圆圈中的图标 + 粗体标题 + 2 行描述，对称重复 3 次。最可识别的 AI 布局。
3. 彩色圆圈中的图标作为区域装饰（SaaS 启动模板外观）
4. 全部居中（所有标题、描述、卡片 `text-align: center`）
5. 所有元素统一大圆角
6. 装饰 blob、浮动圆圈、波浪 SVG 分隔线（如果一个区域感觉空，它需要更好的内容，不是装饰）
7. Emoji 作为设计元素（标题中的火箭、emoji 作为项目符号）
8. 卡片左侧彩色边框（`border-left: 3px solid <accent>`）
9. 通用英雄文案（"欢迎来到 [X]"、"释放...的力量"、"你的一站式解决方案..."）
10. 模具化区域节奏（英雄 → 3 个特征 → 评价 → 定价 → CTA，每个区域同高）

来源：[OpenAI "Designing Delightful Frontends with GPT-5.4"](https://developers.openai.com/blog/designing-delightful-frontends-with-gpt-5-4)（2026 年 3 月）+ gstack 设计方法论。

在 Phase 6 结束时记录基线设计评分和 AI slop 评分。

---

## 输出结构

```
~/.gstack/projects/$SLUG/designs/design-audit-{YYYYMMDD}/
├── design-audit-{domain}.md                  # Structured report
├── screenshots/
│   ├── first-impression.png                  # Phase 1
│   ├── {page}-annotated.png                  # Per-page annotated
│   ├── {page}-mobile.png                     # Responsive
│   ├── {page}-tablet.png
│   ├── {page}-desktop.png
│   ├── finding-001-before.png                # Before fix
│   ├── finding-001-target.png                # Target mockup (if generated)
│   ├── finding-001-after.png                 # After fix
│   └── ...
└── design-baseline.json                      # For regression mode
```

---

## Design Outside Voices（并行）

**自动：** 当 Codex 可用时，外部声音自动运行。无需选择加入。

**检查 Codex 可用性：**
```bash
which codex 2>/dev/null && echo "CODEX_AVAILABLE" || echo "CODEX_NOT_AVAILABLE"
```

**如果 Codex 可用**，同时启动两个声音：

1. **Codex 设计声音**（通过 Bash）：
```bash
TMPERR_DESIGN=$(mktemp /tmp/codex-design-XXXXXXXX)
_REPO_ROOT=$(git rev-parse --show-toplevel) || { echo "ERROR: not in a git repo" >&2; exit 1; }
codex exec "Review the frontend source code in this repo. Evaluate against these design hard rules:
- Spacing: systematic (design tokens / CSS variables) or magic numbers?
- Typography: expressive purposeful fonts or default stacks?
- Color: CSS variables with defined system, or hardcoded hex scattered?
- Responsive: breakpoints defined? calc(100svh - header) for heroes? Mobile tested?
- A11y: ARIA landmarks, alt text, contrast ratios, 44px touch targets?
- Motion: 2-3 intentional animations, or zero / ornamental only?
- Cards: used only when card IS the interaction? No decorative card grids?

First classify as MARKETING/LANDING PAGE vs APP UI vs HYBRID, then apply matching rules.

LITMUS CHECKS — answer YES/NO:
1. Brand/product unmistakable in first screen?
2. One strong visual anchor present?
3. Page understandable by scanning headlines only?
4. Each section has one job?
5. Are cards actually necessary?
6. Does motion improve hierarchy or atmosphere?
7. Would design feel premium with all decorative shadows removed?

HARD REJECTION — flag if ANY apply:
1. Generic SaaS card grid as first impression
2. Beautiful image with weak brand
3. Strong headline with no clear action
4. Busy imagery behind text
5. Sections repeating same mood statement
6. Carousel with no narrative purpose
7. App UI made of stacked cards instead of layout

Be specific. Reference file:line for every finding." -C "$_REPO_ROOT" -s read-only -c 'model_reasoning_effort="high"' --enable web_search_cached 2>"$TMPERR_DESIGN"
```
使用 5 分钟超时（`timeout: 300000`）。命令完成后，读取 stderr：
```bash
cat "$TMPERR_DESIGN" && rm -f "$TMPERR_DESIGN"
```

2. **Claude 设计 subagent**（通过 Agent tool）：
分派一个 subagent，使用以下 prompt：
"Review the frontend source code in this repo. You are an independent senior product designer doing a source-code design audit. Focus on CONSISTENCY PATTERNS across files rather than individual violations:
- Are spacing values systematic across the codebase?
- Is there ONE color system or scattered approaches?
- Do responsive breakpoints follow a consistent set?
- Is the accessibility approach consistent or spotty?

For each finding: what's wrong, severity (critical/high/medium), and the file:line."

**错误处理（全部非阻断）：**
- **Auth 失败：** 如果 stderr 包含"auth"、"login"、"unauthorized"或"API key"："Codex 认证失败。运行 `codex login` 认证。"
- **超时：** "Codex 在 5 分钟后超时。"
- **空响应：** "Codex 没有返回响应。"
- 任何 Codex 错误时：仅使用 Claude subagent 输出继续，标记 `[single-model]`。
- 如果 Claude subagent 也失败："外部声音不可用——继续主评审。"

在 `CODEX SAYS (design source audit):` 标题下呈现 Codex 输出。
在 `CLAUDE SUBAGENT (design consistency):` 标题下呈现 subagent 输出。

**综合——Litmus 记分卡：**

使用与 /plan-design-review 相同的记分卡格式（如上所示）。从两个输出填充。
将发现合并到分类中，带 `[codex]` / `[subagent]` / `[cross-model]` 标签。

**记录结果：**
```bash
~/.claude/skills/gstack/bin/gstack-review-log '{"skill":"design-outside-voices","timestamp":"'"$(date -u +%Y-%m-%dT%H:%M:%SZ)"'","status":"STATUS","source":"SOURCE","commit":"'"$(git rev-parse --short HEAD)"'"}'
```
将 STATUS 替换为 "clean" 或 "issues_found"，SOURCE 替换为 "codex+subagent"、"codex-only"、"subagent-only" 或 "unavailable"。

## Phase 7: 分类

按影响对所有发现的发现排序，然后决定修复哪些：

- **高影响：** 先修复。这些影响第一印象并损害用户信任。
- **中等影响：** 接下来修复。这些降低抛光度并被潜意识感知。
- **打磨：** 有时间则修复。这些分开好和伟大。

标记无法从源代码修复的发现（例如第三方组件问题、需要团队提供文案的内容问题）为"延期"，无论影响如何。

---

## Phase 8: 修复循环

对每个可修复的发现，按影响顺序：

### 8a. 定位来源

```bash
# 搜索 CSS 类、组件名、样式文件
# Glob 匹配受影响页面的文件模式
```

- 找到负责设计问题的源文件
- **仅**修改与发现直接相关的文件
- 优先 CSS/样式更改而非结构组件更改

### 8a.5. 目标 Mockup（如果 DESIGN_READY）

如果 gstack designer 可用且发现涉及视觉布局、层次或间距（不是仅 CSS 值修复如错误颜色或字号），生成一个目标 mockup 展示修正版本应该是什么样子：

```bash
$D generate --brief "<描述修复了发现的页面/组件，引用 DESIGN.md 约束>" --output "$REPORT_DIR/screenshots/finding-NNN-target.png"
```

向用户展示："这是当前状态（截图），这是它应该的样子（mockup）。现在我将修复源代码以匹配。"

此步骤是可选的——对琐碎 CSS 修复（错误 hex 颜色、缺失 padding 值）跳过。用于仅从描述无法看出预期设计的发现。

### 8b. 修复

- 阅读源代码，理解上下文
- 做**最小修复**——解决设计问题的最小更改
- 如果在 8a.5 中生成了目标 mockup，用它作为修复的视觉参考
- 优先仅 CSS 更改（更安全、更可逆）
- 不要重构周围的代码、添加功能或"改进"无关的东西

### 8c. 提交

```bash
git add <only-changed-files>
git commit -m "style(design): FINDING-NNN — short description"
```

- 每个修复一个提交。永远不要捆绑多个修复。
- 消息格式：`style(design): FINDING-NNN — 简短描述`

### 8d. 重新测试

导航回受影响的页面并验证修复：

```bash
$B goto <affected-url>
$B screenshot "$REPORT_DIR/screenshots/finding-NNN-after.png"
$B console --errors
$B snapshot -D
```

对每个修复拍摄**前后截图对**。

### 8e. 分类

- **verified**：重新测试确认修复有效，无新错误引入
- **best-effort**：应用了修复但无法完全验证（例如需要特定浏览器状态）
- **reverted**：检测到回归 → `git revert HEAD` → 将发现标记为"deferred"

### 8e.5. 回归测试（design-review 变体）

设计修复通常仅是 CSS。仅为涉及 JavaScript 行为更改的修复生成回归测试——坏掉的下拉菜单、动画失败、条件渲染、交互状态问题。

对仅 CSS 修复：完全跳过。CSS 回归通过重新运行 /design-review 捕获。

如果修复涉及 JS 行为：遵循与 /qa Phase 8e.5 相同的程序（研究现有测试模式、编写编码确切 bug 条件的回归测试、运行它、通过则提交或失败则延期）。提交格式：`test(design): regression test for FINDING-NNN`。

### 8f. 自我监管（停下来评估）

每 5 个修复后（或任何回退后），计算设计修复风险等级：

```
DESIGN-FIX RISK:
  从 0% 开始
  每次回退:                        +15%
  每次仅 CSS 文件更改:          +0%   （安全——仅此样式）
  每次 JSX/TSX/组件文件更改: +5%   每个文件
  修复 10 之后:                  +1%   每个额外修复
  触碰不相关文件:           +20%
```

**如果风险 > 20%：** 立即停止。向用户展示你做了什么。询问是否继续。

**硬性上限：30 个修复。** 30 个修复后停止，无论是否有剩余发现。

---

## Phase 9: 最终设计审计

应用所有修复后：

1. 在所有受影响页面上重新运行设计审计
2. 如果在修复循环期间生成了目标 mockup 且 `DESIGN_READY`：运行 `$D verify --mockup "$REPORT_DIR/screenshots/finding-NNN-target.png" --screenshot "$REPORT_DIR/screenshots/finding-NNN-after.png"` 将修复结果与目标比较。在报告中包含通过/失败。
3. 计算最终设计评分和 AI slop 评分
4. **如果最终评分比基线更差：** 醒目警告——某些东西回退了

---

## Phase 10: 报告

将报告写入 `$REPORT_DIR`（已在 setup phase 设置）：

**主要：** `$REPORT_DIR/design-audit-{domain}.md`

**同时写入摘要到项目索引：**
```bash
eval "$(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)" && mkdir -p ~/.gstack/projects/$SLUG
```
将一行摘要写入 `~/.gstack/projects/{slug}/{user}-{branch}-design-audit-{datetime}.md`，带指向 `$REPORT_DIR` 中完整报告的指针。

**每发现附加项**（超出标准设计审计报告）：
- 修复状态：verified / best-effort / reverted / deferred
- 提交 SHA（如果修复了）
- 更改的文件（如果修复了）
- 前后截图（如果修复了）

**摘要部分：**
- 总发现数
- 应用的修复（verified: X, best-effort: Y, reverted: Z）
- 延期发现
- 设计评分变化：基线 → 最终
- AI slop 评分变化：基线 → 最终

**PR 摘要：** 包含适合 PR 描述的一行摘要：
> "Design review found N issues, fixed M. Design score X → Y, AI slop score X → Y."

---

## Phase 11: TODOS.md 更新

如果 repo 有 `TODOS.md`：

1. **新的延期设计发现** → 作为 TODO 添加，带影响级别、类别和描述
2. **已修复且之前在 TODOS.md 中的发现** → 标注 "Fixed by /design-review on {branch}, {date}"

---

## Capture Learnings

如果你在此 session 发现了非明显模式、陷阱或架构洞察，为未来 session 记录它：

```bash
~/.claude/skills/gstack/bin/gstack-learnings-log '{"skill":"design-review","type":"TYPE","key":"SHORT_KEY","insight":"DESCRIPTION","confidence":N,"source":"SOURCE","files":["path/to/relevant/file"]}'
```

**类型：** `pattern`（可复用的方法）、`pitfall`（不该做什么）、`preference`（用户声明）、`architecture`（结构决策）、`tool`（库/框架洞察）、`operational`（项目环境/CLI/工作流知识）。

**来源：** `observed`（你在代码中找到的）、`user-stated`（用户告诉你的）、`inferred`（AI 推理）、`cross-model`（Claude 和 Codex 都同意）。

**置信度：** 1-10。诚实点。你在代码中验证过的观察模式是 8-9。你不太确定的推理是 4-5。用户明确声明的偏好是 10。

**files:** 包含此 learning 引用的具体文件路径。这支持陈旧性检测：如果这些文件之后被删除，learning 可以被标记。

**仅记录真正的发现。** 不要记录显而易见的东西。不要记录用户已经知道的东西。一个好的测试：这个洞察能否在未来 session 中节省时间？如果是，记录它。

## Additional Rules (design-review 特定)

11. **干净工作树是必需的。** 如果不干净，在继续之前使用 AskUserQuestion 提供提交/暂存/中止。
12. **每个修复一个提交。** 永远不要将多个设计修复捆绑到一个提交中。
13. **仅在 Phase 8e.5 中生成回归测试时修改测试。** 永远不要修改 CI 配置。永远不要修改现有测试——只创建新测试文件。
14. **回退回归。** 如果一个修复让事情更糟，立即 `git revert HEAD`。
15. **自我监管。** 遵循设计修复风险启发式。如有疑问，停下来问。
16. **CSS 优先。** 优先 CSS/样式更改而非结构组件更改。仅 CSS 更改更安全、更可逆。
17. **DESIGN.md 导出。** 如果用户接受 Phase 2 的提议，你**可以**编写 DESIGN.md 文件。
# /design-review: Design Audit → Fix → Verify

## Phase 2 详细：设计系统提取

读取所有设计相关文件，提取设计 token：

### 设计 token 提取

**字体：** 从 CSS 变量、`font-family` 属性或 Google Fonts 链接提取。
**颜色：** 从 `--color` CSS 变量或硬编码色值提取。
**间距：** 从 `padding`、`margin`、`gap` 值提取。
**断点：** 从 `@media` 查询提取。
**圆角：** 从 `border-radius` 值提取。
**阴影：** 从 `box-shadow` 值提取。

输出格式：提取 token 和它们的 CSS 自定义属性映射。

## Phase 3 详细：逐页视觉审计

对每个页面应用检查清单：

### 排版审查
- 标题层级：`<h1>` 到 `<h6>` 是否正确使用？
- 字体大小：标题、正文、标签是否一致？
- 行高：可读的 line-height？
- 字重：粗体用于强调，不是随机使用？

### 颜色审查
- 文字/背景对比度至少 4.5:1（AA 级）？
- 调色板一致？没有随机颜色？
- 暗色模式正确处理？

### 布局审查
- 对齐：文字和组件正确对齐？
- 间距：组件之间的间距遵循间距比例？
- 响应式：页面在所有断点看起来正常？

### 组件审查
- 按钮：一致的内边距、圆角、颜色？
- 输入框：一致的边框、高度、焦点状态？
- 卡片：一致的背景、边框、阴影、圆角？

### 图像和媒体
- 图片是真实内容还是占位符？
- 视频/iframe 正确处理？
- 图标一致性（SVG vs 字体图标 vs emoji）？

## Phase 4 详细：交互流程审评

测试用户流：

1. **导航流：** 用户可以轻松找到他们需要的吗？
2. **表单流：** 填写表单是否顺畅？验证清晰？错误消息有帮助？
3. **空状态：** 零数据时页面看起来正常？
4. **加载状态：** 加载指示器显示？
5. **错误状态：** 错误处理优雅？恢复路径清晰？

**使用浏览工具测试：**
```bash
$B goto <page-url>
$B snapshot -i -a
$B click <interactive-element>
$B snapshot -D
```

## Phase 5 详细：跨页一致性

比较所有审计页面的一致性：

- **排版跨页：** 标题/正文字体大小和行高一致？
- **颜色跨页：** 相同元素类型使用相同颜色？
- **组件跨页：** 按钮/卡片/输入框跨页一致？
- **间距跨页：** 页面之间间距一致？
- **布局模式跨页：** 页面结构遵循一致模式？

**每个不一致：**
`[不一致] {描述}：页面 A 有 {X}，页面 B 有 {Y}`

## Phase 6 详细：编译报告

产出带评分的结构化审计摘要。

**审计评分（每个类别 0-10）：**
排版、颜色、间距、布局、组件、内容、一致性、响应式。

**报告格式：**
```
设计审计报告
══════════════════════════════
整体：{N}/10

问题
[排版] {描述}
[颜色] {描述}

响应式状态
375px：✓ 正常
768px：⚠ {问题}
1024px：✓ 正常
1440px：✓ 正常
```

## Phase 7：分类

将每个发现的问题分类为：

- **HIGH [→FIX]：** 机械可修复的（缺失 outline、使用!important、小于 16px 的字体、对比度失败）。自动修复。
- **MEDIUM [→ASK]：** 需要设计判断的（间距调整、布局重组、组件变更）。AskUserQuestion。
- **LOW [→NOTE]：** 意图检测（可能但不确定的问题）。注记"可能...通过视觉验证或运行完整审查确认"。

**AI 垃圾检测：**
1. 默认渐变（紫色/蓝色）而不是设计规范
2. 通用 SaaS 组件（3 列功能网格+图标）
3. Emoji 作为视觉元素
4. 库存占位符图片
5. 装饰性 blob/波浪不在原始设计中

每个检测到的：`[AI 垃圾] {element}：{问题}。修复：{建议}`

## Phase 8：修复循环

### Step 8a：自动修复 HIGH

对所有 HIGH 分类机械可修复项，直接应用修复。

### Step 8b：修复 ASK 项

AskUserQuestion：
```
1. [排版] app-header h1：期望 'Satoshi'，实际 'Inter'。→ A) 修复 B) 跳过
2. [布局] 导航在 768px 溢出。建议汉堡菜单。→ A) 修复 B) 跳过
```

### Step 8c：验证修复

在修复后重新测试。确认修复工作没有引入新回归。

## Phase 9：最终设计审计

修复后运行审计，确认修复。对比修复前评分，展示改进。

## Phase 10：报告

输出最终审计报告。包括：
- 最终评分
- 已修复的问题列表
- 跳过的剩余项
- 建议的后续步骤

## Phase 11：TODOS.md 更新

检查审计中发现的问题是否应该成为 TODOS.md 条目。对于跳过的项或需要人工输入的项，创建 TODO。

## 设计外部声音（并行）

当从 plan-ceo-review 或 plan-eng-review 调用时可用。运行 Codex 或其他独立模型获取独立设计意见。

## Capture Learnings

（标准学习记录格式...类型、来源、置信度、文件引用。）

## 附加规则（design-review 专用）

1. DESIGN.md 是事实来源。所有审计针对它。
2. 截图是证据。始终内联显示截图。
3. 机械修复无争议。颜色、间距、字体...可证明不匹配。
4. AI 垃圾始终自动修复。
5. 不自动更改：布局结构变更、内容变更、缺失状态、组件架构变更。
