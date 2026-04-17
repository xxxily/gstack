# Data Migration Specialist 审核清单

范围：当 SCOPE_MIGRATIONS=true 时
输出：JSON 对象，每行一个 finding。Schema：
{"severity":"CRITICAL|INFORMATIONAL","confidence":N,"path":"file","line":N,"category":"data-migration","summary":"...","fix":"...","fingerprint":"path:line:data-migration","specialist":"data-migration"}
可选字段：line、fix、fingerprint、evidence、test_stub。
如果没有 finding：输出 `NO FINDINGS`，不要输出其他内容。

---

## 类别

### 可逆性
- 此迁移能否在不丢失数据的情况下回滚？
- 是否有对应的 down/rollback 迁移？
- 回滚是真正撤销了更改，还是只是空操作？
- 回滚是否会破坏当前应用程序代码？

### 数据丢失风险
- 删除仍包含数据的列（应先添加弃用期）
- 更改会截断数据的列类型（varchar(255) → varchar(50)）
- 删除表时未验证是否有代码引用它
- 重命名列时未更新所有引用（ORM、原始 SQL、视图）
- 对已有 NULL 值的列添加 NOT NULL 约束（需要先回填）

### 锁持续时间
- 对大型表执行 ALTER TABLE 时未使用 CONCURRENTLY（PostgreSQL）
- 对超过 10 万行的表添加索引时未使用 CONCURRENTLY
- 可以合并为一次锁获取的多个 ALTER TABLE 语句
- 在流量高峰时段获取排他锁的模式更改

### 回填策略
- 没有 DEFAULT 值的新 NOT NULL 列（需要在约束之前回填）
- 需要批量填充的带有计算默认值的新列
- 缺少针对现有记录的回填脚本或 rake task
- 一次性更新所有行而不是分批的回填（会锁定表）

### 索引创建
- 在生产表上 CREATE INDEX 时未使用 CONCURRENTLY
- 重复索引（新索引覆盖的列与现有索引相同）
- 新的外键列缺少索引
- 在不该用部分索引的地方使用了部分索引，或反之

### 多阶段安全
- 必须与应用程序代码按特定顺序部署的迁移
- 破坏当前运行代码的模式更改（先部署代码，然后迁移）
- 假设存在部署边界的迁移（旧代码 + 新模式 = 崩溃）
- 缺少在滚动部署期间处理混合新旧代码的 feature flag
