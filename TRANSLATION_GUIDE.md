# Gstack 翻译规范

## 输出格式

每个英文源文件 `.md` 对应生成一个同名中文文件 `{basename}-zh.md`，放在同一目录下。

**不生成双语对照版**——原版和中文版并排放置，方便对照审查。

## 翻译风格

### 技术术语保留
以下技术术语**不翻译**，保留英文：
- Skill, Agent, Subagent, Prompt, PR, CI/CD, Pipeline, Template
- GitHub 相关：PR, Issue, Commit, Branch, Merge, Rebase
- 开发：TDD, LSP, AST, MCP, CLI, API, SDK, RPM, QPS, SLO, SLI
- 平台：Claude Code, Codex, Copilot, Gemini, OpenAI, Supabase

### 专有名词
- gstack（产品名）保留小写 gstack
- 人名、品牌名保留英文
- 代码块、shell 命令、yaml/json/toml 配置**完全不动**

### 翻译原则
1. **Markdown 格式完全保留**：标题层级、列表、表格、代码块、链接等
2. **代码块原样不动**：包括注释中的内容也不翻译（避免代码逻辑受影响）
3. **链接地址不动**：只翻译链接文本，不改 URL
4. **语气**：专业、简洁、技术文档风格
5. **准确性优先**：宁可直译，不要过度意译导致歧义
6. **上下文一致**：同一术语在全文中保持一致的译法

### 不翻译的内容
- 代码块（``` 包裹的内容）
- 行内代码（` 包裹的内容）
- YAML/JSON/TOML 值中的技术标识符
- 文件路径、命令名
- 正则表达式

### SKILL.md 文件特殊规则
- `# Role: ...` 中的角色名可意译为中文，但保留英文括号备注
- Skill 的触发条件（Trigger phrases）保留英文
- 表格中的英文命令/短语保持原样或标注原文
- 工具名（Read, Edit, Bash 等）保留英文

## 文件命名

| 源文件 | 翻译文件 |
|--------|---------|
| `README.md` | `README-zh.md` |
| `office-hours/SKILL.md` | `office-hours/SKILL-zh.md` |
| `review/checklist.md` | `review/checklist-zh.md` |
| `docs/skills.md` | `docs/skills-zh.md` |

规则：在 `.md` 前插入 `-zh`。
