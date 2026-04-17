# Security Specialist 审核清单

范围：当 SCOPE_AUTH=true 或（SCOPE_BACKEND=true 且 diff > 100 行）时
输出：JSON 对象，每行一个 finding。Schema：
{"severity":"CRITICAL|INFORMATIONAL","confidence":N,"path":"file","line":N,"category":"security","summary":"...","fix":"...","fingerprint":"path:line:security","specialist":"security"}
可选字段：line、fix、fingerprint、evidence、test_stub。
如果没有 finding：输出 `NO FINDINGS`，不要输出其他内容。

---

本清单比主 CRITICAL 检查更深入。主 agent 已经检查了 SQL 注入、竞态条件、LLM 信任和枚举完整性。本 specialist 专注于 auth/authz 模式、加密误用和攻击面扩展。

## 类别

### 信任边界的输入验证
- 在控制器/处理程序级别接受用户输入而未进行验证
- 查询参数直接用于数据库查询或文件路径
- 接受请求体字段而未进行类型检查或 schema 验证
- 文件上传没有类型/大小/内容验证
- 处理 webhook payload 时未进行签名验证

### 认证与授权绕过
- 端点缺少认证中间件（检查路由定义）
- 默认"允许"而非"拒绝"的授权检查
- 角色升级路径（用户可以修改自己的角色/权限）
- 直接对象引用漏洞（用户 A 通过更改 ID 访问用户 B 的数据）
- 会话固定或会话劫持机会
- Token/API 密钥验证未检查过期时间

### 注入向量（SQL 之外）
- 通过子进程调用进行命令注入，参数由用户控制
- 模板注入（Jinja2、ERB、Handlebars），带有用户输入
- 目录查询中的 LDAP 注入
- 通过用户控制的 URL 进行 SSRF（fetch、重定向、webhook 目标）
- 通过用户控制的文件路径进行路径遍历（../../etc/passwd）
- 通过用户控制的值进行 HTTP 头部注入

### 加密误用
- 安全敏感操作使用弱哈希算法（MD5、SHA1）
- 用于 token 或密钥的可预测随机数（Math.random、rand()）
- 对密钥、token 或摘要使用非常量时间比较（==）
- 硬编码加密密钥或 IV
- 密码哈希缺少盐值

### 密钥暴露
- 源代码中的 API 密钥、token 或密码（即使在注释中）
- 应用程序日志或错误消息中记录的密钥
- URL 中的凭据（查询参数或 URL 中的 basic auth）
- 返回给用户的错误响应中的敏感数据
- 预期加密时以明文存储 PII

### 通过逃逸途径的 XSS
- Rails：对由用户控制的数据使用 .html_safe、raw()
- React：dangerouslySetInnerHTML 带有用户内容
- Vue：v-html 带有用户内容
- Django：对用户输入使用 |safe、mark_safe()
- 通用：使用未清理数据赋值 innerHTML

### 反序列化
- 反序列化不可信数据（pickle、Marshal、YAML.load、可执行类型的 JSON.parse）
- 接受来自用户输入或外部 API 的序列化对象，未进行 schema 验证
