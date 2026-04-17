# Performance Specialist 审核清单

范围：当 SCOPE_BACKEND=true 或 SCOPE_FRONTEND=true 时
输出：JSON 对象，每行一个 finding。Schema：
{"severity":"CRITICAL|INFORMATIONAL","confidence":N,"path":"file","line":N,"category":"performance","summary":"...","fix":"...","fingerprint":"path:line:performance","specialist":"performance"}
可选字段：line、fix、fingerprint、evidence、test_stub。
如果没有 finding：输出 `NO FINDINGS`，不要输出其他内容。

---

## 类别

### N+1 查询
- 在循环中遍历 ActiveRecord/ORM 关联，未使用预加载（.includes、joinedload、include）
- 迭代块（each、map、forEach）中的数据库查询，可以批量处理
- 触发延迟加载关联的嵌套序列化器
- 按字段查询而非批处理的 GraphQL 解析器（检查是否使用了 DataLoader）

### 缺少数据库索引
- 在没有索引的列上使用新的 WHERE 子句（检查迁移文件或 schema）
- 在未索引的列上使用新的 ORDER BY
- 复合查询（WHERE a AND b）没有复合索引
- 添加了外键列但没有索引

### 算法复杂度
- O(n^2) 或更差的模式：集合上的嵌套循环、Array.map 内的 Array.find
- 可以使用 hash/map/set 查找的重复线性搜索
- 循环中的字符串拼接（使用 join 或 StringBuilder）
- 对大型集合多次排序或过滤，而一次就足够

### Bundle 体积影响（前端）
- 已知较重的新生产依赖（moment.js、lodash 完整版、jquery）
- 桶导入（import from 'library'）而非深层导入（import from 'library/specific'）
- 提交的大型静态资源（图片、字体）未优化
- 路由级分块缺少代码分割

### 渲染性能（前端）
- Fetch 瀑布：可以并行的顺序 API 调用（Promise.all）
- 不稳定引用导致的不必要重新渲染（渲染中创建新的对象/数组）
- 在昂贵计算上缺少 React.memo、useMemo 或 useCallback
- 在循环中读取然后写入 DOM 属性导致的布局抖动
- 视口下方图片缺少 loading="lazy"

### 缺少分页
- 返回无界结果的列表端点（无 LIMIT、无分页参数）
- 没有 LIMIT 的数据库查询，随数据量增长而增长
- API 响应嵌入完整的嵌套对象，而不是带扩展能力的 ID

### 异步上下文中的阻塞
- 异步函数中的同步 I/O（文件读取、子进程、HTTP 请求）
- 基于事件循环的处理程序中的 time.sleep() / Thread.sleep()
- CPU 密集型计算阻塞主线程，未卸载到 worker
