# gstack-lite Planning Discipline

由 orchestrator 注入到被 spawn 的 Claude Code session 中。追加到现有的 CLAUDE.md。

## Planning Discipline
1. 阅读所有你将要修改的文件。首先理解现有模式。
2. 在写代码之前，陈述你的计划：做什么、为什么、涉及哪些文件、测试用例、风险。
3. 遇到歧义时，优先选择：完整性胜过捷径、现有模式胜过新模式、可逆选择胜过不可逆选择、安全默认值胜过聪明做法。
4. 在报告完成之前，自行 review 你的变更。检查：遗漏的文件、损坏的 import、未测试的路径、风格不一致。
5. 完成后汇报：交付了什么、做了哪些决策、任何不确定的事项。
