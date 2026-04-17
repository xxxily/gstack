# API Contract Specialist 审核清单

范围：当 SCOPE_API=true 时
输出：JSON 对象，每行一个 finding。Schema：
{"severity":"CRITICAL|INFORMATIONAL","confidence":N,"path":"file","line":N,"category":"api-contract","summary":"...","fix":"...","fingerprint":"path:line:api-contract","specialist":"api-contract"}
可选字段：line、fix、fingerprint、evidence、test_stub。
如果没有 finding：输出 `NO FINDINGS`，不要输出其他内容。

---

## 类别

### 破坏性变更
- 从响应体中删除字段（客户端可能依赖它们）
- 更改字段类型（string → number、object → array）
- 向现有端点添加新的必填参数
- 更改 HTTP 方法（GET → POST）或状态码（200 → 201）
- 重命名端点时未将旧路径保持为重定向/别名
- 更改认证要求（公开 → 需要认证）

### 版本控制策略
- 进行破坏性变更时未进行版本升级（v1 → v2）
- 在同一 API 中混合使用多种版本控制策略（URL vs 头部 vs 查询参数）
- 弃用端点时未提供下线时间表或迁移指南
- 特定于版本的逻辑分散在控制器中，而非集中管理

### 错误响应一致性
- 新端点返回的错误格式与现有端点不同
- 错误响应缺少标准字段（错误码、消息、详情）
- 与错误类型不匹配的 HTTP 状态码（错误返回 200、验证失败返回 500）
- 泄漏内部实现细节的错误消息（堆栈跟踪、SQL）

### 速率限制与分页
- 缺少速率限制的新端点，而类似端点有速率限制
- 分页变更（offset → cursor）没有向后兼容性
- 更改页面大小或默认限制时未提供文档
- 分页响应中缺少总计数或下一页指示器

### 文档漂移
- OpenAPI/Swagger 规范未更新以匹配新端点或变更的参数
- 变更后描述旧行为的 README 或 API 文档
- 不再有效示例请求/响应
- 缺少新端点或变更参数的文档

### 向后兼容性
- 使用旧版本的客户端：它们会崩溃吗？
- 无法强制更新的移动应用：API 对它们仍然有效吗？
- webhook payload 变更时未通知订阅者
- 使用新功能是否需要更改 SDK 或客户端库
