# 设计系统 — gstack

## 产品背景
- **这是什么：** gstack 社区网站 — gstack 是一款将 Claude Code 转变为虚拟工程团队的 CLI 工具
- **目标用户：** 发现 gstack 的开发者、现有社区成员
- **领域/行业：** 开发者工具（同类：Linear、Raycast、Warp、Zed）
- **项目类型：** 社区仪表板 + 营销网站

## 美学方向
- **风格定位：** 工业风/实用主义 — 功能优先、数据密集、等宽字体作为个性字体
- **装饰程度：** 刻意经营 — 表面添加微妙的噪点/纹理质感以增强材质感
- **氛围：** 由注重匠心的开发者打造的严肃工具。温暖而非冰冷。CLI 的传承就是品牌。
- **参考站点：** formulae.brew.sh（竞品，但我们的是实时交互的）、Linear（暗色 + 克制）、Warp（暖色点缀）

## 字体
- **展示/主标题：** Satoshi（Black 900 / Bold 700）— 几何中带有温度，独特的字母形态（小写 'a' 和 'g'）。不是 Inter，不是 Geist。从 Fontshare CDN 加载。
- **正文：** DM Sans（Regular 400 / Medium 500 / Semibold 600）— 干净、易读，比几何展示字体略友好。从 Google Fonts 加载。
- **UI/标签：** DM Sans（与正文相同）
- **数据/表格：** JetBrains Mono（Regular 400 / Medium 500）— 个性字体。支持等宽数字。等宽字体应该突出展示，不应只隐藏在代码块中。从 Google Fonts 加载。
- **代码：** JetBrains Mono
- **加载方式：** Google Fonts 加载 DM Sans + JetBrains Mono，Fontshare 加载 Satoshi。使用 `display=swap`。
- **字号比例：**
  - Hero: 72px / clamp(40px, 6vw, 72px)
  - H1: 48px
  - H2: 32px
  - H3: 24px
  - H4: 18px
  - 正文: 16px
  - 小字: 14px
  - 说明: 13px
  - 微小: 12px
  - 纳米: 11px（JetBrains Mono 标签）

## 颜色
- **策略：** 克制 — 琥珀色强调点稀有而有意义。仪表板数据使用色彩；框架保持中性。
- **主色（暗色模式）：** amber-500 #F59E0B — 温暖、有活力，读起来像"终端光标"
- **主色（亮色模式）：** amber-600 #D97706 — 在白色背景下更深以确保对比度
- **主文字点缀色（暗色模式）：** amber-400 #FBBF24
- **主文字点缀色（亮色模式）：** amber-700 #B45309
- **中性色：** 冷色调锌灰色
  - zinc-50: #FAFAFA（最亮）
  - zinc-400: #A1A1AA
  - zinc-600: #52525B
  - zinc-800: #27272A
  - 表面色（暗色）：#141414
  - 基底色（暗色）：#0C0C0C
  - 表面色（亮色）：#FFFFFF
  - 基底色（亮色）：#FAFAF9
- **语义色：** 成功 #22C55E、警告 #F59E0B、错误 #EF4444、信息 #3B82F6
- **暗色模式：** 默认。近黑色基底（#0C0C0C），表面卡片 #141414，边框 #262626。
- **亮色模式：** 暖石色基底（#FAFAF9），白色表面卡片，石色边框（#E7E5E4）。琥珀色强调移至 amber-600 以增强对比度。

## 间距
- **基础单位：** 4px
- **密度：** 舒适 — 不拥挤（不是 Bloomberg Terminal），不宽松（不是营销网站）
- **比例：** 2xs(2px) xs(4px) sm(8px) md(16px) lg(24px) xl(32px) 2xl(48px) 3xl(64px)

## 布局
- **策略：** 仪表板使用网格规范，着陆页使用编辑风格 Hero
- **网格：** 大尺寸以上 12 列，移动端 1 列
- **最大内容宽度：** 1200px (6xl)
- **圆角：** sm:4px, md:8px, lg:12px, full:9999px
  - 卡片/面板：lg (12px)
  - 按钮/输入框：md (8px)
  - 标签/胶囊：full (9999px)
  - Skill 条：sm (4px)

## 动效
- **策略：** 极简功能性 — 仅使用有助于理解的过渡效果。仪表板的实时数据流本身就是动效。
- **缓动：** 进入 (ease-out / cubic-bezier(0.16,1,0.3,1)) 退出 (ease-in) 移动 (ease-in-out)
- **时长：** 微动效(50-100ms) 短(150ms) 中(250ms) 长(400ms)
- **动画元素：** 实时数据流脉冲（2s 无限循环）、Skill 条填充（600ms ease-out）、悬停状态（150ms）

## 纹理质感
为整个页面添加微妙的噪点叠加以增强材质感：
- 暗色模式：不透明度 0.03
- 亮色模式：不透明度 0.02
- 使用 SVG feTurbulence 滤镜作为 body::after 的 CSS background-image
- pointer-events: none, position: fixed, z-index: 9999

## 决策记录
| 日期 | 决策 | 理由 |
|------|------|------|
| 2026-03-21 | 初始设计系统 | 由 /design-consultation 创建。工业美学，温暖琥珀色点缀，Satoshi + DM Sans + JetBrains Mono。 |
| 2026-03-21 | 亮色模式 amber-600 | amber-500 在白色背景上过于明亮/褪色；amber-700 过于偏棕/土黄。amber-600 是最佳选择。 |
| 2026-03-21 | 纹理质感 | 为扁平暗色表面增添材质感。避免陷入"通用 SaaS 模板"的同质化。 |
