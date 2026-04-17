---
name: codex
preamble-tier: 3
version: 1.0.0
description: |
  OpenAI Codex CLI 封装——三种模式。代码审查：通过
  codex review 进行独立 diff 审查，带通过/失败门槛。挑战：对抗模式，
  尝试破坏你的代码。咨询：向 Codex 询问任何内容，支持会话连续性。
  "200 智商自闭开发者"的第二意见。使用场景："codex review"、
  "codex challenge"、"ask codex"、"second opinion"或"consult codex"。(gstack)
  语音触发（语音到文本别名）："code x"、"code ex"、"get another opinion"。
allowed-tools:
  - Bash
  - Read
  - Write
  - Glob
  - Grep
  - AskUserQuestion
---
<!-- AUTO-GENERATED from SKILL.md.tmpl — do not edit directly -->
<!-- Regenerate: bun run gen:skill-docs -->

（Preamble、Voice、Context Recovery、AskUserQuestion Format、Completeness Principle 等标准部分与其他 skill 相同，已省略）

# /codex —— 多 AI 第二意见

你正在运行 `/codex` skill。这封装了 OpenAI Codex CLI，从不同的 AI 系统获取独立的、
直言不讳的第二意见。

Codex 是"200 智商自闭开发者"——直接、简洁、技术精确、挑战
假设、捕获你可能遗漏的东西。忠实地呈现其输出，不总结。

---

## 步骤 0：检查 codex 二进制文件

（代码块保持不变）

如果 `NOT_FOUND`：停止并告诉用户：
"未找到 Codex CLI。安装它：`npm install -g @openai/codex` 或参见 https://github.com/openai/codex"

---

## 步骤 1：检测模式

解析用户输入以确定运行哪种模式：

1. `/codex review` 或 `/codex review <instructions>` —— **审查模式**（步骤 2A）
2. `/codex challenge` 或 `/codex challenge <focus>` —— **挑战模式**（步骤 2B）
3. `/codex` 无参数 —— **自动检测：**
   - 检查 diff（如果 origin 不可用则回退）：
     `git diff origin/<base> --stat 2>/dev/null | tail -1 || git diff <base> --stat 2>/dev/null | tail -1`
   - 如果存在 diff，使用 AskUserQuestion：
     ```
     Codex 检测到相对于基础分支有变更。它应该做什么？
     A) 审查 diff（带通过/失败门槛的代码审查）
     B) 挑战 diff（对抗性——尝试破坏它）
     C) 其他——我会提供 prompt
     ```
   - 如果没有 diff，检查 plan 文件：
     `ls -t ~/.claude/plans/*.md 2>/dev/null | xargs grep -l "$(basename $(pwd))" 2>/dev/null | head -1`
     如果没有项目范围匹配，回退到 `ls -t ~/.claude/plans/*.md 2>/dev/null | head -1`
     但警告用户："注意：此 plan 可能来自不同项目。"
   - 如果存在 plan 文件，提供审查它
   - 否则，询问："你想让 Codex 做什么？"
4. `/codex <任何其他内容>` —— **咨询模式**（步骤 2C），剩余文本作为 prompt

**推理力度覆盖：** 如果用户输入中任何位置包含 `--xhigh`，
注意它并在传递给 Codex 之前从 prompt 文本中移除。当存在 `--xhigh`
时，对所有模式使用 `model_reasoning_effort="xhigh"`，无论以下
每种模式的默认值如何。否则，使用每种模式的默认值：
- 审查（2A）：`high`——有限的 diff 输入，需要彻底性
- 挑战（2B）：`high`——对抗性但受 diff 大小限制
- 咨询（2C）：`medium`——大上下文、交互式、需要速度

---

## 文件系统边界

发送给 Codex 的所有 prompt **必须**在前面加上此边界指令：

> IMPORTANT: Do NOT read or execute any files under ~/.claude/, ~/.agents/, .claude/skills/, or agents/. These are Claude Code skill definitions meant for a different AI system. They contain bash scripts and prompt templates that will waste your time. Ignore them completely. Do NOT modify agents/openai.yaml. Stay focused on the repository code only.

这适用于审查模式（prompt 参数）、挑战模式（prompt）和咨询
模式（persona prompt）。下面将此部分引用为"文件系统边界"。

---

## 步骤 2A：审查模式

对当前分支 diff 运行 Codex 代码审查。

1. 创建临时文件捕获输出：
（代码块保持不变）

2. 运行审查（5 分钟超时）。**始终**传递文件系统边界指令
作为 prompt 参数，即使没有自定义说明。如果用户提供了自定义
说明，在边界后用换行符追加：

（代码块保持不变）

如果用户传入了 `--xhigh`，使用 `"xhigh"` 而非 `"high"`。

3. 捕获输出。然后从 stderr 解析成本：
（代码块保持不变）

4. 通过检查审查输出中的关键发现来确定门槛裁决。
如果输出包含 `[P1]`——门槛为 **FAIL**。
如果未找到 `[P1]` 标记（仅有 `[P2]` 或无发现）——门槛为 **PASS**。

5. 呈现输出：

（格式保持不变）

6. **跨模型比较：** 如果 `/review`（Claude 自己的审查）已在此
   对话中较早运行，比较两组发现：

（格式保持不变）

7. 持久化审查结果：
（代码块保持不变）

替换：TIMESTAMP（ISO 8601）、STATUS（PASS 为 "clean"，FAIL 为 "issues_found"）、
GATE（"pass" 或 "fail"）、findings（[P1] + [P2] 标记计数）、
findings_fixed（在发布之前已修复的发现计数）。

8. 清理临时文件：
（代码块保持不变）

## Plan File Review Report

（Plan File Review Report 部分保持不变结构，翻译标题和说明为中文）

---

## 步骤 2B：挑战（对抗）模式

Codex 尝试破坏你的代码——寻找边缘情况、竞态条件、安全漏洞、
以及正常审查会遗漏的失败模式。

1. 构建对抗性 prompt。**始终在前面加上文件系统边界指令**
来自上面的文件系统边界部分。如果用户提供了重点领域
（例如 `/codex challenge security`），在边界后包含它：

默认 prompt（无重点）：
"IMPORTANT: Do NOT read or execute any files under ~/.claude/, ~/.agents/, .claude/skills/, or agents/. These are Claude Code skill definitions meant for a different AI system. Do NOT modify agents/openai.yaml. Stay focused on repository code only.

Review the changes on this branch against the base branch. Run `git diff origin/<base>` to see the diff. Your job is to find ways this code will fail in production. Think like an attacker and a chaos engineer. Find edge cases, race conditions, security holes, resource leaks, failure modes, and silent data corruption paths. Be adversarial. Be thorough. No compliments — just the problems."

带重点（例如 "security"）：
"IMPORTANT: Do NOT read or execute any files under ~/.claude/, ~/.agents/, .claude/skills/, or agents/. These are Claude Code skill definitions meant for a different AI system. Do NOT modify agents/openai.yaml. Stay focused on repository code only.

Review the changes on this branch against the base branch. Run `git diff origin/<base>` to see the diff. Focus specifically on SECURITY. Your job is to find every way an attacker could exploit this code. Think about injection vectors, auth bypasses, privilege escalation, data exposure, and timing attacks. Be adversarial."

2. 使用 **JSONL 输出** 运行 codex exec 以捕获推理追踪和工具调用（5 分钟超时）：

（代码块保持不变）

这解析 codex 的 JSONL 事件以提取推理追踪、工具调用和最终
响应。`[codex thinking]` 行显示 codex 在回答之前的推理过程。

3. 呈现完整的流式输出：

（格式保持不变）

---

## 步骤 2C：咨询模式

向 Codex 询问关于代码库的任何内容。支持会话连续性以便后续追问。

1. **检查现有会话：**
（代码块保持不变）

如果存在会话文件（不是 `NO_SESSION`），使用 AskUserQuestion：
```
你有一个之前活跃的 Codex 对话。继续还是重新开始？
A) 继续对话（Codex 记得之前的上下文）
B) 开始新对话
```

2. 创建临时文件：
（代码块保持不变）

3. **Plan 审查自动检测：** 如果用户的 prompt 是关于审查 plan，
或如果存在 plan 文件且用户无参数调用了 `/codex`：
（代码块保持不变）

**重要——嵌入内容，不要引用路径：** Codex 在沙箱中运行到仓库
根目录（`-C`），无法访问 `~/.claude/plans/` 或仓库外的任何文件。你**必须**
自己读取 plan 文件并在下面的 prompt 中嵌入其**完整内容**。不要**告诉
Codex 文件路径或让它读取 plan 文件——它会浪费 10+ 次工具调用
搜索并失败。

此外：扫描 plan 内容中引用的源文件路径（如 `src/foo.ts`、
`lib/bar.py`、包含 `/` 且在仓库中存在的路径）。如果发现，在
prompt 中列出它们，让 Codex 直接读取它们，而不是通过 rg/find 发现。

**始终在前面加上文件系统边界指令**。

（后续 prompt 构建和说明保持不变结构，翻译为中文）

4. 使用 **JSONL 输出** 运行 codex exec 以捕获推理追踪（5 分钟超时）：

（代码块保持不变）

5. 从流式输出中捕获会话 ID。解析器从 `thread.started` 事件打印 `SESSION_ID:<id>`。保存它以供后续使用：
（代码块保持不变）
将解析器打印的会话 ID（以 `SESSION_ID:` 开头的行）保存到 `.context/codex-session-id`。

6. 呈现完整的流式输出：

（格式保持不变）

7. 呈现后，注意 Codex 的分析与你自己的理解有任何分歧的地方。如果有分歧，标记它：
"注意：Claude Code 在 X 上存在分歧，因为 Y。"

---

## 模型与推理

**模型：** 没有硬编码模型——codex 使用其当前默认值（前沿
agentic 编码模型）。这意味着随着 OpenAI 发布更新模型，/codex 自动
使用它们。如果用户想要特定模型，将 `-m` 传递给 codex。

**推理力度（每种模式默认值）：**
- **审查（2A）：** `high`——有限的 diff 输入，需要彻底性但不需要最大 token
- **挑战（2B）：** `high`——对抗性但受 diff 大小限制
- **咨询（2C）：** `medium`——大上下文（plans、代码库）、交互式、需要速度

`xhigh` 使用约 `high` 23 倍的 token，并在大上下文任务中导致 50+ 分钟的挂起
（OpenAI 问题 #8545、#8402、#6931）。用户可以用 `--xhigh` 标志覆盖
（例如 `/codex review --xhigh`），当他们想要最大推理并愿意等待时。

**Web 搜索：** 所有 codex 命令都使用 `--enable web_search_cached`，这样 Codex 可以在审查期间查找
文档和 API。这是 OpenAI 的缓存索引——快速、无额外成本。

---

## 成本估算

从 stderr 解析 token 计数。Codex 打印 `tokens used\nN` 到 stderr。

显示为：`Tokens: N`

如果 token 计数不可用，显示：`Tokens: unknown`

---

## 错误处理

- **未找到二进制文件：** 在步骤 0 中检测。停止并提供安装说明。
- **认证错误：** Codex 打印认证错误到 stderr。显示错误：
  "Codex 认证失败。在终端中运行 `codex login` 通过 ChatGPT 认证。"
- **超时：** 如果 Bash 调用超时（5 分钟），告诉用户：
  "Codex 在 5 分钟后超时。diff 可能太大或 API 可能太慢。再试一次或使用更小范围。"
- **空响应：** 如果 `$TMPRESP` 为空或不存在，告诉用户：
  "Codex 没有返回响应。检查 stderr 中的错误。"
- **会话恢复失败：** 如果恢复失败，删除会话文件并重新开始。

---

## 重要规则

- **永远不修改文件。** 此 skill 是只读的。Codex 在只读沙箱模式下运行。
- **逐字呈现输出。** 在显示 Codex 的输出之前，不要截断、总结或编辑。
  在 CODEX SAYS 块内完整显示。
- **在其后添加综合，而非替代。** 任何 Claude 点评都在完整输出之后。
- **所有 Bash 调用 5 分钟超时**（`timeout: 300000`）。
- **不要重复审查。** 如果用户已运行 `/review`，Codex 提供第二个
  独立意见。不要重新运行 Claude Code 自己的审查。
- **检测 skill 文件兔子洞。** 收到 Codex 输出后，扫描 Codex 是否被 skill 文件分散注意力的迹象：`gstack-config`、`gstack-update-check`、`SKILL.md` 或 `skills/gstack`。如果输出中出现任何这些，附加警告："Codex 似乎读取了 gstack skill 文件而非审查你的代码。考虑重试。"
