---
name: investigate
preamble-tier: 2
version: 1.0.0
description: |
  系统调试，带根因调查。四个阶段：investigate、
  analyze、hypothesize、implement。铁律：没有根因就不修复。
  使用场景："debug this"、"fix this bug"、"why is this broken"、
  "investigate this error"或"root cause analysis"。
  当用户报告错误、500 错误、堆栈跟踪、意外行为、"昨天还好好的"，
  或排查为什么某功能不工作时，主动调用此 skill（不要直接调试）。(gstack)
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - AskUserQuestion
  - WebSearch
hooks:
  PreToolUse:
    - matcher: "Edit"
      hooks:
        - type: command
          command: "bash ${CLAUDE_SKILL_DIR}/../freeze/bin/check-freeze.sh"
          statusMessage: "正在检查调试范围边界..."
    - matcher: "Write"
      hooks:
        - type: command
          command: "bash ${CLAUDE_SKILL_DIR}/../freeze/bin/check-freeze.sh"
          statusMessage: "正在检查调试范围边界..."
---
<!-- AUTO-GENERATED from SKILL.md.tmpl — do not edit directly -->
<!-- Regenerate: bun run gen:skill-docs -->

（Preamble、Voice、Context Recovery、AskUserQuestion Format、Completeness Principle、Completion Status Protocol、Operational Self-Improvement、Telemetry、Plan Mode 等标准部分与其他 skill 相同，已省略）

# 系统调试

## 铁律

**没有根因调查，不做修复。**

修复症状只会制造打地鼠式的调试。每个未触及根因的修复都会让下一个 bug 更难找到。找到根因，再修复。

---

## 阶段 1：根因调查

在形成任何假设之前先收集上下文。

1. **收集症状：** 阅读错误消息、堆栈跟踪和复现步骤。如果用户没有提供足够的上下文，通过 AskUserQuestion 一次只问一个问题。

2. **阅读代码：** 从症状追溯代码路径到潜在原因。使用 Grep 查找所有引用，使用 Read 理解逻辑。

3. **检查最近的更改：**
   ```bash
   git log --oneline -20 -- <affected-files>
   ```
   之前能工作吗？什么变了？回归意味着根因在 diff 中。

4. **复现：** 能否确定性地触发 bug？如果不能，在继续之前收集更多证据。

## Prior Learnings

搜索之前会话的相关 learnings：

（代码块保持不变）

如果 `CROSS_PROJECT` 为 `unset`（首次）：使用 AskUserQuestion：

> gstack 可以搜索此机器上其他项目的 learnings，找到
> 可能适用的模式。这保持本地（数据不会离开你的机器）。
> 推荐单人开发者使用。如果你在多个客户端代码库上工作，
> 且跨项目交叉污染会引起问题时请跳过。

选项：
- A) 启用跨项目 learnings（推荐）
- B) 仅限项目作用域的 learnings

如果选择 A：运行 `~/.claude/skills/gstack/bin/gstack-config set cross_project_learnings true`
如果选择 B：运行 `~/.claude/skills/gstack/bin/gstack-config set cross_project_learnings false`

然后用相应标志重新运行搜索。

如果找到 learnings，将其纳入分析。当 review 发现
与过去的 learning 匹配时，显示：

**"已应用 prior learning：[key]（置信度 N/10，来自 [date]）"**

这让复合效果可见。用户应该看到 gstack 正在他们的代码库上变得越来越智能。

输出：**"根因假设：..."**——关于什么错了以及为什么的、具体的、可验证的主张。

---

## 范围锁定

形成根因假设后，将编辑锁定到受影响的模块，以防止范围蔓延。

（代码块保持不变）

**如果 FREEZE_AVAILABLE：** 识别包含受影响文件的最小目录。将其写入 freeze 状态文件：

（代码块保持不变）

将 `<detected-directory>` 替换为实际的目录路径（例如 `src/auth/`）。告诉用户："本调试会话限制在 `<dir>/` 内编辑。这样可以防止对不相关代码的更改。运行 `/unfreeze` 解除限制。"

如果 bug 跨越整个仓库或范围确实不清楚，跳过锁定并注明原因。

**如果 FREEZE_UNAVAILABLE：** 跳过范围锁定。编辑不受限制。

---

## 阶段 2：模式分析

检查此 bug 是否匹配已知模式：

| 模式 | 特征 | 在哪查找 |
|---------|-----------|---------------|
| 竞态条件 | 间歇性、依赖时序 | 对共享状态的并发访问 |
| Nil/null 传播 | NoMethodError、TypeError | 可选值缺少防护 |
| 状态损坏 | 数据不一致、部分更新 | 事务、回调、hooks |
| 集成失败 | 超时、意外响应 | 外部 API 调用、服务边界 |
| 配置漂移 | 本地可工作，staging/prod 失败 | 环境变量、feature flags、数据库状态 |
| 缓存过期 | 显示旧数据、清除缓存后修复 | Redis、CDN、浏览器缓存、Turbo |

同时检查：
- `TODOS.md` 中的已知问题
- 同一区域的 `git log` 中以前的修复——**相同文件中反复出现的 bug 是架构坏味道**，不是巧合

**外部模式搜索：** 如果 bug 不匹配以上已知模式，使用 WebSearch 搜索：
- "{framework} {通用错误类型}"——**先脱敏：**剥离主机名、IP、文件路径、SQL、客户数据。搜索错误类别，而非原始消息。
- "{library} {component} 已知问题"

如果 WebSearch 不可用，跳过此搜索并继续进行假设测试。如果发现文档化的解决方案或已知依赖 bug，将其作为候选假设在阶段 3 中呈现。

---

## 阶段 3：假设测试

在写**任何**修复之前，验证你的假设。

1. **确认假设：** 在疑似根因处添加临时日志语句、断言或调试输出。运行复现。证据是否匹配？

2. **如果假设错误：** 在形成下一个假设之前，考虑搜索该错误。**先脱敏**——剥离主机名、IP、文件路径、SQL 片段、客户标识符和任何内部/专有数据。仅搜索通用错误类型和框架上下文："{component} {脱敏错误类型} {framework version}"。如果错误消息过于具体而无法安全脱敏，跳过搜索。如果 WebSearch 不可用，跳过并继续。然后回到阶段 1。收集更多证据。不要猜测。

3. **三振出局规则：** 如果 3 个假设都失败，**停止**。使用 AskUserQuestion：
   ```
   已测试 3 个假设，没有一个匹配。这可能是架构问题
   而非简单 bug。

   A) 继续调查——我有一个新假设：[描述]
   B) 升级给人工审查——需要熟悉系统的人
   C) 添加日志并等待——在这个区域打桩，下次捕获它
   ```

**红旗**——如果看到以下任何情况，放慢节奏：
- "暂时快速修复"——不存在"暂时"。要么修好要么升级。
- 在追踪数据流之前提出修复——你在猜。
- 每个修复都在别处引出新问题——层级错了，不是代码错了。

---

## 阶段 4：实现

确认根因后：

1. **修复根因，而非症状。** 消除实际问题的最小变更。

2. **最小 diff：** 触及的文件最少，变更的行数最少。抵制重构邻近代码的冲动。

3. **编写回归测试**：
   - **不修复时失败**（证明测试有意义）
   - **有修复时通过**（证明修复有效）

4. **运行完整测试套件。** 粘贴输出。不允许回归。

5. **如果修复触及 >5 个文件：** 使用 AskUserQuestion 标记影响范围：
   ```
   此修复触及 N 个文件。对于 bug 修复来说，这影响范围太大了。
   A) 继续——根因确实跨越这些文件
   B) 拆分——现在修复关键路径，其余推迟
   C) 重新思考——也许有更针对性的方法
   ```

---

## 阶段 5：验证与报告

**全新验证：** 复现原始 bug 场景并确认已修复。这是必须的。

运行测试套件并粘贴输出。

输出结构化的调试报告：
```
调试报告
════════════════════════════════════════
症状：         [用户观察到的]
根因：         [实际出了什么问题]
修复：         [改了什么，含 file:line 引用]
证据：         [测试输出、复现尝试显示修复有效]
回归测试：    [新测试的 file:line]
相关：         [TODOS.md 条目、同一区域之前的 bug、架构备注]
状态：         DONE | DONE_WITH_CONCERNS | BLOCKED
════════════════════════════════════════
```

## 捕获 Learnings

如果你在此会话中发现了非显而易见的模式、陷阱或架构洞见，
为后续会话记录它：

（代码块保持不变）

**类型：** `pattern`（可复用方法）、`pitfall`（不要做什么）、`preference`
（用户声明）、`architecture`（结构决策）、`tool`（library/framework 洞见）、
`operational`（项目环境/CLI/workflow 知识）。

**来源：** `observed`（你在代码中找到的）、`user-stated`（用户告诉你的）、
`inferred`（AI 推理）、`cross-model`（Claude 和 Codex 都同意）。

**置信度：** 1-10。诚实一点。你在代码中验证的观察到的模式是 8-9。
你不太确定的推理是 4-5。用户明确声明的偏好是 10。

**files：** 包含此 learning 涉及的具体文件路径。这支持
过时检测：如果这些文件之后被删除，learning 可以被标记。

**仅记录真正的发现。** 不要记录显而易见的东西。不要记录用户已经知道的东西。一个好的测试：这个洞见能否在未来会话中节省时间？如果可以，记录它。

---

## 重要规则

- **3+ 次修复失败 → 停止并质疑架构。** 是架构问题，不是假设失败。
- **绝不应用无法验证的修复。** 如果不能复现并确认，不要发布。
- **永远不要说"这应该能修复"。** 验证并证明。运行测试。
- **如果修复触及 >5 个文件 → 使用 AskUserQuestion** 在继续之前标记影响范围。
- **完成状态：**
  - DONE——找到根因，应用修复，编写回归测试，所有测试通过
  - DONE_WITH_CONCERNS——已修复但无法完全验证（如间歇性 bug、需 staging 环境）
  - BLOCKED——调查后根因仍不清楚，已升级
