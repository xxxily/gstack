# 为 gstack 添加新 Host

gstack 使用声明式的 host 配置系统。每个受支持的 AI 编码代理（Claude、Codex、Factory、Kiro、OpenCode、Slate、Cursor、OpenClaw）都通过一个类型化的 TypeScript 配置对象定义。添加新 host 只需要创建一个文件并重新导出它。对生成器、安装脚本或工具无需进行任何代码更改。

## 工作原理

```
hosts/
├── claude.ts        # 主 host
├── codex.ts         # OpenAI Codex CLI
├── factory.ts       # Factory Droid
├── kiro.ts          # Amazon Kiro
├── opencode.ts      # OpenCode
├── slate.ts         # Slate（Random Labs）
├── cursor.ts        # Cursor
├── openclaw.ts      # OpenClaw（混合模式：config + adapter）
└── index.ts         # 注册表：导入所有 host，派生 Host 类型
```

每个配置文件导出一个 `HostConfig` 对象，告诉生成器：
- 生成的 Skill 放哪里（路径）
- 如何转换 frontmatter（允许列表/拒绝列表字段）
- 要重写哪些 Claude 相关的引用（路径、工具名）
- 检测哪个二进制文件以实现自动安装
- 要抑制哪些 resolver 章节
- 安装时要创建哪些符号链接

生成器、安装脚本、平台检测、卸载脚本、健康检查、worktree 拷贝和测试都从这些配置中读取。它们中没有一个包含针对特定 host 的代码。

## 分步：添加新 host

### 1. 创建配置文件

复制一个现有配置作为起点。`hosts/opencode.ts` 是一个最简示例。`hosts/factory.ts` 展示了工具重写和条件字段。`hosts/openclaw.ts` 展示了针对不同工具模型的 host 的 adapter 模式。

创建 `hosts/myhost.ts`：

```typescript
import type { HostConfig } from '../scripts/host-config';

const myhost: HostConfig = {
  name: 'myhost',
  displayName: 'MyHost',
  cliCommand: 'myhost',        // 用于 `command -v` 检测的二进制文件名
  cliAliases: [],              // 替代的二进制文件名

  globalRoot: '.myhost/skills/gstack',
  localSkillRoot: '.myhost/skills/gstack',
  hostSubdir: '.myhost',
  usesEnvVars: true,           // 仅 Claude 为 false（使用字面量 ~ 路径）

  frontmatter: {
    mode: 'allowlist',         // 'allowlist' 只保留列出的字段
    keepFields: ['name', 'description'],
    descriptionLimit: null,    // 对于有限制的 host 可设为 1024
  },

  generation: {
    generateMetadata: false,   // 仅 Codex 为 true（openai.yaml）
    skipSkills: ['codex'],     // codex Skill 仅限 Claude
  },

  pathRewrites: [
    { from: '~/.claude/skills/gstack', to: '~/.myhost/skills/gstack' },
    { from: '.claude/skills/gstack', to: '.myhost/skills/gstack' },
    { from: '.claude/skills', to: '.myhost/skills' },
  ],

  runtimeRoot: {
    globalSymlinks: ['bin', 'browse/dist', 'browse/bin', 'gstack-upgrade', 'ETHOS.md'],
    globalFiles: { 'review': ['checklist.md', 'TODOS-format.md'] },
  },

  install: {
    prefixable: false,
    linkingStrategy: 'symlink-generated',
  },

  learningsMode: 'basic',
};

export default myhost;
```

### 2. 注册到 index

编辑 `hosts/index.ts`：

```typescript
import myhost from './myhost';

// 添加到 ALL_HOST_CONFIGS 数组：
export const ALL_HOST_CONFIGS: HostConfig[] = [
  claude, codex, factory, kiro, opencode, slate, cursor, openclaw, myhost
];

// 添加到重新导出：
export { claude, codex, factory, kiro, opencode, slate, cursor, openclaw, myhost };
```

### 3. 添加到 .gitignore

将 `.myhost/` 添加到 `.gitignore`（生成的 Skill 文档会被 git 忽略）。

### 4. 生成和验证

```bash
# 为新 host 生成 Skill 文档
bun run gen:skill-docs --host myhost

# 验证输出是否存在且没有 .claude/skills 泄漏
ls .myhost/skills/gstack-*/SKILL.md
grep -r ".claude/skills" .myhost/skills/ | head -5
# （应该为空）

# 为所有 host 生成（包含新的 host）
bun run gen:skill-docs --host all

# 健康面板会显示新的 host
bun run skill:check
```

### 5. 运行测试

```bash
bun test test/gen-skill-docs.test.ts
bun test test/host-config.test.ts
```

参数化的冒烟测试会自动拾取新的 host。无需编写任何测试代码。它们会验证：输出存在、无路径泄漏、frontmatter 有效、新鲜度检查通过、codex Skill 被排除。

### 6. 更新 README.md

在适当的部分添加新 host 的安装说明。

## 配置字段参考

完整 `HostConfig` 接口参见 `scripts/host-config.ts`，每个字段都有 JSDoc 注释。

关键字段：

| 字段 | 用途 |
|-------|---------|
| `frontmatter.mode` | `allowlist`（只保留列出的字段）或 `denylist`（剥离列出的字段） |
| `frontmatter.descriptionLimit` | 最大字符数，`null` 表示无限制 |
| `frontmatter.descriptionLimitBehavior` | `error`（构建失败）、`truncate`（截断）、`warn`（警告） |
| `frontmatter.conditionalFields` | 基于模板值添加字段（例如 sensitive → disable-model-invocation） |
| `frontmatter.renameFields` | 重命名模板字段（例如 voice-triggers → triggers） |
| `pathRewrites` | 内容的字面量 replaceAll。顺序很重要。 |
| `toolRewrites` | 重写 Claude 工具名（例如 "use the Bash tool" → "run this command"） |
| `suppressedResolvers` | 对此 host 返回空内容的 resolver 函数 |
| `coAuthorTrailer` | Git commit 的 co-author 标识 |
| `boundaryInstruction` | 跨模型调用时的反 prompt 注入警告 |
| `adapter` | 用于复杂变换的 adapter 模块路径 |

## Adapter 模式（适用于不同工具模型的 host）

如果字符串替换的工具重写不够用（host 具有根本不同的工具语义），请使用 adapter 模式。参见 `hosts/openclaw.ts` 和 `scripts/host-adapters/openclaw-adapter.ts`。

Adapter 在所有通用重写之后作为后处理步骤运行。它导出 `transform(content: string, config: HostConfig): string`。

## 验证

`scripts/host-config.ts` 中的 `validateHostConfig()` 函数检查：
- 名称：小写字母数字加连字符
- CLI 命令：字母数字加连字符/下划线
- 路径：仅为安全字符（字母数字、`.`、`/`、`$`、`{}`、`~`、`-`、`_`）
- 配置之间没有重复的名称、hostSubdirs 或 globalRoots

运行 `bun run scripts/host-config-export.ts validate` 来检查所有配置。
