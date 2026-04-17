# gstack x OpenClaw 集成

gstack 将 OpenClaw 集成作为方法论来源，而非移植的代码库。OpenClaw 的 ACP 运行时原生地派生 Claude Code 会话。gstack 提供规划和纪律，让这些会话更高效。

这是一个轻量级协议，通过 prompt 文本编码。没有守护进程。没有 JSON-RPC。没有兼容性矩阵。Prompt 就是桥梁。

## 架构

```
  OpenClaw                               gstack 仓库
  ─────────────────────                    ──────────────
  编排器：消息传递、                       方法论 + 规划的
  日历、内存、EA                          真实来源
        │                                        │
        ├── 原生 Skill（对话式）                  ├── 生成原生 Skill
        │   office-hours、ceo-review、            │   通过 gen-skill-docs 流程
        │   investigate、retro                   │
        │                                        ├── 生成 gstack-lite
        ├── sessions_spawn(runtime: "acp")       │   （规划纪律）
        │       │                                │
        │       └── Claude Code                  ├── 生成 gstack-full
        │           └── gstack 安装在            │   （完整流程）
        │               ~/.claude/skills/gstack  │
        │                                        └── docs/OPENCLAW.md（本文件）
        └── 分发路由（AGENTS.md）
```

## 分发路由

OpenClaw 在派生时决定使用哪个层级的 gstack 支持：

| 层级 | 适用场景 | Prompt 前缀 |
|------|---------|------------|
| **Simple** | 单文件编辑、拼写修改、配置变更 | 不注入 gstack 上下文 |
| **Medium** | 多文件功能、重构 | 追加 gstack-lite CLAUDE.md |
| **Heavy** | 需要特定 gstack Skill | "Load gstack. Run /X" |
| **Full** | 完整功能、目标、项目 | 追加 gstack-full 流程 |
| **Plan** | "帮我规划一个 Claude Code 项目" | 追加 gstack-plan 流程 |

### 决策启发式

- 能用少于 10 行代码完成吗？ -> **Simple**
- 涉及多个文件但方法显而易见吗？ -> **Medium**
- 用户指定了特定的 Skill（/cso、/review、/qa）吗？ -> **Heavy**
- 是一个功能、项目或目标（而非任务）吗？ -> **Full**
- 用户想要规划 Claude Code 项目但暂不实施吗？ -> **Plan**

### 分发路由指南（用于 AGENTS.md）

完整的可粘贴部分位于 `openclaw/agents-gstack-section.md`。复制到你的 OpenClaw AGENTS.md 中即可。

关键行为规则（这些放在分发层级之前）：

1. **始终派生，绝不重定向。** 当用户要求使用任何 gstack Skill 时，始终派生 Claude Code 会话。永远不要告诉用户打开 Claude Code。
2. **确定仓库。** 如果用户指定了仓库，设置工作目录。如果未知，询问是哪个仓库。
3. **Autoplan 端到端运行。** 派生会话，让其运行完整流程，在聊天中汇报结果。用户永远不需要离开 Telegram。

### CLAUDE.md 冲突处理

在已有 CLAUDE.md 的仓库中派生 Claude Code 时，将 gstack-lite/full 追加为新章节。不要替换仓库的现有指令。

## gstack 为 OpenClaw 生成什么

所有产物都存放在 `openclaw/` 目录下，由 `bun run gen:skill-docs --host openclaw` 生成：

### gstack-lite（Medium 层级）
`openclaw/gstack-lite-CLAUDE.md` — 约 15 行规划纪律：
1. 修改前阅读每个文件
2. 编写 5 行计划：做什么、为什么、涉及哪些文件、测试用例、风险
3. 使用决策原则解决歧义
4. 汇报前进行自我审查
5. 完成报告：交付了什么、做了哪些决策、有什么不确定的

A/B 测试验证：耗时 2 倍，输出质量显著提升。

### gstack-full（Full 层级）
`openclaw/gstack-full-CLAUDE.md` — 链式调用现有 gstack Skill：
1. 阅读 CLAUDE.md 并理解项目
2. 运行 /autoplan（CEO + eng + design review）
3. 实施已批准的规划
4. 运行 /ship 创建 PR
5. 汇报 PR URL 和决策内容

### gstack-plan（Plan 层级）
`openclaw/gstack-plan-CLAUDE.md` — 完整审查环节，不实施：
1. 运行 /office-hours 产出设计文档
2. 运行 /autoplan（CEO + eng + design + DX review + codex adversarial）
3. 将审查后的规划保存到 `plans/<project-slug>-plan-<date>.md`
4. 汇报：规划路径、摘要、关键决策、建议的下一步

编排器将规划链接持久化到其自身的内存存储中（brain 仓库、知识库或 AGENTS.md 中配置的任何内容）。当用户准备好构建时，派生一个 FULL 会话来引用已保存的规划。

### 原生的方法论 Skill
发布到 ClawHub。使用 `clawhub install` 安装：
- `gstack-openclaw-office-hours` — 产品审查（6 个强制问题）
- `gstack-openclaw-ceo-review` — 战略挑战（10 部分审查，4 种模式）
- `gstack-openclaw-investigate` — 操作型调试（4 阶段方法论）
- `gstack-openclaw-retro` — 操作型回顾（每周审查）

源代码位于 gstack 仓库中的 `openclaw/skills/`。这些是针对 OpenClaw 对话场景手工改编的 gstack 方法论。不含 gstack 基础设施（无 browse、无遥测、无前导语）。

## 派生会话检测

当 Claude Code 在 OpenClaw 派生的会话中运行时，应设置 `OPENCLAW_SESSION` 环境变量。gstack 检测到此变量后会调整行为：
- 跳过交互式提示（自动选择推荐选项）
- 跳过升级检查和遥测提示
- 专注于任务完成和散文报告

在 sessions_spawn 中设置环境变量：`env: { OPENCLAW_SESSION: "1" }`

## 安装

对于 OpenClaw 用户：告诉你的 OpenClaw 代理"为 openclaw 安装 gstack"。

代理应该：
1. 将 gstack-lite CLAUDE.md 安装到其编码会话模板中
2. 安装 4 个原生方法论 Skill
3. 将分发路由规则添加到 AGENTS.md
4. 通过一次测试派生来验证

对于 gstack 开发者：`./setup --host openclaw` 输出本文档。实际产物由 `bun run gen:skill-docs --host openclaw` 生成。

## 我们不做的事情

- 没有分发守护进程（ACP 处理会话派生）
- 没有 Clawvisor 中继（不需要安全层）
- 没有双向 learnings 桥接（brain 仓库就是知识存储）
- 没有 JSON schema 或协议版本控制
- 没有来自 gstack 的 SOUL.md（OpenClaw 有自己的）
- 没有完整的 Skill 移植（编码 Skill 保持 Claude Code 原生）
