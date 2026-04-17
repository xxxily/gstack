# Pre-Landing Review 清单

## 说明

审查 `git diff origin/main` 输出，查找以下问题。具体说明——引用 `file:line` 并建议修复。跳过没问题的部分。只标记真正的问题。

**两遍审查：**
- **第一遍（CRITICAL）：** 先运行 SQL & 数据安全、竞态条件、LLM 输出信任边界、Shell 注入和枚举完整性。最高严重程度。
- **第二遍（INFORMATIONAL）：** 运行以下剩余类别。较低严重程度但仍需处理。
- **specialist 类别（由并行子 agent 处理，不在此清单中）：** 测试缺口、死代码、魔数、条件副作用、性能与 Bundle 体积影响、加密与熵。参见 `review/specialists/`。

所有发现都通过 Fix-First Review 执行行动：明显的机械修复自动应用，真正模糊的问题批量合并为单个用户问题。

**输出格式：**

```
Pre-Landing Review: N issues (X critical, Y informational)

**AUTO-FIXED:**
- [file:line] 问题 → 已应用修复

**NEEDS INPUT:**
- [file:line] 问题描述
  Recommended fix: 建议的修复
```

如果没有发现问题：`Pre-Landing Review: No issues found.`

保持简洁。每个问题：一行描述问题，一行给出修复。不要前言、不要总结、不要"整体看起来不错"。

---

## 审查类别

### 第一遍 — CRITICAL

#### SQL & 数据安全
- SQL 中的字符串插值（即使值是 `.to_i`/`.to_f`——也要使用参数化查询（Rails：sanitize_sql_array/Arel；Node：prepared statements；Python：parameterized queries））
- TOCTOU 竞态：应先检查后设置的模式应使用原子 `WHERE` + `update_all`
- 绕过模型验证直接写入数据库（Rails：update_column；Django：QuerySet.update()；Prisma：原始查询）
- N+1 查询：循环/视图中使用的关联缺少预加载（Rails：.includes()；SQLAlchemy：joinedload()；Prisma：include）

#### 竞态条件与并发
- 没有唯一约束或捕获重复键错误并重试的读-检查-写（例如 `where(hash:).first` 然后 `save!`，未处理并发插入）
- find-or-create 没有唯一的 DB 索引——并发调用会创建重复项
- 状态转换不使用原子 `WHERE old_status = ? UPDATE SET new_status`——并发更新可能跳过或双重应用转换
- 不安全 HTML 渲染（Rails：.html_safe/raw()；React：dangerouslySetInnerHTML；Vue：v-html；Django：|safe/mark_safe）作用于由用户控制的数据（XSS）

#### LLM 输出信任边界
- LLM 生成的值（电子邮件、URL、名称）写入数据库或传递给邮件程序时未进行格式验证。在持久化之前添加轻量级守卫（`EMAIL_REGEXP`、`URI.parse`、`.strip`）。
- 结构化工具输出（数组、hash）在数据库写入之前未进行类型/形状检查。
- 获取 LLM 生成的 URL 时没有 allowlist——如果 URL 指向内部网络，存在 SSRF 风险（Python：`urllib.parse.urlparse` → 在 `requests.get`/`httpx.get` 之前将主机名与 blocklist 进行比较）
- LLM 输出存储到知识库或向量数据库时未进行清理——存在存储的 prompt 注入风险

#### Shell 注入（Python 特定）
- `subprocess.run()` / `subprocess.call()` / `subprocess.Popen()` 使用 `shell=True` 且命令字符串中使用 f-string/`.format()` 插值——改为使用参数数组
- `os.system()` 带有变量插值——替换为使用参数数组的 `subprocess.run()`
- `eval()` / `exec()` 作用于 LLM 生成的代码而没有沙箱

#### 枚举与值完整性
当 diff 引入新的枚举值、状态字符串、层级名称或类型常量时：
- **追踪每个消费者。** 读取（不要只是 grep——阅读）每个对该值进行 switch、过滤或显示的文件。如果任何消费者没有处理新值，标记它。常见遗漏：在前端下拉列表中添加了值，但后端模型/计算方法没有持久化它。
- **检查 allowlist/过滤数组。** 搜索包含兄弟值的数组或 `%w[]` 列表（例如，如果向层级添加"revise"，找到每个 `%w[quick lfg mega]` 并验证在需要的地方包含"revise"）。
- **检查 `case`/`if-elsif` 链。** 如果现有代码根据枚举分支，新值是否会落入错误的默认值？
方法：使用 Grep 查找所有对兄弟值的引用（例如 grep"lfg"或"mega"以找到所有层级消费者）。阅读每个匹配项。此步骤需要阅读 diff 之外的代码。

### 第二遍 — INFORMATIONAL

#### Async/Sync 混合（Python 特定）
- `async def` 端点中的同步 `subprocess.run()`、`open()`、`requests.get()`——阻塞事件循环。改用 `asyncio.to_thread()`、`aiofiles` 或 `httpx.AsyncClient`。
- 异步函数中的 `time.sleep()`——使用 `asyncio.sleep()`
- 异步上下文中的同步 DB 调用，未使用 `run_in_executor()` 包装

#### 列/字段名称安全
- 将 ORM 查询（`.select()`、`.eq()`、`.gte()`、`.order()`）中的列名与实际 DB schema 进行验证——错误的列名会静默返回空结果或抛出被吞没的错误
- 检查查询结果上的 `.get()` 调用使用的列名是否与实际选择的列名一致
- 在有 schema 文档时进行交叉引用

#### 死代码与一致性（仅限版本/changelog——其他项目由 maintainability specialist 处理）
- PR 标题与 VERSION/CHANGELOG 文件之间的版本不匹配
- CHANGELOG 条目不准确地描述变更（例如，"从 X 更改为 Y"，但 X 从未存在过）

#### LLM Prompt 问题
- prompt 中使用 0 索引列表（LLM 可靠地返回 1 索引）
- 列出可用工具/能力的 prompt 文本与实际连接的 `tool_classes`/`tools` 数组不匹配
- 多处声明的 word/token 限制可能漂移

#### 完整性缺口
- 完整版本花费不到 30 分钟 CC 时间的快捷实现（例如部分枚举处理、不完整的错误路径、缺少的边界情况，这些添加起来很简单）
- 仅提供人工团队工作量评估的选项——应同时显示人工和 CC+gstack 时间
- 测试覆盖率缺口，添加缺失的测试是"湖泊"而非"海洋"（例如缺少负面路径测试、缺少与正常路径结构匹配的边界情况测试）
- 以 80-90% 实现的功能，100% 实现只需要少量额外代码

#### 时间窗口安全
- 假设"今天"覆盖 24 小时的日期键查找——在 PT 时间 8am 报告时，今天键下只看到 midnight→8am
- 相关功能之间时间窗口不匹配——一个使用小时桶，另一个对同一数据使用日键

#### 边界处的类型强制
- 跨越 Ruby→JSON→JS 边界的值，类型可能发生变化（数值 vs 字符串）——hash/摘要输入必须规范化类型
- 序列化之前未调用 `.to_s` 或等效方法的 hash/摘要输入——`{ cores: 8 }` 和 `{ cores: "8" }` 产生不同的 hash

#### 视图/前端
- 部分中的内联 `<style>` 块（每次渲染都重新解析）
- 视图中的 O(n*m) 查找（循环中的 `Array#find` 而非 `index_by` hash）
- 对 DB 结果进行 Ruby 端的 `.select{}` 过滤，本可以用 `WHERE` 子句完成（除非有意避免使用前导通配符 `LIKE`）

#### 分发与 CI/CD Pipeline
- CI/CD 工作流变更（`.github/workflows/`）：验证构建工具版本是否符合项目要求、artifact 名称/路径是否正确、secret 使用 `${{ secrets.X }}` 而非硬编码值
- 新 artifact 类型（CLI 二进制文件、库、包）：验证存在发布/发布工作流并针对正确的平台
- 跨平台构建：验证 CI 矩阵覆盖所有目标 OS/arch 组合，或文档说明哪些未测试
- 版本标签格式一致性：`v1.2.3` vs `1.2.3`——必须在 VERSION 文件、git 标签和发布脚本之间保持一致
- 发布步骤幂等性：重新运行发布工作流不应失败（例如在 `gh release create` 之前先 `gh release delete`）

**不要标记：**
- 存在自动部署 Pipeline 的 Web 服务（Docker 构建 + K8s 部署）
- 不在团队外部使用的内部工具
- 仅测试 CI 变更（添加测试步骤，而非发布步骤）

---

## 严重程度分类

```
CRITICAL (最高严重程度):       INFORMATIONAL (主 agent):       SPECIALIST (并行子 agent):
├─ SQL & 数据安全              ├─ Async/Sync 混合              ├─ Testing specialist
├─ 竞态条件与并发              ├─ 列/字段名称安全              ├─ Maintainability specialist
├─ LLM 输出信任边界            ├─ 死代码（仅版本）             ├─ Security specialist
├─ Shell 注入                  ├─ LLM Prompt 问题              ├─ Performance specialist
└─ 枚举与值完整性              ├─ 完整性缺口                   ├─ Data Migration specialist
                               ├─ 时间窗口安全                 ├─ API Contract specialist
                               ├─ 边界处的类型强制             └─ Red Team (有条件)
                               ├─ 视图/前端
                               └─ 分发与 CI/CD Pipeline

所有发现都通过 Fix-First Review 执行行动。严重程度决定
展示顺序和 AUTO-FIX vs ASK 的分类——critical
发现倾向于 ASK（风险更大），informational 发现
倾向于 AUTO-FIX（更机械）。
```

---

## Fix-First 启发式

此启发式被 `/review` 和 `/ship` 共同引用。它决定 agent 是自动修复发现还是询问用户。

```
AUTO-FIX（agent 无需询问即可修复）:    ASK（需要人类判断）:
├─ 死代码/未使用的变量                ├─ 安全（auth、XSS、注入）
├─ N+1 查询（缺少预加载）              ├─ 竞态条件
├─ 与代码矛盾的过时注释                ├─ 设计决策
├─ 魔数 → 命名常量                     ├─ 大型修复（>20 行）
├─ 缺少 LLM 输出验证                 ├─ 枚举完整性
├─ 版本/路径不匹配                     ├─ 删除功能
├─ 赋值但从未读取的变量               └─ 更改用户可见行为的任何内容
└─ 内联样式、O(n*m) 视图查找
```

**经验法则：** 如果修复是机械性的，高级工程师会在不讨论的情况下应用它，那就是 AUTO-FIX。如果合理的工程师对修复可能有分歧，那就是 ASK。

**critical 发现默认倾向于 ASK**（它们本质上风险更大）。
**informational 发现默认倾向于 AUTO-FIX**（它们更机械）。

---

## 抑制项——不要标记这些

- 当冗余无害且有助于可读性时，"X 与 Y 冗余"（例如 `present?` 与 `length > 20` 冗余）
- "添加注释解释为什么选择此阈值/常量"——阈值在调整期间会变化，注释会过时
- "此断言可以更严格"，当断言已经覆盖了该行为时
- 仅建议一致性的更改（将值包装在条件中以匹配另一个常量的保护方式）
- "正则表达式不处理边界情况 X"，当输入受限且 X 在实践中从未发生时
- "测试同时练习多个守卫"——没问题，测试不需要隔离每个守卫
- 评估阈值更改（max_actionable、min 分数）——这些经过经验调整并不断变化
- 无害的空操作（例如对从不在数组中的元素执行 `.reject`）
- 你正在审查的 diff 中已解决的任何内容——在评论之前阅读完整的 diff
