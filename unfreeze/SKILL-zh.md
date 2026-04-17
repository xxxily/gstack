---
name: unfreeze
version: 0.1.0
description: |
  清除由 /freeze 设置的 freeze 边界，允许再次对所有目录进行编辑。
  当你想在不结束 session 的情况下扩大编辑范围时使用。
  当用户要求 "unfreeze"、"unlock edits"、"remove freeze" 或 "allow all edits" 时使用。(gstack)
allowed-tools:
  - Bash
  - Read
---
<!-- AUTO-GENERATED from SKILL.md.tmpl — do not edit directly -->
<!-- Regenerate: bun run gen:skill-docs -->

# /unfreeze — 清除 Freeze 边界

移除由 `/freeze` 设置的编辑限制，允许对所有目录进行编辑。

```bash
mkdir -p ~/.gstack/analytics
echo '{"skill":"unfreeze","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","repo":"'$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo "unknown")'"}'  >> ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true
```

## 清除边界

```bash
STATE_DIR="${CLAUDE_PLUGIN_DATA:-$HOME/.gstack}"
if [ -f "$STATE_DIR/freeze-dir.txt" ]; then
  PREV=$(cat "$STATE_DIR/freeze-dir.txt")
  rm -f "$STATE_DIR/freeze-dir.txt"
  echo "Freeze boundary cleared (was: $PREV). Edits are now allowed everywhere."
else
  echo "No freeze boundary was set."
fi
```

告诉用户结果。注意 `/freeze` hook 在当前 session 内仍然注册 —— 它们只是会允许所有操作，因为状态文件不存在了。要重新 freeze，再次运行 `/freeze`。
