# DX Hall of Fame 参考

仅读取当前 review pass 对应的部分。不要加载整个文件。

## Pass 1：入门体验

**黄金标准：**
- **Stripe**：7 行代码即可为卡片收费。登录时文档会自动预填充你的测试 API 密钥。Stripe Shell 在文档页面内运行 CLI。无需本地安装。
- **Vercel**：`git push` = 部署到全球 CDN 的实时站点，附带 HTTPS。每个 PR 都有预览 URL。一条 CLI 命令：`vercel`。
- **Clerk**：`<SignIn />`、`<SignUp />`、`<UserButton />`。3 个 JSX 组件，开箱即用的 auth，支持电子邮件、社交登录、MFA。
- **Supabase**：创建 Postgres 表，立即自动生成 REST API + Realtime + 自文档化文档。
- **Firebase**：`onSnapshot()`。3 行代码即可在客户端之间实现实时同步，内置离线持久化。
- **Twilio**：控制台中的虚拟电话。发送/接收短信无需购买号码、无需信用卡。结果：激活率提升 62%。

**反模式：**
- 在任何价值之前就要求邮箱验证（中断流程）
- 沙箱之前需要信用卡
- "选择你自己的冒险"式多路径（决策疲劳；一条黄金路径胜出）
- API 密钥隐藏在设置中（Stripe 将它们预填充到代码示例中）
- 没有语言切换的静态代码示例
- 文档站点与管理面板分离（上下文切换）

## Pass 2：API/CLI/SDK 设计

**黄金标准：**
- **Stripe 前缀 ID**：`ch_` 用于 charge、`cus_` 用于 customer。自文档化。不可能传递错误类型的 ID。
- **Stripe 可扩展对象**：默认返回 ID 字符串。`expand[]` 可内联获取完整对象。嵌套扩展最多 4 层。
- **Stripe 幂等键**：在变更操作时传递 `Idempotency-Key` 头部。安全重试。没有"我是否重复扣费了？"的焦虑。
- **Stripe API 版本控制**：首次调用将账户固定到当天的版本。通过 `Stripe-Version` 头部按请求测试新版本。
- **GitHub CLI**：自动检测终端 vs 管道。终端中人类可读，管道输出时以制表符分隔。`gh pr <tab>` 显示所有 PR 操作。
- **SwiftUI 渐进式披露**：`Button("Save") { save() }` 到完整自定义，每个层级使用相同的 API。
- **htmx**：HTML 属性替代 JS。总共 14KB。`hx-get="/search" hx-trigger="keyup changed delay:300ms"`。零构建步骤。
- **shadcn/ui**：将源代码复制到你的项目中。你拥有的每一行代码。没有依赖、没有版本冲突。

**反模式：**
- 多嘴 API：用户可见的一个操作需要 5 次调用
- 命名不一致：`/users`（复数）vs `/user/123`（单数）vs `/create-order`（URL 中的动词）
- 隐式失败：200 OK 但错误嵌套在响应体中
- 上帝端点：47 个参数组合，不同子集有不同的行为
- 需要阅读文档才能使用的 API：首次调用前需要 3 页文档 = 过于繁琐

## Pass 3：错误消息与调试

**错误质量的三个层级：**

**Tier 1，Elm（对话式编译器）：**
```
-- TYPE MISMATCH ---- src/Main.elm
I cannot do addition with String values like this one:
42|   "hello" + 1
      ^^^^^^^
Hint: To put strings together, use the (++) operator instead.
```
第一人称，完整句子，精确位置，建议修复，进一步阅读。

**Tier 2，Rust（带注释的源码）：**
```
error[E0308]: mismatched types
 --> src/main.rs:4:20
help: consider borrowing here
  |
4 |     let name: &str = &get_name();
  |                       +
```
错误代码链接到教程。主要 + 次要标签。帮助部分显示精确编辑。

**Tier 3，Stripe API（结构化带 doc_url）：**
```json
{"error":{"type":"invalid_request_error","code":"resource_missing","message":"No such customer: 'cus_nonexistent'","param":"customer","doc_url":"https://stripe.com/docs/error-codes/resource-missing"}}
```
五个字段，零歧义。

**公式：** 发生了什么 + 为什么 + 如何修复 + 在哪了解更多 + 导致问题的实际值。

**反模式：** TypeScript 将"你是不是想..."放在长错误链的最底部。最具操作性的信息应该最先出现。

## Pass 4：文档与学习

**黄金标准：**
- **Stripe 文档**：三列布局（导航/内容/实时代码）。登录时注入 API 密钥。语言切换器在所有页面间持久化。悬停高亮。Stripe Shell 用于浏览器内 API 调用。构建并开源了 Markdoc。功能在文档最终确定之前不会发布。文档贡献会影响绩效考核。
- 52% 的开发者因缺乏文档而受阻（Postman 2023）
- 拥有世界级文档的公司采用率提升 2.5 倍
- "文档即产品"：与功能一起发布，否则功能不发布

## Pass 5：升级与迁移路径

**黄金标准：**
- **Next.js**：`npx @next/codemod upgrade major`。一条命令升级 Next.js、React、React DOM，运行所有相关 codemod。
- **AG Grid**：从 v31+ 开始的每个版本都包含一个 codemod。
- **Stripe API 版本控制**：内部一个代码库。按账户进行版本固定。破坏性变更永远不会让你意外。
- **Martin Fowler 的 Pipeline 模式**：组合小型、可测试的转换，而非单一单体 codemod。
- Maven Central 中 21.9% 的破坏性变更没有文档记录（Ochoa 等，2021）

## Pass 6：开发者环境与工具

**黄金标准：**
- **Bun**：比 npm install 快 100 倍，比 Node.js 运行时快 4 倍。速度就是 DX。
- 平均每天 87 次中断；每次需要 25 分钟恢复。开发者每天只编码 2-4 小时。
- DXI 每改善 1 分 = 每位开发者每周节省 13 分钟。
- **GitHub Copilot**：任务完成速度提升 55.8%。PR 时间从 9.6 天缩短到 2.4 天。

## Pass 7：社区与生态

- 开发者工具需要约 14 次曝光才会购买（Matt Biilmann，Netlify）。与季度 OKR 周期不兼容。
- 拥有强大开发者体验的团队有 4-5 倍的性能乘数（DevEx 框架）。

## Pass 8：DX 测量

**三个学术框架：**
1. **SPACE**（Microsoft Research，2021）：满意度、性能、活动、沟通、效率。至少测量 3 个维度。
2. **DevEx**（ACM Queue，2023）：反馈循环、认知负载、心流状态。结合感知 + 工作流数据。
3. **Fagerholm & Munch**（IEEE，2012）：认知、情感、动机。心理"三部曲"。

## Claude Code Skill DX 清单

在 review Claude Code skill、MCP 服务器或 AI agent 工具的计划时使用。

- [ ] **AskUserQuestion 设计**：每次调用一个问题。重新锚定上下文（项目、分支、任务）。浏览器移交用于视觉反馈。
- [ ] **状态存储**：全局（~/.tool/）vs 每个项目（$SLUG/）vs 每个会话。追加模式的 JSONL 用于审计追踪。
- [ ] **渐进式同意**：带标记文件的一次性提示。不再重复询问。可逆。
- [ ] **自动升级**：带缓存 + 延后回退的版本检查。迁移脚本。内联提供。
- [ ] **Skill 组合**：收益链。review 链。内联调用并跳过部分。
- [ ] **错误恢复**：从故障恢复。保留部分结果。检查点安全。
- [ ] **会话连续性**：时间线事件。压缩恢复。跨会话学习。
- [ ] **有界自主性**：明确的运行限制。对破坏性操作强制升级。审计追踪。

参考实现：gstack 的 design-shotgun 循环、自动升级流程、渐进式同意、分层存储。
