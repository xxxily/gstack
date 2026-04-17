# Design Review 清单（精简版）

> **DESIGN_METHODOLOGY 的子集**——在此处添加项目时，也需要同步更新 `scripts/gen-skill-docs.ts` 中的 `generateDesignMethodology()`，反之亦然。

## 说明

此清单适用于 **diff 中的源代码**——而非渲染后的输出。读取每个已修改的前端文件（完整文件，而不只是 diff 片段），并标记反模式。

**触发条件：** 仅在 diff 涉及前端文件时运行此清单。使用 `gstack-diff-scope` 检测：

```bash
source <(~/.claude/skills/gstack/bin/gstack-diff-scope <base> 2>/dev/null)
```

如果 `SCOPE_FRONTEND=false`，静默跳过整个 design review。

**DESIGN.md 校准：** 如果 repo 根目录存在 `DESIGN.md` 或 `design-system.md`，请先读取。所有发现都基于项目声明的设计系统进行校准。DESIGN.md 明确认可的模式不会被标记。如果没有 DESIGN.md，使用通用设计原则。

---

## 置信度层级

每个项目都标注了检测置信度：

- **[HIGH]** — 可通过 grep/模式匹配可靠检测。确定性发现。
- **[MEDIUM]** — 可通过模式聚合或启发式检测。标记为发现，但预期有一定噪音。
- **[LOW]** — 需要理解视觉意图。呈现为："可能的问题——视觉验证或运行 /design-review。"

---

## 分类

**AUTO-FIX**（仅机械 CSS 修复——高置信度，不需要设计判断）：
- `outline: none` 没有替代 → 添加 `outline: revert` 或 `&:focus-visible { outline: 2px solid currentColor; }`
- 新 CSS 中的 `!important` → 删除并修复特异性
- 正文文本 `font-size` < 16px → 提升到 16px

**ASK**（其他所有——需要设计判断）：
- 所有 AI slop 发现、排版结构、间距选择、交互状态缺口、DESIGN.md 违规

**低置信度项目** → 呈现为"可能：[描述]。视觉验证或运行 /design-review。"永远不要 AUTO-FIX。

---

## 输出格式

```
Design Review: N issues (X auto-fixable, Y need input, Z possible)

**AUTO-FIXED:**
- [file:line] 问题 → 已应用修复

**NEEDS INPUT:**
- [file:line] 问题描述
  Recommended fix: 建议的修复

**POSSIBLE (verify visually):**
- [file:line] 可能的问题——用 /design-review 验证
```

可选：`test_stub`——使用项目的测试框架为此 finding 生成的骨架测试代码。

如果没有发现问题：`Design Review: No issues found.`

如果没有前端文件变更：静默跳过，无输出。

---

## 类别

### 1. AI Slop 检测（6 项）—— 最高优先级

这些是 AI 生成的 UI 的典型痕迹，受重视的工作室的设计师绝不会发布。

- **[MEDIUM]** 紫色/紫罗兰/靛蓝渐变背景或蓝到紫的配色方案。查找 `linear-gradient`，值在 `#6366f1`–`#8b5cf6` 范围内，或解析为紫色/紫罗兰的 CSS 自定义属性。

- **[LOW]** 3 列功能网格：带颜色的圆形中的图标 + 粗体标题 + 2 行描述，对称重复 3 次。查找恰好有 3 个子元素的 grid/flex 容器，每个子元素都包含圆形元素 + 标题 + 段落。

- **[LOW]** 带颜色圆形中的图标作为装饰。查找带有 `border-radius: 50%` + 背景色、用作图标装饰容器的元素。

- **[HIGH]** 全部居中：所有标题、描述和卡片使用 `text-align: center`。Grep 查找 `text-align: center` 密度——如果超过 60% 的文本容器使用居中对齐，标记它。

- **[MEDIUM]** 所有元素使用统一的圆滑边框半径：相同的大半径（16px 及以上）统一应用于卡片、按钮、输入框、容器。聚合 `border-radius` 值——如果超过 80% 使用相同的 ≥16px 值，标记它。

- **[MEDIUM]** 通用 hero 文案："Welcome to [X]"、"Unlock the power of..."、"Your all-in-one solution for..."、"Revolutionize your..."、"Streamline your workflow"。Grep HTML/JSX 内容查找这些模式。

### 2. 排版（4 项）

- **[HIGH]** 正文文本 `font-size` < 16px。Grep 查找 `body`、`p`、`.text` 或基础样式上的 `font-size` 声明。低于 16px（或 base 为 16px 时的 1rem）的值会被标记。

- **[HIGH]** diff 中引入了超过 3 种字体族。统计不同的 `font-family` 声明。如果变更文件中出现超过 3 种独特字体族，标记它。

- **[HIGH]** 标题层级跳跃：同一文件/组件中 `h1` 后跟着 `h3`，跳过了 `h2`。检查 HTML/JSX 中的标题标签。

- **[HIGH]** 黑名单字体：Papyrus、Comic Sans、Lobster、Impact、Jokerman。Grep `font-family` 查找这些名称。

### 3. 间距与布局（4 项）

- **[MEDIUM]** 任意间距值不在 4px 或 8px 刻度上，当 DESIGN.md 指定了间距刻度时。对照声明的刻度检查 `margin`、`padding`、`gap` 值。仅在 DESIGN.md 定义了刻度时标记。

- **[MEDIUM]** 固定宽度没有响应式处理：容器上的 `width: NNNpx` 没有 `max-width` 或 `@media` 断点。移动端有水平滚动风险。

- **[MEDIUM]** 文本容器缺少 `max-width`：正文或段落容器没有设置 `max-width`，允许行超过 75 个字符。检查文本包装器上的 `max-width`。

- **[HIGH]** 新 CSS 规则中的 `!important`。Grep 查找新增行中的 `!important`。几乎总是应该正确修复的特异性逃逸通道。

### 4. 交互状态（3 项）

- **[MEDIUM]** 交互元素（按钮、链接、输入框）缺少 hover/focus 状态。检查新交互元素样式是否存在 `:hover` 和 `:focus` 或 `:focus-visible` 伪类。

- **[HIGH]** `outline: none` 或 `outline: 0` 没有替代的 focus 指示器。Grep 查找 `outline:\s*none` 或 `outline:\s*0`。这移除了键盘可访问性。

- **[LOW]** 交互元素上的触摸目标 < 44px。检查按钮和链接上的 `min-height`/`min-width`/`padding`。需要从多个属性计算有效大小——仅凭代码置信度较低。

### 5. DESIGN.md 违规（3 项，有条件）

仅在 `DESIGN.md` 或 `design-system.md` 存在时应用：

- **[MEDIUM]** 不在声明的调色板中的颜色。将变更 CSS 中的颜色值与 DESIGN.md 中定义的调色板进行比较。

- **[MEDIUM]** 不在声明的排版部分中的字体。将 `font-family` 值与 DESIGN.md 的字体列表进行比较。

- **[MEDIUM]** 不在声明的间距刻度范围内的间距值。将 `margin`/`padding`/`gap` 值与 DESIGN.md 的间距刻度进行比较。

---

## 抑制项

不要标记：
- DESIGN.md 中明确记录为有意选择的模式
- 第三方/vendor CSS 文件（node_modules、vendor 目录）
- CSS reset 或 normalize 样式表
- 测试 fixture 文件
- 生成/压缩的 CSS
