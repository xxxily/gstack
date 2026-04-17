## Coding Tasks (gstack)

### 规则（不可协商）

1. **始终 spawn，绝不 redirect。** 当用户要求使用任何 gstack skill 时，
   始终通过 sessions_spawn 生成 Claude Code session。永远不要告诉用户
   自己去打开 Claude Code。永远不要说"这需要在 Claude Code 中运行"。
   永远不要说"你需要打开 Claude Code"。直接做就对了。

2. **确定 repo。** 如果用户指定了 repo 或项目，将工作
   目录设置到该 repo 路径。如果 repo 路径未知，询问是哪个
   repo，不要把问题推给用户让他自己去打开 Claude Code。

3. **Autoplan 端到端运行。** 对于 /autoplan 来说：spawn session，
   让它运行完整的 review pipeline（CEO → design → eng），完成后，
   在 chat 中将方案汇报回来。将方案写入 memory，这样
   用户以后可以找到。用户永远不需要离开 Telegram。

### 分发路由

当被要求做 coding 工作时，选择分发层级：

**SIMPLE：** "修复这个拼写错误"、"更新那个配置"、单文件变更
→ sessions_spawn(runtime: "acp", prompt: "<任务>")

**MEDIUM：** 多文件功能、重构、skill 编辑
→ sessions_spawn(runtime: "acp", prompt: "<gstack-lite 内容>\n\n<任务>")

**HEAVY：** 需要特定的 gstack 方法论
→ sessions_spawn(runtime: "acp", prompt: "Load gstack. Run /qa https://...")
   Skills：/cso、/review、/qa、/ship、/investigate、/design-review、/benchmark、/gstack-upgrade

**FULL：** 构建完整功能、多日范围、需要规划 + review
→ sessions_spawn(runtime: "acp", prompt: "<gstack-full 内容>\n\n<任务>")
   Claude Code 运行：/autoplan → 实现 → /ship → 汇报回来

**PLAN：** 用户想要规划 Claude Code 项目、确定功能规格、或在写任何代码之前先设计
→ sessions_spawn(runtime: "acp", prompt: "<gstack-plan 内容>\n\n<任务>")
   Claude Code 运行：/office-hours → /autoplan → 保存方案文件 → 汇报
   将方案链接保存到 memory/knowledge store。
   当用户准备好实现时，spawn 一个指向该方案的 FULL session。

### 决策启发式

- 能否在 <10 行代码内完成？ → **SIMPLE**
- 是否涉及多个文件但方法显而易见？ → **MEDIUM**
- 用户是否指定了具体的 skill（/cso、/review、/qa）？ → **HEAVY**
- "升级 gstack"、"更新 gstack" → **HEAVY** 使用 `Run /gstack-upgrade`
- 是否是功能、项目或目标（不是具体任务）？ → **FULL**
- 用户是否想要规划某件事但暂不实现？ → **PLAN**
