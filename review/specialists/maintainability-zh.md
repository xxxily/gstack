# Maintainability Specialist 审核清单

范围：始终开启（每次 review）
输出：JSON 对象，每行一个 finding。Schema：
{"severity":"INFORMATIONAL","confidence":N,"path":"file","line":N,"category":"maintainability","summary":"...","fix":"...","fingerprint":"path:line:maintainability","specialist":"maintainability"}
可选字段：line、fix、fingerprint、evidence、test_stub。
如果没有 finding：输出 `NO FINDINGS`，不要输出其他内容。

---

## 类别

### 死代码与未使用的导入
- 在已修改文件中赋值但从未读取的变量
- 定义但从未调用的函数/方法（使用 Grep 在整个 repo 中检查）
- 更改后不再被引用的导入/requires
- 被注释掉的代码块（要么删除，要么解释为什么存在）

### 魔数与字符串耦合
- 在逻辑中使用的裸数字字面量（阈值、限制、重试次数）——应该是命名常量
- 用作查询过滤器或条件的错误消息字符串
- 应该是配置的硬编码 URL、端口或主机名
- 跨多个文件重复出现的字面量值

### 过时的注释与文档字符串
- 在此 diff 中代码更改后描述旧行为的注释
- 引用已完成工作的 TODO/FIXME 注释
- 参数列表与当前函数签名不匹配的文档字符串
- 注释中不再匹配代码流的 ASCII 图

### DRY 违规
- 在 diff 中多次出现的相似代码块（3 行及以上）
- 复制粘贴模式，使用共享 helper 会更清晰
- 跨测试文件重复的配置或设置逻辑
- 重复的条件链，可以用查找表或 map 替代

### 条件副作用
- 根据条件分支的代码路径，但其中一个分支遗漏了副作用
- 声称发生了某个操作的日志消息，但该操作有条件地跳过了
- 状态转换中一个分支更新相关记录，另一个分支不更新
- 仅在正常路径触发的事件发射，遗漏了错误/边界路径

### 模块边界违规
- 深入另一个模块的内部实现（访问按约定为私有的方法）
- 应该通过 service/model 的控制器/视图中直接进行数据库查询
- 应该通过接口通信的组件之间的紧密耦合
