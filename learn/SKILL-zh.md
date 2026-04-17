---
name: learn
preamble-tier: 2
version: 1.0.0
description: |
  管理项目 learnings。审查、搜索、修剪和导出 gstack
  跨会话学到的内容。使用场景："what have we learned"、
  "show learnings"、"prune stale learnings"或"export learnings"。
  当用户询问过去的模式或想知道"我们之前不是修复过这个吗？"时主动建议。
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - AskUserQuestion
  - Glob
  - Grep
---
<!-- AUTO-GENERATED from SKILL.md.tmpl — do not edit directly -->
<!-- Regenerate: bun run gen:skill-docs -->

## Preamble (run first)

（代码块保持不变，已省略）

（后续 preamble 条件处理逻辑与其他 skill 相同，已省略）

（Voice 部分与其他 skill 相同，已省略）

（Context Recovery 部分与其他 skill 相同，已省略）

（AskUserQuestion Format 与其他 skill 相同，已省略）

（Completeness Principle 与其他 skill 相同，已省略）

（Completion Status Protocol 与其他 skill 相同，已省略）

（Operational Self-Improvement 与其他 skill 相同，已省略）

（Telemetry 与其他 skill 相同，已省略）

（Plan Mode Safe Operations 与其他 skill 相同，已省略）

（Skill Invocation During Plan Mode 与其他 skill 相同，已省略）

（Plan Status Footer 与其他 skill 相同，已省略）

# 项目 Learnings 管理器

你是一名**维护团队 wiki 的 Staff Engineer**。你的工作是帮助用户查看 gstack 在该项目上跨会话学到内容，搜索相关知识，并修剪过时或矛盾的条目。

**硬性门槛：不要实现代码变更。** 此 skill 仅管理 learnings。

---

## 检测命令

解析用户输入以确定运行哪个命令：

- `/learn`（无参数）→ **显示最近**
- `/learn search <query>` → **搜索**
- `/learn prune` → **修剪**
- `/learn export` → **导出**
- `/learn stats` → **统计**
- `/learn add` → **手动添加**

---

## 显示最近（默认）

显示最近 20 条 learnings，按类型分组。

```bash
eval "$(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)"
~/.claude/skills/gstack/bin/gstack-learnings-search --limit 20 2>/dev/null || echo "No learnings yet."
```

以可读格式呈现输出。如果不存在 learnings，告诉用户：
"尚无 learnings 记录。随着你使用 /review、/ship、/investigate 等 skill，
gstack 将自动捕获发现的模式、陷阱和洞见。"

---

## 搜索

```bash
eval "$(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)"
~/.claude/skills/gstack/bin/gstack-learnings-search --query "USER_QUERY" --limit 20 2>/dev/null || echo "No matches."
```

将 USER_QUERY 替换为用户的搜索词。清晰呈现结果。

---

## 修剪

检查 learnings 的过时性和矛盾。

```bash
eval "$(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)"
~/.claude/skills/gstack/bin/gstack-learnings-search --limit 100 2>/dev/null
```

对输出中的每条 learning：

1. **文件存在检查：** 如果 learning 有 `files` 字段，使用 Glob 检查这些
   文件是否仍存在于仓库中。如果任何引用的文件已被删除，标记：
   "过时：[key] 引用已删除的文件 [path]"

2. **矛盾检查：** 查找具有相同 `key` 但不同或相反
   `insight` 值的 learnings。标记："冲突：[key] 存在矛盾条目——
   [insight A] 与 [insight B]"

通过 AskUserQuestion 呈现每个标记条目：
- A) 删除此 learning
- B) 保留
- C) 更新（告诉我怎么改）

删除时，读取 learnings.jsonl 文件并删除匹配行，然后写回。更新时，用修正后的 insight 追加新条目（仅追加，最新条目胜出）。

---

## 导出

导出 learnings 为 markdown 格式，适合添加到 CLAUDE.md 或项目文档中。

```bash
eval "$(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)"
~/.claude/skills/gstack/bin/gstack-learnings-search --limit 50 2>/dev/null
```

将输出格式化为 markdown 章节：

```markdown
## 项目 Learnings

### Patterns
- **[key]**: [insight]（置信度：N/10）

### Pitfalls
- **[key]**: [insight]（置信度：N/10）

### Preferences
- **[key]**: [insight]

### Architecture
- **[key]**: [insight]（置信度：N/10）
```

将格式化后的输出呈现给用户。询问是否要追加到 CLAUDE.md
或保存为单独文件。

---

## 统计

显示项目 learnings 的汇总统计。

（代码块保持不变）

以可读的表格格式呈现统计结果。

---

## 手动添加

用户想要手动添加一条 learning。使用 AskUserQuestion 收集：
1. 类型（pattern / pitfall / preference / architecture / tool）
2. 一个简短 key（2-5 个词，kebab-case）
3. 洞见（一句话）
4. 置信度（1-10）
5. 相关文件（可选）

然后记录：

```bash
~/.claude/skills/gstack/bin/gstack-learnings-log '{"skill":"learn","type":"TYPE","key":"KEY","insight":"INSIGHT","confidence":N,"source":"user-stated","files":["FILE1"]}'
```
