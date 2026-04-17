# TODOS

## 侧边栏安全

### 机器学习提示注入分类器

**做什么：** 通过 @huggingface/transformers v4（WASM 后端）添加 DeBERTa-v3-base-prompt-injection-v2，作为 Chrome 侧边栏的 ML 防御层。可复用的 `browse/src/security.ts` 模块，包含 `checkInjection()` API。包括金丝雀令牌、攻击日志、盾牌图标、特殊遥测（检测时即使遥测关闭也会触发 AskUserQuestion），以及 BrowseSafe-bench 红队测试工具（3,680 个对抗性用例源自 Perplexity）。

**为什么：** PR 1 修复了架构（命令白名单、XML 框架、参数修复）。但攻击者仍然可以欺骗 Claude 导航到钓鱼网站或通过允许的 browse 命令泄露可见页面数据。ML 分类器能捕获架构控制无法察觉的提示注入模式。94.8% 准确率，99.6% 召回率，通过 WASM 约 50-100ms 推理。纵深防御。

**背景：** 完整设计文档包含行业研究、开源工具格局、Codex 审查发现和雄心勃勃的 Bun 原生愿景（通过 FFI + Apple Accelerate 实现 5ms 推理）：[`docs/designs/ML_PROMPT_INJECTION_KILLER.md`](docs/designs/ML_PROMPT_INJECTION_KILLER.md)。CEO 计划包含范围决策：`~/.gstack/projects/garrytan-gstack/ceo-plans/2026-03-28-sidebar-prompt-injection-defense.md`。

**工作量：** L（人类：约 2 周 / CC：约 3-4 小时）
**优先级：** P0
**依赖：** 侧边栏安全修复 PR（命令白名单 + XML 框架 + 参数修复）必须先合入

## 构建者精神

### 首次"先搜索再构建"引导

**做什么：** 添加 `generateSearchIntro()` 函数（类似 `generateLakeIntro()`），在首次使用时介绍"先搜索再构建"原则，附带博客文章链接。

**为什么：** "煮沸海洋"有一个引导流程，链接到文章并标记 `.completeness-intro-seen`。"先搜索再构建"应该有相同的模式以提高可发现性。

**背景：** 阻塞于待写的博客文章。文章写好后，添加带有 `.search-intro-seen` 标记文件的引导流程。参考模式：`generateLakeIntro()` 位于 gen-skill-docs.ts:176。

**工作量：** S
**优先级：** P2
**依赖：** 关于"先搜索再构建"的博客文章

## Chrome DevTools MCP 集成

### 真实 Chrome 会话访问

**做什么：** 集成 Chrome DevTools MCP，连接到用户真实的 Chrome 会话，带有真实 Cookie、真实状态，无需 Playwright 中间层。

**为什么：** 目前，有头模式启动一个全新的 Chromium 配置文件。用户必须手动登录或导入 Cookie。Chrome DevTools MCP 连接到用户实际的 Chrome——即时访问所有已认证站点。这是 AI Agent 浏览器自动化的未来。

**背景：** Google 在 Chrome 146+（2025 年 6 月）发布了 Chrome DevTools MCP。它提供截屏、控制台消息、性能追踪、Lighthouse 审计，以及通过用户真实浏览器的完整页面交互。gstack 应该用它来获取真实会话访问，同时保留 Playwright 用于无头 CI/测试工作流。

潜在新 Skill：
- `/debug-browser`：带源码映射堆栈跟踪的 JS 错误追踪
- `/perf-debug`：性能追踪、Core Web Vitals、网络瀑布图

可能对大多数用例替代 `/setup-browser-cookies`，因为用户的真实 Cookie 已经在那里了。

**工作量：** L（人类：约 2 周 / CC：约 2 小时）
**优先级：** P0
**依赖：** Chrome 146+、DevTools MCP 服务器已安装

## Browse

### 将 server.ts 打包到编译后的二进制文件中

**做什么：** 完全消除 `resolveServerScript()` 回退链——将 server.ts 打包到编译后的 browse 二进制文件中。

**为什么：** 当前的回退链（检查 cli.ts 相邻位置，检查全局安装）很脆弱，在 v0.3.2 中引发了 Bug。单个编译后的二进制文件更简单、更可靠。

**背景：** Bun 的 `--compile` 标志可以打包多个入口点。服务器当前在运行时通过文件路径查找来解析。打包后完全消除了这个解析步骤。

**工作量：** M
**优先级：** P2
**依赖：** 无

### 会话（隔离的浏览器实例）

**做什么：** 隔离的浏览器实例，具有独立的 Cookie/存储/历史记录，可按名称寻址。

**为什么：** 启用不同用户角色的并行测试、A/B 测试验证和干净的认证状态管理。

**背景：** 需要 Playwright 浏览器上下文隔离。每个会话获得自己的上下文，具有独立的 Cookie/localStorage。是视频录制（干净的上下文生命周期）和认证库的前提条件。

**工作量：** L
**优先级：** P3

### 视频录制

**做什么：** 将浏览器交互录制为视频（开始/停止控制）。

**为什么：** QA 报告和 PR 正文中的视频证据。目前被推迟是因为 `recreateContext()` 会销毁页面状态。

**背景：** 需要会话来实现干净的上下文生命周期。Playwright 支持每个上下文的视频录制。还需要 WebM → GIF 转换以便嵌入 PR。

**工作量：** M
**优先级：** P3
**依赖：** 会话功能

### v20 加密格式支持

**做什么：** AES-256-GCM 支持，用于未来 Chromium Cookie DB 版本（当前为 v10）。

**为什么：** 未来的 Chromium 版本可能会改变加密格式。前瞻性支持可防止未来崩溃。

**工作量：** S
**优先级：** P3

### 状态持久化 — 已发布

~~**做什么：** 将 Cookie + localStorage 保存到 JSON 文件中，用于可重现的测试会话。~~

`$B state save/load` 已在 v0.12.1.0 发布。V1 仅保存 Cookie + URL（不保存 localStorage，这会在导航之前加载时崩溃）。文件位于 `.gstack/browse-states/{name}.json`，权限 0o600。加载将替换当前会话（先关闭所有页面）。名称清理为 `[a-zA-Z0-9_-]`。

**剩余：** V2 localStorage 支持（需要导航前注入策略）。
**完成：** v0.12.1.0 (2026-03-26)

### 认证库

**做什么：** 加密凭据存储，按名称引用。LLM 永远看不到密码。

**为什么：** 安全性——目前认证凭据流经 LLM 上下文。库将秘密信息排除在 AI 视野之外。

**工作量：** L
**优先级：** P3
**依赖：** 会话功能、状态持久化

### Iframe 支持 — 已发布

~~**做什么：** `frame <sel>` 和 `frame main` 命令用于跨框架交互。~~

`$B frame` 已在 v0.12.1.0 发布。支持 CSS 选择器、@ref、`--name` 和 `--url` 模式匹配。所有读/写/快照命令中的执行目标抽象化（`getActiveFrameOrPage()`）。退出上下文在导航、标签切换、恢复时清除。分离帧自动恢复。仅页面操作（goto、screenshot、viewport）在帧上下文中时抛出明确错误。

**完成：** v0.12.1.0 (2026-03-26)

### 语义定位器

**做什么：** `find role/label/text/placeholder/testid` 附带关联操作。

**为什么：** 比 CSS 选择器或 ref 编号更有弹性的元素选择。

**工作量：** M
**优先级：** P4

### 设备模拟预设

**做什么：** `set device "iPhone 16 Pro"` 用于移动/平板测试。

**为什么：** 响应式布局测试无需手动调整视口大小。

**工作量：** S
**优先级：** P4

### 网络模拟/路由

**做什么：** 拦截、阻止和模拟网络请求。

**为什么：** 测试错误状态、加载状态和离线行为。

**工作量：** M
**优先级：** P4

### 下载处理

**做什么：** 点击后下载并控制路径。

**为什么：** 端到端测试文件下载流程。

**工作量：** S
**优先级：** P4

### 内容安全

**做什么：** `--max-output` 截断，`--allowed-domains` 过滤。

**为什么：** 防止上下文窗口溢出并将导航限制在安全域名。

**工作量：** S
**优先级：** P4

### 流式传输（WebSocket 实时预览）

**做什么：** 基于 WebSocket 的实时预览，用于结对浏览会话。

**为什么：** 实现实时协作——人类可以观察 AI 浏览。

**工作量：** L
**优先级：** P4

### 有头模式 + Chrome 扩展 — 已发布

`$B connect` 在有头模式下启动 Playwright 捆绑的 Chromium，并自动加载 gstack Chrome 扩展。`$B handoff` 现在产生相同的结果（扩展 + 侧边栏）。侧边栏聊天限定在 `--chat` 标志之后。

### `$B watch` — 已发布

Claude 以被动只读模式观察用户浏览行为并定期截取快照。`$B watch stop` 退出并携带摘要。在观察期间阻止变更命令。

### 侧边栏侦察/文件投送中继 — 已发布

侧边栏 Agent 将结构化消息写入 `.context/sidebar-inbox/`。工作区 Agent 通过 `$B inbox` 读取。消息格式：`{type, timestamp, page, userMessage, sidebarSessionId}`。

### 多 Agent 标签隔离

**做什么：** 两个 Claude 会话连接到同一个浏览器，每个在不同的标签上操作。无交叉污染。

**为什么：** 启用在同一个浏览器中对不同标签并行执行 /qa + /design-review。

**背景：** 需要并发有头连接的标签所有权模型。Playwright 可能无法干净地支持两个持久上下文。需要调研。

**工作量：** L（人类：约 2 周 / CC：约 2 小时）
**优先级：** P3
**依赖：** 有头模式（已发布）

### 侧边栏 Agent 需要 Write 工具 + 更好的错误可见性 — 已发布

**做什么：** 侧边栏 Agent（`sidebar-agent.ts`）的两个问题：（1）`--allowedTools` 硬编码为 `Bash,Read,Glob,Grep`，缺少 `Write`。Claude 在被要求时无法创建文件（如 CSV）。（2）当 Claude 出错或返回空时，侧边栏 UI 什么都不显示，只有一个绿点。没有错误消息，没有"我尝试了但失败了"，什么都没有。

**完成：** v0.15.4.0 (2026-04-04)。Write 工具已添加到 allowedTools。40+ 空 catch 块被替换为带 `[gstack sidebar]`、`[gstack bg]`、`[browse]`、`[sidebar-agent]` 前缀的控制台日志记录，覆盖全部 4 个文件（sidepanel.js、background.js、server.ts、sidebar-agent.ts）。错误占位符文本现在以红色显示。认证令牌过期刷新 Bug 已修复。

### 侧边栏直接 API 调用（消除 claude -p 启动代价）

**做什么：** 每条侧边栏消息都启动一个全新的 `claude -p` 进程（约 2-3 秒冷启动开销）。对于"点击 @e24"这样的操作来说这太荒谬了。直接调用 Anthropic API 将在亚秒级完成。

**为什么：** `claude -p` 的启动代价是：进程启动（约 100ms）+ CLI 初始化（约 500ms-1s）+ API 连接（约 200ms）+ 首个 token。模型路由（Sonnet 用于操作）有帮助但不能解决 CLI 开销。

**背景：** `server.ts:spawnClaude()` 构建参数并写入队列文件。`sidebar-agent.ts:askClaude()` 启动 `claude -p`。替换为直接 `fetch('https://api.anthropic.com/...')` 附带工具使用。需要 browse 服务器可以访问 `ANTHROPIC_API_KEY`。

**工作量：** M（人类：约 1 周 / CC：约 30 分钟）
**优先级：** P2
**依赖：** 无

### Chrome Web Store 发布

**做什么：** 将 gstack browse Chrome 扩展发布到 Chrome Web Store 以便更方便安装。

**为什么：** 目前通过 chrome://extensions 侧载。Web Store 让安装变得一键完成。

**工作量：** S
**优先级：** P4
**依赖：** Chrome 扩展通过侧载证明其价值

### Linux Cookie 解密 — 部分已发布

~~**做什么：** GNOME Keyring / kwallet / DPAPI 支持，用于非 macOS 的 Cookie 导入。~~

Linux Cookie 导入已在 v0.11.11.0（Wave 3）发布。支持 Linux 上的 Chrome、Chromium、Brave、Edge，使用 GNOME Keyring (libsecret) 和 "peanuts" 回退方案。Windows DPAPI 支持仍然搁置。

**剩余：** Windows Cookie 解密（DPAPI）。需要完全重写——PR #64 是 1346 行且已过时。

**工作量：** L（仅 Windows）
**优先级：** P4
**已完成（Linux）：** v0.11.11.0 (2026-03-23)

## Ship

### GitLab 支持用于 /land-and-deploy

**做什么：** 为 `/land-and-deploy` Skill 添加 GitLab MR 合并 + CI 轮询支持。目前在 15+ 个地方使用 `gh pr view`、`gh pr checks`、`gh pr merge` 和 `gh run list/view`——每个都需要使用 `glab ci status`、`glab mr merge` 等的 GitLab 条件路径。

**为什么：** 没有这个功能，GitLab 用户可以 `/ship`（创建 MR）但无法 `/land-and-deploy`（合并 + 验证）。完成端到端的 GitLab 故事。

**背景：** `/retro`、`/ship` 和 `/document-release` 现在通过多平台 `BASE_BRANCH_DETECT` 解析器支持 GitLab。`/land-and-deploy` 有更深入的 GitHub 专属语义（合并队列、通过 `gh pr checks` 的必需检查、部署工作流轮询），在 GitLab 上形态不同。`glab` CLI (v1.90.0) 支持 `glab mr merge`、`glab ci status`、`glab ci view`，但输出格式不同，且没有合并队列概念。

**工作量：** L
**优先级：** P2
**依赖：** 无（BASE_BRANCH_DETECT 多平台解析器已完成）

### 多提交 CHANGELOG 完整度评估

**做什么：** 添加一个周期性 E2E 评估，创建一个包含 5+ 次提交、跨越 3+ 个主题（功能、清理、基础设施）的分支，运行 /ship 的第 5 步 CHANGELOG 生成，并验证 CHANGELOG 涵盖了所有主题。

**为什么：** v0.11.22 中修复的 Bug（garrytan/ship-full-commit-coverage）表明，在长分支上 /ship 的 CHANGELOG 生成偏向最近的提交。提示修复添加了交叉检查，但没有测试覆盖多提交失败模式。现有的 `ship-local-workflow` E2E 测试仅使用单提交分支。

**背景：** 将是一个 `periodic` 级别的测试（约 $4/次，非确定性，因为它测试 LLM 指令遵循）。设置：创建裸远端、克隆、在功能分支上跨不同主题添加 5+ 提交、通过 `claude -p` 运行第 5 步、验证 CHANGELOG 输出覆盖所有主题。模式：`test/skill-e2e-workflow.test.ts` 中的 `ship-local-workflow`。

**工作量：** M
**优先级：** P3
**依赖：** 无

### Ship 日志 — /ship 运行的持久记录

**做什么：** 在每次 /ship 运行结束时，将结构化的 JSON 条目追加到 `.gstack/ship-log.json`（版本、日期、分支、PR URL、审查发现、Greptile 统计、已完成的 TODOS、测试结果）。

**为什么：** /retro 缺乏关于发布速度的结构化数据。Ship 日志支持：每周 PR 趋势、审查发现率、Greptile 信号随时间变化、测试套件增长。

**背景：** /retro 已经读取 greptile-history.md——相同模式。评估持久化（eval-store.ts）显示了 JSON 追加模式在代码库中已存在。ship 模板中约 15 行。

**工作量：** S
**优先级：** P2
**依赖：** 无

### PR 正文中的截屏验证

**做什么：** /ship 第 7.5 步：推送后对关键页面进行截屏，嵌入 PR 正文。

**为什么：** PR 中的视觉证据。审查者无需本地部署就能看到变化。

**背景：** 第 3.6 阶段的一部分。需要 S3 上传用于图片托管。

**工作量：** M
**优先级：** P2
**依赖：** /setup-gstack-upload

## Review

### 内联 PR 批注

**做什么：** /ship 和 /review 使用 `gh api` 创建 PR 审查批注，在特定的 file:line 位置发布内联审查评论。

**为什么：** 行级批注比顶层评论更有可操作性。PR 线程变成 Greptile、Claude 和人类审查者之间的逐行对话。

**背景：** GitHub 通过 `gh api repos/$REPO/pulls/$PR/reviews` 支持内联审查批注。自然地与第 3.6 阶段视觉批注配对。

**工作量：** S
**优先级：** P2
**依赖：** 无

### Greptile 训练反馈导出

**做什么：** 将 greptile-history.md 聚合为机器可读的 JSON 摘要，包含误报模式，可导出给 Greptile 团队用于模型改进。

**为什么：** 形成反馈闭环——Greptile 可以使用误报数据停止在你的代码库上犯同样的错误。

**背景：** 之前是 P3 未来想法。升级为 P2，因为现在有了 greptile-history.md 数据基础设施。信号数据已经在收集中；这只是让它可导出。约 40 行。

**工作量：** S
**优先级：** P2
**依赖：** 积累足够的误报数据（10+ 条目）

### 使用带标注截屏的视觉审查

**做什么：** /review 第 4.5 步：浏览 PR 的预览部署，使用带标注的截屏、与生产环境对比、检查响应式布局、验证可访问性树。

**为什么：** 视觉差异可以捕获代码审查遗漏的布局回归。

**背景：** 第 3.6 阶段的一部分。需要 S3 上传用于图片托管。

**工作量：** M
**优先级：** P2
**依赖：** /setup-gstack-upload

## QA

### QA 趋势追踪

**做什么：** 对 baseline.json 进行跨时间比较，检测 QA 运行之间的回归。

**为什么：** 发现质量趋势——应用在变好还是变差？

**背景：** QA 已经能写入结构化报告。这增加了跨运行比较。

**工作量：** S
**优先级：** P2

### CI/CD QA 集成

**做什么：** `/qa` 作为 GitHub Action 步骤，如果健康分数下降则使 PR 失败。

**为什么：** CI 中的自动化质量门。合并前捕获回归。

**工作量：** M
**优先级：** P2

### 智能默认 QA 级别

**做什么：** 运行几次后，检查 index.md 中用户通常选择的级别，跳过 AskUserQuestion。

**为什么：** 减少重复用户的摩擦。

**工作量：** S
**优先级：** P2

### 可访问性审计模式

**做什么：** 用于集中可访问性测试的 `--a11y` 标志。

**为什么：** 超越一般 QA 检查清单的专用可访问性测试。

**工作量：** S
**优先级：** P3

### 为 GitHub 以外的提供商生成 CI/CD

**做什么：** 将 CI/CD 引导扩展为生成 GitLab CI（`.gitlab-ci.yml`）、CircleCI（`.circleci/config.yml`）和 Bitrise 流水线。

**为什么：** 不是所有项目都使用 GitHub Actions。通用 CI/CD 引导将使测试构建对所有人都可用。

**背景：** V1 仅支持 GitHub Actions。检测逻辑已经检查 `.gitlab-ci.yml`、`.circleci/`、`bitrise.yml` 并跳过并附带信息性注释。每个提供商需要在 `generateTestBootstrap()` 中编写约 20 行模板文本。

**工作量：** M
**优先级：** P3
**依赖：** 测试引导（已发布）

### 自动升级弱测试（★）为强测试（★★★）

**做什么：** 当第 3.4 步覆盖率审计识别出现有的★评级测试（冒烟/琐碎断言）时，生成改进版本，测试边界情况和错误路径。

**为什么：** 许多代码库有技术上存在但无法捕获真实 Bug 的测试——`expect(component).toBeDefined()` 并没有测试行为。升级这些测试缩小了"有测试"和"有好测试"之间的差距。

**背景：** 需要测试覆盖率审计的质量评分规则。修改现有测试文件比创建新文件风险更大——需要仔细比对以确保升级后的测试仍然通过。考虑创建配套测试文件而不是修改原文件。

**工作量：** M
**优先级：** P3
**依赖：** 测试质量评分（已发布）

## Retro

### 部署健康追踪（retro + browse）

**做什么：** 截屏生产环境状态、检查性能指标（页面加载时间）、统计关键页面的控制台错误、在 retro 窗口内追踪趋势。

**为什么：** Retro 除了代码指标之外还应包括生产健康度。

**背景：** 需要 browse 集成。截屏 + 指标输入到 retro 输出中。

**工作量：** L
**优先级：** P3
**依赖：** Browse 会话

## 基础设施

### /setup-gstack-upload Skill（S3 存储桶）

**做什么：** 配置 S3 存储桶用于图片托管。视觉 PR 批注的一次性设置。

**为什么：** 是 /ship 和 /review 中视觉 PR 批注的前提条件。

**工作量：** M
**优先级：** P2

### gstack-upload 辅助工具

**做什么：** `browse/bin/gstack-upload` — 上传文件到 S3，返回公开 URL。

**为什么：** 所有需要在 PR 中嵌入图片的 Skill 通用的实用工具。

**工作量：** S
**优先级：** P2
**依赖：** /setup-gstack-upload

### WebM 转 GIF 转换

**做什么：** 基于 ffmpeg 的 WebM → GIF 转换，用于 PR 中的视频证据。

**工作量：** S
**优先级：** P3
**依赖：** 视频录制

### 将 worktree 隔离扩展到 Claude E2E 测试

**做什么：** 为 `runSkillTest()` 添加 `useWorktree?: boolean` 选项，使任何 Claude E2E 测试都可以选择 worktree 模式以获取完整仓库上下文，而非 tmpdir fixtures。

**为什么：** 一些 Claude E2E 测试（CSO 审计、review-sql-injection）创建了最小假仓库，但使用完整仓库上下文会产生更真实的结果。基础设施已存在（`describeWithWorktree()` 在 e2e-helpers.ts 中）——这将其扩展到 session-runner 级别。

**背景：** WorktreeManager 在 v0.11.12.0 发布。目前只有 Gemini/Codex 测试使用 worktrees。Claude 测试使用 planted-bug 修复仓库，这对他们的目的是正确的，但想要真实仓库上下文的新测试今天就可以使用 `describeWithWorktree()`。这个 TODO 是通过 `runSkillTest()` 上的标志使其更加简单。

**工作量：** M（人类：约 2 天 / CC：约 20 分钟）
**优先级：** P3
**依赖：** Worktree 隔离（v0.11.12.0 已发布）

### E2E 模型固定 — 已发布

~~**做什么：** 将 E2E 测试固定到 claude-sonnet-4-6 以降低成本，添加 retry:2 以应对不稳定的 LLM 响应。~~

已发布：默认模型改为 Sonnet 用于结构测试（约 30），Opus 保留用于质量测试（约 10）。添加了 `--retry 2`。`EVALS_MODEL` 环境变量用于覆盖。添加了 `test:e2e:fast` 级别。添加了速率限制遥测（first_response_ms、max_inter_turn_ms）和 wall_clock_ms 追踪到 eval-store。

### 评估 Web 仪表板

**做什么：** `bun run eval:dashboard` 提供本地 HTML 页面带图表：成本趋势、检测率、通过/失败历史。

**为什么：** 可视化图表比 CLI 工具更适合发现趋势。

**背景：** 读取 `~/.gstack-dev/evals/*.json`。约 200 行 HTML + 通过 Bun HTTP 服务器的 chart.js。

**工作量：** M
**优先级：** P3
**依赖：** 评估持久化（v0.3.6 已发布）

### CI/CD QA 质量门

**工作量：** M
**优先级：** P2
**依赖：** 无

### 跨平台 URL 打开辅助工具

**做什么：** `gstack-open-url` 辅助脚本——检测平台，使用 `open`（macOS）或 `xdg-open`（Linux）。

**为什么：** 首次 Completeness Principle 引导使用 macOS `open` 来启动文章。如果 gstack 将来支持 Linux，这会静默失败。

**工作量：** S（人类：约 30 分钟 / CC：约 2 分钟）
**优先级：** P4
**依赖：** 无

### 基于 CDP 的 DOM 变更检测用于 ref 失效

**做什么：** 使用 Chrome DevTools Protocol `DOM.documentUpdated` / MutationObserver 事件在 DOM 变更时主动使过期 ref 失效，无需显式 `snapshot` 调用。

**为什么：** 当前的 ref 失效检测（异步 count() 检查）仅在操作时捕获过期 ref。CDP 突变检测会在 ref 过期时主动发出警告，完全避免 SPA 重渲染时的 5 秒超时。

**背景：** ref 失效修复的第 1+2 部分（RefEntry 元数据 + 通过 count() 急切验证）已发布。这是第 3 部分——最有野心的部分。需要 CDP 会话与 Playwright 配对、MutationObserver 桥接，以及小心的性能调优以避免每次 DOM 变更的开销。

**工作量：** L
**优先级：** P3
**依赖：** ref 失效第 1+2 部分（已发布）

## Office Hours / 设计

### 设计文档 → Supabase 团队存储同步

**做什么：** 将设计文档（`*-design-*.md`）添加到 Supabase 同步管线中，与测试计划、retro 快照和 QA 报告一起。

**为什么：** 跨团队设计发现规模化。本地 `~/.gstack/projects/$SLUG/` 关键字 grep 发现对同机用户现在有效，但 Supabase 同步使其对整个团队有效。重复的想法会浮现，每个人都能看到已探索的内容。

**背景：** /office-hours 将设计文档写入 `~/.gstack/projects/$SLUG/`。团队存储已经同步测试计划、retro 快照、QA 报告。设计文档遵循相同的模式——只需添加一个同步适配器。

**工作量：** S
**优先级：** P2
**依赖：** `garrytan/team-supabase-store` 分支合入 main

### /yc-prep Skill

**做什么：** 帮助创始人在 /office-hours 识别到强信号后准备 YC 申请的 Skill。从设计文档中提取，结构化回答 YC 申请问题，运行模拟面试。

**为什么：** 形成闭环。/office-hours 识别创始人，/yc-prep 帮助他们更好地申请。设计文档已经包含了 YC 申请所需的大部分原始材料。

**工作量：** M（人类：约 2 周 / CC：约 2 小时）
**优先级：** P2
**依赖：** office-hours 创始人发现引擎先发布

## 设计审查

### /plan-design-review + /qa-design-review + /design-consultation — 已发布

作为 v0.5.0 在 main 发布。包括 `/plan-design-review`（仅报告的设计审计）、`/qa-design-review`（审计 + 修复循环）和 `/design-consultation`（交互式 DESIGN.md 创建）。`{{DESIGN_METHODOLOGY}}` 解析器提供共享的 80 项设计审计检查清单。

### /plan-eng-review 中的设计外部声音

**做什么：** 将并行双声模式（Codex + Claude 子Agent）扩展到 /plan-eng-review 的架构审查部分。

**为什么：** 设计滩头（v0.11.3.0）证明跨模型共识对主观审查有效。架构审查在权衡决策中具有类似的主观性。

**背景：** 依赖于设计滩头的经验教训。如果 Litmus 计分卡格式证明有效，将其适配到架构维度（耦合度、扩展性、可逆性）。

**工作量：** S
**优先级：** P3
**依赖：** 设计外部声音已发布（v0.11.3.0）

### /qa 视觉回归检测中的外部声音

**做什么：** 为 /qa 添加 Codex 设计声音，在 Bug 修复验证期间检测视觉回归。

**为什么：** 修复 Bug 时，修复可能引入代码级检查无法发现的视觉回归。Codex 可以在重新测试期间标记"修复破坏了响应式布局"。

**背景：** 依赖于 /qa 具有设计感知。目前 /qa 侧重于功能测试。

**工作量：** M
**优先级：** P3
**依赖：** 设计外部声音已发布（v0.11.3.0）

## Document-Release

### 从 /ship 自动调用 /document-release — 已发布

在 v0.8.3 中发布。第 8.5 步添加到 `/ship`——创建 PR 后，`/ship` 自动读取 `document-release/SKILL.md` 并执行文档更新工作流。零摩擦文档更新。

### `{{DOC_VOICE}}` 共享解析器

**做什么：** 在 gen-skill-docs.ts 中创建占位符解析器，编码 gstack 语音指南（友好、用户导向、以好处为先）。注入到 /ship 第 5 步、/document-release 第 5 步，并从 CLAUDE.md 引用。

**为什么：** DRY——语音规则目前内联存在于 3 个位置（CLAUDE.md CHANGELOG 风格部分、/ship 第 5 步、/document-release 第 5 步）。当语音演变时，三者会漂移。

**背景：** 与 `{{QA_METHODOLOGY}}` 相同的模式——共享块注入到多个模板中以防止漂移。gen-skill-docs.ts 中约 20 行。

**工作量：** S
**优先级：** P2
**依赖：** 无

## Ship Confidence 仪表板

### 智能审查相关性检测 — 部分已发布

~~**做什么：** 根据分支变更自动检测 4 个审查中哪些是相关的（如果没有 CSS/视觉变更则跳过设计审查，如果仅计划则跳过代码审查）。~~

`bin/gstack-diff-scope` 已发布——将 diff 分类为 SCOPE_FRONTEND、SCOPE_BACKEND、SCOPE_PROMPTS、SCOPE_TESTS、SCOPE_DOCS、SCOPE_CONFIG。design-review-lite 在没有前端文件变更时跳过。仪表板集成的条件行显示是后续工作。

**剩余：** 仪表板条件行显示（当 SCOPE_FRONTEND=false 时隐藏 "Design Review: NOT YET RUN"）。扩展到 Eng Review（仅限文档时跳过）和 CEO Review（仅限配置时跳过）。

**工作量：** S
**优先级：** P3
**依赖：** gstack-diff-scope（已发布）

## Codex

### Codex→Claude 反向伙伴检查 Skill

**做什么：** 基于 Codex 的 Skill（`.agents/skills/gstack-claude/SKILL.md`），运行 `claude -p` 以从 Claude 获得独立的第二意见——与今天 `/codex` 从 Claude Code 所做的相反。

**为什么：** Codex 用户应该获得与 Claude 用户通过 `/codex` 获得的相同的跨模型挑战。目前流程是单向的（Claude→Codex）。Codex 用户无法获得 Claude 的第二意见。

**背景：** `/codex` Skill 模板（`codex/SKILL.md.tmpl`）展示了模式——它将 `codex exec` 与 JSONL 解析、超时处理和结构化输出包装在一起。反向 Skill 将使用类似的基础设施包装 `claude -p`。将通过 `gen-skill-docs --host codex` 生成到 `.agents/skills/gstack-claude/`。

**工作量：** M（人类：约 2 周 / CC：约 30 分钟）
**优先级：** P1
**依赖：** 无

## 完整度

### 完整度指标仪表板

**做什么：** 追踪在 gstack 会话中 Claude 选择完整选项与捷径的频率。聚合到显示随时间完整度趋势的仪表板。

**为什么：** 没有度量，我们无法知道 Completeness Principle 是否有效。可能发现模式（例如，某些 Skill 仍然偏向捷径）。

**背景：** 需要记录选择（例如，当 AskUserQuestion 解决时追加到 JSONL 文件）、解析它们并显示趋势。与评估持久化类似模式。

**工作量：** M（人类）/ S（CC）
**优先级：** P3
**依赖：** Boil the Lake 已发布（v0.6.1）

## 安全与可观察性

### 按需 Hook Skills（/careful、/freeze、/guard） — 已发布

~~**做什么：** 三个新 Skill，使用 Claude Code 的会话级 PreToolUse hooks 添加按需安全护栏。~~

以 `/careful`、`/freeze`、`/guard` 和 `/unfreeze` 在 v0.6.5 发布。包括 Hook 触发率遥测（仅模式名，无命令内容）和内联 Skill 激活遥测。

### Skill 使用遥测 — 已发布

~~**做什么：** 追踪哪些 Skill 被调用、频率、来自哪个仓库。~~

在 v0.6.5 发布。gen-skill-docs.ts 中的 TemplateContext 将 Skill 名称烘焙到前言遥测行中。Analytics CLI（`bun run analytics`）用于查询。/retro 集成显示本周使用的 Skill。

### /investigate 范围调试增强（取决于遥测）

**做什么：** 对 /investigate 自动冻结的六项增强，取决于遥测显示冻结 Hook 在实际调试会话中确实触发。

**为什么：** /investigate v0.7.1 自动冻结正在调试的模块的编辑。如果遥测显示 Hook 经常触发，这些增强会让体验更智能。如果它从未触发，说明问题不是真实的，这些不值得构建。

**背景：** 所有项目都是 `investigate/SKILL.md.tmpl` 中的散文添加。没有新脚本。

**工作量：** M（全部 6 项合计）
**优先级：** P3
**依赖：** 遥测数据显示冻结 Hook 在实际 /investigate 会话中触发

## 上下文智能

### 上下文恢复前言

**工作量：** S（人类：约 30 分钟 / CC：约 5 分钟）
**优先级：** P1
**依赖：** 无
**关键文件：** `scripts/resolvers/preamble.ts`

### 会话时间线

**工作量：** S（人类：约 1 小时 / CC：约 5 分钟）
**优先级：** P1
**依赖：** 无
**关键文件：** `scripts/resolvers/preamble.ts`、`retro/SKILL.md.tmpl`

### 跨会话上下文注入

**工程量：** S（人类：约 2 小时 / CC：约 10 分钟）
**优先级：** P2
**依赖：** 上下文恢复前言

### /checkpoint Skill

**工作量：** M（人类：约 1 周 / CC：约 30 分钟）
**优先级：** P2
**依赖：** 上下文恢复前言
**关键文件：** 新 `checkpoint/SKILL.md.tmpl`，`scripts/gen-skill-docs.ts`

### Session Intelligence Layer 设计文档

**工作量：** S（人类：约 2 小时 / CC：约 15 分钟）
**优先级：** P1
**依赖：** 无

## 健康

### /health — 项目健康仪表板

**工作量：** M（人类：约 1 周 / CC：约 30 分钟）
**优先级：** P1
**依赖：** 无
**关键文件：** 新 `health/SKILL.md.tmpl`，`scripts/gen-skill-docs.ts`

### /health 作为 /ship 门禁

**工作量：** S（人类：约 1 小时 / CC：约 5 分钟）
**优先级：** P2
**依赖：** /health Skill

## 蜂群

### 蜂群原语 — 可复用的多Agent调度

**工作量：** L（人类：约 2 周 / CC：约 2 小时）
**优先级：** P2
**依赖：** 无

## 重构

### /refactor-prep — 重构前 Token 卫生

**工作量：** M（人类：约 1 周 / CC：约 30 分钟）
**优先级：** P2
**依赖：** 无

## Factory Droid

### 为 Factory Droid 提供 Browse MCP 服务器

**工作量：** L（人类：约 1 周 / CC：约 5 小时）
**优先级：** P1
**依赖：** --host factory (选项 A，将在 v0.13.4.0 发布)

### .agent/skills/ 双输出用于跨Agent兼容

**工作量：** S
**优先级：** P3
**依赖：** --host factory

### 与 Skills 并列的自定义 Droid 定义

**工作量：** M
**优先级：** P3
**依赖：** --host factory

## GStack Browser

### 反 Bot 隐身：Playwright CDP 补丁（rebrowser 风格）

**工作量：** L（人类：约 2 周 / CC：约 3 小时）
**优先级：** P1
**依赖：** 无

### Chromium 分叉（CDP 补丁的长期替代方案）

**工作量：** XL（人类：约 1 季度 / CC：约 2-3 周的集中工作）
**优先级：** P2
**依赖：** CDP 补丁首先证明反 Bot 隐身的价值

## 已完成

### CI 评估管线（v0.9.9.0）
**已完成：** v0.9.9.0

### 部署管线（v0.9.8.0）
**已完成：** v0.9.8.0

### 第 1 阶段：基础（v0.2.0）
**已完成：** v0.2.0

### 第 2 阶段：增强浏览器（v0.2.0）
**已完成：** v0.2.0

### 第 3 阶段：QA 测试 Agent（v0.3.0）
**已完成：** v0.3.0

### 第 3.5 阶段：浏览器 Cookie 导入（v0.3.x）
**已完成：** v0.3.1

### E2E 测试成本追踪
**已完成：** v0.3.6

### 自动升级模式 + 智能更新检查
**已完成：** v0.3.8
