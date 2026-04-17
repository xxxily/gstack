# 设计：GStack 自学习基础设施

由 /office-hours + /plan-ceo-review + /plan-eng-review 生成，日期 2026-03-28
更新：2026-04-01（会话智能之后，经 Codex 审核）
分支：garrytan/ce-features
仓库：gstack
状态：ACTIVE
模式：开源/社区

## 问题陈述

GStack 跨会话运行 30+ skill，但它们之间什么都不学习。/review
会话捕获 N+1 查询模式，下次 /review 在同一代码库上
从零开始。/ship 运行发现测试命令，每次未来的 /ship
重新发现它。/investigate 发现棘手的竞态条件，没有未来的会话
知道它。

每个 AI 编码工具都有这个问题。Cursor 有每用户记忆。Claude Code 有
CLAUDE.md。Windsurf 有持久上下文。但它们都不复合。它们都不
结构化它们学到的内容。它们都不在 skill 之间共享知识。

## 我们正在构建什么

跨会话和 skill 复合的每项目制度知识。
结构化的、类型化的、带有信心分数的 learnings，每个 gstack skill 都可以读取和
写入。目标：在同一代码库上经过 20 次会话后，gstack 了解每个
架构决策、每个过去的 bug 模式，以及每次出错的情况。

## 北极星

/autoship（第 5 版）。一个命令中的完整工程团队。描述一个特性、
批准 plan，其他一切都是自动的。/autoship 没有
learnings（R1）、review 质量（R2）、会话持久化（R3）和自适应仪式
（R4）就无法工作。第 1-4 版是让 /autoship 真正工作的基础设施。

## 受众

用 AI 构建的 YC 创始人。每周在真实代码库上运行 gstack 20+ 次
并注意到它重复问同一个问题的人。

## 差异化

| 工具 | 记忆模型 | 范围 | 结构 |
|------|-------------|-------|-----------|
| Cursor | 每用户聊天记忆 | 每会话 | 非结构化 |
| CLAUDE.md | 静态文件 | 每项目 | 手动 |
| Windsurf | 持久上下文 | 每会话 | 非结构化 |
| **GStack** | **每项目 JSONL** | **跨会话、跨 skill** | **类型化、评分、衰减** |

---

## 状态系统

gstack 有四个不同的持久化层。它们共享存储模式
（`~/.gstack/projects/$SLUG/` 中的 JSONL）但服务于不同目的：

| 系统 | 文件 | 存储内容 | 写入者 | 读取者 |
|--------|------|---------------|------------|---------|
| **Learnings** | `learnings.jsonl` | 制度知识（陷阱、模式、偏好） | 所有 skill | 所有 skill（preamble） |
| **时间线** | `timeline.jsonl` | 事件历史（skill 开始/完成、分支、结果） | Preamble（自动） | /retro、preamble 上下文恢复 |
| **Checkpoints** | `checkpoints/*.md` | 工作状态快照（决策、剩余工作、文件） | /checkpoint、/ship、/investigate | Preamble 上下文恢复、/checkpoint 恢复 |
| **健康** | `health-history.jsonl` | 随时间变化的代码质量分数（每工具、综合） | /health | /retro、/ship（门控）、/health（趋势） |

这些不重叠。Learnings = 你知道什么。时间线 = 发生了什么。
Checkpoints = 你在哪里。健康 = 代码有多好。每个回答不同的问题。

---

## 发布路线图

### 第 1 版："GStack 学习"（v0.13-0.14）— 已发布

**标题：** 每次会话让下一次更聪明。

已发布内容：
- `~/.gstack/projects/{slug}/learnings.jsonl` 处的 Learnings 持久化
- 用于手动 review、搜索、修剪、导出的 `/learn` skill
- 所有 review 发现的信心校准（1-10 分，带显示规则）
- 观察/推断 learnings 的信心衰减（1 分/30 天）
- 跨项目 learnings 发现（可选择，AskUserQuestion 同意）
- 当 review 匹配过去 learnings 时的"Learning applied"标注
- 集成到 /review、/ship、/plan-*、/office-hours、/investigate、/retro

Schema：
```json
{
  "ts": "2026-03-28T12:00:00Z",
  "skill": "review",
  "type": "pitfall",
  "key": "n-plus-one-activerecord",
  "insight": "Always check includes() for has_many in list endpoints",
  "confidence": 8,
  "source": "observed",
  "branch": "feature-x",
  "commit": "abc1234",
  "files": ["app/models/user.rb"]
}
```

类型：`pattern` | `pitfall` | `preference` | `architecture` | `tool`
来源：`observed` | `user-stated` | `inferred` | `cross-model`

架构：仅追加 JSONL。重复项在读取时解决（"最新获胜者"
每个 key+type）。无写入时变更，无竞态条件。

### 第 2 版："Review Army"（v0.14.3-0.14.4）— 已发布

**标题：** 每个 PR 上的 10 个 specialist reviewer。

已发布内容：
- 7 个并行 specialist 子 agent：始终开启（testing、maintainability）+
  有条件（security、performance、data-migration、API contract、design）+
  red team（大 diff / critical 发现）
- JSON 结构化发现，带有信心分数 + 跨 agent 指纹去重
- 每次 review 记录 PR 质量分数（0-10）+ /retro 趋势
- 学习驱动的 specialist prompt，过去陷阱按领域注入
- 多 specialist 共识突出显示，确认的发现得到提升
- 通过 PLAN_COMPLETION_AUDIT 增强交付完整性
- 清单重构：CRITICAL 类别保留在主通道中，specialist
  类别提取到 review/specialists/ 中的专注清单

### 第 2.5 版："Review Army 扩展" — 尚未发布

**标题：** 在 R2 证明稳定后发布。检查核心循环的表现。

预检查：审核 R2 质量指标（PR 质量分数、specialist 命中率、
误报率、E2E 测试稳定性）。如果核心循环有问题，先修复。

发布内容：
- E1：自适应 specialist 门控，自动跳过 0 发现记录的 specialist。
  通过 gstack-learnings-log 存储每个项目的命中率。用户可以用 --security 等强制。
- E3：测试 stub 生成，每个 specialist 在发现旁边输出 TEST_STUB。
  从项目检测框架（Jest/Vitest/RSpec/pytest/Go test）。
  流入 Fix-First：AUTO-FIX 应用修复 + 创建测试文件。
- E5：跨 review 发现去重，读取 gstack-review-read 获取以前的 review 条目。
  抑制匹配先前用户跳过发现的发现。
- E7：specialist 性能跟踪，通过 gstack-review-log 记录每个 specialist 指标。
  时间线集成：specialist 运行出现在 timeline.jsonl 中用于 /retro 趋势。

### 第 3 版："会话智能"（v0.15.0）— 已发布

**标题：** 你的 AI 会话记得发生了什么。

已发布内容：
- 会话时间线：每个 skill 自动向
  `~/.gstack/projects/$SLUG/timeline.jsonl` 记录开始/完成事件。仅本地，从不发送到任何地方，
  无论遥测设置如何始终开启。
- 上下文恢复：压缩或会话启动后，preamble 列出最近的 CEO
  plans、checkpoints 和 reviews。Agent 读取最新的以恢复上下文。
- 跨会话注入：preamble 打印当前分支的 LAST_SESSION 和 LATEST_CHECKPOINT。
  你在输入任何内容之前就看到你在哪里停止的。
- 预测性 skill 建议：如果你最后 3 次会话遵循一个模式
  （review、ship、review），gstack 建议你可能接下来想要什么。
- 会话启动时合成的"欢迎回来"上下文消息。
- `/checkpoint` skill：保存/恢复/列出工作状态快照。跨分支
  列表用于 Conductor 工作空间在 agent 之间交接。
- `/health` skill：代码质量记分牌，包装项目工具（tsc、biome、
  knip、shellcheck、tests）。综合 0-10 分、趋势跟踪、分数下降时的改进
  建议。
- 时间线二进制文件：`bin/gstack-timeline-log` 和 `bin/gstack-timeline-read`。
- 路由规则：/checkpoint 和 /health 添加到 preamble skill 路由。

设计文档：`docs/designs/SESSION_INTELLIGENCE.md`

### 第 4 版："自适应仪式" — 尚未发布

**标题：** GStack 尊重你的时间，同时不危及你的安全。

仪式和信任是分开的。仪式 = PR 经过的 review/测试/QA
步骤集合。信任 = 确定应用哪个仪式级别的策略引擎。
它们交互但不合并。

发布内容：

**仪式级别：**
- FULL：所有 specialist、adversarial、Codex 结构化 review、覆盖率审计、plan
  完成。适用于大 diff、新特性、迁移、auth 变更。
- STANDARD：adversarial + Codex、覆盖率审计、plan 完成。适用于中等 diff、
  典型特性工作。
- FAST：仅 adversarial。适用于小型、经过良好测试的、受信任项目上的变更。

**信任策略引擎：**
- 范围感知信任。信任是按变更类别获得的，而不是全局的。仅文档 PR 的干净历史
  不会为迁移 PR 购买信任。
- 变更类别检测：docs、tests、config、frontend、backend、migrations、auth、
  infra。每个类别有自己的信任阈值。
- 信任信号：连续干净的 review（按类别）、/health 分数稳定性、
  回归频率、测试覆盖率趋势。
- 信任从不会快速通道：迁移、auth/权限变更、新 API 端点、
  基础设施变更。无论信任级别如何，这些始终获得 FULL 仪式。
- 逐渐降级，而不是二元重置。单次回归不会重置所有信任。
  它将对该变更类别的信任降低一个级别。

**范围评估：**
- /review、/ship、/autoplan 中基于
  diff 大小、触及文件和变更类别的 TINY/SMALL/MEDIUM/LARGE 分类。
- 仪式级别 = f(范围、信任、变更类别)。

**TODO 生命周期：**
- /triage 用于交互式审批传入 TODO
- /resolve 用于通过并行 agent 批量解决

### 第 5 版："/autoship — 一个命令，完整特性" — 尚未发布

**标题：** 描述一个特性。批准 plan。其他一切都是自动的。

/autoship 是可恢复的状态机，不是线性管道。review 和 QA 可以
将工作送回构建/修复。压缩可以中断任何阶段。系统必须
优雅恢复。

```
                     ┌──────────┐
                     │  START   │
                     └────┬─────┘
                          │
                     ┌────▼─────┐
                     │ /office- │
                     │  hours   │
                     └────┬─────┘
                          │
                     ┌────▼─────┐
                     │/autoplan │ ◄── 单一审批门
                     └────┬─────┘
                          │
               ┌──────────▼──────────┐
               │       BUILD         │ ◄── /checkpoint 自动保存
               └──────────┬──────────┘
                          │
               ┌──────────▼──────────┐
               │      /health        │ ◄── 质量门
               │   (分数 >= 7.0)    │
               └──────────┬──────────┘
                          │ 失败 → 回到 BUILD
               ┌──────────▼──────────┐
               │      /review        │
               └──────────┬──────────┘
                          │ ASK 项 → 回到 BUILD
               ┌──────────▼──────────┐
               │        /qa          │
               └──────────┬──────────┘
                          │ 发现 bug → 回到 BUILD
               ┌──────────▼──────────┐
               │       /ship         │
               └──────────┬──────────┘
                          │
               ┌──────────▼──────────┐
               │ /checkpoint 归档    │ ◄── 保留，不销毁
               └─────────────────────┘
```

发布内容：
- 带有上述状态机的 /autoship 自主管道。
  每个阶段向 timeline.jsonl 写入。每个阶段之前 checkpoint 自动保存。
  压缩恢复：上下文恢复读取 checkpoint + 时间线，在
  最后完成的阶段恢复。
- 完成时 checkpoint 归档（不是删除）。恢复状态被保留
  用于调试失败的 autoship 运行。
- /ideate 头脑风暴 skill（并行发散 agent + adversarial 过滤）
- /plan-eng-review 中的研究 agent（代码库分析师、历史分析师、
  最佳实践研究员、learnings 研究员）

依赖：R1（研究 agent 的 learnings）、R2（质量 review army）、
R3（持久化的会话智能）、R4（速度的自适应仪式）。

### 第 6 版："执行工作室" — 尚未发布

**标题：** 并行执行基础设施。

发布内容：
- Swarm 编排：多 worktree 并行构建。基于 /checkpoint 的 Conductor
  工作空间交接（R3）。编排器 skill 将
  独立工作流分配给并行 agent，每个 agent 有自己的 worktree。
- Codex 构建委托：根据任务类型（样板、测试生成、机械重构）自动检测何时将实现委托给 Codex
  CLI。
- PR 反馈解决：跨 review 平台的并行评论解决。
- /onboard：从代码库分析自动生成的贡献者指南。
- /triage-prs：维护者的批量 PR 分类。

### 第 7 版："设计与媒体" — 尚未发布

**标题：** 视觉设计集成。

发布内容：
- Figma 设计同步（像素匹配迭代循环）
- 特性视频录制（自动生成的 PR 演示）
- 跨平台可移植性（Copilot、Kiro、Windsurf 输出）

---

## 风险登记

### 代理信号作为跳过审查的许可
（由 Codex review 识别，2026-04-01）

/health 分数、干净的 review 历史和时间线模式是有用的信号。
它们不是安全的证明。如果这些信号影响仪式降低 AND /autoship，
失败模式是罕见的、静默的、高严重性的错误。缓解措施：
- 某些变更类别从不快速通道（迁移、auth、infra、新端点）。
- 信任逐渐降级，而不是二元重置。
- /autoship 始终在每个项目的第一次运行上运行 FULL 仪式。信任是获得的。

### 陈旧的上下文恢复
（由 Codex review 识别，2026-04-01）

上下文恢复可以注入错误分支状态、过时的 plans 或无效的
checkpoints。缓解措施：
- Checkpoints 在 YAML frontmatter 中包含分支名称。上下文恢复按
  当前分支过滤。
- 时间线 grep 在显示 LAST_SESSION 之前按分支过滤。
- 陈旧 artifact 检测：如果 checkpoint >7 天，注明它可能
  过时而不是呈现为当前状态。

### 需要的验证指标
（由 Codex review 识别，2026-04-01）

在发布 R4（自适应仪式）之前，测量：
- 预测建议准确度（用户是否运行了建议的 skill？）
- 信任策略误跳过率（快速通道的 PR 是否有合并后问题？）
- 上下文恢复准确度（恢复的上下文是否匹配实际状态？）
- /health 分数与实际代码质量的相关性（高分数是否预测
  更少的生产 bug？）

这些指标应在 R3 使用期间收集，并在 R4 发布前审核。

---

## 已确认的灵感

自学习路线图的灵感来自 Nico Bailon 的 [Compound Engineering](https://github.com/nicobailon/compound-engineering) 项目的想法。他们对 learnings 持久化、并行 review agent 和自主管道的探索催化了 GStack 方法的设计。我们将每个概念适配到 GStack 的模板系统、风格和架构中，而不是直接移植。
