---
name: document-release
preamble-tier: 2
version: 1.0.0
description: |
  发布后文档更新。读取所有项目文档，交叉引用
  diff，更新 README/ARCHITECTURE/CONTRIBUTING/CLAUDE.md 以匹配已发布内容，
  润色 CHANGELOG 语气，清理 TODOS，并可选 bump VERSION。使用场景
  "update the docs"、"sync documentation"或"post-ship docs"。
  在 PR 合并或代码发布后主动建议。(gstack)
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - AskUserQuestion
---
<!-- AUTO-GENERATED from SKILL.md.tmpl — do not edit directly -->
<!-- Regenerate: bun run gen:skill-docs -->

（Preamble、Voice、Context Recovery、AskUserQuestion Format、Completeness Principle 等标准部分与其他 skill 相同，已省略）

# Document Release：发布后文档更新

你正在运行 `/document-release` 工作流。此流程在 `/ship` 之后运行（代码已提交，PR
存在或即将存在），但在 **PR 合并之前**。你的工作：确保项目中
每个文档文件都准确、最新，并以友好、用户优先的语气撰写。

你大部分是自动化的。对明显的事实性变更直接进行。仅在有风险或
主观决策时停止并询问。

**仅在这些情况停止：**
- 有风险/可疑的文档变更（叙述性、哲学性、安全性、删除、大幅重写）
- VERSION bump 决策（如果尚未 bump）
- 要添加的新 TODOS 条目
- 属于叙述性的跨文档矛盾（而非事实性的）

**永远不停止：**
- diff 中明确的事实性更正
- 向表格/列表中添加条目
- 更新路径、计数、版本号
- 修复过时的交叉引用
- CHANGELOG 语气润色（措辞微调）
- 标记 TODOS 为完成
- 跨文档事实性不一致（如版本号不匹配）

**绝不做：**
- 覆盖、替换或重新生成 CHANGELOG 条目——仅润色措辞，保留所有内容
- 不问就 bump VERSION——始终使用 AskUserQuestion 进行版本变更
- 对 CHANGELOG.md 使用 `Write` 工具——始终使用 `Edit` 并精确匹配 `old_string`

---

## 步骤 1：预检与 Diff 分析

1. 检查当前分支。如果在基础分支上，**中止**："你在基础分支上。请从 feature 分支运行。"

2. 收集关于变更的上下文：

（代码块保持不变）

3. 发现仓库中的所有文档文件：

（代码块保持不变）

4. 将变更分类为与文档相关的类别：
   - **新功能**——新文件、新命令、新 skills、新能力
   - **行为变更**——修改的服务、更新的 API、配置变更
   - **已删除功能**——已删除的文件、已删除的命令
   - **基础设施**——构建系统、测试基础设施、CI

5. 输出简短摘要："正在分析 M 个 commit 中的 N 个文件变更。找到 K 个需要审查的文档文件。"

---

## 步骤 2：逐文件文档审计

读取每个文档文件并对照 diff 进行交叉引用。使用以下通用启发式方法
（适应你所在的任何项目——这些不是 gstack 专属的）：

**README.md：**
- 是否描述了 diff 中可见的所有功能和能力？
- 安装/设置说明是否与变更一致？
- 示例、演示和使用说明是否仍然有效？
- 故障排除步骤是否仍然准确？

**ARCHITECTURE.md：**
- ASCII 图和组件描述是否与当前代码匹配？
- 设计决策和"为什么"的解释是否仍然准确？
- 保持保守——仅更新与 diff 明确矛盾的内容。架构文档
  描述不太可能频繁变更的内容。

**CONTRIBUTING.md——新贡献者冒烟测试：**
- 像全新贡献者一样走完设置说明。
- 列出的命令是否准确？每个步骤能成功吗？
- 测试层级描述是否与当前测试基础设施匹配？
- 工作流描述（开发设置、操作学习等）是否最新？
- 标记任何会让首次贡献者失败或困惑的内容。

**CLAUDE.md / 项目说明：**
- 项目结构部分是否与实际文件树匹配？
- 列出的命令和脚本是否准确？
- 构建/测试说明是否与 package.json（或等价物）中的匹配？

**任何其他 .md 文件：**
- 阅读文件，确定其目的和受众。
- 对照 diff 交叉引用，检查是否与文件所述内容矛盾。

对每个文件，将需要的更新分类为：

- **自动更新**——diff 明确需要的更正：向表格中添加条目、更新文件路径、修复计数、更新项目结构树。
- **询问用户**——叙述性变更、删除章节、安全模型变更、大幅重写（一个章节中超过 ~10 行）、模糊相关性、添加全新章节。

---

## 步骤 3：应用自动更新

直接使用 Edit 工具进行所有明确的事实性更新。

对每个修改的文件，输出单行摘要描述**具体变更了什么**——不只是"更新了 README.md"，而是"README.md：在 skills 表格中添加了 /new-skill，skill 计数从 9 更新为 10。"

**不自动更新：**
- README 介绍或项目定位
- ARCHITECTURE 哲学或设计理由
- 安全模型描述
- 不要从任何文档中删除整个章节

---

## 步骤 4：询问有风险/可疑变更

对步骤 2 中识别的每个有风险或可疑的更新，使用 AskUserQuestion，附带：
- 上下文：项目名称、分支、哪个文档文件、我们要审查什么
- 具体的文档决策
- `RECOMMENDATION: Choose [X] because [one-line reason]`
- 包含 C) Skip —— 保持原样的选项

每次回答后立即应用已批准的变更。

---

## 步骤 5：CHANGELOG 语气润色

**关键——永远不要覆盖 CHANGELOG 条目。**

此步骤润色语气。它**不**重写、替换或重新生成 CHANGELOG 内容。

曾发生过 agent 在不应替换时替换了现有 CHANGELOG 条目的事故。此 skill 绝不能这样做。

**规则：**
1. 首先阅读整个 CHANGELOG.md。了解已有的内容。
2. 仅修改现有条目中的措辞。绝不删除、重新排序或替换条目。
3. 绝不从头重新生成 CHANGELOG 条目。条目由 `/ship` 根据实际
   diff 和 commit 历史编写。它是事实来源。你在润色散文，不是
   重写历史。
4. 如果条目看起来错误或 incomplete，使用 AskUserQuestion——不要**静默修复。
5. 使用 Edit 工具精确匹配 `old_string`——绝不使用 Write 覆盖 CHANGELOG.md。

**如果此分支中 CHANGELOG 未被修改：** 跳过此步骤。

**如果此分支中 CHANGELOG 被修改**，审查条目的语气：

- **销售测试：** 用户阅读每个 bullet 时是否会想"哦不错，我想试试"？如果不会，
  重写措辞（而非内容）。
- 以用户现在能**做**什么开头——而非实现细节。
- "你现在可以..."而非"重构了..."
- 标记并重写任何读起来像 commit message 的条目。
- 内部/贡献者变更属于单独的 "### For contributors" 子章节。
- 自动修复小的语气调整。如果重写会改变含义，使用 AskUserQuestion。

---

## 步骤 6：跨文档一致性和可发现性检查

在单独审计每个文件之后，进行跨文档一致性检查：

1. README 的功能/能力列表是否与 CLAUDE.md（或项目说明）描述的匹配？
2. ARCHITECTURE 的组件列表是否与 CONTRIBUTING 的项目结构描述匹配？
3. CHANGELOG 的最新版本是否与 VERSION 文件匹配？
4. **可发现性：** 每个文档文件是否可以从 README.md 或 CLAUDE.md 访问？如果
   ARCHITECTURE.md 存在但 README 和 CLAUDE.md 都没有链接到它，标记它。每个文档
   都应该能从两个入口点文件之一发现。
5. 标记文档之间的矛盾。自动修复明确的事实性不一致（如
   版本不匹配）。对叙述性矛盾使用 AskUserQuestion。

---

## 步骤 7：TODOS.md 清理

这是对 `/ship` 步骤 5.5 的补充。如果可用，读取 `review/TODOS-format.md` 了解规范的 TODO 条目格式。

如果 TODOS.md 不存在，跳过此步骤。

1. **已完成但未标记的条目：** 将 diff 与开放的 TODO 条目交叉引用。如果 TODO
   明显被此分支的变更完成，将其移至 Completed 部分并附带
   `**Completed:** vX.Y.Z.W (YYYY-MM-DD)`。保持保守——仅标记 diff 中有明确
   证据的条目。

2. **需要描述更新的条目：** 如果 TODO 引用了被
   显著修改的文件或组件，其描述可能过时。使用 AskUserQuestion 确认 TODO
   是否应更新、完成或保持原样。

3. **新的延期工作：** 检查 diff 中的 `TODO`、`FIXME`、`HACK` 和 `XXX` 注释。对
   每个代表有意义的延期工作的注释（不是琐碎的内联注释），使用
   AskUserQuestion 询问是否应捕获到 TODOS.md 中。

---

## 步骤 8：VERSION Bump 询问

**关键——不问就不 bump VERSION。**

1. **如果 VERSION 不存在：** 静默跳过。

2. 检查 VERSION 是否已在此分支上被修改：

（代码块保持不变）

3. **如果 VERSION 未被 bump：** 使用 AskUserQuestion：
   - RECOMMENDATION: 选择 C（跳过），因为纯文档变更很少需要版本 bump
   - A) Bump PATCH（X.Y.Z+1）——如果文档变更与代码变更一起发布
   - B) Bump MINOR（X.Y+1.0）——如果这是一个重大的独立发布
   - C) 跳过——不需要版本 bump

4. **如果 VERSION 已被 bump：** 不要**静默跳过。相反，检查 bump
   是否仍覆盖此分支上变更的完整范围：

   a. 阅读当前 VERSION 的 CHANGELOG 条目。它描述了哪些功能？
   b. 阅读完整 diff（`git diff <base>...HEAD --stat` 和 `git diff <base>...HEAD --name-only`）。
      是否有未提及的重大变更（新功能、新 skills、新命令、重大重构）？
   c. **如果 CHANGELOG 条目涵盖了所有内容：** 跳过——输出"VERSION：已 bump 到
      vX.Y.Z，覆盖所有变更。"
   d. **如果有未涵盖的重大变更：** 使用 AskUserQuestion 解释当前
       版本涵盖的内容与新内容的对比，并询问：
       - RECOMMENDATION: 选择 A，因为新变更值得自己的版本
       - A) Bump 到下一个 patch（X.Y.Z+1）——给新变更它们自己的版本
       - B) 保持当前版本——将新变更添加到现有 CHANGELOG 条目
       - C) 跳过——保持版本不变，稍后处理

   关键洞察：为"功能 A"设置的 VERSION bump 不应静默吸收"功能 B"
   如果功能 B 足够重要，值得自己的版本条目。

---

## 步骤 9：提交与输出

**首先检查空状态：** 运行 `git status`（永远不要使用 `-uall`）。如果之前的任何步骤都没有修改文档文件，输出"所有文档都是最新的。"并退出，不提交。

**提交：**

1. 按名称暂存修改的文档文件（永远不要 `git add -A` 或 `git add .`）。
2. 创建单个 commit：

（代码块保持不变）

3. 推送到当前分支：

（代码块保持不变）

**PR/MR body 更新（幂等、竞态安全）：**

1. 读取现有的 PR/MR body 到 PID 唯一的临时文件（使用步骤 0 检测的平台）：

（GitHub 和 GitLab 代码块保持不变）

2. 如果临时文件已经包含 `## Documentation` 章节，用更新内容替换该章节。如果没有，在末尾追加一个 `## Documentation` 章节。

3. Documentation 章节应包含 **文档 diff 预览**——对每个修改的文件，描述具体变更了什么。

4. 将更新的 body 写回：

（GitHub 和 GitLab 代码块保持不变）

5. 清理临时文件：

（代码块保持不变）

6. 如果 `gh pr view` / `glab mr view` 失败（不存在 PR/MR）：跳过并提示"未找到 PR/MR——跳过 body 更新。"
7. 如果 `gh pr edit` / `glab mr update` 失败：警告"无法更新 PR/MR body——文档变更已在
   commit 中。"然后继续。

**结构化文档健康摘要（最终输出）：**

输出可扫描的摘要，显示每个文档文件的状态：

```
文档健康状况：
  README.md       [状态]（[详情]）
  ARCHITECTURE.md [状态]（[详情]）
  CONTRIBUTING.md [状态]（[详情]）
  CHANGELOG.md    [状态]（[详情]）
  TODOS.md        [状态]（[详情]）
  VERSION         [状态]（[详情]）
```

状态可选值：
- Updated——附带变更描述
- Current——不需要变更
- Voice polished——措辞调整
- Not bumped——用户选择跳过
- Already bumped——版本由 /ship 设置
- Skipped——文件不存在

---

## 重要规则

- **编辑前先阅读。** 修改文件前始终读取完整内容。
- **永远不要覆盖 CHANGELOG。** 仅润色措辞。绝不删除、替换或重新生成条目。
- **永远不要静默 bump VERSION。** 始终询问。即使已 bump，也要检查是否覆盖完整变更范围。
- **明确说明变更了什么。** 每次编辑都获得单行摘要。
- **通用启发式，非项目特定。** 审计检查适用于任何仓库。
- **可发现性很重要。** 每个文档文件都应从 README 或 CLAUDE.md 可达。
- **语气：友好、用户优先、不晦涩。** 像向没见过代码的聪明人解释一样写。
