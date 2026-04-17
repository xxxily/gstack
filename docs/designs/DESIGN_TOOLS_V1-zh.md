# 设计：gstack 视觉设计生成（`design` 二进制文件）

由 /office-hours 于 2026-03-26 生成
分支：garrytan/agent-design-tools
仓库：gstack
状态：DRAFT
模式：内部创业

## 背景

gstack 的设计 Skill（/office-hours、/design-consultation、/plan-design-review、/design-review）都产出设计的**文本描述** — DESIGN.md 文件带色值、规划文档带像素规格的散文描述、ASCII 艺术线框。创建者是一位在 OmniGraffle 中手动设计 HelloSign 的设计师，对此感到尴尬。

价值单位是错误的。用户不需要更丰富的设计语言 — 他们需要一个可执行的视觉产物，改变对话从"你喜欢这个需求吗？"变成"这就是那个页面吗？"

## 问题陈述

设计 Skill 用文本来描述设计而不是展示它。Argus UX  overhaul 规划就是例子：487 行详细的情感弧线规格、字体选择、动画时序 — 零视觉产出。一个"设计"的 AI 编码 Agent 应该产出你能看到并能直观反应的东西。

## 需求证据

创建者/主要用户发现当前的输出令人尴尬。每次设计 Skill 会话都以散文结束，而应该是一个 mockup。GPT Image API 现在可以生成像素级精确的 UI mockup，文本渲染准确 — 过去为纯文本输出辩护的能力差距已不复存在。

## 最小可行切入点

一个编译后的 TypeScript 二进制文件（`design/dist/design`），封装 OpenAI Images/Responses API，可从 Skill 模板中通过 `$D` 调用（模仿现有 `$B` 浏览二进制模式）。优先集成顺序：/office-hours → /plan-design-review → /design-consultation → /design-review。

## 已达成共识的前提

1. GPT Image API（通过 OpenAI Responses API）是正确的引擎。Google Stitch SDK 作为备选。
2. **设计 Skill 默认开启视觉 mockup**，可轻松跳过 — 而非需要选择开启。（根据 Codex 挑战修改。）
3. 集成是一个共享工具（非每个 Skill 重新实现）— 任何 Skill 都可以调用的 `design` 二进制。
4. 优先级：/office-hours 第一，然后是 /plan-design-review、/design-consultation、/design-review。

## 跨模型视角（Codex）

Codex 独立验证了核心论点："失败不是在 markdown 内输出质量问题，而是当前价值单位错了。"关键贡献：
- 挑战了前提 #2（选择开启 → 默认开启）— 已接受
- 提出了基于视觉的质量门控：使用 GPT-4o vision 验证生成的 mockup 是否有不可读文本、缺失部分、布局损坏，自动重试一次
- 划定了 48 小时原型范围：共享的 `visual_mockup.ts` 工具，仅限 /office-hours + /plan-design-review，hero mockup + 2 个变体

## 推荐方案：`design` 二进制（Approach B）

### 架构

**共享浏览二进制的编译和分发模式**（bun build --compile、安装脚本、Skill 模板中的 $VARIABLE 解析），但架构上更简单 — 没有持久守护服务器、没有 Chromium、没有健康检查、没有 token 认证。Design 二进制是一个无状态 CLI，进行 OpenAI API 调用并将 PNG 写入磁盘。会话状态（用于多轮迭代）是一个 JSON 文件。

**新依赖：** `openai` npm 包（添加到 `devDependencies`，而非运行时依赖）。Design 二进制与浏览分开编译，这样 openai 不会增大浏览二进制。

```
design/
├── src/
│   ├── cli.ts            # 入口，命令分发
│   ├── commands.ts       # 命令注册表（文档和验证的真实来源）
│   ├── generate.ts       # 从结构化 brief 生成 mockup
│   ├── iterate.ts        # 围绕现有 mockup 进行多轮迭代
│   ├── variants.ts       # 从 brief 生成 N 个设计变体
│   ├── check.ts          # 基于视觉的质量门控（GPT-4o）
│   ├── brief.ts          # 结构化 brief 类型 + 组装辅助
│   └── session.ts        # 会话状态（response ID 用于多轮）
├── dist/
│   ├── design            # 编译后的二进制
│   └── .version          # Git hash
└── test/
    └── design.test.ts     # 集成测试
```

### 命令

```bash
# 从结构化 brief 生成 hero mockup
$D generate --brief "编码评估工具的仪表盘。深色主题、奶油色点缀。显示：构建者名称、分数徽章、叙事信、分数卡片。目标：技术用户。" --output /tmp/mockup-hero.png

# 生成 3 个设计变体
$D variants --brief "..." --count 3 --output-dir /tmp/mockups/

# 用反馈迭代现有 mockup
$D iterate --session /tmp/design-session.json --feedback "让分数卡片更大，把叙事移到分数上方" --output /tmp/mockup-v2.png

# 基于视觉的质量检查（返回 PASS/FAIL + 问题）
$D check --image /tmp/mockup-hero.png --brief "带构建者名称、分数徽章、叙事的仪表盘"

# 一次性带质量门控 + 自动重试
$D generate --brief "..." --output /tmp/mockup.png --check --retry 1

# 通过 JSON 文件传递结构化 brief
$D generate --brief-file /tmp/brief.json --output /tmp/mockup.png

# 生成对比面板 HTML 供用户审阅
$D compare --images /tmp/mockups/variant-*.png --output /tmp/design-board.html

# 引导式 API key 设置 + 冒烟测试
$D setup
```

**Brief 输入模式：**
- `--brief "普通文本"` — 自由文本 prompt（简单模式）
- `--brief-file path.json` — 结构化 JSON，匹配 `DesignBrief` 接口（丰富模式）
- Skill 构建 JSON brief 文件，写入 /tmp，传递 `--brief-file`

**所有命令都在 `commands.ts` 中注册**，包括 `--check` 和 `--retry` 作为 `generate` 的标志。

### 设计探索工作流（来自 eng review）

工作流是顺序执行的，而非并行。PNG 用于视觉探索（面向人类），HTML 线框用于实施（面向 Agent）：

```
1. $D variants --brief "..." --count 3 --output-dir /tmp/mockups/
   → 生成 2-5 个 PNG mockup 变体

2. $D compare --images /tmp/mockups/*.png --output /tmp/design-board.html
   → 生成 HTML 对比面板（规格如下）

3. $B goto file:///tmp/design-board.html
   → 用户在 headed Chrome 中审阅所有变体

4. 用户选择喜欢的、评分、评论、点击 [Submit]
   Agent 轮询: $B eval document.getElementById('status').textContent
   Agent 读取: $B eval document.getElementById('feedback-result').textContent
   → 无需剪贴板、无需粘贴。Agent 直接从页面读取反馈。

5. Claude 通过 DESIGN_SKETCH 生成 HTML 线框匹配已通过的方向
   → Agent 从可检查的 HTML 实施，而非不透明的 PNG
```

### 对比面板设计规格（来自 /plan-design-review）

**分类：APP UI**（任务导向，实用页面）。无产品品牌。

**布局：单列，全宽 mockup。** 每个变体获得完整的视口宽度以最大化图像保真度。用户垂直滚动浏览变体。

```
┌─────────────────────────────────────────────────────────────┐
│  标题栏                                                        │
│  "Design Exploration" . 项目名称 . "3 个变体"                 │
│  模式指示器：[广泛探索] | [匹配 DESIGN.md]                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌───────────────────────────────────────────────────────┐  │
│  │              变体 A（全宽）                              │  │
│  │         [ mockup PNG，max-width: 1200px ]               │  │
│  ├───────────────────────────────────────────────────────┤  │
│  │ (●) 选择   ★★★★☆   [你喜欢/不喜欢的内容？____]        │  │
│  │            [更多类似的]                                  │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                             │
│  ┌───────────────────────────────────────────────────────┐  │
│  │              变体 B（全宽）                              │  │
│  │         [ mockup PNG，max-width: 1200px ]               │  │
│  ├───────────────────────────────────────────────────────┤  │
│  │ ( ) 选择   ★★★☆☆   [你喜欢/不喜欢的内容？____]        │  │
│  │            [更多类似的]                                  │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                             │
│  ...（滚动查看更多变体）                                       │
│                                                             │
│  ─── 分隔线 ────────────────────────────────────────        │
│  整体方向（可选，默认折叠）                                     │
│  [textarea，3 行，聚焦时扩展]                                  │
│                                                             │
│  ─── 重新生成栏（#f7f7f7 背景）──────────────────────        │
│  "想探索更多吗？"                                              │
│  [完全不同的]  [匹配我的设计]  [自定义：______]                │
│                                      [重新生成 ->]            │
│  ─────────────────────────────────────────────────────────  │
│                                        [ ✓ 提交 ]            │
└─────────────────────────────────────────────────────────────┘
```

**视觉规格：**
- 背景：#fff。无阴影、无卡片边框。变体分隔线：1px #e5e5e5。
- 字体：系统字体栈。标题：16px semibold。标签：14px semibold。反馈占位符：13px regular #999。
- 星级评分：5 个可点击的星星，已填充=#000，未填充=#ddd。不彩色、无动画。
- 单选按钮"选择"：显式偏好选择。每个变体一个，互斥。
- "更多类似的"按钮：每个变体单独按钮，触发重新生成并以该变体风格为种子。
- 提交按钮：#000 背景、白色文字、右对齐。单一 CTA。
- 重新生成栏：#f7f7f7 背景，与反馈区域在视觉上区分。
- 最大宽度：mockup 图片 1200px 居中。边距：两侧 24px。

**交互状态：**
- 加载中（页面在图片就绪前打开）：每个卡片的骨架屏脉冲，显示"正在生成变体 A..."。星星/textarea/选择被禁用。
- 部分失败（3 个中 2 个成功）：显示成功的，为失败的创建错误卡片并带每个变体的 [重试]。
- 提交后："反馈已提交！返回你的编码 Agent。"页面保持打开。
- 重新生成：平滑过渡，淡出旧变体，骨架脉冲，淡入新变体。滚动重置到顶部。之前的反馈清除。

**反馈 JSON 结构**（写入隐藏的 #feedback-result 元素）：
```json
{
  "preferred": "A",
  "ratings": { "A": 4, "B": 3, "C": 2 },
  "comments": {
    "A": "喜欢间距，头部看起来不错",
    "B": "太复杂了，但配色不错",
    "C": "完全错误的氛围"
  },
  "overall": "选择 A，把 CTA 放大",
  "regenerated": false
}
```

**可访问性：** 星级评分可键盘导航（方向键）。textarea 有标签（"Variant A 的反馈"）。提交/重新生成可通过键盘访问并带可见的聚焦环。所有文字在白色背景上为 #333+。

**响应式：** >1200px：舒适的边距。768-1200px：较紧的边距。<768px：全宽、无横向滚动。

**截图同意（仅 $D evolve 首次）：** "这将把线上站点的截图发送给 OpenAI 用于设计演进。[继续] [不再询问]" 存储在 ~/.gstack/config.yaml 的 design_screenshot_consent 中。

为什么按顺序：Codex 对抗审查发现位图 PNG 对 Agent 是不透明的（无 DOM、无状态、无可比较的结构）。HTML 线框保留了回到代码的桥接。PNG 供人类说"是的，就是这样。"HTML 供 Agent 说"我知道如何构建这个。"

### 关键设计决策

**1. 无状态 CLI，非守护进程**
浏览需要持久的 Chromium 实例。Design 只是 API 调用 — 不需要服务器。多轮迭代的会话状态是写入 `/tmp/design-session-{id}.json` 的 JSON 文件，包含 `previous_response_id`。
- **会话 ID：** 从 `${PID}-${timestamp}` 生成，通过 `--session` 标志传递
- **发现：** `generate` 命令创建会话文件并打印其路径；`iterate` 通过 `--session` 读取
- **清理：** /tmp 中的会话文件是临时的（操作系统清理）；不需要显式清理

**2. 结构化 brief 输入**
Brief 是 Skill 散文和图像生成之间的接口。Skill 从设计上下文中构建它：
```typescript
interface DesignBrief {
  goal: string;           // "编码评估工具的仪表盘"
  audience: string;       // "技术用户、YC 合伙人"
  style: string;          // "深色主题、奶油色点缀、极简"
  elements: string[];     // ["构建者名称", "分数徽章", "叙事信"]
  constraints?: string;   // "最大宽度 1024px，移动优先"
  reference?: string;     // 现有截图或 DESIGN.md 摘录的路径
  screenType: string;     // "desktop-dashboard" | "mobile-app" | "landing-page" | 等
}
```

**3. 设计 Skill 中默认开启**
Skill 默认生成 mockup。模板包含跳过语言：
```
正在生成提议设计的视觉 mockup...（如果不需要视觉，说"skip"）
```

**4. 视觉质量门控**
生成后，可选将图像通过 GPT-4o vision 检查：
- 文本可读性（标签/标题是否清晰可读？）
- 布局完整性（所有请求的元素是否都存在？）
- 视觉连贯性（看起来像真实的 UI 而非拼贴画？）
失败时自动重试一次。如果仍然失败，则附带警告展示。

**5. 输出位置：探索在 /tmp 中，批准的最终版在 `docs/designs/`**
- 探索变体放入 `/tmp/gstack-mockups-{session}/`（临时的、不提交）
- 只有**用户批准的最终版** mockup 才保存到 `docs/designs/`（纳入版本控制）
- 默认输出目录可通过 CLAUDE.md 的 `design_output_dir` 设置配置
- 文件名模式：`{skill}-{description}-{timestamp}.png`
- 如果不存在则创建 `docs/designs/`（mkdir -p）
- 设计文档引用已提交的图片路径
- 始终通过 Read 工具向用户展示（在 Claude Code 中内联渲染图片）
- 避免仓库膨胀：只有已批准的设计被提交，而非每个探索变体
- 回退：如果不在 git 仓库中，保存到 `/tmp/gstack-mockup-{timestamp}.png`

**6. 信任边界确认**
默认开启的生成会将设计 brief 文本发送到 OpenAI。与现有的 HTML 线框路径（完全本地）相比，这是一个新的外部数据流。Brief 仅包含抽象的设计描述（目标、风格、元素），不包含源代码或用户数据。来自 $B 的截图不会发送到 OpenAI（DesignBrief 中的 reference 字段是本地文件路径，由 Agent 使用，不上传到 API）。在 CLAUDE.md 中记录这一点。

**7. 速率限制缓解**
变体生成使用交错并行：通过 `Promise.allSettled()` 带延迟启动每个 API 调用，间隔 1 秒。这避免了图像生成的 5-7 RPM 速率限制，同时比完全串行更快。如果任何调用返回 429，则使用指数退避重试（2s、4s、8s）。

### 模板集成

**添加到现有 resolver：** `scripts/resolvers/design.ts`（不是新文件）
- 添加 `generateDesignSetup()` 用于 `{{DESIGN_SETUP}}` 占位符（模仿 `generateBrowseSetup()`）
- 添加 `generateDesignMockup()` 用于 `{{DESIGN_MOCKUP}}` 占位符（完整探索工作流）
- 所有 design resolver 保存在一个文件中（与现有代码库约定一致）

**新的 HostPaths 条目：** `types.ts`
```typescript
// claude host：
designDir: '~/.claude/skills/gstack/design/dist'
// codex host：
designDir: '$GSTACK_DESIGN'
```
注意：Codex 运行时设置（`setup` 脚本）还必须导出 `GSTACK_DESIGN` 环境变量，类似于设置 `GSTACK_BROWSE` 的方式。

**`$D` 解析 bash 块**（由 `{{DESIGN_SETUP}}` 生成）：
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
```
如果 `DESIGN_NOT_AVAILABLE`：Skill 回退到 HTML 线框生成（现有 `DESIGN_SKETCH` 模式）。Design mockup 是渐进增强，不是硬性要求。

**现有 resolver 中的新函数：** `scripts/resolvers/design.ts`
- 添加 `generateDesignSetup()` 用于 `{{DESIGN_SETUP}}` — 模仿 `generateBrowseSetup()` 模式
- 添加 `generateDesignMockup()` 用于 `{{DESIGN_MOCKUP}}` — 完整的生成+检查+展示工作流
- 所有 design resolver 保存在一个文件中（与现有代码库约定一致）

### Skill 集成（优先顺序）

**1. /office-hours** — 替换 Visual Sketch 部分
- 选择方案后（Phase 4），生成 hero mockup + 2 个变体
- 通过 Read 工具展示全部三个，请用户选择
- 如果需要则迭代
- 将选择的 mockup 与设计文档一起保存

**2. /plan-design-review** — "更好的样子"
- 当对某个设计维度评分 <7/10 时，生成 mockup 展示 10/10 的样子
- 并排对比：当前（通过 $B 截图）vs. 提议（通过 $D mockup）

**3. /design-consultation** — 设计系统预览
- 生成提议设计系统的视觉预览（字体、颜色、组件）
- 用正式 mockup 替换 /tmp HTML 预览页面

**4. /design-review** — 设计意图对比
- 从规划/DESIGN.md 规格生成"设计意图" mockup
- 与线上站点截图进行视觉差异对比

### 要创建的文件

| 文件 | 用途 |
|------|---------|
| `design/src/cli.ts` | 入口，命令分发 |
| `design/src/commands.ts` | 命令注册表 |
| `design/src/generate.ts` | 通过 Responses API 的 GPT Image 生成 |
| `design/src/iterate.ts` | 带会话状态的多轮迭代 |
| `design/src/variants.ts` | 生成 N 个设计变体 |
| `design/src/check.ts` | 基于视觉的质量门控 |
| `design/src/brief.ts` | 结构化 brief 类型 + 辅助 |
| `design/src/session.ts` | 会话状态管理 |
| `design/src/compare.ts` | HTML 对比面板生成器 |
| `design/test/design.test.ts` | 集成测试（mock OpenAI API） |
| （无 — 添加到现有 `scripts/resolvers/design.ts`） | `{{DESIGN_SETUP}}` + `{{DESIGN_MOCKUP}}` resolver |

### 要修改的文件

| 文件 | 变更 |
|------|--------|
| `scripts/resolvers/types.ts` | 在 `HostPaths` 中添加 `designDir` |
| `scripts/resolvers/index.ts` | 注册 DESIGN_SETUP + DESIGN_MOCKUP resolver |
| `package.json` | 添加 `design` 构建命令 |
| `setup` | 与浏览并行构建 design 二进制 |
| `scripts/resolvers/preamble.ts` | 为 Codex host 添加 `GSTACK_DESIGN` 环境变量导出 |
| `test/gen-skill-docs.test.ts` | 更新 DESIGN_SKETCH 测试套件以适配新的 resolver |
| `setup` | 添加 design 二进制构建 + Codex/Kiro 资产链接 |
| `office-hours/SKILL.md.tmpl` | 用 `{{DESIGN_MOCKUP}}` 替换 Visual Sketch 部分 |
| `plan-design-review/SKILL.md.tmpl` | 添加 `{{DESIGN_SETUP}}` + 低分维度的 mockup 生成 |

### 可复用的现有代码

| 代码 | 位置 | 用途 |
|------|----------|----------|
| 浏览 CLI 模式 | `browse/src/cli.ts` | 命令分发架构 |
| `commands.ts` 注册表 | `browse/src/commands.ts` | 单一真实来源模式 |
| `generateBrowseSetup()` | `scripts/resolvers/browse.ts` | `generateDesignSetup()` 模板 |
| `DESIGN_SKETCH` resolver | `scripts/resolvers/design.ts` | `DESIGN_MOCKUP` resolver 模板 |
| HostPaths 系统 | `scripts/resolvers/types.ts` | 多 host 路径解析 |
| 构建流水线 | `package.json` 构建脚本 | `bun build --compile` 模式 |

### API 详情

**生成：** OpenAI Responses API 带 `image_generation` 工具
```typescript
const response = await openai.responses.create({
  model: "gpt-4o",
  input: briefToPrompt(brief),
  tools: [{ type: "image_generation", size: "1536x1024", quality: "high" }],
});
// 从响应输出项中提取图像
const imageItem = response.output.find(item => item.type === "image_generation_call");
const base64Data = imageItem.result; // base64 编码的 PNG
fs.writeFileSync(outputPath, Buffer.from(base64Data, "base64"));
```

**迭代：** 相同 API 带 `previous_response_id`
```typescript
const response = await openai.responses.create({
  model: "gpt-4o",
  input: feedback,
  previous_response_id: session.lastResponseId,
  tools: [{ type: "image_generation" }],
});
```
**注意：** 通过 `previous_response_id` 的多轮图像迭代是一个需要通过原型验证的假设。Responses API 支持对话线程，但是否保留生成图像的视觉上下文用于编辑式迭代在文档中未确认。**回退：** 如果多轮不起作用，`iterate` 回退到使用原始 brief + 积累反馈在单个 prompt 中重新生成。

**检查：** GPT-4o vision
```typescript
const check = await openai.chat.completions.create({
  model: "gpt-4o",
  messages: [{
    role: "user",
    content: [
      { type: "image_url", image_url: { url: `data:image/png;base64,${imageData}` } },
      { type: "text", text: `检查此 UI mockup。Brief：${brief}。文字可读吗？所有元素都存在吗？看起来像真实的 UI 吗？返回 PASS 或 FAIL 和问题。` }
    ]
  }]
});
```

**成本：** 每次设计会话约 $0.10-$0.40（1 个 hero + 2 个变体 + 1 次质量检查 + 1 次迭代）。与每个 Skill 调用中已有的 LLM 成本相比可忽略不计。

### 认证（通过冒烟测试验证）

**Codex OAuth token 不适用于图像生成。** 测试于 2026-03-26：Images API 和 Responses API 都拒绝 `~/.codex/auth.json` access_token，错误为 "Missing scopes: api.model.images.request"。Codex CLI 也没有原生图像生成能力。

**认证解析顺序：**
1. 读取 `~/.gstack/openai.json` → `{ "api_key": "sk-..." }`（文件权限 0600）
2. 回退到 `OPENAI_API_KEY` 环境变量
3. 如果都不存在 → 引导设置流程：
   - 告诉用户："Design mockup 需要一个带有图像生成权限的 OpenAI API key。在 platform.openai.com/api-keys 获取。"
   - 提示用户粘贴 key
   - 以 0600 权限写入 `~/.gstack/openai.json`
   - 运行冒烟测试（生成 1024x1024 测试图像）验证 key 可用
   - 冒烟测试通过则继续。失败则显示错误并回退到 DESIGN_SKETCH。
4. 如果认证存在但 API 调用失败 → 回退到 DESIGN_SKETCH（现有 HTML 线框方案）。Design mockup 是渐进增强，永远不是硬性要求。

**新命令：** `$D setup` — 引导式 API key 设置 + 冒烟测试。随时可以运行来更新 key。

## 原型中需要验证的假设

1. **图像质量：** "像素级精确 UI mockup" 是理想状态。GPT 图像生成可能无法可靠产出准确的文本渲染、对齐和间距达到真正的 UI 保真度。视觉质量门控有帮助，但"足以用来实施"的成功标准需要在完整 Skill 集成前通过原型验证。
2. **多轮迭代：** `previous_response_id` 是否保留视觉上下文未经验证（见 API 详情部分）。
3. **成本模型：** 估算 $0.10-$0.40/次需要真实世界验证。

**原型验证计划：** 构建 Commit 1（核心生成 + 检查），在不同屏幕类型上运行 10 个设计 brief，评估输出质量后再进入 Skill 集成。

## CEO 扩展范围（通过 /plan-ceo-review SCOPE EXPANSION 接受）

### 1. 设计记忆 + 探索宽度控制
- 从已批准的 mockup 自动提取视觉语言到 DESIGN.md
- 如果 DESIGN.md 存在，未来的 mockup 约束到已建立的设计语言
- 如果没有 DESIGN.md（启动阶段），在多个不同方向间 WIDE 探索
- 渐进约束：设计越成熟，探索范围越窄
- 对比面板获得 REGENERATE 区域，带探索控制：
  - "完全不同的东西"（广泛探索）
  - "更像选项 ___"（围绕最喜欢的做窄探索）
  - "匹配我现有的设计"（约束到 DESIGN.md）
  - 自由文本输入用于具体的方向变更
  - 重新生成刷新页面，Agent 轮询新的提交

### 2. Mockup 差异比较
- `$D diff --before old.png --after new.png` 生成视觉 diff
- 并排对比，高亮变更区域
- 使用 GPT-4o vision 识别差异
- 用于：/design-review、迭代反馈、PR 审查

### 3. 截图到 Mockup 演进
- `$D evolve --screenshot current.png --brief "让它更平静"`
- 获取线上站点截图，生成 mockup 展示它应该是什么样的
- 从现实出发，而非空白画布
- 连接 /design-review 批评与视觉修复议之间的桥梁

### 4. 设计意图验证
- 在 /design-review 期间，将已批准的 mockup（docs/designs/）叠加到线上截图上
- 高亮偏差："你设计了 X，你构建了 Y，这是差距"
- 闭合完整循环：设计 -> 实现 -> 视觉验证
- 结合 $B 截图 + $D diff + vision 分析

### 5. 响应式变体
- `$D variants --brief "..." --viewports desktop,tablet,mobile`
- 自动在多个视口大小生成 mockup
- 对比面板显示响应式网格以便同时批准
- 让响应式设计从 mockup 阶段起就是一等公民

### 6. 设计到代码的 Prompt
- 对比面板批准后，自动生成结构化的实施 prompt
- 通过 vision 分析从批准的 PNG 中提取颜色、字体、布局
- 与 DESIGN.md 和 HTML 线框结合成结构化规格
- 弥合"已批准的设计"与"Agent 开始编码"之间零解读差距的桥梁

### 未来引擎（不在本规划范围内）
- Magic Patterns 集成（从现有设计提取模式）
- Variant API（发布后，多变体 React 代码 + 预览）
- Figma MCP（双向设计文件访问）
- Google Stitch SDK（免费 TypeScript 替代方案）

## 未解决问题

1. Variant 发布 API 后，集成路径是什么？（design 二进制中的独立引擎，还是独立的 Variant 二进制？）
2. Magic Patterns 应该如何集成？（$D 中的另一个引擎，还是独立工具？）
3. 在什么时间点 design 二进制需要插件/引擎架构来支持多个生成后端？

## 成功标准

- 在 UI 想法上运行 `/office-hours` 产出实际的 PNG mockup 与设计文档一起
- 运行 `/plan-design-review` 以 mockup 展示"更好的样子"，而不是散文
- Mockup 质量足够好，开发者可以据此实施
- 质量门控捕获明显损坏的 mockup 并重试
- 每次设计会话的成本保持在 $0.50 以下

## 分发方案

Design 二进制与浏览二进制一起编译和分发：
- `bun build --compile design/src/cli.ts --outfile design/dist/design`
- 在 `./setup` 和 `bun run build` 期间构建
- 通过现有的 `~/.claude/skills/gstack/` 安装路径进行符号链接

## 下一步（实施顺序）

### Commit 0：原型验证（构建基础设施前必须通过）
- 单文件原型脚本（约 50 行），向 GPT Image API 发送 3 个不同的设计 brief
- 验证：文本渲染质量、布局准确性、视觉连贯性
- 如果输出是"令人尴尬的糟糕 AI 艺术"，停止。重新评估方案。
- 这是在构建 8 个基础设施文件前验证核心假设的最便宜的方法。

### Commit 1：Design 二进制核心（generate + check + compare）
- `design/src/` 包含 cli.ts、commands.ts、generate.ts、check.ts、brief.ts、session.ts、compare.ts
- 认证模块（读取 ~/.gstack/openai.json、回退环境变量、引导设置流程）
- `compare` 命令生成带每个变体反馈 textarea 的 HTML 对比面板
- `package.json` 构建命令（与浏览分开 `bun build --compile`）
- `setup` 脚本集成（包括 Codex + Kiro 资产链接）
- 带 mock OpenAI API 服务器的单元测试

### Commit 2：变体 + 迭代
- `design/src/variants.ts`、`design/src/iterate.ts`
- 交错并行生成（每个启动间隔 1s，429 时指数退避）
- 多轮会话状态管理
- 迭代流程 + 速率限制处理测试

### Commit 3：模板集成
- 在现有 `scripts/resolvers/design.ts` 中添加 `generateDesignSetup()` + `generateDesignMockup()`
- 在 `scripts/resolvers/types.ts` 的 `HostPaths` 中添加 `designDir`
- 在 `scripts/resolvers/index.ts` 中注册 DESIGN_SETUP + DESIGN_MOCKUP
- 在 `scripts/resolvers/preamble.ts`（Codex host）中添加 GSTACK_DESIGN 环境变量导出
- 更新 `test/gen-skill-docs.test.ts`（DESIGN_SKETCH 测试套件）
- 重新生成 SKILL.md 文件

### Commit 4：/office-hours 集成
- 用 `{{DESIGN_MOCKUP}}` 替换 Visual Sketch 部分
- 顺序工作流：生成变体 → $D compare → 用户反馈 → DESIGN_SKETCH HTML 线框
- 将已批准的 mockup 保存到 docs/designs/（仅批准的那个，不保存探索版）

### Commit 5：/plan-design-review 集成
- 添加 `{{DESIGN_SETUP}}` 和低分维度的 mockup 生成
- "10/10 的样子" mockup 对比

### Commit 6：设计记忆 + 探索宽度控制（CEO 扩展）
- mockup 批准后，通过 GPT-4o vision 提取视觉语言
- 将提取的颜色、字体、间距、布局模式写入/更新 DESIGN.md
- 如果 DESIGN.md 存在，将其作为约束上下文喂给所有未来的 mockup prompt
- 在对比面板 HTML 中添加 REGENERATE 区域（chiclets + 自由文本 + 刷新循环）
- brief 构建中的渐进约束逻辑

### Commit 7：Mockup 差异比较 + 设计意图验证（CEO 扩展）
- `$D diff` 命令：接收两个 PNG，使用 GPT-4o vision 识别差异，生成叠加层
- `$D verify` 命令：通过 $B 截图线上站点，与 docs/designs/ 中已批准的 mockup 进行 diff
- 集成到 /design-review 模板：当存在已批准的 mockup 时自动验证

### Commit 8：截图到 Mockup 演进（CEO 扩展）
- `$D evolve` 命令：接收截图 + brief，生成"它应该是什么样子"的 mockup
- 将截图作为参考图像发送给 GPT Image API
- 集成到 /design-review："修复应该是什么样子"的视觉提案

### Commit 9：响应式变体 + 设计到代码的 Prompt（CEO 扩展）
- `$D variants` 的 `--viewports` 标志用于多尺寸生成
- 对比面板响应式网格布局
- 批准后自动生成结构化的实施 prompt
- 对批准的 PNG 进行 vision 分析，提取颜色、字体、布局用于 prompt

## TODO

告诉 Variant 去构建一个 API。作为他们的投资者："我在构建一个工作流，AI Agent 以编程方式生成视觉设计。GPT Image API 现在可用 — 但我更愿意使用 Variant，因为多变体方法更适合设计探索。发布一个 API 端点：prompt 进，React 代码 + 预览图像出。我成为你的第一个集成合作伙伴。"

## 验证

1. `bun run build` 编译 `design/dist/design` 二进制
2. `$D generate --brief "开发者工具的落地页" --output /tmp/test.png` 产出真实的 PNG
3. `$D check --image /tmp/test.png --brief "落地页"` 返回 PASS/FAIL
4. `$D variants --brief "..." --count 3 --output-dir /tmp/variants/` 产出 3 个 PNG
5. 在 UI 想法上运行 `/office-hours` 产出内联 mockup
6. `bun test` 通过（Skill 验证、gen-skill-docs）
7. `bun run test:evals` 通过（E2E 测试）

## 我怎么注意到你的思考方式

- 你对文本描述和 ASCII 艺术说了"那不算设计"。那是设计师的直觉 — 你知道描述一件事和展示一件事之间的区别。大多数构建 AI 工具的人不会注意到这个差距，因为他们从来不是设计师。
- 你优先选择了 /office-hours — 上游杠杆点。如果头脑风暴产出真实 mockup，每个下游 Skill（/plan-design-review、/design-review）就有了可供参考的视觉产物，而不是重新解读散文。
- 你投资了 Variant 并立刻想到"他们应该有个 API。"那是投资者作为用户的思维 — 你不仅在评估公司，还在设计他们的产品如何融入你的工作流。
- 当 Codex 挑战选择开启前提时，你立刻接受了。没有自我辩护。这是通往正确答案的最快路径。

## 规格审查结果

文档经历了 1 轮对抗审查。捕获并修复了 11 个问题。
质量评分：7/10 → 修复后估计 8.5/10。

已修复的问题：
1. 声明了 OpenAI SDK 依赖
2. 指定了图像数据提取路径（response.output item 形状）
3. --check 和 --retry 标志在命令注册表中正式注册
4. 指定了 brief 输入模式（纯文本 vs JSON 文件）
5. 修复了 resolver 文件矛盾（添加到现有 design.ts）
6. 注明了 HostPaths Codex 环境变量设置
7. "模仿浏览"重构为"共享编译/分发模式"
8. 会话状态已指定（ID 生成、发现、清理）
9. "像素级精确"标记为需要原型验证的假设
10. 多轮迭代标记为未经验证且有回退计划
11. 完全指定 $D 发现 bash 块并包含回退到 DESIGN_SKETCH

## Eng 审查完成摘要

- 第 0 步：范围挑战 — 范围按原样接受（完整二进制，用户否决了缩小建议）
- 架构审查：发现 5 个问题（openai 依赖分离、优雅降级、输出目录配置、认证模型、信任边界）
- 代码质量审查：发现 1 个问题（8 个文件 vs 5 个，保留 8 个）
- 测试审查：生成了图表，识别了 42 个差距，写了测试计划
- 性能审查：发现 1 个问题（带交错启动的并行变体）
- 不在范围内：Google Stitch SDK 集成、Figma MCP、Variant API（延期）
- 已存在的：浏览 CLI 模式、DESIGN_SKETCH resolver、HostPaths 系统、gen-skill-docs 流水线
- 外部声音：4 轮（Claude 结构化 12 个问题，Codex 结构化 8 个问题，Claude 对抗 1 个致命缺陷，Codex 对抗 1 个致命缺陷）。关键洞察：顺序 PNG→HTML 工作流解决了"不透明位图"致命缺陷。
- 失败模式：0 个关键差距（所有已识别的失败模式都有错误处理 + 测试计划）
- Lake Score：7/7 推荐项选择了完整选项

## GSTACK 审查报告

| 审查 | 触发 | 为什么 | 运行次数 | 状态 | 发现 |
|--------|---------|-----|------|--------|----------|
| Office Hours | `/office-hours` | 设计头脑风暴 | 1 | 完成 | 4 个前提，1 个修改（Codex：选择开启->默认开启） |
| CEO 审查 | `/plan-ceo-review` | 范围与策略 | 1 | 通过 | EXPANSION：提议 6 个，接受 6 个，延期 0 个 |
| Eng 审查 | `/plan-eng-review` | 架构与测试（必需） | 1 | 通过 | 7 个问题，0 个关键差距，4 个外部声音 |
| Design 审查 | `/plan-design-review` | UI/UX 差距 | 1 | 通过 | 评分：2/10 -> 8/10，做出 5 个决策 |
| 外部声音 | structured + adversarial | 独立挑战 | 4 | 完成 | 顺序 PNG->HTML 工作流，信任边界注明 |

**CEO 扩展：** 设计记忆 + 探索宽度、Mockup 差异比较、截图演进、设计意图验证、响应式变体、设计到代码的 Prompt。
**设计决策：** 单列全宽布局、每卡片"更多类似的"、显式单选"选择"、平滑淡入重新生成、骨架屏加载状态。
**未解决：** 0
**结论：** CEO + ENG + DESIGN 通过。准备实施。从 Commit 0（原型验证）开始。
