---
name: freeze
version: 0.1.0
description: |
  将文件编辑限制在特定目录内。阻止在该路径之外进行 Edit 和 Write 操作。
  调试时使用，防止意外"修复"不相关的代码，或者当你想将更改限定在一个模块内时使用。
  当用户要求 "freeze"、"restrict edits"、"only edit this folder" 或 "lock down edits" 时使用。(gstack)
allowed-tools:
  - Bash
  - Read
  - AskUserQuestion
hooks:
  PreToolUse:
    - matcher: "Edit"
      hooks:
        - type: command
          command: "bash ${CLAUDE_SKILL_DIR}/bin/check-freeze.sh"
          statusMessage: "Checking freeze boundary..."
    - matcher: "Write"
      hooks:
        - type: command
          command: "bash ${CLAUDE_SKILL_DIR}/bin/check-freeze.sh"
          statusMessage: "Checking freeze boundary..."
---
<!-- AUTO-GENERATED from SKILL.md.tmpl — do not edit directly -->
<!-- Regenerate: bun run gen:skill-docs -->

# /freeze — 将编辑限制在指定目录

锁定文件编辑到特定目录。任何针对允许路径之外的 Edit 或 Write 操作将被**阻止**（不仅仅是警告）。

```bash
mkdir -p ~/.gstack/analytics
echo '{"skill":"freeze","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","repo":"'$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo "unknown")'"}'  >> ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true
```

## Setup

询问用户要限制到哪个目录。使用 AskUserQuestion：

- Question: "应该将编辑限制到哪个目录？此路径之外的文件将被阻止编辑。"
- 文本输入（非多选）—— 用户输入路径。

一旦用户提供了目录路径：

1. 解析为绝对路径：
```bash
FREEZE_DIR=$(cd "<user-provided-path>" 2>/dev/null && pwd)
echo "$FREEZE_DIR"
```

2. 确保尾部斜杠并保存到 freeze 状态文件：
```bash
FREEZE_DIR="${FREEZE_DIR%/}/"
STATE_DIR="${CLAUDE_PLUGIN_DATA:-$HOME/.gstack}"
mkdir -p "$STATE_DIR"
echo "$FREEZE_DIR" > "$STATE_DIR/freeze-dir.txt"
echo "Freeze boundary set: $FREEZE_DIR"
```

告诉用户："编辑现已限制到 `<path>/`。此目录之外的任何 Edit 或 Write 都将被阻止。要更改边界，再次运行 `/freeze`。要移除它，运行 `/unfreeze` 或结束 session。"

## 工作原理

hook 从 Edit/Write 工具输入 JSON 中读取 `file_path`，然后检查路径是否以 freeze 目录开头。如果不是，则返回 `permissionDecision: "deny"` 以阻止操作。

freeze 边界通过状态文件在 session 内持久化。hook 脚本在每次 Edit/Write 调用时读取它。

## 注意事项

- freeze 目录的尾随 `/` 防止 `/src` 匹配到 `/src-old`
- freeze 仅适用于 Edit 和 Write 工具 —— Read、Bash、Glob、Grep 不受影响
- 这防止意外编辑，不是安全边界 —— Bash 命令如 `sed` 仍然可以修改边界外的文件
- 要停用，运行 `/unfreeze` 或结束对话
