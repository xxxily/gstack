# gstack-plan：完整 Review Gauntlet

由 orchestrator 注入，当用户想要规划 Claude Code 项目时使用。
追加到现有的 CLAUDE.md。

## Planning Pipeline
1. 阅读 CLAUDE.md，理解项目上下文。
2. 运行 /office-hours 生成设计文档（问题陈述、前提假设、备选方案）。
3. 运行 /autoplan 审查设计方案（CEO + eng + design + DX review + codex adversarial）。
4. 将最终审查通过的方案保存到文件中，供 orchestrator 后续引用。
   写入路径：当前仓库中的 plans/<project-slug>-plan-<date>.md。
   包含设计文档、所有 review 决策和实现顺序。
5. 向 orchestrator 汇报：
   - 方案文件路径
   - 一段话总结设计了什么以及关键决策
   - 已接受的 scope expansion（如有）
   - 推荐的下一步（通常是：spawn 一个新 session 使用 gstack-full 来实现）

不要实现任何内容。这仅是规划阶段。
orchestrator 会将方案链接保存到其自身的 memory/knowledge store 中。
