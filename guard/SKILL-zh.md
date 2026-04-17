---
name: guard
version: 0.1.0
description: |
  完整安全模式：破坏性命令警告 + 目录范围的编辑限制。
  将 /careful（在执行 rm -rf、DROP TABLE、force-push 等操作前警告）与
  /freeze（阻止指定目录之外的编辑）结合使用。在接触生产环境或调试线上系统时用于最大安全防护。
  当用户要求 "guard mode"、"full safety"、"lock it down" 或 "maximum safety" 时使用。(gstack)
allowed-tools:
  - Bash
  - Read
  - AskUserQuestion
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "bash ${CLAUDE_SKILL_DIR}/../careful/bin/check-careful.sh"
          statusMessage: "Checking for destructive commands..."
    - matcher: "Edit"
      hooks:
        - type: command
          command: "bash ${CLAUDE_SKILL_DIR}/../freeze/bin/check-freeze.sh"
          statusMessage: "Checking freeze boundary..."
    - matcher: "Write"
      hooks:
        - type: command
          command: "bash ${CLAUDE_SKILL_DIR}/../freeze/bin/check-freeze.sh"
          statusMessage: "Checking freeze boundary..."
---
<!-- AUTO-GENERATED from SKILL.md.tmpl — do not edit directly -->
<!-- Regenerate: bun run gen:skill-docs -->

# /guard — 完整安全模式

同时激活破坏性命令警告和目录范围的编辑限制。这是 `/careful` + `/freeze` 在一个命令中的组合。

**依赖说明：** 此 skill 引用了兄弟 `/careful` 和 `/freeze` skill 目录中的 hook 脚本。两者都必须安装（它们由 gstack setup 脚本一起安装）。

```bash
mkdir -p ~/.gstack/analytics
echo '{"skill":"guard","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","repo":"'$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo "unknown")'"}'  >> ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true
```

## Setup

询问用户要限制到哪个目录。使用 AskUserQuestion：

- Question: "Guard mode：编辑应限制到哪个目录？破坏性命令警告始终开启。选定路径之外的文件将被阻止编辑。"
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

告诉用户：
- "**Guard mode 已激活。** 两项保护现已运行："
- "1. **破坏性命令警告** —— rm -rf、DROP TABLE、force-push 等将在执行前发出警告（你可以覆盖）"
- "2. **编辑边界** —— 文件编辑限制在 `<path>/` 内。此目录之外的编辑被阻止。"
- "要移除编辑边界，运行 `/unfreeze`。要停用所有保护，结束 session。"

## 受保护内容

请参阅 `/careful` 获取完整的破坏性命令模式列表和安全例外。
请参阅 `/freeze` 了解编辑边界强制的工作原理。
