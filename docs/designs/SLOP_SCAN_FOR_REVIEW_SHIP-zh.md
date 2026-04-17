# 设计：slop-scan 在 /review 和 /ship 中的集成

状态：已推迟
创建日期：2026-04-09
依赖：slop-diff 脚本（scripts/slop-diff.ts，已合并）

## 问题

slop-scan 发现仅在你手动运行 `bun run slop:diff` 时可见。它们
应该在代码审查和发布期间自动显示，与 SQL 安全
和信任边界检查的方式相同。

## 集成点

### /review（步骤 4，清单通过之后）

在 critical/informational 清单通过之后运行 `bun run slop:diff`。将新
发现内联显示在其他 review 输出中：

```
Pre-Landing Review: 3 issues（1 critical，2 informational）

AI Slop: +2 新发现，-0 已移除
  browse/src/new-feature.ts
    defensive.empty-catch: 2 个位置
      line 42: 空 catch，boundary=filesystem
      line 87: 空 catch，boundary=process
```

分类：INFORMATIONAL（从不阻塞合并，仅显示模式）。

Fix-First 启发式适用：如果发现是围绕文件操作的空 catch，
用 `safeUnlink()` 自动修复。如果是扩展代码中的 catch-and-log，则跳过
（根据 CLAUDE.md 指南，这是正确的模式）。

### /ship（步骤 3.5，pre-landing review + PR body）

与 /review 相同的集成。此外，在 PR body 中显示一行摘要：

```markdown
## Pre-Landing Review
- 2 个问题已自动修复，0 个需要输入
- AI Slop: +0 新发现 / -3 已移除 ✓
```

### Review Readiness Dashboard

不要添加行。Slop 是对 diff 的诊断，而不是独立"运行"的 review。
它显示在 Eng Review 输出内部，而不是作为自己独立的仪表板条目。

## 什么自动修复 vs 什么跳过

遵循 CLAUDE.md "Slop-scan" 部分。摘要：

**自动修复（真正的质量改进）：**
- `fs.unlinkSync` 周围的空 catch → 替换为 `safeUnlink()`
- `process.kill` 周围的空 catch → 替换为 `safeKill()`
- 没有封闭 try 的 `return await` → 移除 `await`
- URL 解析周围的无类型 catch → 添加 `instanceof TypeError` 检查

**跳过（slop-scan 标记的正确模式）：**
- 即发即弃浏览器操作（page.close、bringToFront）上的 `.catch(() => {})`
- Chrome 扩展代码中的 catch-and-log（未捕获的错误会使扩展崩溃）
- 关闭/紧急路径中的 `safeUnlinkQuiet`（吞下所有错误是正确的）
- 委托给活动会话的传递包装器（API 稳定性层）

## 实现说明

- `scripts/slop-diff.ts` 已经处理了大部分工作（基于 worktree 的 base
  比较、行号不敏感的指纹识别、优雅回退）
- review/ship skill 运行 bash 块。集成方式为：运行脚本、解析
  输出、包含在 review 发现中
- 如果 slop-scan 未安装（`npx slop-scan` 失败），静默跳过
- 脚本始终退出 0（诊断，从不阻塞）

## 工作量估算

| 任务 | 人工 | CC+gstack |
|------|-------|-----------|
| 添加到 review/SKILL.md.tmpl | 2 小时 | 10 分钟 |
| 添加到 ship/SKILL.md.tmpl | 2 小时 | 10 分钟 |
| 添加到 review/checklist.md | 1 小时 | 5 分钟 |
| 用实际 PR 测试 | 2 小时 | 15 分钟 |
| 重新生成 SKILL.md 文件 | — | 1 分钟 |
