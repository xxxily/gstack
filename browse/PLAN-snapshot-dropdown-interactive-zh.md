# 计划：Snapshot 下拉/自动补全交互元素检测

## 问题

`snapshot -i` 在现代 Web 应用中会遗漏下拉/自动补全项。这些元素：
1. 通常是带有点击处理函数但没有语义 ARIA 角色的 `<div>`/`<li>`
2. 存在于动态创建的 portal/popover（浮动容器）内部
3. 不出现在 Playwright 的可访问性树（`ariaSnapshot()`）中

`-C` 标志（cursor-interactive 扫描）就是为此设计的，但是：
- 需要单独的 flag —— 使用 `-i` 的 Agent 不会自动获得它
- 跳过那些有 ARIA role 的元素（即使 ARIA 树也遗漏了它们）
- 不会优先检测下拉项所在的 popover/portal 容器

## 根因

Playwright 的 `ariaSnapshot()` 基于浏览器的可访问性树构建。动态渲染的 popover（React portals、Radix Popover 等）可能不在可访问性树中，如果：
- 组件没有设置 ARIA 角色
- portal 渲染在初始 `body` 定位器子树之外的时机问题
- 浏览器在 DOM 变更后还没有更新可访问性树

## 变更

### 1. 使用 `-i` 标志时自动启用 cursor-interactive 扫描

**文件：** `browse/src/snapshot.ts`

当传入 `-i`（interactive）时，自动包含 cursor-interactive 扫描。这意味着当 Agent 请求交互元素时，始终能看到可点击的非 ARIA 元素。

`-C` 标志仍作为独立选项，用于非交互 snapshot。

```
if (opts.interactive) {
  opts.cursorInteractive = true;
}
```

### 2. 添加 popover/portal 优先扫描

**文件：** `browse/src/snapshot.ts`（在 cursor-interactive evaluate 块内）

在通用的 cursor:pointer 扫描之前，专门扫描可见的浮动容器（popover、下拉菜单、菜单）并将其所有直接子元素包含为交互元素：

浮动容器的检测启发式规则：
- `position: fixed` 或 `position: absolute` 且 `z-index >= 10`
- 具有 `role="listbox"`、`role="menu"`、`role="dialog"`、`role="tooltip"`、`[data-radix-popper-content-wrapper]`、`[data-floating-ui-portal]` 等
- 最近在 DOM 中出现（不在初始页面加载时）
- 可见（`offsetParent !== null` 或 `position: fixed`）

对于每个浮动容器，包含满足以下条件的子元素：
- 有文本内容
- 可见
- 具有 cursor:pointer 或 onclick 或 role="option" 或 role="menuitem"
- 标记原因为 `popover-child` 以便清晰

### 3. 移除 cursor-interactive 扫描中的 `hasRole` 跳过

**文件：** `browse/src/snapshot.ts`

当前：`if (hasRole) continue;` —— 跳过任何具有 ARIA role 的元素，假设 ARIA 树已捕获它。

问题：如果 ARIA 树**遗漏**了该元素（时机、portal、不良 DOM 结构），它就会在两个系统之间都漏掉。

修复：仅当元素的 role 在 `INTERACTIVE_ROLES` 中**且**它确实被主 refMap 捕获时才跳过。否则包含它。

由于我们无法轻易在 `page.evaluate()` 内部检查 refMap，更简单的修复：对于检测到的浮动容器内的元素，完全移除 `hasRole` 跳过。对于浮动容器外的元素，保持 `hasRole` 跳过不变（以避免在正常页面内容中产生重复）。

### 4. 添加下拉测试 fixture 和测试

**文件：** `browse/test/fixtures/dropdown.html`

HTML 页面包含：
- 一个 combobox 输入，在 focus/type 时显示下拉菜单
- 下拉项作为带有点击处理函数的 `<div>`（无 ARIA 角色）
- 下拉项作为带有 `role="option"` 的 `<li>`
- 一个 React-portal 风格的容器（`position: fixed`，高 z-index）

**文件：** `browse/test/snapshot.test.ts`

新测试用例：
- 在下拉页面上使用 `snapshot -i`，通过 cursor 扫描找到下拉项
- 在下拉页面上使用 `snapshot -i`，包含 popover-child 元素
- 来自下拉扫描的 `@c` refs 可点击
- 即使在 ARIA 树遗漏时，也能捕获浮动容器内带有 ARIA role 的元素

## 发布风险

**低。** `-C` 扫描是增量式的 —— 它只添加 `@c` refs，从不移除 `@e` refs。将其自动与 `-i` 一起启用会增加输出大小，但 Agent 已经能处理混合 ref 类型。

**一个关注点：** `-C` 扫描查询**所有**元素（`document.querySelectorAll('*')`），在重型页面上可能较慢。对于 popover 专用扫描，我们限制在检测到的浮动容器内的元素，这部分很快（小子树）。

## 测试

```bash
cd /data/gstack/browse && bun test snapshot
```

## 变更的文件

1. `browse/src/snapshot.ts` —— 使用 -i 时自动启用 -C，popover 扫描，移除浮动容器内的 hasRole 跳过
2. `browse/test/fixtures/dropdown.html` —— 新测试 fixture
3. `browse/test/snapshot.test.ts` —— 新的下拉/popover 测试用例
