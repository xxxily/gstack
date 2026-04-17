# TODOS.md 格式参考

`/ship`（步骤 5.5）和 `/plan-ceo-review`（TODOS.md 更新部分）引用的标准 TODOS.md 格式参考，确保 TODO 项目结构一致。

---

## 文件结构

```markdown
# TODOS

## <Skill/组件>     ← 例如 ## Browse、## Ship、## Review、## Infrastructure
<按 P0 优先排序的项目，然后是 P1、P2、P3、P4>

## Completed
<已完成的项目，带完成标注>
```

**分区：** 按 skill 或组件组织（`## Browse`、`## Ship`、`## Review`、`## QA`、`## Retro`、`## Infrastructure`）。在每个分区内，按优先级对项目排序（P0 在最上）。

---

## TODO 项目格式

每个项目都是其分区下的 H3：

```markdown
### <标题>

**What：** 一行描述工作内容。

**Why：** 它解决的具体问题或解锁的价值。

**Context：** 足够的细节，让 3 个月后接手的人了解动机、当前状态和从何处开始。

**Effort：** S / M / L / XL
**Priority：** P0 / P1 / P2 / P3 / P4
**Depends on：** <前置条件，或"None">
```

**必填字段：** What、Why、Context、Effort、Priority
**可选字段：** Depends on、Blocked by

---

## 优先级定义

- **P0** — 阻塞：在下次发布之前必须完成
- **P1** — 关键：应在本周期内完成
- **P2** — 重要：在 P0/P1 清除后完成
- **P3** — 锦上添花：在获取采用/使用数据后重新审视
- **P4** — 有朝一日：好主意，不紧急

---

## 已完成项目格式

当项目完成时，将其移至 `## Completed` 分区，保留其原始内容并追加：

```markdown
**Completed：** vX.Y.Z (YYYY-MM-DD)
```
