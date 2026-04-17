# Testing Specialist 审核清单

范围：始终开启（每次 review）
输出：JSON 对象，每行一个 finding。Schema：
{"severity":"CRITICAL|INFORMATIONAL","confidence":N,"path":"file","line":N,"category":"testing","summary":"...","fix":"...","fingerprint":"path:line:testing","specialist":"testing"}
可选字段：line、fix、fingerprint、evidence、test_stub。
如果没有 finding：输出 `NO FINDINGS`，不要输出其他内容。

---

## 类别

### 缺少负面路径测试
- 处理错误、拒绝或无效输入的新代码路径，没有对应的测试
- 未被测试的守卫子句和提前返回
- try/catch、rescue 或 error boundary 中的错误分支，没有失败路径测试
- 在代码中断言的权限/auth 检查，但未测试"拒绝"情况

### 缺少边界情况覆盖
- 边界值：零、负数、最大整数、空字符串、空数组、nil/null/undefined
- 单元素集合（循环中的差一错误）
- 面向用户输入中的 Unicode 和特殊字符
- 没有竞态条件测试的并发访问模式

### 测试隔离违规
- 测试之间共享可变状态（类变量、全局单例、未清理的 DB 记录）
- 依赖顺序的测试（按顺序通过，随机化后失败）
- 依赖系统时钟、时区或 locale 的测试
- 进行真实网络调用而不是使用 stub/mock 的测试

### 不稳定测试模式
- 依赖时间的断言（sleep、setTimeout、带有紧张超时时间的 waitFor）
- 对无序结果的排序断言（hash 键、Set 迭代、async 解析顺序）
- 依赖外部服务（API、数据库）且没有回退机制的测试
- 使用随机化测试数据但没有种子控制的测试

### 缺少安全执行测试
- 控制器中的 auth/authz 检查，没有测试"未授权"情况
- 速率限制逻辑，没有证明确实能阻止的测试
- 输入清理，没有针对恶意输入的测试
- CSRF/CORS 配置，没有集成测试

### 覆盖率缺口
- 新的公开方法/函数，测试覆盖为零
- 已修改的方法，现有测试只覆盖旧行为，不覆盖新分支
- 从多处调用的工具函数，仅通过间接方式测试
