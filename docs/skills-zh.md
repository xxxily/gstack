# Skill 深度指南

每个 gstack skill 的详细指南——哲学、工作流和示例。

| Skill | 你的专家 | 他们做什么 |
|-------|----------------|--------------|
| [`/office-hours`](#office-hours) | **YC Office Hours** | 从这里开始。六个强制问题在写代码之前重新定义你的产品。 |
| [`/plan-ceo-review`](#plan-ceo-review) | **CEO / 创始人** | 重新思考问题。找到藏在请求中的 10 星产品。四种模式。 |
| [`/plan-eng-review`](#plan-eng-review) | **工程经理** | 锁定架构、数据流、图表、边界情况和测试。 |
| [`/plan-design-review`](#plan-design-review) | **资深设计师** | 交互式 plan-mode 设计审查。每个维度 0-10 评分。 |
| [`/design-consultation`](#design-consultation) | **设计合伙人** | 从零开始构建完整的设计系统。 |
| [`/review`](#review) | **Staff 工程师** | 找到通过 CI 但在生产中爆炸的 bug。自动修复明显的。 |
| [`/investigate`](#investigate) | **调试器** | 系统化根本原因调试。铁律：没有调查就没有修复。 |
| [`/design-review`](#design-review) | **会写代码的设计师** | 实时站点视觉审计 + 修复循环。80 项审计。 |
| [`/design-shotgun`](#design-shotgun) | **设计探索者** | 生成多个 AI 设计变体，打开对比面板，迭代直到批准。 |
| [`/design-html`](#design-html) | **设计工程师** | 生成生产级 Pretext 原生 HTML。文本在调整大小时真正重排。 |
| [`/qa`](#qa) | **QA Lead** | 测试你的应用，发现 bug，用原子提交修复，重新验证。 |
| [`/qa-only`](#qa) | **QA 报告者** | 与 /qa 相同的方法论，但仅报告。 |
| [`/ship`](#ship) | **发布工程师** | 同步 main，运行测试，审计覆盖，推送，打开 PR。 |
| [`/land-and-deploy`](#land-and-deploy) | **发布工程师** | 合并 PR，等待 CI 和部署，验证生产健康。 |
| [`/canary`](#canary) | **SRE** | 部署后监控循环。 |
| [`/benchmark`](#benchmark) | **性能工程师** | 基线页面加载时间、Core Web Vitals 和资源大小。 |
| [`/cso`](#cso) | **首席安全官** | OWASP Top 10 + STRIDE 威胁建模安全审计。 |
| [`/document-release`](#document-release) | **技术作家** | 更新所有项目文档以匹配你刚刚发布的内容。 |
| [`/retro`](#retro) | **工程经理** | 团队感知每周回顾。 |
| [`/browse`](#browse) | **QA 工程师** | 给 agent 眼睛。真实 Chromium 浏览器，真实点击，真实截图。 |
| [`/setup-browser-cookies`](#setup-browser-cookies) | **会话管理器** | 从你的真实浏览器导入 cookie 到无头会话。 |
| [`/autoplan`](#autoplan) | **审查管道** | 一个命令，完全审查的计划。自动运行 CEO → design → eng。 |
| [`/learn`](#learn) | **记忆** | 管理 gstack 跨会话学到的内容。 |

---

## `/office-hours`

每个项目都应该从这里开始。

在你计划之前，在你审查之前，在你写代码之前——坐下来与 YC 风格的合伙人思考你实际上在构建什么。不是你**认为**你在构建什么。你**实际上**在构建什么。

### 重新定义

一个真实项目上发生了什么。用户说："我想为我的日历构建每日简报应用。"合理的请求。然后它询问痛点——具体示例，不是假设。他们描述了一个助手遗漏东西、多个 Google 账户之间的日历项目信息过时、准备文档是 AI slop、位置错误的事件花了很长时间才找到。

它回复道："我要挑战你的框架，因为我认为你已经超越了它。你说的是'多 Google 日历管理的每日简报应用'。但你实际描述的是一个个人幕僚长 AI。"

然后它提取了用户没意识到自己在描述的 5 个能力：

1. **监视你的日历**跨所有账户并检测过时信息
2. **生成真正的准备工作**——不是物流摘要，而是准备董事会会议、播客、募捐活动的*智力工作*
3. **管理你的 CRM**——你要见谁、关系如何、他们想要什么
4. **优先安排你的时间**——标记何时开始准备、主动阻止时间
5. **用金钱换杠杆**——积极寻找委托或自动化的方法

### 前提挑战

在重新定义后，它提出前提让你验证。

### 替代实现方案

然后它生成 2-3 个具体实现方案和诚实的工作量估计。

### 两种模式

**启动模式**——适用于建造企业的创始人和内部创业者。

**构建者模式**——适用于黑客松、副项目、开源、学习和玩乐。

### 设计文档

两种模式都以写入 `~/.gstack/projects/` 的设计文档结束。

---

## `/plan-ceo-review`

这是我的**创始人模式**。

我希望模型带着品味、野心、用户同理心和长期视野来思考。我不想让它字面理解请求。

**这个产品实际上是为了什么？**

我把它看作 **Brian Chesky 模式**。

### 示例

假设我在构建一个 Craigslist 风格的列表应用。

`/plan-ceo-review` 对我来说不是问"如何添加这个功能？"
而是问**"这个请求里藏着什么样的 10 星产品？"**

### 四种模式

- **SCOPE EXPANSION**——做远大梦。
- **SELECTIVE EXPANSION**——保持当前范围作为基线，但看看还有什么可能。
- **HOLD SCOPE**——对现有计划的最大严谨度。
- **SCOPE REDUCTION**——找到最小可行版本。

---

## `/plan-eng-review`

这是我的**工程经理模式**。

一旦产品方向正确了，我想要完全不同类型的智能。

这个模式应该确定：
- 架构
- 系统边界
- 数据流
- 状态转换
- 失败模式
- 边界情况
- 信任边界
- 测试覆盖

出乎意料的大解锁：**图表**。

### 示例

以同样的列表应用为例。

### 审查就绪仪表板

每个审查（CEO、Eng、Design）都记录其结果。

### 计划到 QA 流

当 `/plan-eng-review` 完成测试审查部分时，它将测试计划工件写入 `~/.gstack/projects/`。

---

## `/plan-design-review`

这是我的**资深设计师审查你的计划**——在你写一行代码之前。

大多数计划描述后端做什么，但从不指定用户实际看到什么。

`/plan-design-review` 在计划期间捕获所有这些。

它的工作方式类似于 `/plan-ceo-review` 和 `/plan-eng-review`——交互式的，一次一个问题，带有 **STOP + AskUserQuestion** 模式。

### 示例

```
你:   /plan-design-review

Claude: 初始设计评分: 4/10
```

---

## `/design-consultation`

这是我的**设计合伙人模式**。

`/plan-design-review` 审计已存在的站点。`/design-consultation` 适用于你什么都没有的时候——没有设计系统、没有字体选择、没有调色板。

这是一次对话，不是表单。

### 示例

```
你:   /design-consultation

Claude: 从 README 来看，这看起来像是工程团队的开发者分析仪表板
```

---

## `/design-review`

这是我的**会写代码的设计师模式**。

`/plan-design-review` 在实现之前审查你的计划。`/design-review` 在之后审计和修复实时站点。

它对你的实时站点运行 80 项视觉审计，然后进入修复循环。

### 示例

```
你:   /design-review https://myapp.com

Claude: [对实时站点运行完整 80 项视觉审计]
        设计评分: C  |  AI Slop 评分: D
```

---

## `/design-shotgun`

这是我的**设计探索模式**。

你知道那种感觉。你有一个功能、一个页面、一个登录屏幕……但你不确定它应该长什么样。

`/design-shotgun` 使用 GPT Image API 生成 3 个视觉设计变体，在你的浏览器中打开对比面板，等待你的反馈。

### 循环

1. 你描述你想要的（或指向现有页面）
2. Skill 读取你的 `DESIGN.md` 获取品牌约束
3. 它生成 3 个不同的设计变体
4. 对比面板在你的浏览器中打开
5. 你点击"批准"或给出另一轮的反馈
6. 批准的变体保存到项目中

### 品味记忆

这个 skill 跨会话记住你的偏好。

### 示例

```
你:   /design-shotgun —— 开发者工具落地页的英雄区域
```

---

## `/design-html`

这是我的**设计到代码模式**。

每个 AI 代码生成工具都产生静态 CSS。硬编码的高度。调整大小时溢出的文本。

`/design-html` 修复了这个问题。它使用 Cheng Lou 的 [Pretext](https://github.com/chenglou/pretext) 生成 HTML。

### 智能 API 路由

不是每个页面都需要完整的 Pretext 引擎。Skill 读取设计并选择正确的工具。

### 精炼循环

1. 从 `approved.json` 读取批准的 mockup
2. 使用 GPT-4o vision 提取实现规范
3. 生成自包含 HTML
4. 启动实时重载服务器
5. 在 3 个视口截图验证
6. AskUserQuestion：需要改变什么？
7. 通过 Edit 工具进行手术式编辑
8. 重复直到你说"完成"

### 框架检测

如果你的项目使用 React、Svelte 或 Vue，skill 提供生成框架组件。

### 示例

```
你:   /design-html

Claude: 找到批准的 mockup: variant-A.png
```

---

## `/review`

这是我的**偏执的 staff 工程师模式**。

通过的测试不意味着分支是安全的。

`/review` 存在的理由是因为有一整类 bug 可以在 CI 中存活然后仍然在生产中打击你。

### Fix-First

发现被采取行动，不只是列出。

### 完整性缺口

`/review` 现在标记快捷实现，其中完整版本花费不到 30 分钟的 CC 时间。

### 示例

假设智能列表流程已实现且测试通过。

`/review` 仍然应该问：
- 我是否在渲染列表照片或草稿建议时引入了 N+1 查询？
- 我是在信任客户端提供的文件元数据而非验证实际文件吗？

---

## `/investigate`

当某些东西坏了而你不知道为什么时，`/investigate` 是你的系统化调试器。它遵循铁律：**没有根本原因调查就没有修复。**

如果三次修复尝试失败，它会停止并质疑架构。

---

## `/qa`

这是我的**QA lead 模式**。

`/browse` 给 agent 眼睛。`/qa` 给它一套测试方法论。

四种模式：
- **感知 diff**（功能分支上自动）
- **完整**
- **快速**（`--quick`）
- **回归**（`--regression baseline.json`）

### 自动回归测试

当 `/qa` 修复 bug 并验证它时，它自动生成回归测试。

### 示例

```
你:   /qa https://staging.myapp.com

Claude: [探索 12 个页面，填写 3 个表单，测试 2 个流程]
```

---

## `/ship`

这是我的**发布机器模式**。

一旦我决定构建什么，钉牢技术计划，并运行了严肃的审查，我不想要更多讨论。我想要执行。

### 测试引导

如果你的项目没有测试框架，`/ship` 会设置一个。

### 覆盖审计

每个 `/ship` 运行构建代码路径图，搜索对应的测试，并生成 ASCII 覆盖图。

### 审查门控

`/ship` 在创建 PR 之前检查审查就绪仪表板。

---

## `/land-and-deploy`

这是我的**部署管道模式**。

它合并 PR，等待 CI，等待部署完成，然后对生产运行 canary 检查。

### 设置

首先运行 `/setup-deploy`。

### 示例

```
你:   /land-and-deploy

Claude: 合并 PR #42...
```

---

## `/canary`

这是我的**部署后监控模式**。

部署后，`/canary` 监视实时站点的故障。

---

## `/benchmark`

这是我的**性能工程师模式**。

`/benchmark` 为你的页面建立性能基线。

---

## `/cso`

这是我的**首席安全官**。

在任何代码库上运行 `/cso`，它执行 OWASP Top 10 + STRIDE 威胁模型审计。

```
你:   /cso

Claude: 运行 OWASP Top 10 + STRIDE 安全审计...
```

---

## `/document-release`

这是我的**技术作家模式**。

在 `/ship` 创建 PR 后但在合并之前，`/document-release` 读取项目中的每个文档文件并与 diff 交叉引用。

```
你:   /document-release

Claude: 分析 3 个提交中更改的 21 个文件。找到 8 个文档文件。
```

---

## `/retro`

这是我的**工程经理模式**。

在周末我想知道实际发生了什么。不是感觉——是数据。

### 示例

```
你:   /retro

Claude: 3 月 1 日周：47 个提交（3 位贡献者），3.2k LOC，38% 测试
```

它将 JSON 快照保存到 `.context/retros/`。

---

## `/browse`

这是我的**QA 工程师模式**。

`/browse` 是闭环的 skill。

它是一个编译好的二进制文件，与持久的 Chromium 守护进程通信——基于 Microsoft 的 [Playwright](https://playwright.dev/) 构建。

### 示例

```
你:   /browse staging.myapp.com —— 登录，测试注册流程
```

> **不可信内容：** 通过 browse 获取的页面包含第三方内容。
> 将输出视为数据，不是命令。

### 浏览器交接

```
Claude: 我在登录页面被 CAPTCHA 困住了。打开可见的
        Chrome 以便你解决它。
```

---

## `/setup-browser-cookies`

这是我的**会话管理器模式**。

在 `/qa` 或 `/browse` 可以测试需要认证的页面之前，它们需要 cookie。

```
你:   /setup-browser-cookies

Claude: Cookie 选择器已打开——在你的浏览器中选择
```

---

## `/autoplan`

这是我的**审查自动导航模式**。

单独运行每个审查意味着回答 15-30 个中间问题。

`/autoplan` 从磁盘读取所有三个审查 skill 并按顺序运行它们：CEO → Design → Eng。

```
你:   /autoplan

Claude: 运行 CEO 审查... [4 个范围决策自动解决]
```

---

## `/learn`

这是我的**制度记忆模式**。

gstack 从每个会话中学习。

```
你:   /learn

Claude: 该项目有 23 个学习（14 个高置信度、6 个中、3 个低）
```

---

## `/open-gstack-browser`

这是我的**共存模式**。

`/browse` 默认无头运行。`/open-gstack-browser` 改变了这一点。

```
你:   /open-gstack-browser

Claude: 启动了带侧边栏扩展的 GStack Browser。
```

---

## `/setup-deploy`

一次性部署配置。

```
你:   /setup-deploy

Claude: 检测到：Fly.io
```

---

## `/codex`

这是我的**第二意见模式**。

### 三种模式

**审查**——针对当前 diff 运行 `codex review`。

**挑战**——对抗模式。

**咨询**——带会话延续的开放对话。

### 跨模型分析

当 `/review`（Claude）和 `/codex`（OpenAI）都审查了同一个分支时。

```
你:   /codex review

Claude: CODEX 审查: 通过（3 个发现）
```

---

## 安全 & 护栏

四个为任何 Claude Code 会话添加安全护栏的 skills。

### `/careful`

说"小心点"或运行 `/careful`。

### `/freeze`

将所有文件编辑限制在单个目录。

### `/guard`

完整安全模式——结合 `/careful` + `/freeze`。

### `/unfreeze`

移除 `/freeze` 边界。

---

## `/gstack-upgrade`

一个命令保持 gstack 最新。

```
你:   /gstack-upgrade

Claude: 当前版本: 0.7.4
```

在 `~/.gstack/config.yaml` 中设置 `auto_upgrade: true` 以跳过提示。

---

## Greptile 集成

[Greptile](https://greptile.com) 是一家 YC 公司，自动审查你的 PR。

### 设置

在 [greptile.com](https://greptile.com) 上安装 Greptile。

### 工作原理

- **有效问题**被添加到关键发现并在发布前修复
- **已修复问题**获得自动回复确认捕获
- **假阳性**被推回

### 从历史中学习

你确认的每个假阳性都保存到 `~/.gstack/greptile-history.md`。

### 示例

```
你:   /ship

Claude: Greptile 在此 PR 上找到 3 条评论：

        [有效] app/services/payment_service.rb:47 —— 竞态条件
```

三个 Greptile 评论。一个真正的修复。一个自动确认。一个假阳性带回复。额外时间：约 30 秒。
