# Changelog

## [0.17.0.0] - 2026-04-14

### Added
- **UX 行为基础。** 每个设计 skill 现在都考虑用户实际如何行为，而不仅仅是界面看起来怎样。一个共享的 `{{UX_PRINCIPLES}}` 解析器将 Steve Krug 的《点石成金》提炼为可操作的指导：扫描行为、满意即可、善意储备、导航寻路、主干测试。注入到 /design-html、/design-shotgun、/design-review 和 /plan-design-review。
- **6 项可用性测试融入 design-review。** 方法论现在运行主干测试、3 秒扫描、页面区域测试、废话检测带字数统计、无脑选择审计、善意储备跟踪带可视化仪表盘。
- **第一人称叙述模式。** 设计审查报告现在读起来像可用性顾问在观察某人使用你的站点。带有防 slop 护栏：如果 agent 无法命名具体元素，它就在生成陈词滥调。
- **`$B ux-audit` 命令。** 独立 UX 结构提取。
- **`snapshot -H` / `--heatmap` 标志。** 彩色覆盖截图。
- **Token 上限强制。** `gen-skill-docs` 现在在任何生成的 SKILL.md 超过 100KB（约 25K tokens）时发出警告。

### Changed
- **Krug 的始终/从不规则**添加到设计硬性规则中：从不用占位符当标签、从不浮动标题、始终区分已访问链接、从不使用小于 16px 的正文字体。
- **Plan-design-review 参考文献**现在包括 Steve Krug、Ginny Redish（放飞文字）和 Caroline Jarrett（有效的表单设计）。

## [0.16.4.0] - 2026-04-13

### Added
- **Cookie 来源固定。** 导入特定域名的 cookie 时，JS 执行现在被阻止在不匹配的页面上。防止 prompt injection 导航到攻击者站点并窃取 cookie 的攻击。来自 @halbert04 的 3 个 PR。
- **命令审计日志。** 每个 browse 命令现在在 `~/.gstack/.browse/browse-audit.jsonl` 中获得持久的取证跟踪。来自 @halbert04。
- **Cookie 域名跟踪。** gstack 现在跟踪从哪些域导入了 cookie。

### Fixed
- **文件写入中的符号链接绕过。** `validateOutputPath` 现在在写入前用 `lstatSync` 检查文件本身。来自 @Hybirdss。
- **Cookie 导入路径绕过。** 两个问题：相对路径绕过所有验证，符号链接解析缺失。来自 @urbantech。
- **安装脚本中的 Shell 注入。** `gstack-settings-hook` 现在使用环境变量而非直接插值。来自 @garagon。
- **表单字段凭证泄漏。** 快照脱敏现在检查字段名和 id 是否匹配敏感模式。来自 @garagon。
- **学习记录 prompt 注入。** 三个修复：输入验证、注入模式检测、跨项目信任门控。来自 @Ziadstr。
- **IPv6 元数据绕过。** 添加了十六进制编码形式的元数据 IP 到阻止列表。来自 @mehmoodosman。
- **会话文件全局可读。** `/tmp` 中的设计会话文件现在创建为 0600（仅所有者）。来自 @garagon。
- **setup 中的冻结锁文件。** `bun install` 现在使用 `--frozen-lockfile`。来自 @halbert04。
- **Dockerfile chmod 修复。** 移除了重复的递归 `chmod -R 1777 /tmp`。来自 @Gonzih。
- **Cookie 导入中的硬编码 /tmp。** 现在使用 `os.tmpdir()`，修复了 Windows 支持。

### Security
- 关闭了 14 个安全问题（#665-#675 等）和 17 个社区安全 PR。安全波次 3：12 个修复，7 位贡献者。

## [0.16.3.0] - 2026-04-09

### Changed
- **AI slop 清理。** 运行 [slop-scan](https://github.com/benvinegar/slop-scan)，从 100 个发现降至 90 个。修复了 `safeUnlink()` 和 `safeKill()` 工具函数等。

### Added
- **`bun run slop:diff`** 显示分支上相比 main 新增的 slop-scan 发现。
- **Slop-scan 使用指南**在 CLAUDE.md 中。
- **设计文档**用于未来的 slop-scan 集成。

## [0.16.2.0] - 2026-04-09

### Added
- **Office hours 现在记住你。** 关闭体验根据会话次数自适应。首次：完整 YC 建议和创始人资源。2-3 次："欢迎回来。上次你在做 [你的项目]。" 4-7 次：跨整个旅程的弧线级回调。8+ 次：数据自己说话。
- **构建者档案**跟踪你的 office hours 旅程。
- **构建者到创始人的推动**针对重复处于构建者模式的用户。
- **旅程匹配的资源。**
- **构建者旅程摘要**在会话 5+ 时自动生成并在浏览器中打开。
- **全局资源去重。**

### Fixed
- package.json 版本现在与 VERSION 文件保持同步。

## [0.16.1.0] - 2026-04-08

### Fixed
- Cookie 选择器不再泄漏浏览服务器认证 token。现在使用一次性代码交换 + HttpOnly 会话 cookie。（由 Horoshi 在 Vagabond Research 报告，CVSS 7.8）

## [0.16.0.0] - 2026-04-07

### Added
- **浏览器数据平台。** 6 个新的 browse 命令，将 gstack 浏览器从"点击按钮的工具"变为完整的抓取和数据提取工具。
- `media` 命令：发现页面上所有图片、视频和音频元素。
- `data` 命令：提取嵌入在页面中的结构化数据。
- `download` 命令：使用浏览器会话 cookie 获取任何 URL。
- `scrape` 命令：批量下载页面中的所有媒体。
- `archive` 命令：通过 CDP 将完整页面保存为 MHTML。
- `scroll --times N`：自动重复滚动。
- `screenshot --base64`：返回内联数据 URI 格式的截图。
- **网络响应体捕获。** `network --capture` 拦截 API 响应体。
- `GET /file` 端点：远程配对 agent 现在可以通过 HTTP 获取下载的文件。

### Changed
- 配对 agent 现在默认获得完全访问权限（读+写+管理+元数据）。
- 路径验证提取到共享 `path-security.ts` 模块。

## [0.15.16.0] - 2026-04-06

### Added
- 通过 TabSession 实现每个 tab 的状态隔离。

### Changed
- 处理器签名现在接受 TabSession 用于每个 tab 的操作。

### Fixed
- codex-review E2E 测试现在只复制相关的 ~6KB/148 行，从 8 个 Read 调用降到 1 个。

## [0.15.15.1] - 2026-04-06

### Fixed
- pair-agent 隧道在 15 秒后断开。
- `$B connect` 因"domains is not defined"崩溃。

## [0.15.15.0] - 2026-04-06

社区安全波次：来自 4 位贡献者的 8 个 PR。

### Added
- Cookie 值脱敏（token、API 密钥、JWT、会话密钥）。
- IPv6 ULA 前缀阻止。
- 每个 tab 的取消信号。
- 父进程看门狗用于浏览服务器。
- 卸载说明。
- CSS 值验证阻止注入攻击。
- 队列条目 schema 验证。
- 视口尺寸钳位和等待超时钳位。
- Cookie 域名验证。
- DocumentFragment 基于 tab 切换。
- `pollInProgress` 重入保护。
- 750+ 行新的安全回归测试。
- Supabase 迁移 003：列级 GRANT。

### Fixed
- Windows: `extraEnv` 现在传递到 Windows 启动器。
- Windows: 欢迎页面提供内联 HTML 而非 `about:blank` 重定向。
- Headed 模式：即使没有 Origin 头也返回认证 token。
- `frame --url` 现在转义用户输入（ReDoS 修复）。
- 注解截图路径验证现在解决符号链接。
- 认证 token 从健康广播中移除。
- `/health` 端点不再暴露 `currentUrl` 或 `currentMessage`。
- 会话 ID 在用于文件路径前验证。
- SIGTERM/SIGKILL 升级。

### For contributors
- 队列文件创建权限 0o700/0o600。
- `escapeRegExp` 工具从 meta-commands 导出。

## [0.15.14.0] - 2026-04-05

### Fixed
- **`gstack-team-init` 现在检测并移除 vendored gstack 副本。**
- **`/gstack-upgrade` 遵守 team mode。**
- **`team_mode` 配置键。**

## [0.15.13.0] - 2026-04-04 — Team Mode

团队现在可以自动让每个开发者保持相同的 gstack 版本。不再需要将 342 个文件 vendored 到仓库中。

### Added
- **`./setup --team`。** 注册 `SessionStart` hook，在每个 Claude Code 会话开始时自动更新 gstack。
- **`./setup -q` / `--quiet`。** 抑制所有信息输出。
- **`gstack-team-init` 命令。** 生成仓库级引导文件。
- **`gstack-settings-hook` 助手。** 用于在 `settings.json` 中添加/移除 hook 的 DRY 工具。
- **`gstack-session-update` 脚本。** SessionStart hook 目标。
- **Vendoring 弃用警告。**

### Changed
- **Vendoring 被弃用。** README 不再推荐将 gstack 复制到仓库中。

## [0.15.12.0] - 2026-04-05 — 内容安全：4 层 Prompt 注入防御

### Added
- **内容信封包装。** 每个由 scoped agent 读取的页面都被包装在 `═══ BEGIN UNTRUSTED WEB CONTENT ═══` / `═══ END UNTRUSTED WEB CONTENT ═══` 标记中。
- **隐藏元素剥离。** CSS 隐藏元素和 ARIA label 注入被检测并从文本输出中剥离。
- **数据标记。** 文本命令输出获得会话范围的水印。
- **内容过滤钩子。** 可扩展的过滤管道。
- **快照分割格式。**
- **指令块中的 SECURITY 部分。**
- **47 个内容安全测试。**

### Changed
- `handleCommand` 重构。
- `attrs` 添加到 `PAGE_CONTENT_COMMANDS`。

### Fixed
- `snapshot -i` 现在自动包含光标交互元素。
- 快照正确捕获浮动容器内的项目。
- 链命令现在检查 `newtab` 的域名限制。
- 嵌套链命令被拒绝。
- 速率限制对链子命令的豁免。
- 隧道存活性验证。
- 所有 16 个预先存在的测试失败已修复。

## [0.15.11.0] - 2026-04-05

### Changed
- `/ship` 重新运行现在执行每个验证步骤。
- `/ship` 现在在预着陆审查期间运行完整的 Review Army 专家分派。

### Added
- 跨审查发现在 `/ship` 中的去重。
- `/document-release` 后的 PR 正文刷新。

### Fixed
- Review Army diff 大小启发式现在计算插入+删除。

### For contributors
- 提取跨审查去重到共享 `{{CROSS_REVIEW_DEDUPT}}` 解析器。

## [0.15.10.0] - 2026-04-05 — 原生 OpenClaw Skills + ClawHub 发布

### Added
- **4 个原生 OpenClaw skills 在 ClawHub 上。** 通过 `clawhub install` 安装。
- **AGENTS.md 分派修复。**

### Changed
- OpenClaw `includeSkills` 清空。

## [0.15.9.0] - 2026-04-05 — OpenClaw 集成 v2

### Added
- **gstack-lite 规划纪律。** 15 行 CLAUDE.md。
- **gstack-full 管道模板。**
- **4 个原生方法论 skills。**
- **4 层分派路由。**
- **生成会话检测。**
- **includeSkills host 配置字段。**
- **docs/OPENCLAW.md。**

### Changed
- OpenClaw host 配置更新。

## [0.15.8.1] - 2026-04-05 — 社区 PR 分类 + 错误优化

### Fixed
- **所有设计命令上的友好 OpenAI 组织错误。**

### Added
- **>128KB 回归测试用于 Codex 会话发现。**

### For contributors
- 关闭了 12 个冗余社区 PR。

## [0.15.8.0] - 2026-04-04 — 更智能的审查

### Added
- **跨审查发现去重。**
- **测试框架建议。**
- **自适应专家门控。**
- **审查日志中每个专家的统计。**

## [0.15.7.0] - 2026-04-05 — 安全波次 1

### Fixed
- **设计服务器仅绑定 localhost。**
- **阻止 /api/reload 上的路径遍历。**
- **`/inspector/events` 上的认证门控。**
- **设计反馈中的 prompt 注入防御。**
- **文件和目录权限加固。**
- **setup 符号链接创建中的 TOCTOU 竞态。**
- **CORS 通配符移除。**
- **Cookie 选择器强制认证。**
- **DNS 重绑定保护检查 IPv6。**
- **validateOutputPath 中的符号链接绕过。**
- **restoreState 上的 URL 验证。**
- **遥测端点使用 anon 密钥。**
- **killAgent 真正杀掉子进程。**

## [0.15.6.2] - 2026-04-04 — 反跳过审查规则

### Added
- **所有 4 个审查 skill 中的反跳过规则。**
- **CEO 审查标题修复。**

## [0.15.6.1] - 2026-04-04

### Fixed
- **Skill 前缀自我修复。** setup 现在作为最终一致性检查运行 `gstack-relink`。

## [0.15.6.0] - 2026-04-04 — 声明式多宿主平台

### Added
- **声明式 host 配置系统。** 每个 host 是一个类型化的 `HostConfig` 对象。
- **4 个新的 hosts：OpenCode、Slate、Cursor、OpenClaw。**
- **OpenClaw adapter。**
- **106 个新测试。**
- **`host-config-export.ts` CLI。**
- **贡献者 `/gstack-contrib-add-host` skill。**
- **Golden-file baselines。**
- **README 中每个 host 的安装说明。**

### Changed
- **`gen-skill-docs.ts` 现在是配置驱动的。**

### Fixed
- **Sidebar E2E 测试现在自包含。**

## [0.15.5.0] - 2026-04-04 — 交互式 DX 审查 + Plan Mode Skill 修复

### Added
- **开发者角色审问。**
- **共情叙事。**
- **竞争性 DX 基准。**
- **神奇时刻设计。**
- **三种审查模式。**
- **摩擦点旅程追踪。**
- **首次开发者角色扮演。**

### Fixed
- **Plan mode 期间的 skill 调用。**

## [0.15.4.0] - 2026-04-03 — Autoplan DX 集成 + 文档

### Added
- **/autoplan 中的 DX 审查。**
- **README 中的"哪个审查？"对比表。**
- **安装指令中的 `/plan-devex-review` 和 `/devex-review`。**

### Changed
- **Autoplan 管道顺序。** 现在是 CEO → Design → Eng → DX。

## [0.15.3.0] - 2026-04-03 — 开发者体验审查

### Added
- **/plan-devex-review skill。**
- **/devex-review skill。**
- **DX 名人堂参考。**
- **`{{DX_FRAMEWORK}}` 解析器。**
- **仪表板中的 DX 审查。**

## [0.15.2.1] - 2026-04-02 — Setup 运行迁移

### Fixed
- **Setup 运行待处理迁移。**
- **空格安全的迁移循环。**
- **全新安装跳过迁移。**
- **未来迁移守卫。**
- **缺失 VERSION 守卫。**

## [0.15.2.0] - 2026-04-02 — 语音友好的 Skill 触发器

### Added
- **10 个 skill 的语音触发器。**
- **`voice-triggers:` YAML 字段。**
- **README 中的语音输入部分。**
- **CONTRIBUTING.md 中记录的 `voice-triggers`。**

## [0.15.1.0] - 2026-04-01 — 没有 Shotgun 的设计

### Changed
- **`/design-html` 从任何起点工作。** 三种路由模式。
- **缺少上下文时使用 AskUserQuestion。**

### Fixed
- **Skill 现在被发现为顶层名称。**

## [0.15.0.0] - 2026-04-01 — 会话智能

### Added
- **会话时间线。** 每个 skill 自动记录开始/完成事件到 `timeline.jsonl`。
- **上下文恢复。**
- **跨会话注入。**
- **预测性 skill 建议。**
- **欢迎回来消息。**
- **`/checkpoint` skill。**
- **`/health` skill。**
- **时间线二进制工具。**
- **路由规则。**

## [0.14.6.0] - 2026-03-31 — 递归自我改进

### Added
- **运营自我改进。**
- **前言中的学习摘要。**
- **13 个 skill 现在学习。**

### Changed
- **贡献者模式被替代。**

### Fixed
- **learnings-show E2E 测试 slug 不匹配。**

## [0.14.5.0] - 2026-03-31 — Ship 幂等性 + Skill 前缀修复

### Fixed
- **`/ship` 现在是幂等的（#649）。**
- **Skill 前缀实际上修补了 SKILL.md 中的 `name:`（#620, #578）。**
- **`gen-skill-docs` 在前缀修补需要重新应用时发出警告。**
- **PR 幂等性检查开放状态。**
- **`--no-prefix` 排序 bug。**

### Added
- **`bin/gstack-patch-names` 共享助手。**

### For contributors
- 4 个单元测试、2 个测试用于 gen-skill-docs 前缀警告、1 个 E2E 测试。

## [0.14.4.0] - 2026-03-31 — Review Army：并行专家审查者

### Added
- **7 个专家审查者并行运行。**
- **JSON 发现 schema。**
- **基于指纹的去重。**
- **PR 质量评分。**
- **3 个新的 diff 范围信号。**
- **学习通知的专家 prompts。**
- **14 个新的 diff 范围测试。**
- **7 个新的 E2E 测试。**

### Changed
- **审查清单重构。**
- **交付完整性增强。**

## [0.14.3.0] - 2026-03-31 — 始终开启的对抗审查 + 范围漂移 + Plan Mode 设计工具

### Added
- **始终开启的对抗审查。**
- **`/ship` 中的范围漂移检测。**
- **Plan Mode 安全操作。**

### Changed
- **对抗退出拆分。**
- **跨模型紧张格式。**
- **范围漂移现在是共享解析器。**

## [0.14.2.0] - 2026-03-30 — Sidebar CSS 检查器 + 每个 Tab 的 Agent

### Added
- **Sidebar 中的 CSS 检查器。**
- **实时样式编辑。**
- **每个 tab 的 agent。**
- **Tab 跟踪。**
- **LLM 驱动的页面清理。**
- **漂亮的截图。**
- **停止按钮。**
- **CSP 回退。**

### Fixed
- **检查器消息允许列表。**
- **粘性导航保留。**
- **Agent 不会停止。**
- **焦点窃取。**
- **聊天消息去重。**

### Changed
- **Sidebar 横幅。**
- **输入占位符。**
- **系统 prompts。**

## [0.14.1.0] - 2026-03-30 — 对比面板是选择器

### Changed
- **对比面板现在是强制性的。**
- **AskUserQuestion 是等待，不是选择器。**
- **Serve 失败回退改进。**

### Fixed
- **面板 URL 纠正。**

## [0.14.0.0] - 2026-03-30 — 设计到代码

### Added
- **`/design-html` skill。**
- **vendor 化的 Pretext。**
- **设计管道链式连接。**

### Changed
- **`/plan-design-review` 的下一步扩展。**

## [0.13.10.0] - 2026-03-29 — Office Hours 获得阅读清单

### Added
- **/office-hours 关闭时的轮换创始人资源。**
- **资源去重日志。**
- **资源选择分析。**
- **浏览器打开提供。**

### Fixed
- **构建脚本 chmod 安全网。**

## [0.13.9.0] - 2026-03-29 — 可组合 Skills

### Added
- **`{{INVOKE_SKILL:skill-name}}` 解析器。**
- **参数化解析器支持。**
- **`{{CHANGELOG_WORKFLOW}}` 解析器。**
- **Frontmatter `name:` 用于 skill 注册。**
- **主动 skill 路由。**
- **带注解的配置文件。**

### Changed
- **BENEFITSFROM 现在委托给 INVOKE_SKILL。**
- **/plan-ceo-review 中途会话回退使用 INVOKE_SKILL。**
- **更强的路由语言。**

### Fixed
- **配置 grep 锚定到行首。**

## [0.13.8.0] - 2026-03-29 — 安全审查轮次 2

### Fixed
- **信任边界标记是逃逸证明的。**

### Added
- **内容信任边界标记。**
- **扩展发送者验证。**
- **CDP 仅绑定 localhost。**
- **校验和验证的 bun 安装。**

### Removed
- **Factory Droid 支持。**

## [0.13.7.0] - 2026-03-29 — 社区波次

### Fixed
- **遥测关闭意味着各处都关闭。**
- **`find -delete` 替换为 POSIX `-exec rm`。**
- **不再有先发制人的上下文警告。**
- **Sidebar 安全测试更新。**
- **`gstack-relink` 不再双重前缀化 `gstack-upgrade`。**

### Added
- **Skill 可发现性。**
- **`/ship` 中的功能信号检测。**
- **Sidebar Write 工具。**
- **Sidebar stderr 捕获。**
- **`bin/gstack-relink`。**
- **`bin/gstack-open-url`。**

## [0.13.6.0] - 2026-03-29 — GStack 学习

### Added
- **项目学习系统。**
- **`/learn` skill。**
- **置信度校准。**
- **"已应用学习"标注。**
- **跨项目发现。**
- **置信度衰减。**
- **前言中的学习计数。**
- **5 个版本的路线图设计文档。**

## [0.13.5.1] - 2026-03-29 — Gitignore .factory

### Changed
- **停止跟踪 `.factory/` 目录。**

## [0.13.5.0] - 2026-03-29 — Factory Droid 兼容

### Added
- **Factory Droid 支持（`--host factory`）。**
- **`--host all` 标志。**
- **`gstack-platform-detect` 二进制文件。**
- **敏感 skill 安全性。**
- **Factory CI 新鲜度检查。**
- **运营工具中的 Factory 感知。**

### Changed
- **重构多宿主生成。**
- **构建脚本使用 `--host all`。**
- **Factory 的工具名称翻译。**

## [0.13.4.0] - 2026-03-29 — Sidebar 防御

### Fixed
- **Sidebar agent 现在尊重服务器端参数。**

### Added
- **XML prompt 框架与信任边界。**
- **Bash 命令允许列表。**
- **Sidebar 默认使用 Opus。**
- **ML prompt 注入防御设计文档。**

## [0.13.3.0] - 2026-03-28 — 锁死

### Fixed
- **依赖现在被锁定。** `bun.lock` 已提交和跟踪。
- **`gstack-slug` 不再在非 git 仓库外崩溃。**
- **`./setup` 不再在 CI 中挂起。**
- **Browse CLI 在 Windows 上工作。**
- **`/ship` 和 `/review` 找到你的设计文档。**
- **`/autoplan` 双声音实际工作。**

### Added
- **CLAUDE.md 中的社区 PR 护栏。**

## [0.13.2.0] - 2026-03-28 — 用户主权

### Added
- **ETHOS.md 中的用户主权原则。**
- **/autoplan 中的用户挑战类别。**
- **安全/可行性警告框架。**
- **所有 skill 声音中的用户主权声明。**

### Changed
- **跨模型紧张模板不再说"你对谁对的评估。"**
- **/autoplan 现在有两个门，不是一个。**
- **决策审计跟踪现在跟踪分类。**

## [0.13.1.0] - 2026-03-28 — 纵深防御

### Fixed
- **从 `/health` 端点移除认证 token。**
- **Cookie 选择器数据路由现在需要 Bearer 认证。**
- **收紧 `/refs` 和 `/activity/*` 上的 CORS。**
- **状态文件 7 天后自动过期。**
- **扩展使用 `textContent` 而非 `innerHTML`。**
- **路径验证在边界检查之前解析符号链接。**
- **冻结钩使用可移植路径解析。**
- **Shell 配置脚本验证输入。**

### Added
- 20 个回归测试。

## [0.13.0.0] - 2026-03-27 — 你的 Agent 现在可以设计

### Added
- **设计二进制（`$D`）。**
- **对比面板。**
- **`/design-shotgun` skill。**
- **`$D serve` 命令。**
- **`$D gallery` 命令。**
- **设计记忆。**
- **视觉差异。**
- **截图演进。**
- **响应式变体。**
- **设计到代码 prompt。**

### Changed
- **/office-hours 现在默认生成视觉 mockup 探索。**
- **/plan-design-review 使用 `{{DESIGN_SHOTGUN_LOOP}}`。**
- **/design-consultation 使用 `{{DESIGN_SHOTGUN_LOOP}}`。**
- **对比面板提交后生命周期。**

### For contributors
- 设计二进制源码、新文件、测试、设计文档、模板解析器。

## [0.12.12.0] - 2026-03-27 — 安全审计合规

### Fixed
- **示例中不再有硬编码凭证。**
- **遥测调用是有条件的。**
- **Bun 安装是版本锁定的。**
- **不可信内容警告。**
- **review.ts 中的数据流文档。**

### Removed
- **gen-skill-docs.ts 中的 2017 行死代码。**

### For contributors
- 新的 `test:audit` 脚本。

## [0.12.11.0] - 2026-03-27 — Skill 前缀现在是你的选择

### Added
- **首次安装时的交互式前缀选择。**
- **`--prefix` 标志。**
- **反向符号链接清理。**
- **命名空间感知的 skill 建议。**

### Fixed
- **`gstack-config` 在 Linux 上工作。**
- **死的欢迎消息。**

### For contributors
- 8 个新的结构测试。

## [0.12.10.0] - 2026-03-27 — Codex 文件系统边界

### Fixed
- **Codex 留在仓库中。**
- **兔子洞检测。**
- **5 个回归测试。**

## [0.12.9.0] - 2026-03-27 — 社区 PR：更快的安装、Skill 命名空间、卸载

### Added
- **卸载脚本。**
- **/review 中的 Python 安全模式。**
- **Office-hours 在没有 Codex 时也工作。**

### Changed
- **更快的安装（约 30 秒）。**
- **Skills 以 `gstack-` 前缀命名。**

### Fixed
- **Windows 端口竞态条件。**
- **package.json 版本同步。**

## [0.12.8.1] - 2026-03-27 — zsh 全局兼容

### Fixed
- **`.github/workflows/` 全局替换为 `find`。**
- **`~/.gstack/` 和 `~/.claude/` 全局用 `setopt` 保护。**
- **测试框架检测全局保护。**

## [0.12.8.0] - 2026-03-27 — Codex 不再审查错误的项目

### Fixed
- **Codex exec 急切解析仓库根目录。**
- **`codex review` 也获得 cwd 保护。**
- **静默回退替换为硬失败。**

### Removed
- **gen-skill-docs.ts 中的死解析器副本。**

### Added
- **回归测试。**

## [0.12.7.0] - 2026-03-27 — 社区 PR + 安全加固

### Added
- **Skill 发现中的隐藏文件过滤。**
- **review-log 中的 JSON 验证门控。**
- **遥测输入清理。**
- **特定于 host 的合作者尾部。**
- **10 个新的安全测试。**

### Fixed
- **以 `./` 开头的文件路径不再被视为 CSS 选择器。**
- **构建链弹性。**
- **更新检查器回退。**
- **不稳定的 E2E 测试稳定化。**
- **移除了不可靠的 `journey-think-bigger` 路由测试。**

### For contributors
- 新的 CLAUDE.md 规则。

## [0.12.6.0] - 2026-03-27 — Sidebar 知道你在哪个页面

### Fixed
- **Sidebar 使用真实的 tab URL。**
- **URL 消毒。**
- **重新连接时杀死过时的 sidebar agents。**

### Added
- **`/connect-chrome` 的飞行前清理。**
- **Sidebar agent 测试套件（36 个测试）。**

## [0.12.5.1] - 2026-03-27 — Eng Review 现在告诉你并行化什么

### Added
- **`/plan-eng-review` 中的工作树并行化策略。**

## [0.12.5.0] - 2026-03-26 — 修复 Codex 挂起：30 分钟等待消失

### Fixed
- **计划文件现在对 Codex sandbox 可见。**
- **流式输出实际流式传输。**
- **合理的推理努力默认值。**
- **`--xhigh` 覆写在所有模式中工作。**

## [0.12.4.0] - 2026-03-26 — /ship 中的完整提交覆盖

### Fixed
- **/ship 步骤 5（CHANGELOG）：现在强制提交枚举。**
- **/ship 步骤 8（PR 正文）：从"来自 CHANGELOG 的要点"改为逐提交覆盖。**

## [0.12.3.0] - 2026-03-26 — 语音指令：每个 Skill 听起来都像构建者

### Added
- **所有 25 个 skill 中的语音指令。**
- **上下文依赖的语气。**
- **具体性标准。**
- **用户结果连接。**
- **LLM 评估测试。**

## [0.12.2.0] - 2026-03-26 — 自信部署：首次运行干跑

### Added
- **首次运行干跑。**
- **优先 staging 选项。**
- **配置衰减检测。**
- **内联审查门控。**
- **合并队列感知。**
- **CI 自动部署检测。**

### Changed
- **完整副本重写。**
- **语音 & 语气部分。**

## [0.12.1.0] - 2026-03-26 — 更智能的浏览：网络空闲、状态持久化、Iframes

### Added
- **网络空闲检测。**
- **`$B state save/load`。**
- **`$B frame` 命令。**
- **管道格式链。**

### Changed
- **链循环后空闲等待。**

### Fixed
- **Iframe ref 作用域。**
- **分离的 frame 恢复。**
- **状态加载重置 frame 上下文。**
- **frame 命令中的 elementHandle 泄漏。**
- **上传命令感知 frame。**

## [0.12.0.0] - 2026-03-26 — Headed 模式 + Sidebar Agent

### Added
- **带 sidebar agent 的 headed 模式。**
- **个人自动化。**
- **Chrome 扩展。**
- **`/connect-chrome` skill。**

### Changed
- **Sidebar agent 无门控。**
- **Agent 超时提高到 5 分钟。**

## [0.11.21.0] - 2026-03-26

### Fixed
- **`/autoplan` 审查现在计入 ship 准备门控。**
- **`/ship` 不再告诉你"先运行 /review"。**
- **`/land-and-deploy` 现在检查所有 8 种审查类型。**
- **仪表板外部声音行现在工作。**
- **`/codex review` 现在跟踪过时。**
- **`/autoplan` 不再硬编码"clean"状态。**

## [0.11.20.0] - 2026-03-26

### Added
- **GitLab 支持用于 `/retro` 和 `/ship`。**
- **GitHub Enterprise 和自托管 GitLab 检测。**
- **`/document-release` 在 GitLab 上工作。**
- **`/land-and-deploy` 的 GitLab 安全门。**

### Fixed
- **去重的 gen-skill-docs 解析器。**

## [0.11.19.0] - 2026-03-24

### Fixed
- **自动升级不再损坏。**
- **Codex 审查现在在正确的仓库中运行。**

### Added
- **900 字符早期警告测试。**

## [0.11.18.2] - 2026-03-24

### Fixed
- **Windows 浏览守护进程修复。**

## [0.11.18.1] - 2026-03-24

### Changed
- **每个问题一个决策——无处不在。**

## [0.11.18.0] - 2026-03-24 — 带着牙齿 Ship

### Added
- **/ship 中的测试覆盖门控。**
- **/review 中的覆盖警告。**
- **计划完成审计。**
- **感知计划的范围漂移检测。**
- **通过 /qa-only 自动验证。**
- **共享计划文件发现。**
- **Ship 指标日志。**
- **/retro 中的计划完成。**

## [0.11.17.0] - 2026-03-24 — 更清晰的 Skill 描述 + 主动选择退出

### Changed
- **Skill 描述现在干净易读。**
- **你现在可以选择退出主动 skill 建议。**

### Fixed
- **遥测源标记不再崩溃。**

## [0.11.16.1] - 2026-03-24 — 安装 ID 隐私修复

### Fixed
- **安装 ID 现在是随机 UUID 而非主机名哈希。**
- **RLS 验证脚本处理边缘情况。**

## [0.11.16.0] - 2026-03-24 — 更智能的 CI + 遥测安全

### Changed
- **CI 默认只运行门控测试——定期测试每周运行。**
- **全局 touchfiles 现在是粒度的。**
- **新的 `test:gate` 和 `test:periodic` 脚本。**
- **遥测同步使用 `GSTACK_SUPABASE_URL`。**
- **游标前进是安全的。**

### Fixed
- **遥测 RLS 策略收紧。**
- **社区仪表盘更快且服务器缓存。**

### For contributors
- `E2E_TIERS` 映射、移除的 `EVALS_FAST`、新的周期工作流、新的迁移、新的冒烟测试。

## [0.11.15.0] - 2026-03-24 — 计划审查和 Codex 的 E2E 测试覆盖

### Added
- **E2E 测试验证计划审查报告出现在计划底部。**
- **E2E 测试验证 Codex 在每个计划 skill 中被提供。**

### For contributors
- 新的 E2E 测试。

## [0.11.14.0] - 2026-03-24 — Windows 浏览修复

### Fixed
- **浏览引擎现在在 Windows 上工作。**
- **健康检查首先在所有平台上运行。**
- **启动错误被记录到磁盘。**
- **Windows 上禁用 Chromium sandbox。**

### For contributors
- 新的测试。

## [0.11.13.0] - 2026-03-24 — 工作树隔离 + 基础设施优雅

### Added
- **E2E 测试现在在 git 工作树中运行。**
- **收获去重。**
- **`describeWithWorktree()` 助手。**

### Changed
- **Gen-skill-docs 现在是模块化解析器管道。**
- **评估结果是项目范围的。**

### For contributors
- WorktreeManager、12 个新单元测试、`GLOBAL_TOUCHFILES` 更新。

## [0.11.12.0] - 2026-03-24 — 三声音 Autoplan

### Added
- **每个 autoplan 阶段的双声音。**
- **阶段级联上下文。**
- **结构化共识表。**
- **跨阶段综合。**
- **顺序强制。**
- **阶段过渡摘要。**
- **降级矩阵。**

## [0.11.11.0] - 2026-03-23 — 社区波次 3

### Added
- **Chrome 多配置文件 cookie 导入。**
- **Linux Chromium cookie 导入。**
- **浏览会话中的 Chrome 扩展。**
- **项目范围的 gstack 安装。**
- **分发管道检查。**
- **动态 skill 发现。**
- **自动触发守卫。**

### Fixed
- **浏览服务器启动崩溃。**
- **skill 前置中的 zsh 全局错误。**
- **`--force` 现在实际上强制升级。**
- **/review 范围漂移检测中的三点差异。**
- **CI 工作流 YAML 解析。**

### Community
- 感谢 @osc, @Explorer1092 等贡献者。

## [0.11.10.0] - 2026-03-23 — CI 评估在 Ubicloud 上

### Added
- **E2E 评估现在在每个 PR 上在 CI 中运行。**
- **3x 更快的评估运行。**
- **Docker CI 镜像。**

### Fixed
- **路由测试现在在 CI 中工作。**

### For contributors
- `EVALS_CONCURRENCY=40`、Ubicloud 运行器、`workflow_dispatch` 触发。

## [0.11.9.0] - 2026-03-23 — Codex Skill 加载修复

### Fixed
- **Codex 不再因"无效 SKILL.md"拒绝 gstack skills。**
- **`package.json` 版本现在与 `VERSION` 同步。**

### Added
- **Codex E2E 测试现在断言没有 skill 加载错误。**
- **README 中的 Codex 故障排除条目。**

### For contributors
- 验证测试、一次性迁移、P1 TODO。

## [0.11.8.0] - 2026-03-23 — zsh 兼容修复

### Fixed
- **gstack skills 现在在 zsh 中没有错误地工作。**

### Added
- **zsh 全局安全性的回归测试。**

## [0.11.7.0] - 2026-03-23 — /review → /ship 交接修复

### Fixed
- **`/review` 现在满足 ship 准备门控。**
- **Ship 中止提示现在提到两个审查选项。**

### For contributors
- 4 个新验证测试。

## [0.11.6.0] - 2026-03-23 — 基础设施优先安全审计

### Added
- **`/cso` v2。**
- **两种审计模式。**
- **主动验证。**
- **趋势跟踪。**
- **Diff 范围审计。**
- **3 个 E2E 测试。**

### Changed
- **扫描前的栈检测。**
- **正确的工具使用。**

## [0.11.5.2] - 2026-03-22 — 外部声音

### Added
- **计划审查现在提供独立第二意见。**
- **跨模型紧张检测。**
- **审查就绪仪表板中的外部声音。**

### Changed
- **`/plan-eng-review` Codex 集成升级。**

## [0.11.5.1] - 2026-03-23 — 内联 Office Hours

### Changed
- **不再"打开另一个窗口"用于 /office-hours。**
- **移除交接笔记基础设施。**

## [0.11.5.0] - 2026-03-23 — Bash 兼容修复

### Fixed
- **`gstack-review-read` 和 `gstack-review-log` 不再在 bash 下崩溃。**
- **所有 SKILL.md 模板已更新。**
- **添加了回归测试。**

## [0.11.4.0] - 2026-03-22 — Office Hours 中的 Codex

### Added
- **你的头脑风暴现在获得第二意见。**
- **设计文档中的跨模型视角。**
- **新的创始人信号：用推理捍卫的前提。**

## [0.11.3.0] - 2026-03-23 — 设计外部声音

### Added
- **每个设计审查现在获得第二意见。**
- **OpenAI 的设计硬性规则内置。**
- **每个 PR 中的 Codex 设计声音。**
- **/office-hours 头脑风暴中的外部声音。**
- **AI slop 黑名单提取为共享常量。**

## [0.11.2.0] - 2026-03-22 — Codex 正常工作

### Fixed
- **Codex 启动时不再显示"超过最大长度 1024 字符"。**
- **不再有重复的 skill 发现。**
- **旧的直接安装自动迁移。**
- **Sidecar 目录不再作为 skill 链接。**

### Added
- **Repo 本地 Codex 安装。**
- **Kiro CLI 支持。**
- **`.agents/` 现在被 gitignore。**

### Changed
- **`GSTACK_DIR` 重命名。**
- **CI 验证 Codex 生成。**

## [0.11.1.1] - 2026-03-22 — 计划文件始终显示审查状态

### Added
- **每个计划文件现在显示审查状态。**

## [0.11.1.0] - 2026-03-22 — 全局 Retro：跨项目 AI 编码回顾

### Added
- **`/retro global`。**
- **全局 retro 中每个项目的个人贡献。**
- **`gstack-global-discover`。**

### Fixed
- **发现脚本只读取会话文件的前几 KB。**
- **Claude Code 会话计数现在准确。**
- **周窗口现在是午夜对齐的。**

## [0.11.0.0] - 2026-03-22 — /cso：零噪声安全审计

### Added
- **`/cso` —— 你的首席安全官。**
- **零噪声假阳性过滤。**
- **独立的发现验证。**
- **`browse storage` 现在自动脱敏密钥。**
- **Azure 元数据端点被阻止。**

### Fixed
- **`gstack-slug` 针对 shell 注入加固。**
- **DNS 重绑定保护。**
- **并发服务器启动竞态修复。**
- **更智能的存储脱敏。**
- **CI 工作流 YAML lint 错误修复。**

### For contributors
- 社区 PR 分类流程、存储脱敏测试覆盖。

## [0.10.2.0] - 2026-03-22 — Autoplan 深度修复

### Fixed
- **`/autoplan` 现在产生全深度审查而非压缩为一行。**
- **CEO 和 Eng 阶段的执行清单。**
- **预门控验证捕获跳过的输出。**
- **测试审查永远不能被跳过。**

## [0.10.1.0] - 2026-03-22 — 测试覆盖目录

### Added
- **测试覆盖审计现在在所有地方工作——计划、ship 和审查。**
- **`/review` 步骤 4.75——测试覆盖图。**
- **内置的 E2E 测试建议。**
- **回归检测铁律。**
- **`/ship` 失败分类。**
- **测试框架自动检测。**

### Fixed
- **gstack 不再在没有 `origin` 远程的仓库中崩溃。**
- **`REPO_MODE` 在助手不输出任何内容时正确默认。**

## [0.10.0.0] - 2026-03-22 — Autoplan

### Added
- **`/autoplan` —— 一个命令，完全审查的计划。**

## [0.9.8.0] - 2026-03-21 — 部署管道 + E2E 性能

### Added
- **`/land-and-deploy`。**
- **`/canary`。**
- **`/benchmark`。**
- **`/setup-deploy`。**
- **`/review` 现在包含性能 & 包影响分析。**

### Changed
- **E2E 测试现在快 3-5 倍。**
- **所有 E2E 测试上的 `--retry 2`。**
- **`test:e2e:fast` 层级。**
- **E2E 计时遥测。**

### Fixed
- **`plan-design-review-plan-mode` 不再竞态。**
- **`ship-local-workflow` 不再浪费 6/15 回合。**
- **`design-consultation-core` 不再在同义词部分失败。**

## [0.9.7.0] - 2026-03-21 — 计划文件审查报告

### Added
- **每个计划文件现在显示哪些审查已运行。**
- **审查日志现在捕获更丰富的数据。**

## [0.9.6.0] - 2026-03-21 — 自动缩放的对抗审查

### Changed
- **审查彻底程度现在根据 diff 大小自动缩放。**
- **Claude 现在有对抗模式。**
- **审查仪表板显示"对抗"而非"Codex 审查"。**

## [0.9.5.0] - 2026-03-21 — 构建者精神

### Added
- **ETHOS.md。**
- **每个工作流 skill 现在在推荐之前搜索。**
- **尤里卡时刻。**
- **`/office-hours` 添加景观感知阶段。**
- **`/plan-eng-review` 添加搜索检查。**
- **`/investigate` 在假设失败时搜索。**
- **`/design-consultation` 三层综合。**
- **当交接给 `/office-hours` 时 CEO 审查保存上下文。**

## [0.9.4.1] - 2026-03-20

### Changed
- **`/retro` 不再对 PR 大小唠叨。**

## [0.9.4.0] - 2026-03-20 — Codex 审查默认开启

### Changed
- **Codex 代码审查现在在 `/ship` 和 `/review` 中自动运行。**
- **所有 Codex 操作使用最大推理能力。**
- **Codex 审查错误不会损坏仪表板。**
- **Codex 审查日志包括提交哈希。**

### Fixed
- **防止 Codex 用于 Codex 的递归。**

## [0.9.3.0] - 2026-03-20 — Windows 支持

### Fixed
- **gstack 现在在 Windows 11 上工作。**
- **路径处理在 Windows 上工作。**

### Added
- **用于 Node.js 的 Bun API polyfill。**
- **Node 服务器构建脚本。**

## [0.9.2.0] - 2026-03-20 — Gemini CLI E2E 测试

### Added
- **Gemini CLI 现在端到端测试。**
- **带 10 个单元测试的 Gemini JSONL 解析器。**
- **`bun run test:gemini` 和 `bun run test:gemini:all` 脚本。**

## [0.9.1.0] - 2026-03-20 — 对抗性规格审查 + Skill 链式

### Added
- **你的设计文档现在在给你看之前经过压力测试。**
- **头脑风暴期间的视觉线框。**
- **Skills 现在互相帮助。**
- **规格审查指标。**

## [0.9.0.1] - 2026-03-19

### Changed
- **遥测选择退出现在默认社区模式。**

### Fixed
- **审查日志和遥测现在在 plan mode 期间持久化。**

## [0.9.0] - 2026-03-19 — 适用于 Codex、Gemini CLI 和 Cursor

- **一个安装，四个 agent。**
- **自动检测。**
- **适配 Codex 的输出。**
- **CI 检查两个 host。**

## [0.8.6] - 2026-03-19

### Added
- **你现在可以看到如何使用 gstack。**
- **选择加入的社区遥测。**
- **社区健康仪表盘。**
- **通过更新检查跟踪安装基数。**
- **崩溃聚类。**
- **升级漏斗跟踪。**
- **/retro 现在显示你的 gstack 使用情况。**
- **特定于会话的待处理标记。**

## [0.8.5] - 2026-03-19

### Fixed
- **`/retro` 现在计算完整日历天数。**
- **审查日志不再在带有 `/` 的分支名上中断。**
- **所有 skill 模板现在是平台无关的。**
- **`/ship` 读取 CLAUDE.md 以发现测试命令。**

### Added
- **平台无关设计原则。**
- **CLAUDE.md 中的 `## Testing` 部分。**

## [0.8.4] - 2026-03-19

### Added
- **`/ship` 现在自动同步你的文档。**
- **文档中的六个新 skills。**
- **浏览交接在所有地方都有记录。**
- **主动建议知道所有 skills。**

## [0.8.3] - 2026-03-19

### Added
- **计划审查现在引导你到下一步。**
- **审查知道它们何时过时。**
- **`skip_eng_review` 在所有地方受到尊重。**
- **设计审查精简版现在也跟踪提交。**

### Fixed
- **浏览不再导航到危险 URL。**
- **安装脚本告诉你缺少什么。**
- **`/debug` 重命名为 `/investigate`。**
- **Shell 注入表面减少。**
- **25 个新的安全测试。**

## [0.8.2] - 2026-03-19

### Added
- **当无头浏览器卡住时交接给真正的 Chrome。**
- **3 次连续失败后自动交接提示。**
- **15 个交接功能的新测试。**

### Changed
- `recreateContext()` 重构。
- `browser.close()` 现在有 5 秒超时。

## [0.8.1] - 2026-03-19

### Fixed
- **`/qa` 不再拒绝在后端更改时使用浏览器。**

## [0.8.0] - 2026-03-19 — 多 AI 第二意见

- **`/codex` —— 获得来自完全不同的 AI 的独立第二意见。**
- **无处不在的集成。**
- **主动 skill 建议。**

## [0.7.4] - 2026-03-18

### Changed
- **`/qa` 和 `/design-review` 现在询问如何处理未提交的更改。**

## [0.7.3] - 2026-03-18

### Added
- **一条命令可以打开的安全护栏。**
- **用 `/freeze` 将一个文件夹的编辑锁定。**
- **`/guard` 同时激活两者。**
- **`/debug` 现在自动冻结正在调试的模块。**
- **你现在可以看到使用哪些 skills 以及使用频率。**
- **每周 retros 包括 skill 使用情况。**

## [0.7.2] - 2026-03-18

### Fixed
- `/retro` 日期范围现在对齐到午夜。
- `/retro` 时间戳现在使用本地时区。

## [0.7.1] - 2026-03-19

### Added
- **gstack 现在在自然时刻建议 skills。**
- **生命周期图。**
- **用自然语言选择退出。**
- **11 个旅程阶段 E2E 测试。**
- **触发短语验证。**

### Fixed
- `/debug` 和 `/office-hours` 对自然语言完全不可见。

## [0.7.0] - 2026-03-18 — YC Office Hours

- **`/office-hours` —— 在写一行代码之前与 YC 合伙人坐下来。**
- **`/debug` —— 找到根本原因，不是症状。**

## [0.6.4.1] - 2026-03-18

### Added
- **Skills 现在可以通过自然语言发现。**

## [0.6.4.0] - 2026-03-17

### Added
- **`/plan-design-review` 现在是交互式的——评分 0-10，修复计划。**
- **CEO 审查现在召唤设计师。**
- **15 个 skill 中有 14 个现在有完整测试覆盖。**
- **Bisect 提交风格。**

### Changed
- `/qa-design-review` 重命名为 `/design-review`。

## [0.6.3.0] - 2026-03-17

### Added
- **每个接触前端代码的 PR 现在自动获得设计审查。**
- **`gstack-diff-scope` 对分支中的变更进行分类。**
- **设计审查出现在审查就绪仪表板中。**
- **设计审查检测的 E2E 评估。**

## [0.6.2.0] - 2026-03-17

### Added
- **计划审查现在像世界上最好的人一样思考。**
- **潜在空间激活，不是清单。**

## [0.6.1.0] - 2026-03-17

### Added
- **E2E 和 LLM-judge 测试现在只运行你更改的内容。**
- **`bun run eval:select` 预览将运行哪些测试。**
- **完整性护栏捕获忘记的测试条目。**

### Changed
- `test:evals` 和 `test:e2e` 现在根据 diff 自动选择。

## 0.6.1 — 2026-03-17 — Boil the Lake

- **完整性评分。**
- **双重时间估计。**
- **反模式示例。**
- **首次引导。**
- **审查完整性缺口。**
- **Lake Score。**
- **CEO + Eng 审查双重时间。**

## 0.6.0.1 — 2026-03-17

- **`/gstack-upgrade` 现在自动捕获过时的 vendored 副本。**
- **升级同步更安全。**

### For contributors
- 独立使用部分、更新检查回退。

## 0.6.0 — 2026-03-17

- **100% 测试覆盖是出色 vibe 编码的关键。**
- **每个 bug 修复现在获得回归测试。**
- **自信 Ship——覆盖审计显示哪些被测试、哪些没有。**
- **你的 retro 跟踪测试健康。**
- **设计审查也生成回归测试。**

### For contributors
- 添加了 `generateTestBootstrap()` 解析器、Phase 8e.5 回归测试生成、步骤 3.4 测试覆盖审计、测试健康跟踪、`qa-only` 获得推荐说明、`qa-report-template.md` 获得回归测试部分、26 个新验证测试、2 个新 E2E 评估。

## 0.5.4 — 2026-03-17

- **工程审查现在是完整审查。**
- **Ship 在回答后不再询问审查。**

### For contributors
- 移除 SMALL_CHANGE / BIG_CHANGE 菜单、添加审查门控覆盖持久性、更新 2 个 E2E 测试。

## 0.5.3 — 2026-03-17

- **即使在做梦时你也处于掌控之中。**
- **新模式：SELECTIVE EXPANSION。**
- **你的 CEO 审查愿景被保存，不会丢失。**
- **更智能的 ship 门控。**

### For contributors
- 添加 SELECTIVE EXPANSION 模式、重写 EXPANSION 模式、添加 CEO 计划持久性、模式快速参考表扩展、新的测试。

## 0.5.2 — 2026-03-17

- **你的设计顾问现在承担创造性风险。**
- **在选择之前看到景观。**
- **看起来像你的产品的预览页面。**

## 0.5.1 — 2026-03-17

- **在 Ship 之前知道你的立场。**
- **`/ship` 在创建 PR 之前检查你的审查。**
- **少一件要复制粘贴的事。**
- **截图现在在 QA 和浏览会话中可见。**

### For contributors
- 添加 `{{REVIEW_DASHBOARD}}` 解析器、添加 `bin/gstack-slug` 助手、新的 TODO。

## 0.5.0 — 2026-03-16

- **你的站点刚刚获得设计审查。**
- **它也可以修复发现的问题。**
- **了解你的实际设计系统。**
- **AI Slop 检测是标题指标。**
- **设计回归跟踪。**
- **80 项设计审计清单。**

### For contributors
- 添加 `{{DESIGN_METHODOLOGY}}` 解析器等。

## 0.4.5 — 2026-03-16

- **审查发现现在实际上被修复，不只是列出。**
- **你控制"只管修复"和"先问我"之间的界限。**

### Fixed
- **`$B js` 现在工作。**
- **点击下拉选项不再永远挂起。**
- **当 click 是错误工具时，gstack 告诉你。**

### For contributors
- 门分类→严重性分类重命名、修复优先启发式部分、新的验证测试、提取的助手函数。

## 0.4.4 — 2026-03-16

- **新发布在不到一小时内检测到，不再是半天。**
- **`/gstack-upgrade` 始终检查真实性。**

### For contributors
- 分割 `last-update-check` 缓存 TTL、添加 `--force` 标志、3 个新测试。

## 0.4.3 — 2026-03-16

- **新的 `/document-release` skill。**
- **每个问题现在都清晰明了。**
- **分支名始终正确。**

### For contributors
- 合并 ELI16 规则、添加 `_BRANCH` 检测、添加回归防护测试。

## 0.4.2 — 2026-03-16

- **`$B js "await fetch(...)"` 现在正常工作。**
- **贡献者模式现在反思，不仅仅是反应。**
- **Skills 现在尊重你的分支目标。**
- **`/retro` 在任何默认分支上工作。**
- **新的 `{{BASE_BRANCH_DETECT}}` 占位符。**
- **3 个新的 E2E 冒烟测试。**

### For contributors
- 添加 `hasAwait()` 助手、智能评估包装、6 个新的异步包装单元测试、40 个新的贡献者模式前置验证测试。

## 0.4.1 — 2026-03-16

- **gstack 现在注意到它搞砸的时候。**
- **同时处理多个会话？gstack 跟得上。**
- **每个问题现在都带有推荐。**
- **/review 现在捕获被遗忘的枚举处理器。**

### For contributors
- 重命名 `{{UPDATE_CHECK}}` 为 `{{PREAMBLE}}`、DRY 化了问题格式化。

## 0.4.0 — 2026-03-16

### Added
- **QA-only skill（`/qa-only`）**——仅报告的 QA 模式。
- **QA 修复循环**——`/qa` 现在运行发现-修复-验证循环。
- **计划到 QA 工件流**。
- **`{{QA_METHODOLOGY}}` DRY 占位符。**
- **评估效率指标。**
- **`generateCommentary()` 引擎。**
- **评估列表列。**
- **每个测试的评估摘要。**
- **`judgePassed()` 单元测试。**
- **3 个新的 E2E 测试。**
- **浏览器 ref 过时检测。**
- 3 个新的快照测试。

### Changed
- QA skill 提示重组。
- `formatComparison()` 现在显示每测试轮次和持续时间变化。
- `printSummary()` 显示轮次和持续时间的列。

### Fixed
- 浏览器 ref 过时。

## 0.3.9 — 2026-03-15

### Added
- **`bin/gstack-config` CLI。**
- **智能更新检查。**
- **自动升级模式。**
- **4 选项升级提示。**
- **Vendored 副本同步。**
- 25 个新测试。

### Changed
- README 升级/故障排除部分简化。

## 0.3.8 — 2026-03-14

### Added
- **TODOS.md 作为单一事实来源。**
- **`/ship` 步骤 5.5：TODOS.md 管理。**
- **跨 skill 的 TODOS 感知。**
- **共享 `review/TODOS-format.md`。**
- **Greptile 2 层回复系统。**
- **Greptile 回复模板。**
- **Greptile 升级检测。**
- **Greptile 严重性重新排名。**
- `TODOS-format.md` 引用的静态验证测试。

### Fixed
- **`.gitignore` 追加失败被静默吞没。**

### Changed
- `TODO.md` 被删除。
- `/ship` 和 `/review` 现在引用回复模板。

## 0.3.7 — 2026-03-14

### Added
- **截图元素/区域裁剪。**
- 10 个新测试。

## 0.3.6 — 2026-03-14

### Added
- **E2E 可观察性。**
- **`bun run eval:watch`。**
- **增量评估保存。**
- **机器可读诊断。**
- **API 连接性预检。**
- **`is_error` 检测。**
- **Stream-json NDJSON 解析器。**
- **评估持久化。**
- **评估 CLI 工具。**
- **所有 9 个 skills 转换为 `.tmpl` 模板。**
- **3 层评估套件。**
- **植入 bug 结果测试。**
- 15 个可观察性单元测试。
- plan-ceo-review、plan-eng-review、retro skills 的 E2E 测试。
- 更新检查退出码回归测试。

### Fixed
- **浏览二进制发现对 agent 损坏。**
- **更新检查退出码 1 误导 agent。**
- **browse/SKILL.md 缺少安装块。**
- **plan-ceo-review 超时。**
- 植入的 bug 评估可靠性。

### Changed
- **模板系统扩展。**
- 丰富了 14 个命令描述。
- 安装块检查工作区本地路径。
- LLM 评估 judge 从 Haiku 升级到 Sonnet 4.6。

## 0.3.3 — 2026-03-13

### Added
- **SKILL.md 模板系统。**
- **命令注册表。**
- **快照标志元数据。**
- **1 层静态验证——43 个测试。**
- **2 层 E2E 测试。**
- **3 层 LLM-as-judge 评估。**
- **`bun run skill:check`。**
- **`bun run dev:skill`。**
- **CI 工作流。**
- `bun run gen:skill-docs` 脚本。
- `bun run test:eval`。
- `test/helpers/skill-parser.ts`。
- `test/helpers/session-runner.ts`。
- **ARCHITECTURE.md。**
- **Conductor 集成。**
- **`.env` 传播。**
- `.env.example` 模板。

### Changed
- 构建现在在编译二进制文件之前运行 `gen:skill-docs`。
- `parseSnapshotArgs` 是元数据驱动的。
- `server.ts` 从 `commands.ts` 导入命令集。

## 0.3.2 — 2026-03-13

### Fixed
- Cookie 导入选择器现在返回 JSON 而非 HTML。
- `help` 命令正确路由。
- 来自全局安装的过时服务器不再遮蔽本地更改。
- 崩溃日志路径更新。

### Added
- **感知 diff 的 QA 模式。**
- **项目本地浏览状态。**
- **共享配置模块。**
- **随机端口选择。**
- **二进制版本跟踪。**
- **旧版 /tmp 清理。**
- **Greptile 集成。**
- **本地开发模式。**
- `help` 命令。
- 路由级测试。

### Changed
- 状态文件位置。
- 日志文件位置。
- 原子状态文件写入。
- CLI 传递 `BROWSE_STATE_FILE`。

### Removed
- `CONDUCTOR_PORT` 魔法偏移。
- 端口扫描范围 9400-9409。
- 旧版回退。
- `DEVELOPING_GSTACK.md`。

## 0.3.1 — 2026-03-12

### Phase 3.5：浏览器 Cookie 导入

- `cookie-import-browser` 命令。
- 交互式 cookie 选择器 Web UI。
- 带 `--domain` 标志的直接 CLI 导入。
- `/setup-browser-cookies` skill。
- macOS Keychain 访问。
- 18 个单元测试。

## 0.3.0 — 2026-03-12

### Phase 3：/qa skill——系统化 QA 测试

- 新的 `/qa` skill，6 阶段工作流。
- 三种模式。
- 问题分类法。
- 结构化报告模板。
- 框架检测指导。

### Phase 2：增强浏览器

- 对话框处理。
- 文件上传。
- 元素状态检查。
- 带 ref 标签的注解截图。
- 快照差异。
- 游标交互元素扫描。
- 等待标志。
- 166 个集成测试。

## 0.0.2 — 2026-03-12

- 修复项目本地 `/browse` 安装。
- `setup` 重建过时二进制文件。
- 修复 `chain` 命令吞没真实错误。
- 修复服务器崩溃时的无限制重启循环。
- 限制控制台/网络缓冲区为 50K 条目。
- 修复磁盘刷新在缓冲区达到 50K 上限后静默停止。
- 修复 setup 中的 `ln -snf`。
- 使用 `git fetch && git reset --hard` 替代 `git pull` 进行升级。
- 简化安装。
- 重构 README。
- 六个 skills。

## 0.0.1 — 2026-03-11

初始发布。

- 五个 skills。
- 带 40+ 命令的无头浏览器 CLI。
- 一键安装。
- `setup` 脚本。
