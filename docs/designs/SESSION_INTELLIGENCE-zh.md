# 会话智能层

## 问题

Claude Code 的上下文窗口是临时的。每次会话都从头开始。当
自动压缩在约 167K token 时触发，它保留通用摘要但
销毁文件读取、推理链和中间决策。

gstack 已经生成有价值的 artifacts，它们在磁盘上存活：CEO plans、
eng reviews、design reviews、QA reports、learnings。这些文件包含
塑造当前工作的决策、约束和上下文。但 Claude
不知道它们的存在。压缩之后，影响每个决策的 plans 和 reviews
从上下文中静默消失。

生态正在解决这个问题。claude-mem（9K+ stars）捕获工具使用
并将上下文注入未来会话。Claude HUD 显示实时 agent
状态。Anthropic 自己的 `claude-progress.txt` 模式使用进度文件
agent 在每次会话开始时读取。

没有人解决 **skill 生成的 artifacts** 存活压缩的特定问题。因为没有其他人拥有 gstack 的 artifact 架构。

## 洞察

gstack 已经向 `~/.gstack/projects/$SLUG/` 写入结构化 artifacts：
- CEO plans：`ceo-plans/`
- Design reviews：`design-reviews/`
- Eng reviews：`eng-reviews/`
- Learnings：`learnings.jsonl`
- Skill 使用：`../analytics/skill-usage.jsonl`

缺失的部分不是存储。是意识。preamble 需要告诉
agent："这些文件存在。它们包含你已经做出的决策。
压缩后，重新读取它们。"

## 架构

```
                   ┌─────────────────────────────────────┐
                   │        Claude 上下文窗口              │
                   │   （临时的，约 167K token 限制）      │
                   │                                      │
                   │   压缩触发 ──► 仅摘要                 │
                   └──────────────┬──────────────────────┘
                                  │
                          启动时/压缩后读取
                                  │
                   ┌──────────────▼──────────────────────┐
                   │    ~/.gstack/projects/$SLUG/         │
                   │    （持久化的，存活一切）             │
                   │                                      │
                   │  ceo-plans/         ← /plan-ceo-review
                   │  eng-reviews/       ← /plan-eng-review
                   │  design-reviews/    ← /plan-design-review
                   │  checkpoints/       ← /checkpoint（新）
                   │  timeline.jsonl     ← 每个 skill（新）
                   │  learnings.jsonl    ← /learn
                   └─────────────────────────────────────┘
                                  │
                          每周汇总
                                  │
                   ┌──────────────▼─────────────────────┐
                   │           /retro                     │
                   │  时间线：3 次 /review，2 次 /ship，...  │
                   │  健康趋势：编译 8/10（↑2）            │
                   │  应用的 learnings：本周 4 次           │
                   └─────────────────────────────────────┘
```

## 特性

### 层 1：上下文恢复（preamble，所有 skill）
preamble 中约 10 行散文。压缩或上下文降级后，
agent 检查 `~/.gstack/projects/$SLUG/` 查找最近的 plans、reviews 和
checkpoints。列出目录，读取最新的文件。

成本：接近零。收益：每个 skill 的 plans/reviews 在压缩后存活。

### 层 2：会话时间线（preamble，所有 skill）
每个 skill 向 `timeline.jsonl` 追加一行 JSONL 条目：时间戳、
skill 名称、分支、关键结果。`/retro` 渲染它。

使项目的 AI 辅助工作历史可见。"本周：3 次 /review、
2 次 /ship、1 次 /investigate，跨越 feature-auth 和 fix-billing 分支。"

### 层 3：跨会话注入（preamble，所有 skill）
当新会话在有最近 artifacts 的分支上启动时，preamble
打印一行："上次会话：实现了 JWT auth，3/5 任务完成。
Plan：~/.gstack/projects/$SLUG/checkpoints/latest.md"

agent 在读取任何文件之前就知道你在哪里停止的。

### 层 4：/checkpoint（可选 skill）
工作状态的手动快照：正在做什么、正在编辑的文件、
做出的决策、剩余的。在离开之前、复杂操作之前、工作空间交接或几天后回来时有用。

### 层 5：/health（可选 skill）
代码质量仪表板：类型检查、lint、测试套件、死代码扫描。
综合 0-10 分。随时间跟踪。`/retro` 显示趋势。`/ship`
在可配置阈值上设置门控。

## 复合效应

每个特性独立有用。一起，它们创造了
复合的东西：

会话 1：/plan-ceo-review 生成 plan。保存到磁盘。
会话 2：Agent 在 preamble 后读取 plan。不再重新询问决策。
会话 3：/checkpoint 保存进度。时间线显示 2 次 /review，1 次 /ship。
会话 4：重构中途触发压缩。Agent 重新读取 checkpoint。
           恢复关键决策、类型、剩余工作。继续。
会话 5：/retro 汇总本周。健康趋势：6/10 → 8/10。
           时间线显示 3 个分支上 12 次 skill 调用。

项目的 AI 历史不再是临时的。它持久化、复合，
并使每个未来会话更智能。这就是会话智能层。

## 这不是什么

- 不是 Claude 内置压缩的替代品（那处理会话
  状态；我们处理 gstack artifacts）
- 不是像 claude-mem 那样的完整记忆系统（那通过 SQLite 处理跨会话
  记忆；我们处理结构化 skill artifacts）
- 不是数据库或服务（只是磁盘上的 markdown 文件）

## 研究来源

- [Anthropic: Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)
- [Anthropic: Effective context engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- [claude-mem](https://github.com/thedotmack/claude-mem)
- [Claude HUD](https://github.com/jarrodwatts/claude-hud)
- [CodeScene: Agentic AI coding best practices](https://codescene.com/blog/agentic-ai-coding-best-practice-patterns-for-speed-with-quality)
- [通过 git 持久化状态进行压缩后恢复（Beads）](https://dev.to/jeremy_longshore/building-post-compaction-recovery-for-ai-agent-workflows-with-beads-207l)
