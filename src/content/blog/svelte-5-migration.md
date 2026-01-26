---
title: 'Svelte 5 迁移实战：ikun-ui 的重构之路'
description: '深度解析 ikun-ui 如何适配 Svelte 5：从 Runes API 重构到 Vitest 测试迁移，以及遇到的那些坑。'
pubDate: 'Dec 21 2025'
tags: ["Archive"]
---

随着 Svelte 5 正式发布，带来了革命性的 Runes API 和底层的重大重构。为了保持技术的先进性，我最近对 `ikun-ui` 组件库进行了一次彻底的升级。这不仅仅是一次版本号的提升，更是一次对代码库的全面现代化改造。

截止目前，我们的迁移进度如下：
- **基础设施**: ✅ 100% (Svelte 5 + Vite 6 + ESLint 9)
- **组件兼容性**: ✅ 98% (71 个组件已通过构建)
- **单元测试**: 🟡 81% (438/541 测试通过)

本文将分享这一过程中的核心技术细节。

## 1. 基础设施升级：Vite 6 与 ESLint 9

在开始代码迁移前，我们首先升级了底层的构建工具链。`ikun-ui` 现在全面运行在 **Vite 6** 之上，配合 **Svelte 5** 的编译器。同时，我们将代码规范工具升级到了 **ESLint 9**，这不仅带来了更严格的检查，也修复了旧版本中的诸多潜在问题。

## 2. 单元测试迁移：告别 `$on`，拥抱 Callback Props

测试套件的迁移是本次升级中最耗时的工作。Svelte 5 彻底改变了组件的挂载和事件处理机制，导致我们原有的 `testing-library` 测试大量失效。

### `render` vs `mount`

官方推荐使用新的 `mount` API 代替旧的 `render`。这一变化要求我们重写大部分组件的测试脚手架。

**旧代码 (Svelte 4):**
```javascript
import { render } from '@testing-library/svelte';
import Button from './Button.svelte';

test('click event', async () => {
	const { component } = render(Button);
	let clicked = false;
	component.$on('click', () => clicked = true);
	// ... trigger click
});
```

**新代码 (Svelte 5):**
```javascript
import { mount } from 'svelte';
import Button from './Button.svelte';

test('click event', async () => {
    let clicked = false;
    const target = document.createElement('div');
    const component = mount(Button, {
        target,
        props: {
            onclick: () => clicked = true // 直接传入回调
        }
    });
	// ... trigger click
});
```

### 移除 `svelte/internal` Mock

在 Svelte 4 时代，为了测试某些内部状态，我们曾 hack 了一些 `svelte/internal` 的 mock。在 Svelte 5 中，这些内部 API 已完全改变或移除。我们通过**重构代码逻辑**，移除了所有对内部 API 的依赖，让测试更加纯粹和健壮。像 `Input`、`ColorPicker` 和 `Backtop` 等组件都因此受益，变得更加易于维护。

## 3. 组件重构：Runes 与 Callback Props

为了充分利用 Svelte 5 的性能，我们不仅仅是“修补”组件，而是对核心组件进行了重写。

- **AutoComplete & Switch**: 这两个组件被完全重写，使用了 Runes (`$state`, `$derived`, `$effect`) 来管理状态，并全面转向 callback props (如 `onchange`) 替代事件分发。
- **Form 表单系统**: 这是迁移中最复杂的模块。旧的表单验证严重依赖上下文和 `$on` 事件流。在迁移中，我们面临 `initValue` 初始化时机和事件冒泡机制变更的双重挑战。目前的策略是使用 **Snippets** 替代旧的 Slots，并显式传递上下文对象。

## 4. 遇到的坑与解决方案

### `document is not defined`
在 Vitest 环境配置中，Svelte 5 对 `jsdom` 的集成有了细微变化。必须严格确保环境配置正确，否则任何涉及 DOM 操作的 `mount` 都会失败。我们在 `vitest.config.ts` 中针对 Svelte 5 进行了专项调优。

### Fixtures 与源码引用
在测试 `RadioGroup` 和 `Form` 等复合组件时，我们发现旧的测试 fixtures (测试用例组件) 无法被 Svelte 5 正确编译。解决方案是将 fixtures 显式地迁移为 Svelte 5 语法，确保测试环境与生产环境一致。

## 5. 展望

虽然目前还有约 20% 的测试用例（主要是 `Menu`, `Tour`, `Tabs` 等复杂组件）需要修复，但最艰难的时刻已经过去。

接下来的计划：
1.  **攻克剩余测试**: 重点解决 Menu 和 Tour 的交互测试。
2.  **完善 Form 逻辑**: 彻底解决表单初始化和重置的边缘情况。
3.  **文档更新**: 更新官网文档，向用户介绍新的 API 使用方式。

Svelte 5 带来的性能提升和开发体验改进是显而易见的，这次痛苦的迁移绝对物超所值。

---
*Happy Coding! 🚀*
