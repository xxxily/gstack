# gstack — AI 工程工作流

gstack 是一组 SKILL.md 文件的集合，为 AI Agent 提供软件开发中的结构化角色。每个 Skill 都是一个专精角色：CEO 评审员、工程经理、设计师、QA 负责人、发布工程师、调试专家等。

## 可用 Skills

Skills 位于 `.agents/skills/` 目录中。按名称调用它们（例如 `/office-hours`）。

| Skill | 功能说明 |
|-------|---------|
| `/office-hours` | 从这里开始。在编写代码之前重新审视你的产品构想。 |
| `/plan-ceo-review` | CEO 级别评审：从需求中找出 10 星级产品。 |
| `/plan-eng-review` | 锁定架构、数据流、边界情况和测试。 |
| `/plan-design-review` | 对每个设计维度进行 0-10 评分，解释 10 分是什么样的。 |
| `/design-consultation` | 从零开始构建完整的设计系统。 |
| `/review` | 合并前的 PR 评审。找出能通过 CI 但在生产环境会出问题的 Bug。 |
| `/debug` | 系统化的根因调试。没有调查就没有修复。 |
| `/design-review` | 设计审计 + 修复循环，使用原子提交。 |
| `/qa` | 打开真实浏览器，发现 Bug，修复它们，重新验证。 |
| `/qa-only` | 与 /qa 相同，但仅报告 — 不修改代码。 |
| `/ship` | 运行测试、审查、推送、创建 PR。一条命令搞定。 |
| `/document-release` | 更新所有文档以匹配你刚发布的内容。 |
| `/retro` | 每周回顾，包含按人员分解和发布记录。 |
| `/browse` | 无头浏览器 — 真实 Chromium，真实点击，每条命令约 100ms。 |
| `/setup-browser-cookies` | 从你真实的浏览器导入 Cookie，用于认证测试。 |
| `/careful` | 在执行破坏性命令前发出警告（rm -rf、DROP TABLE、force-push）。 |
| `/freeze` | 锁定编辑范围到一个目录。硬性阻止，不仅仅是警告。 |
| `/guard` | 同时激活 careful + freeze。 |
| `/unfreeze` | 移除目录编辑限制。 |
| `/gstack-upgrade` | 将 gstack 更新到最新版本。 |

## 构建命令

```bash
bun install              # 安装依赖
bun test                 # 运行测试（免费，<5s）
bun run build            # 生成文档 + 编译二进制文件
bun run gen:skill-docs   # 从模板重新生成 SKILL.md 文件
bun run skill:check      # 所有 Skills 的健康仪表板
```

## 关键约定

- SKILL.md 文件是从 `.tmpl` 模板**自动生成**的。编辑模板，不要编辑生成的输出。
- 运行 `bun run gen:skill-docs --host codex` 重新生成 Codex 专属输出。
- browse 二进制文件提供无头浏览器访问。在 Skills 中使用 `$B <command>`。
- 安全 Skills（careful、freeze、guard）使用内联咨询式建议 — 始终在执行破坏性操作前确认。
