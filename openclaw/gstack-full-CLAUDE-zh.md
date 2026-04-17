# gstack-full Pipeline

由 orchestrator 注入，用于完整的功能构建。追加到现有的 CLAUDE.md。

## Full Pipeline
1. 阅读 CLAUDE.md，理解项目上下文。
2. 运行 /autoplan 审查你的方案（CEO + eng + design review pipeline）。
3. 实现已批准的方案。遵循上述 planning discipline。
4. 运行 /ship 创建 PR，包含 tests、changelog 和 version bump。
5. 汇报：PR URL、交付了什么、做了哪些决策、任何不确定的事项。

在 PR 准备好被 review 之前，不要请求人工输入。
