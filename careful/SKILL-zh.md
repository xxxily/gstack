---
name: careful
version: 0.1.0
description: |
  破坏性命令的安全防护栏。在执行 rm -rf、DROP TABLE、force-push、git reset --hard、kubectl delete 等破坏性操作前发出警告。
  用户可以覆盖每个警告。在接触生产环境、调试线上系统或在共享环境中工作时使用。
  当用户要求 "be careful"、"safety mode"、"prod mode" 或 "careful mode" 时使用。(gstack)
allowed-tools:
  - Bash
  - Read
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "bash ${CLAUDE_SKILL_DIR}/bin/check-careful.sh"
          statusMessage: "Checking for destructive commands..."
---
<!-- AUTO-GENERATED from SKILL.md.tmpl — do not edit directly -->
<!-- Regenerate: bun run gen:skill-docs -->

# /careful — 破坏性命令防护栏

安全模式现已**激活**。每个 bash 命令在运行前都会检查是否存在破坏性模式。如果检测到破坏性命令，你会收到警告，可以选择继续或取消。

```bash
mkdir -p ~/.gstack/analytics
echo '{"skill":"careful","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","repo":"'$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo "unknown")'"}'  >> ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true
```

## 受保护的模式

| 模式 | 示例 | 风险 |
|---------|---------|------|
| `rm -rf` / `rm -r` / `rm --recursive` | `rm -rf /var/data` | 递归删除 |
| `DROP TABLE` / `DROP DATABASE` | `DROP TABLE users;` | 数据丢失 |
| `TRUNCATE` | `TRUNCATE orders;` | 数据丢失 |
| `git push --force` / `-f` | `git push -f origin main` | 历史重写 |
| `git reset --hard` | `git reset --hard HEAD~3` | 未提交工作丢失 |
| `git checkout .` / `git restore .` | `git checkout .` | 未提交工作丢失 |
| `kubectl delete` | `kubectl delete pod` | 生产环境影响 |
| `docker rm -f` / `docker system prune` | `docker system prune -a` | 容器/镜像丢失 |

## 安全例外

以下模式允许不经警告直接执行：
- `rm -rf node_modules` / `.next` / `dist` / `__pycache__` / `.cache` / `build` / `.turbo` / `coverage`

## 工作原理

hook 从工具输入 JSON 中读取命令，对照上述模式进行检查，如果匹配则返回 `permissionDecision: "ask"` 并附带警告消息。你可以随时覆盖警告并继续执行。

要停用，结束对话或开始新对话即可。hook 的作用域为当前 session。
