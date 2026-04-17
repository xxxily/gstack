# QA 报告：{APP_NAME}

| 字段 | 值 |
|-------|-------|
| **日期** | {DATE} |
| **URL** | {URL} |
| **分支** | {BRANCH} |
| **提交** | {COMMIT_SHA} ({COMMIT_DATE}) |
| **PR** | {PR_NUMBER} ({PR_URL}) 或"—" |
| **层级** | Quick / Standard / Exhaustive |
| **范围** | {SCOPE 或"完整应用"} |
| **耗时** | {DURATION} |
| **访问页面数** | {COUNT} |
| **截图数** | {COUNT} |
| **框架** | {DETECTED 或"Unknown"} |
| **索引** | [所有 QA 运行](./index.md) |

## 健康分数：{SCORE}/100

| 类别 | 分数 |
|----------|-------|
| Console | {0-100} |
| 链接 | {0-100} |
| 视觉 | {0-100} |
| 功能 | {0-100} |
| UX | {0-100} |
| 性能 | {0-100} |
| 无障碍 | {0-100} |

## 需要修复的前 3 项

1. **{ISSUE-NNN}：{标题}** — {一行描述}
2. **{ISSUE-NNN}：{标题}** — {一行描述}
3. **{ISSUE-NNN}：{标题}** — {一行描述}

## Console 健康

| 错误 | 次数 | 首次出现 |
|-------|-------|------------|
| {错误消息} | {N} | {URL} |

## 摘要

| 严重程度 | 数量 |
|----------|-------|
| Critical | 0 |
| High | 0 |
| Medium | 0 |
| Low | 0 |
| **总计** | **0** |

## 问题

### ISSUE-001：{简短标题}

| 字段 | 值 |
|-------|-------|
| **严重程度** | critical / high / medium / low |
| **类别** | visual / functional / ux / content / performance / console / accessibility |
| **URL** | {页面 URL} |

**描述：** {什么问题，预期 vs 实际。}

**复现步骤：**

1. 导航到 {URL}
   ![步骤 1](screenshots/issue-001-step-1.png)
2. {操作}
   ![步骤 2](screenshots/issue-001-step-2.png)
3. **观察：** {出现的问题}
   ![结果](screenshots/issue-001-result.png)

---

## 已应用的修复（如适用）

| 问题 | 修复状态 | 提交 | 变更文件 |
|-------|-----------|--------|---------------|
| ISSUE-NNN | verified / best-effort / reverted / deferred | {SHA} | {文件} |

### 前后对比证据

#### ISSUE-NNN：{标题}
**修复前：** ![Before](screenshots/issue-NNN-before.png)
**修复后：** ![After](screenshots/issue-NNN-after.png)

---

## 回归测试

| 问题 | 测试文件 | 状态 | 描述 |
|-------|-----------|--------|-------------|
| ISSUE-NNN | path/to/test | committed / deferred / skipped | description |

### 推迟的测试

#### ISSUE-NNN：{标题}
**前置条件：** {触发 bug 的设置状态}
**操作：** {用户做了什么}
**预期：** {正确行为}
**推迟原因：** {原因}

---

## 发布就绪度

| 指标 | 值 |
|--------|-------|
| 健康分数 | {before} → {after} ({delta}) |
| 发现问题数 | N |
| 已应用修复数 | N（已验证：X，尽力而为：Y，回退：Z） |
| 推迟数 | N |

**PR 摘要：** "QA 发现 N 个问题，修复了 M 个，健康分数 X → Y。"

---

## 回归（如适用）

| 指标 | 基线 | 当前 | 差异 |
|--------|----------|---------|-------|
| 健康分数 | {N} | {N} | {+/-N} |
| 问题 | {N} | {N} | {+/-N} |

**自基线以来已修复：** {列表}
**自基线以来的新问题：** {列表}
