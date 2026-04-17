---
name: gstack-upgrade
version: 1.1.0
description: |
  将 gstack 升级到最新版本。检测全局安装与 vendored 安装，
  运行升级，并显示新内容。当用户要求 "upgrade gstack"、
  "update gstack" 或 "get latest version" 时使用。
  语音触发器（语音转文本别名）："upgrade the tools"、"update the tools"、"gee stack upgrade"、"g stack upgrade"。
allowed-tools:
  - Bash
  - Read
  - Write
  - AskUserQuestion
---
<!-- AUTO-GENERATED from SKILL.md.tmpl — do not edit directly -->
<!-- Regenerate: bun run gen:skill-docs -->

# /gstack-upgrade

将 gstack 升级到最新版本并显示新内容。

## 内联升级流程

本节在所有 skill preamble 检测到 `UPGRADE_AVAILABLE` 时被引用。

### 步骤 1: 询问用户（或自动升级）

首先，检查是否启用了自动升级：
```bash
_AUTO=""
[ "${GSTACK_AUTO_UPGRADE:-}" = "1" ] && _AUTO="true"
[ -z "$_AUTO" ] && _AUTO=$(~/.claude/skills/gstack/bin/gstack-config get auto_upgrade 2>/dev/null || true)
echo "AUTO_UPGRADE=$_AUTO"
```

**如果 `AUTO_UPGRADE=true` 或 `AUTO_UPGRADE=1`：** 跳过 AskUserQuestion。记录"Auto-upgrading gstack v{old} → v{new}..."并直接进入步骤 2。如果自动升级期间 `./setup` 失败，从备份（`.bak` 目录）恢复并警告用户："Auto-upgrade failed —— restored previous version. 手动运行 `/gstack-upgrade` 重试。"

**否则**，使用 AskUserQuestion：
- Question: "gstack **v{new}** 可用（你当前是 v{old}）。现在升级吗？"
- Options: ["Yes, upgrade now", "Always keep me up to date", "Not now", "Never ask again"]

**如果 "Yes, upgrade now"：** 进入步骤 2。

**如果 "Always keep me up to date"：**
```bash
~/.claude/skills/gstack/bin/gstack-config set auto_upgrade true
```
告诉用户："Auto-upgrade enabled. Future updates will install automatically." 然后进入步骤 2。

**如果 "Not now"：** 写入具有递增退避的 snooze 状态（首次 snooze = 24 小时，第二次 = 48 小时，第三次及以上 = 1 周），然后继续当前 skill。不再提及升级。
```bash
_SNOOZE_FILE=~/.gstack/update-snoozed
_REMOTE_VER="{new}"
_CUR_LEVEL=0
if [ -f "$_SNOOZE_FILE" ]; then
  _SNOOZED_VER=$(awk '{print $1}' "$_SNOOZE_FILE")
  if [ "$_SNOOZED_VER" = "$_REMOTE_VER" ]; then
    _CUR_LEVEL=$(awk '{print $2}' "$_SNOOZE_FILE")
    case "$_CUR_LEVEL" in *[!0-9]*) _CUR_LEVEL=0 ;; esac
  fi
fi
_NEW_LEVEL=$((_CUR_LEVEL + 1))
[ "$_NEW_LEVEL" -gt 3 ] && _NEW_LEVEL=3
echo "$_REMOTE_VER $_NEW_LEVEL $(date +%s)" > "$_SNOOZE_FILE"
```
注意：`{new}` 是来自 `UPGRADE_AVAILABLE` 输出的远程版本 —— 从更新检查结果中替换它。

告诉用户 snooze 时长："Next reminder in 24h"（或 48h 或 1 周，取决于级别）。提示："在 `~/.gstack/config.yaml` 中设置 `auto_upgrade: true` 以启用自动升级。"

**如果 "Never ask again"：**
```bash
~/.claude/skills/gstack/bin/gstack-config set update_check false
```
告诉用户："Update checks disabled. 运行 `~/.claude/skills/gstack/bin/gstack-config set update_check true` 以重新启用。"
继续当前 skill。

### 步骤 2: 检测安装类型

```bash
if [ -d "$HOME/.claude/skills/gstack/.git" ]; then
  INSTALL_TYPE="global-git"
  INSTALL_DIR="$HOME/.claude/skills/gstack"
elif [ -d "$HOME/.gstack/repos/gstack/.git" ]; then
  INSTALL_TYPE="global-git"
  INSTALL_DIR="$HOME/.gstack/repos/gstack"
elif [ -d ".claude/skills/gstack/.git" ]; then
  INSTALL_TYPE="local-git"
  INSTALL_DIR=".claude/skills/gstack"
elif [ -d ".agents/skills/gstack/.git" ]; then
  INSTALL_TYPE="local-git"
  INSTALL_DIR=".agents/skills/gstack"
elif [ -d ".claude/skills/gstack" ]; then
  INSTALL_TYPE="vendored"
  INSTALL_DIR=".claude/skills/gstack"
elif [ -d "$HOME/.claude/skills/gstack" ]; then
  INSTALL_TYPE="vendored-global"
  INSTALL_DIR="$HOME/.claude/skills/gstack"
else
  echo "ERROR: gstack not found"
  exit 1
fi
echo "Install type: $INSTALL_TYPE at $INSTALL_DIR"
```

安装类型和目录路径将在后续所有步骤中使用。

### 步骤 3: 保存旧版本

使用步骤 2 输出中的安装目录：

```bash
OLD_VERSION=$(cat "$INSTALL_DIR/VERSION" 2>/dev/null || echo "unknown")
```

### 步骤 4: 升级

使用步骤 2 检测到的安装类型和目录：

**对于 git 安装**（global-git、local-git）：
```bash
cd "$INSTALL_DIR"
STASH_OUTPUT=$(git stash 2>&1)
git fetch origin
git reset --hard origin/main
./setup
```
如果 `$STASH_OUTPUT` 包含 "Saved working directory"，警告用户："Note: local changes were stashed. 在 skill 目录中运行 `git stash pop` 以恢复它们。"

**对于 vendored 安装**（vendored、vendored-global）：
```bash
PARENT=$(dirname "$INSTALL_DIR")
TMP_DIR=$(mktemp -d)
git clone --depth 1 https://github.com/garrytan/gstack.git "$TMP_DIR/gstack"
mv "$INSTALL_DIR" "$INSTALL_DIR.bak"
mv "$TMP_DIR/gstack" "$INSTALL_DIR"
cd "$INSTALL_DIR" && ./setup
rm -rf "$INSTALL_DIR.bak" "$TMP_DIR"
```

### 步骤 4.5: 处理本地 vendored 副本

使用步骤 2 中的安装目录。检查是否还有本地 vendored 副本，以及 team mode 是否激活：

```bash
_ROOT=$(git rev-parse --show-toplevel 2>/dev/null)
LOCAL_GSTACK=""
if [ -n "$_ROOT" ] && [ -d "$_ROOT/.claude/skills/gstack" ]; then
  _RESOLVED_LOCAL=$(cd "$_ROOT/.claude/skills/gstack" && pwd -P)
  _RESOLVED_PRIMARY=$(cd "$INSTALL_DIR" && pwd -P)
  if [ "$_RESOLVED_LOCAL" != "$_RESOLVED_PRIMARY" ]; then
    LOCAL_GSTACK="$_ROOT/.claude/skills/gstack"
  fi
fi
_TEAM_MODE=$(~/.claude/skills/gstack/bin/gstack-config get team_mode 2>/dev/null || echo "false")
echo "LOCAL_GSTACK=$LOCAL_GSTACK"
echo "TEAM_MODE=$_TEAM_MODE"
```

**如果 `LOCAL_GSTACK` 非空且 `TEAM_MODE` 为 `true`：** 移除 vendored 副本。Team mode 使用全局安装作为单一真实来源。

```bash
cd "$_ROOT"
git rm -r --cached .claude/skills/gstack/ 2>/dev/null || true
if ! grep -qF '.claude/skills/gstack/' .gitignore 2>/dev/null; then
  echo '.claude/skills/gstack/' >> .gitignore
fi
rm -rf "$LOCAL_GSTACK"
```
告诉用户："已移除 `$LOCAL_GSTACK` 处的 vendored 副本（team mode 活跃 —— 全局安装是真实来源）。准备好后提交 `.gitignore` 更改。"

**如果 `LOCAL_GSTACK` 非空且 `TEAM_MODE` 不是 `true`：** 通过从刚升级的主安装复制来更新它：
```bash
mv "$LOCAL_GSTACK" "$LOCAL_GSTACK.bak"
cp -Rf "$INSTALL_DIR" "$LOCAL_GSTACK"
rm -rf "$LOCAL_GSTACK/.git"
cd "$LOCAL_GSTACK" && ./setup
rm -rf "$LOCAL_GSTACK.bak"
```
告诉用户："同时更新了 `$LOCAL_GSTACK` 处的 vendored 副本 —— 准备好后提交 `.claude/skills/gstack/`。"

如果 `./setup` 失败，从备份恢复并警告用户：
```bash
rm -rf "$LOCAL_GSTACK"
mv "$LOCAL_GSTACK.bak" "$LOCAL_GSTACK"
```
告诉用户："Sync failed —— restored previous version at `$LOCAL_GSTACK`. 手动运行 `/gstack-upgrade` 重试。"

### 步骤 4.75: 运行版本迁移

在 `./setup` 完成后，运行旧版本和新版本之间 any 迁移脚本。迁移处理 `./setup` 单独无法覆盖的状态修复（过时的配置、孤立文件、目录结构变更）。

```bash
MIGRATIONS_DIR="$INSTALL_DIR/gstack-upgrade/migrations"
if [ -d "$MIGRATIONS_DIR" ]; then
  for migration in $(find "$MIGRATIONS_DIR" -maxdepth 1 -name 'v*.sh' -type f 2>/dev/null | sort -V); do
    # 从文件名提取版本: v0.15.2.0.sh → 0.15.2.0
    m_ver="$(basename "$migration" .sh | sed 's/^v//')"
    # 如果此迁移版本比旧版本新则运行
    if [ "$OLD_VERSION" != "unknown" ] && [ "$(printf '%s\n%s' "$OLD_VERSION" "$m_ver" | sort -V | head -1)" = "$OLD_VERSION" ] && [ "$OLD_VERSION" != "$m_ver" ]; then
      echo "Running migration $m_ver..."
      bash "$migration" || echo "  Warning: migration $m_ver had errors (non-fatal)"
    fi
  done
fi
```

迁移是 `gstack-upgrade/migrations/` 中的幂等 bash 脚本。每个命名为 `v{VERSION}.sh`，仅在从旧版本升级时运行。

### 步骤 5: 写入标记 + 清除缓存

```bash
mkdir -p ~/.gstack
echo "$OLD_VERSION" > ~/.gstack/just-upgraded-from
rm -f ~/.gstack/last-update-check
rm -f ~/.gstack/update-snoozed
```

### 步骤 6: 显示新内容

读取 `$INSTALL_DIR/CHANGELOG.md`。找到旧版本和新版本之间的所有版本条目。总结为 5-7 个按主题分组的要点。不要让用户应接不暇 —— 关注面向用户的变更。跳过内部重构，除非它们很重要。

格式：
```
gstack v{new} — upgraded from v{old}!

What's new:
- [要点 1]
- [要点 2]
- ...

Happy shipping!
```

### 步骤 7: 继续

显示新内容后，继续用户最初调用的 skill。升级已完成 —— 无需进一步操作。

---

## 独立使用

当直接作为 `/gstack-upgrade` 调用时（不是从 preamble）：

1. 强制进行新的更新检查（绕过缓存）：
```bash
~/.claude/skills/gstack/bin/gstack-update-check --force 2>/dev/null || \
.claude/skills/gstack/bin/gstack-update-check --force 2>/dev/null || true
```
使用输出确定是否有可用升级。

2. 如果 `UPGRADE_AVAILABLE <old> <new>`：按照上面的步骤 2-6 执行。

3. 如果没有输出（主安装已是最新）：检查是否有过期的本地 vendored 副本。

运行上面的步骤 2 bash 块来检测主安装类型和目录（`INSTALL_TYPE` 和 `INSTALL_DIR`）。然后运行步骤 4.5 检测 bash 块来检查本地 vendored 副本（`LOCAL_GSTACK`）和 team mode 状态（`TEAM_MODE`）。

**如果 `LOCAL_GSTACK` 为空**（没有本地 vendored 副本）：告诉用户 "You're already on the latest version (v{version})."

**如果 `LOCAL_GSTACK` 非空且 `TEAM_MODE` 为 `true`：** 使用步骤 4.5 team-mode 移除 bash 块移除 vendored 副本。告诉用户："Global v{version} is up to date. 已移除过期的 vendored 副本（team mode 活跃）。准备好后提交 `.gitignore` 更改。"

**如果 `LOCAL_GSTACK` 非空且 `TEAM_MODE` 不是 `true`**，比较版本：
```bash
PRIMARY_VER=$(cat "$INSTALL_DIR/VERSION" 2>/dev/null || echo "unknown")
LOCAL_VER=$(cat "$LOCAL_GSTACK/VERSION" 2>/dev/null || echo "unknown")
echo "PRIMARY=$PRIMARY_VER LOCAL=$LOCAL_VER"
```

**如果版本不同：** 按照步骤 4.5 同步 bash 块更新本地副本。告诉用户："Global v{PRIMARY_VER} is up to date. Updated local vendored copy from v{LOCAL_VER} → v{PRIMARY_VER}. 准备好后提交 `.claude/skills/gstack/`。"

**如果版本匹配：** 告诉用户 "You're on the latest version (v{PRIMARY_VER}). Global and local vendored copy are both up to date."
