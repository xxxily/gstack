# Greptile 评论分类

用于获取、过滤和分类 GitHub PR 上 Greptile review 评论的共享参考。`/review`（步骤 2.5）和 `/ship`（步骤 3.75）都引用此文档。

---

## 获取

运行这些命令来检测 PR 并获取评论。两个 API 调用并行运行。

```bash
REPO=$(gh repo view --json nameWithOwner --jq '.nameWithOwner' 2>/dev/null)
PR_NUMBER=$(gh pr view --json number --jq '.number' 2>/dev/null)
```

**如果任一命令失败或为空：** 静默跳过 Greptile 分类。此集成是附加的——没有它也能正常工作。

```bash
# 并行获取行级 review 评论和顶级 PR 评论
gh api repos/$REPO/pulls/$PR_NUMBER/comments \
  --jq '.[] | select(.user.login == "greptile-apps[bot]") | select(.position != null) | {id: .id, path: .path, line: .line, body: .body, html_url: .html_url, source: "line-level"}' > /tmp/greptile_line.json &
gh api repos/$REPO/issues/$PR_NUMBER/comments \
  --jq '.[] | select(.user.login == "greptile-apps[bot]") | {id: .id, body: .body, html_url: .html_url, source: "top-level"}' > /tmp/greptile_top.json &
wait
```

**如果 API 报错或两个端点都没有 Greptile 评论：** 静默跳过。

行级评论上的 `position != null` 过滤器会自动跳过 force-push 代码产生的过时评论。

---

## 抑制检查

推导项目特定的历史路径：
```bash
REMOTE_SLUG=$(browse/bin/remote-slug 2>/dev/null || ~/.claude/skills/gstack/browse/bin/remote-slug 2>/dev/null || basename "$(git rev-parse --show-toplevel 2>/dev/null || pwd)")
PROJECT_HISTORY="$HOME/.gstack/projects/$REMOTE_SLUG/greptile-history.md"
```

如果存在，读取 `$PROJECT_HISTORY`（每个项目的抑制记录）。每行记录一次之前的分类结果：

```
<date> | <repo> | <type:fp|fix|already-fixed> | <file-pattern> | <category>
```

**类别**（固定集合）：`race-condition`、`null-check`、`error-handling`、`style`、`type-safety`、`security`、`performance`、`correctness`、`other`

将每个获取的评论与条目进行匹配，条件：
- `type == fp`（仅抑制已知的误报，不抑制之前已修复的真实问题）
- `repo` 匹配当前 repo
- `file-pattern` 匹配评论的文件路径
- `category` 匹配评论中的问题类型

将匹配的评论标记为 **SUPPRESSED** 并跳过。

如果历史文件不存在或包含无法解析的行，跳过这些行并继续——永远不要因为格式错误的历史文件而失败。

---

## 分类

对于每个未被抑制的评论：

1. **行级评论：** 在指定的 `path:line` 处读取文件及其上下文（±10 行）
2. **顶级评论：** 读取完整的评论正文
3. 对照完整 diff（`git diff origin/main`）和 review 清单交叉引用评论
4. 分类：
   - **VALID & ACTIONABLE** — 当前代码中真实存在的 bug、竞态条件、安全问题或正确性问题
   - **VALID BUT ALREADY FIXED** — 已在分支后续提交中解决的真实问题。识别修复提交的 SHA。
   - **FALSE POSITIVE** — 评论误解了代码、标记了在其他地方处理的内容，或是风格噪音
   - **SUPPRESSED** — 已在上面的抑制检查中过滤

---

## 回复 API

回复 Greptile 评论时，根据评论来源使用正确的端点：

**行级评论**（来自 `pulls/$PR/comments`）：
```bash
gh api repos/$REPO/pulls/$PR_NUMBER/comments/$COMMENT_ID/replies \
  -f body="<reply text>"
```

**顶级评论**（来自 `issues/$PR/comments`）：
```bash
gh api repos/$REPO/issues/$PR_NUMBER/comments \
  -f body="<reply text>"
```

**如果回复 POST 失败**（例如 PR 已关闭、没有写入权限）：警告并继续。不要因为回复失败而停止工作流。

---

## 回复模板

对所有 Greptile 回复使用这些模板。始终包含具体证据——永远不要发布模糊的回复。

### Tier 1（首次回复）—— 友好、包含证据

**对于 FIXES（用户选择修复问题）：**

```
**Fixed** in `<commit-sha>`。

\`\`\`diff
- <旧的有问题行>
+ <新的已修复行>
\`\`\`

**Why：** <1 句话解释问题所在以及修复如何解决它>
```

**对于 ALREADY FIXED（在分支的先前提交中已解决的问题）：**

```
**Already fixed** in `<commit-sha>`。

**What was done：** <1-2 句话描述现有提交如何解决此问题>
```

**对于 FALSE POSITIVES（评论不正确）：**

```
**Not a bug.** <1 句话直接说明为什么这不正确>

**Evidence：**
- <具体的代码引用，显示模式是安全/正确的>
- <例如："nil 检查由 `ActiveRecord::FinderMethods#find` 处理，它会引发 RecordNotFound 而不是返回 nil">

**Suggested re-rank：** 这似乎是 `<style|noise|misread>` 问题，而不是 `<Greptile 所称的类型>`。建议降低严重程度。
```

### Tier 2（Greptile 在先前回复后重新标记）—— 坚定、压倒性证据

当升级检测（见下文）识别到同一线程上有先前 GStack 回复时，使用 Tier 2。包含最大证据来结束讨论。

```
**This has been reviewed and confirmed as [intentional/already-fixed/not-a-bug].**

\`\`\`diff
<显示变更或安全模式的相关完整 diff>
\`\`\`

**Evidence chain：**
1. <file:line 永久链接，显示安全模式或修复>
2. <解决此问题的 commit SHA（如适用）>
3. <架构原理或设计决策（如适用）>

**Suggested re-rank：** 请重新校准——这是 `<actual category>` 问题，而不是 `<claimed category>`。[如需要，附上具体文件变更永久链接>
```

---

## 升级检测

在撰写回复之前，检查此评论线程上是否已有 GStack 回复：

1. **对于行级评论：** 通过 `gh api repos/$REPO/pulls/$PR_NUMBER/comments/$COMMENT_ID/replies` 获取回复。检查是否有任何回复正文包含 GStack 标记：`**Fixed**`、`**Not a bug.**`、`**Already fixed**`。

2. **对于顶级评论：** 扫描获取的问题评论，查找 Greptile 评论后发布的包含 GStack 标记的回复。

3. **如果存在先前 GStack 回复且 Greptile 在同一文件+类别上再次发帖：** 使用 Tier 2（坚定）模板。

4. **如果没有先前 GStack 回复：** 使用 Tier 1（友好）模板。

如果升级检测失败（API 错误、模糊线程）：默认使用 Tier 1。永远不要在模糊情况下升级。

---

## 严重程度评估与重新排名

在对评论进行分类时，同时评估 Greptile 暗示的严重程度是否与实际情况匹配：

- 如果 Greptile 标记为 **security/correctness/race-condition** 问题，但实际是 **style/performance** 细节：在回复中包含 `**Suggested re-rank：**`，请求更正类别。
- 如果 Greptile 将低严重程度的风格问题标记为严重问题：在回复中反驳。
- 始终具体说明重新排名的理由——引用代码和行号，而不是主观意见。

---

## 历史文件写入

在写入之前，确保两个目录都存在：
```bash
REMOTE_SLUG=$(browse/bin/remote-slug 2>/dev/null || ~/.claude/skills/gstack/browse/bin/remote-slug 2>/dev/null || basename "$(git rev-parse --show-toplevel 2>/dev/null || pwd)")
mkdir -p "$HOME/.gstack/projects/$REMOTE_SLUG"
mkdir -p ~/.gstack
```

将每个分类结果追加一行到**两个**文件（每个项目用于抑制，全局用于 retro）：
- `~/.gstack/projects/$REMOTE_SLUG/greptile-history.md`（每个项目）
- `~/.gstack/greptile-history.md`（全局汇总）

格式：
```
<YYYY-MM-DD> | <owner/repo> | <type> | <file-pattern> | <category>
```

示例条目：
```
2026-03-13 | garrytan/myapp | fp | app/services/auth_service.rb | race-condition
2026-03-13 | garrytan/myapp | fix | app/models/user.rb | null-check
2026-03-13 | garrytan/myapp | already-fixed | lib/payments.rb | error-handling
```

---

## 输出格式

在输出头部包含 Greptile 摘要：
```
+ N Greptile comments (X valid, Y fixed, Z FP)
```

对于每个已分类的评论，显示：
- 分类标签：`[VALID]`、`[FIXED]`、`[FALSE POSITIVE]`、`[SUPPRESSED]`
- 文件:行引用（行级）或 `[top-level]`（顶级）
- 一行正文摘要
- 永久链接 URL（`html_url` 字段）
