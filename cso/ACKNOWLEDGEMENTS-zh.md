# 致谢

/cso v2 参考了安全审计领域的多项研究成果。鸣谢如下：

- **[Sentry Security Review](https://github.com/getsentry/skills)** — 基于置信度的报告系统（仅报告 HIGH 置信度的发现）以及"先研究后报告"的方法论（追踪数据流、检查上游验证），验证了我们 8/10 的每日置信度门槛。TimOnWeb 在测试的 5 个安全 Skill 中评价它是唯一值得安装的一个。
- **[Trail of Bits Skills](https://github.com/trailofbits/skills)** — 审计上下文构建方法论（在寻找 bug 之前先建立心理模型）直接启发了 Phase 0。其变体分析概念（发现一个漏洞后搜索整个代码库中相同的模式）启发了 Phase 12 的变体分析步骤。
- **[Shannon by Keygraph](https://github.com/KeygraphHQ/shannon)** — 自主 AI 渗透测试工具，在 XBOW 基准测试中达到 96.15%（104 个 exploit 中成功 100 个）。验证了 AI 可以做真正的安全测试，而非仅仅是清单扫描。我们的 Phase 12 主动验证是该工具在静态分析中的版本。
- **[afiqiqmal/claude-security-audit](https://github.com/afiqiqmal/claude-security-audit)** — AI/LLM 专属安全检查（prompt injection、RAG 投毒、tool calling 权限）启发了 Phase 7。其框架级自动检测（检测"Next.js"而非仅"Node/TypeScript"）启发了 Phase 0 的框架检测步骤。
- **[Snyk ToxicSkills 研究](https://snyk.io/blog/toxicskills-malicious-ai-agent-skills-clawhub/)** — 发现 36% 的 AI agent skill 存在安全缺陷，13.4% 是恶意的，这启发了 Phase 8（Skill 供应链扫描）。
- **[Daniel Miessler 的个人 AI 基础设施](https://github.com/danielmiessler/Personal_AI_Infrastructure)** — 事件响应 playbook 和保护文件概念影响了补救措施和 LLM 安全阶段。
- **[McGo/claude-code-security-audit](https://github.com/McGo/claude-code-security-audit)** — 生成可共享报告和可操作 epic 的想法影响了我们的报告格式演进。
- **[Claude Code Security Pack](https://dev.to/myougatheaxo/automate-owasp-security-audits-with-claude-code-security-pack-4mah)** — 模块化方法（独立的 /security-audit、/secret-scanner、/deps-check skill）验证了这些是不同的关注点。我们的统一方法牺牲了模块化以换取跨阶段推理能力。
- **[Anthropic Claude Code Security](https://www.anthropic.com/news/claude-code-security)** — 多阶段验证和置信度评分验证了我们的并行发现验证方法。在开源项目中发现 500+ 个零日漏洞。
- **[@gus_argon](https://x.com/gus_aragon/status/2035841289602904360)** — 识别出 v1 的关键盲点：没有栈检测（使用全语言模式）、使用 bash grep 而非 Claude Code 的 Grep 工具、`| head -20` 静默截断结果，以及 preamble 臃肿。这些直接塑造了 v2 的栈优先方法和 Grep 工具强制要求。
