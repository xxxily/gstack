---
name: gstack-openclaw-retro
description: 每周工程回顾。分析 commit history、工作模式和代码质量指标，支持持久化历史记录和趋势追踪。支持团队模式，包含每人贡献、表扬和成长空间。当被要求每周 retro、本周交付了什么或工程回顾时使用。
version: 1.0.0
metadata: { "openclaw": { "emoji": "📊" }
}
---

# 每周工程回顾

生成全面的工程回顾，分析 commit history、工作模式和代码质量指标。支持团队模式：识别运行命令的用户，然后分析每位贡献者，包含每人表扬和成长机会。

## 参数

- 默认：过去 7 天
- `24h`：过去 24 小时
- `14d`：过去 14 天
- `30d`：过去 30 天
- `compare`：将当前窗口与之前相同长度的窗口进行比较

## 指令

解析参数确定时间窗口。默认为 7 天。所有时间应报告为用户的**本地时区**。

**午夜对齐窗口：** 对于天数单位，计算本地午夜的绝对开始日期。例如，如果今天是 2026-03-18 且窗口为 7 天，则开始日期为 2026-03-11。对 git log 查询使用 `--since="2026-03-11T00:00:00"`。对于小时单位，使用 `--since="N hours ago"`。

---

### Step 1：收集原始数据

首先，fetch origin 并识别当前用户：

```bash
git fetch origin main --quiet
git config user.name
git config user.email
```

`git config user.name` 返回的名字就是**"你"**……即阅读此 retro 的人。所有其他作者是队友。

并行运行所有这些 git 命令（它们是独立的）：

```bash
# 所有 commits 带时间戳、主题、hash、作者、变更文件数
git log origin/main --since="<window>" --format="%H|%aN|%ae|%ai|%s" --shortstat

# 每个 commit 的 test 与总 LOC 分解，带作者
git log origin/main --since="<window>" --format="COMMIT:%H|%aN" --numstat

# commit 时间戳，用于 session 检测和每小时分布
git log origin/main --since="<window>" --format="%at|%aN|%ai|%s" | sort -n

# 最常变更的文件（hotspot 分析）
git log origin/main --since="<window>" --format="" --name-only | grep -v '^$' | sort | uniq -c | sort -rn

# 从 commit 消息中提取 PR 编号
git log origin/main --since="<window>" --format="%s" | grep -oE '[#!][0-9]+' | sort -t'#' -k1 | uniq

# 每人文件 hotspot
git log origin/main --since="<window>" --format="AUTHOR:%aN" --name-only

# 每人 commit 数量
git shortlog origin/main --since="<window>" -sn --no-merges

# 测试文件数量
find . -name '*.test.*' -o -name '*.spec.*' -o -name '*_test.*' -o -name '*_spec.*' 2>/dev/null | grep -v node_modules | wc -l

# 窗口内变更的测试文件
git log origin/main --since="<window>" --format="" --name-only | grep -E '\.(test|spec)\.' | sort -u | wc -l
```

---

### Step 2：计算指标

计算并在摘要中展示这些指标：

- **Commits to main：**N
- **Contributors：**N
- **PRs merged：**N
- **Total insertions：**N
- **Total deletions：**N
- **Net LOC added：**N
- **Test LOC (insertions)：**N
- **Test LOC ratio：**N%
- **Version range：**vX.Y.Z → vX.Y.Z
- **Active days：**N
- **Detected sessions：**N
- **Avg LOC/session-hour：**N

然后在下方立即显示**每人排行榜**：

```
Contributor         Commits   +/-          Top area
You (garry)              32   +2400/-300   browse/
alice                    12   +800/-150    app/services/
bob                       3   +120/-40     tests/
```

按 commits 降序排列。当前用户始终显示在最前面，标记为 "You (名字)"。

---

### Step 3：Commit 时间分布

以本地时间显示每小时直方图：

```
Hour  Commits  ████████████████
 00:    4      ████
 07:    5      █████
 ...
```

识别：
- 高峰时段
- 空白时段
- 双峰模式（上午/晚上）vs 持续模式
- 深夜编码集群（晚上 10 点后）

---

### Step 4：工作 Session 检测

使用**45 分钟间隔**阈值检测连续 commits 之间的 sessions。

分类 sessions：
- **Deep sessions**（50+ 分钟）
- **Medium sessions**（20-50 分钟）
- **Micro sessions**（<20 分钟，单次 commit）

计算：
- 总活跃编码时间
- 平均 session 长度
- 每活跃小时 LOC

---

### Step 5：Commit 类型分解

按 conventional commit 前缀分类（feat/fix/refactor/test/chore/docs）。以百分比条显示：

```
feat:     20  (40%)  ████████████████████
fix:      27  (54%)  ███████████████████████████
refactor:  2  ( 4%)  ██
```

如果 fix 比例超过 50% 则标记……这可能表明 review 存在缺口，呈现 "快速交付、快速修复" 模式。

---

### Step 6：Hotspot 分析

显示变更最多的前 10 个文件。标记：
- 变更 5 次以上的文件（churn hotspots）
- hotspot 列表中的测试文件 vs 生产文件
- VERSION/CHANGELOG 变更频率

---

### Step 7：PR 大小分布

估计 PR 大小并分类：
- **Small**（<100 LOC）
- **Medium**（100-500 LOC）
- **Large**（500-1500 LOC）
- **XL**（1500+ LOC）

---

### Step 8：专注度评分 + 本周最佳交付

**Focus score：** 触及单一变更最多的顶级目录的 commit 百分比。越高 = 工作越专注。越低 = 上下文切换越分散。

**Ship of the week：** 窗口内 LOC 变更最多的单个 PR。高亮 PR 编号、变更 LOC 数以及其重要性。

---

### Step 9：团队成员分析

对每位贡献者（包括当前用户），计算：

1. **Commits 和 LOC**……总 commits、insertions、deletions、net LOC
2. **专注领域**……他们最常触及的目录/文件（前 3 个）
3. **Commit 类型组合**……他们个人的 feat/fix/refactor/test 分解
4. **Session 模式**……他们什么时候编码（高峰时段）、session 数量
5. **测试纪律**……他们个人的 test LOC ratio
6. **最大交付**……他们影响最大的单个 commit 或 PR

**对于当前用户（"你"）：** 最深入的对待。包含所有 session 分析、时间模式、focus score。以第一人称叙述。

**对于每位队友：** 2-3 句，涵盖他们交付了什么以及他们的模式。然后：

- **表扬**（1-2 个具体事项）：锚定在实际 commits 上。不要说"干得好"……要准确说出哪里好。
- **成长机会**（1 个具体事项）：以提升水平的框架呈现，而非批评。锚定在实际数据上。

**如果是单人 repo：** 跳过团队分解。

**AI 协作：** 如果 commits 有 `Co-Authored-By` AI 尾注，将 "AI-assisted commits" 作为单独的指标追踪。

---

### Step 10：周环比趋势（如果窗口 >= 14d）

按周分组显示趋势：
- 每周 commits（总数和每人）
- 每周 LOC
- 每周 test ratio
- 每周 fix ratio
- 每周 session 数量

---

### Step 11：连续天数追踪

从今天开始倒推，计算至少有 1 个 commit 的连续天数：

```bash
# 团队连续天数
git log origin/main --format="%ad" --date=format:"%Y-%m-%d" | sort -u

# 个人连续天数
git log origin/main --author="<user_name>" --format="%ad" --date=format:"%Y-%m-%d" | sort -u
```

显示两者：
- "团队交付连续天数：47 天"
- "你的交付连续天数：32 天"

---

### Step 12：加载历史并与之前比较

在 `memory/` 中检查是否有之前的 retro 历史：

如果存在之前的 retro，加载最近的一次并计算差异：

```
                     上次        现在         差异
Test ratio:         22%    →    41%         ↑19pp
Sessions:           10     →    14          ↑4
LOC/hour:           200    →    350         ↑75%
Fix ratio:          54%    →    30%         ↓24pp（改善）
```

如果没有之前的 retro，标注 "首次记录 retro，下周再次运行以查看趋势。"

---

### Step 13：保存 Retro 历史

将 JSON 快照保存到 `memory/retro-YYYY-MM-DD.json`，包含指标、作者、版本范围、连续天数和可推特的摘要。

---

### Step 14：撰写叙事

**为 Telegram 格式化**（使用列表、粗体，最终输出中不使用 markdown 表格）。

结构：

**可推特的摘要**（第一行）：
> 3月1日周：47 commits（3 位贡献者），3.2k LOC，38% 测试，12 个 PR，高峰：晚上 10 点 | 连续：47 天

然后各节：

- **总结**……关键指标
- **对比上次 Retro 的趋势**……差异（如果是首次 retro 则跳过）
- **时间与 Session 模式**……团队什么时候编码、session 长度、deep vs micro
- **交付速度**……commit 类型、PR 大小、fix-chain 检测
- **代码质量信号**……test ratio、hotspots、churn
- **专注与亮点**……focus score、本周最佳交付
- **你的本周**……对当前用户的个人深度分析
- **团队分解**……每人分析，含表扬 + 成长（如果是单人则跳过）
- **团队前 3 胜利**……影响最大的交付事项
- **3 项可改进之处**……具体、可操作、锚定在 commits 上
- **下周 3 个习惯**……小、实用、现实（<5 分钟可采纳）

---

## 比较模式

当用户说"比较"时：
- 为当前窗口运行 retro
- 为之前相同长度的窗口运行 retro
- 并排展示指标，用箭头显示改善/回退
- 简要叙述最大变化

---

## 重要规则

- **所有时间使用本地时区。** 永远不要设置 `TZ`。
- **为 Telegram 格式化。** 使用列表和粗体。避免在最终输出中使用 markdown 表格。
- **表扬锚定在 commits 上。** 永远不要在不说出哪里好的情况下说"干得好"。
- **成长区域锚定在数据上。** 永远不要在无证据的情况下批评。
- **保存历史。** 每次 retro 都保存到 `memory/` 用于趋势追踪。
- **完成状态：**
  - DONE……retro 已生成，历史已保存
  - DONE_WITH_CONCERNS……已生成但缺少数据（例如没有之前的 retro 用于比较）
  - BLOCKED……不在 git repo 中或窗口内没有 commits
