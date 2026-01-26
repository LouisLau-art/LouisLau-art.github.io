---
title: 'SvelteKit 重构之旅：救赎 ikun-ui 与前端大迁徙'
description: '记录从 Nuxt 4 迁移到 SvelteKit 的过程，以及如何解决 ikun-ui 在 Svelte 5 下的兼容性危机。'
pubDate: 'Dec 20 2025'
tags: ["Archive"]
---

在一个阳光明媚的下午，我们做出了一个重大的决定：将原有的 **Nuxt 4 + Una UI** 项目完全推倒重来，迁移到 **SvelteKit + ikun-ui** 的技术栈上。这不仅仅是一次简单的框架切换，更是一场关于技术选型、填坑与工程化的冒险。

## 1. 为什么选择 SvelteKit？

虽然 Nuxt 4 非常强大，但 Svelte 5 的 "Runes" 响应式系统和简洁的语法深深吸引了我们。我们希望打造一个更轻量、更直观的 "智慧校园失物招领平台"。

我们的新架构确认如下：

- **框架**: SvelteKit (Svelte 5)
- **数据库**: SQLite + Drizzle ORM
- **样式**: UnoCSS
- **UI 库**: ikun-ui (KUN 基于 Svelte 的 UI 库)

## 2. 遇见拦路虎：ikun-ui 与 Svelte 5 的水土不服

当我们兴致勃勃地搭建好环境（Drizzle 数据库 schema、Session 认证系统都已就绪），准备开始写 UI 时，现实给了我们当头一棒。

ikun-ui (v0.2.8) 主要是为 Svelte 3/4 设计的。在 Svelte 5 环境下，我们遇到了严重的兼容性问题：

1. **TypeScript Enums 报错**: `button` 和 `radio` 组件使用了 TS `enum` 语法，导致构建时解析错误。
2. **`svelte:fragment` 位置错误**: Svelte 5 对 slot 的位置要求更严格，导致 `menu` 等组件直接报错。

### 临时方案：原生 HTML + UnoCSS

为了不卡住开发进度，我们曾一度决定暂时放弃 UI 库，使用 **原生 HTML + UnoCSS** 手写了大部分基础组件。

- 手写了带过渡动画的 Toast
- 手写了 Dropdown 和 Modal
- 虽然能用，但代码量激增，维护成本变高。

## 3. 转折点：Fork 与救赎

由于 ikun-ui 官方仓库已经有 10 个月未更新，而我们又非常喜欢它的设计风格（UnoCSS 优先），我们做出了一个工程师该做的决定：**Fork 它！**

我们决定自己维护一个兼容 Svelte 5 的 `ikun-ui` 版本。

### 修复之路

我们制定了详细的修复计划：

1. **Enums 替换**: 将所有 TypeScript `enum` 替换为 `const Object as const`。
2. **Fragment 修复**: 重构 slot 结构，符合 Svelte 5 标准。
3. **构建优化**: 使用 pnpm workspace 管理依赖。

目前，我们已经成功 Fork 了代码，并准备开始我们的修复工作。这不仅是为了我们的项目，也是希望能为 Svelte 社区贡献一份力量。

## 4. 下一步计划

接下来的日子里，我们将：

- 修复 `ikun-ui` 的核心组件 bug
- 在项目中集成我们要发布的修复版 UI 库
- 继续完善失物招领平台的业务逻辑（AI 图片分析、信用积分系统等）

技术探索的路上总有坎坷，但正是这些挑战让编程变得有趣。敬请期待我们的下一个版本！

---

**Stay tuned!** 🚀
