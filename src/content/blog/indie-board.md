---
title: '从零构建 Indie Board：一次 Bleeding Edge 技术栈的探索之旅'
description: '记录使用 Nuxt 4 + Una UI + Drizzle ORM 构建独立产品发现榜的完整过程，包括技术选型的反复探索和踩坑经验。'
pubDate: 'Dec 17 2025'
tags: ["Archive"]
---

最近我完成了一个小项目 —— **Indie Board**（独立产品发现榜），一个极简的 Product Hunt 风格应用。这篇文章记录了整个开发过程，特别是在技术选型上的反复探索和踩坑经验。

> 🔗 项目地址：[github.com/LouisLau-art/indie-board](https://github.com/LouisLau-art/indie-board)

## 项目目标

构建一个极简的 Web 应用，用于展示和发现独立开发者的产品：

- 📋 产品列表（按点赞数排序）
- ➕ 提交新产品
- 👍 投票系统（防刷机制）
- 🔄 实时更新（轮询）
- 🌙 暗色模式

**技术要求**：使用"激进"的技术栈，能用最新版就用最新版。

---

## 第一阶段：技术选型

### 确定核心框架

这部分比较直接：

| 技术  | 选择                           | 版本            |
| --- | ---------------------------- | ------------- |
| 框架  | Nuxt                         | 4.2.2         |
| UI  | Vue                          | 3.5.25        |
| 数据库 | Better-SQLite3 + Drizzle ORM | 12.x + 0.45.x |
| 运行时 | Bun                          | latest        |

Vue 3.5 带来了两个很棒的特性：

- **Reactive Props Destructure** - 可以直接解构 props 并保持响应性
- **useTemplateRef** - 更优雅的模板引用方式

```vue
<script setup lang="ts">
// Vue 3.5+ Reactive Props Destructure
const { product } = defineProps<{ product: Product }>()

// Vue 3.5+ useTemplateRef
const titleInput = useTemplateRef<HTMLInputElement>('titleInput')
</script>
```

### CSS/UI 方案的纠结

这里开始了漫长的探索之旅...

**初始想法**：使用 UnoCSS（原子化 CSS 引擎）。

UnoCSS 是 Vue 生态的产物，由 Anthony Fu 开发，比 Tailwind CSS 更轻量、更快。

---

## 第二阶段：UI 组件库的反复横跳（6 次尝试！）

### 尝试 1：手写组件 + UnoCSS Shortcuts

最开始我用 UnoCSS 的 `shortcuts` 功能手写了所有组件：

```ts
shortcuts: {
  'btn': 'px-4 py-2 rounded-lg font-medium transition-all duration-200',
  'btn-primary': 'btn bg-emerald-500 hover:bg-emerald-600 text-white',
  'card': 'bg-white dark:bg-gray-800 rounded-2xl shadow-lg p-5',
}
```

**问题**：功能能用，但 UI 看起来很"素"，缺乏精心设计的细节。

### 尝试 2：Naive UI

我想用 UI 组件库来避免"造轮子"。于是尝试了 [Naive UI](https://www.naiveui.com/)：

**优点**：Vue 3 原生，16k+ GitHub stars，使用 CSS-in-JS 与 UnoCSS 无冲突。

**但我有顾虑**：想要更纯粹的 UnoCSS 生态方案。

### 尝试 3：Onu UI

发现了 [Onu UI](https://github.com/onu-ui/onu-ui) —— 专为 UnoCSS 设计的组件库。

**踩坑**：启动报错！

```
ERROR  Cannot convert from keyword to hex
```

原因是 `@onu-ui/preset` 的颜色处理与 UnoCSS 66.x 不兼容。

### 尝试 4：又回到手写组件

经过一番折腾，暂时回到了手写组件的方案。

**这时候我意识到一个关键问题**：

> "折腾来折腾去还是回到了原点，如果我们用了 git 来管理版本，是不是 rollback 会很方便？"

于是立即初始化 Git 并提交一个稳定版本。有了 Git 备份，就可以放心地继续实验了：

```bash
git checkout -b experiment/try-new-ui  # 创建实验分支
# 实验成功 → git checkout main && git merge experiment/try-new-ui
# 实验失败 → git checkout main && git branch -D experiment/try-new-ui
```

### 尝试 5：DaisyUI

尝试 [DaisyUI](https://daisyui.com/)，基于 Tailwind CSS 的语义化组件库，有社区维护的 UnoCSS preset。

**又踩坑了**：

1. DaisyUI 5.x 报错 `addVariant is not a function`（需要 Tailwind 4 API，preset 还没跟进）
2. 降级到 DaisyUI 4.12.24 后可以工作
3. 但发现 `@unocss/preset-uno` 在 UnoCSS 66 中已被弃用！

在 [awesome-unocss](https://github.com/unocss-community/awesome-unocss) 发现：

> `@unocss/preset-uno` - Deprecated: Use `presetWind3` from `@unocss/preset-wind3` instead in @66.0.0

这说明我们使用的 preset 已经过时了。

### 尝试 6：Una UI（最终方案）

最终发现了 [Una UI](https://unaui.com/) —— **专为 Nuxt 设计的 UnoCSS 原生 UI 框架**！

```json
{
  "@una-ui/nuxt": "^1.0.0-alpha.12"
}
```

**Una UI 的优势**：

- ✅ 专为 Nuxt 3/4 设计
- ✅ 原生 UnoCSS 支持（内置 preset，无需手动配置）
- ✅ 提供完整的组件库（UCard, UButton, UInput, UAlert, UIcon, UBadge 等）
- ✅ 内置暗色模式支持

配置极其简单：

```ts
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@una-ui/nuxt'],
  una: {
    prefix: 'U',
    ui: { primary: 'emerald' },
  },
})
```

**终于成功了！** 不需要单独配置 UnoCSS，Una UI 已经包含了一切。

---

## 第三阶段：修复细节问题

### 字体问题

默认主题可能使用非 sans-serif 字体，导致中英文不协调。

**解决方案**：在全局样式中覆盖：

```css
html {
  font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', 
    'PingFang SC', 'Hiragino Sans GB', 'Microsoft YaHei', sans-serif;
}
```

### TypeScript 类型问题

Nitro 的 `defineNitroPlugin` 报 TypeScript 错误。

**原因**：Nuxt 的类型是动态生成的，需要运行 `bunx nuxi prepare` 后 IDE 才能识别。

---

## 经验教训

### 1. 先用 Git，再实验

在尝试新技术之前，**一定要先提交一个稳定版本**。这样即使实验失败，也能轻松回滚。

### 2. "Bleeding Edge" 有代价

- **生态兼容性问题**：第三方库可能还没跟进
- **文档滞后**
- **社区支持有限**

**建议**：核心框架用最新版，辅助库要检查兼容性。

### 3. 关注官方弃用公告

我在用 `@unocss/preset-uno` 时不知道它已在 UnoCSS 66 中被弃用。通过查看 [awesome-unocss](https://github.com/unocss-community/awesome-unocss) 才发现应该用 `presetWind3`。

### 4. 不要执着于"完美"的技术栈

我在 UI 方案上反复横跳了 **6 次**：

1. 手写 UnoCSS → 太素
2. Naive UI → 不够"纯粹 UnoCSS"
3. Onu UI → 兼容性问题
4. 又手写 → 回到原点
5. DaisyUI → 可以工作，但 preset 过时
6. **Una UI → 最终方案** ✅

### 5. 版本号要查证

手写版本号之前，先用 `npm view <package> version` 确认。

---

## 最终技术栈

| 类别     | 技术             | 版本                |
| ------ | -------------- | ----------------- |
| 框架     | Nuxt           | 4.2.2             |
| UI 框架  | Vue            | 3.5.25            |
| UI 组件  | Una UI         | 1.0.0-alpha.12    |
| CSS 引擎 | UnoCSS         | 内置于 Una UI        |
| ORM    | Drizzle ORM    | 0.45.1            |
| 数据库    | Better-SQLite3 | 12.5.0            |
| 运行时    | Bun            | latest            |

---

## 总结

这次项目最大的收获不是最终的代码，而是在技术选型上的探索过程。

**记住几个关键点**：

1. ✅ 用 Git 管理实验，放心大胆尝试
2. ✅ 检查第三方库对最新版的支持情况
3. ✅ 关注官方弃用公告（如 awesome-unocss）
4. ✅ 不要为了"技术纯粹性"牺牲开发效率
5. ✅ 版本号要查证，不要凭印象

最后，项目地址：[github.com/LouisLau-art/indie-board](https://github.com/LouisLau-art/indie-board)

欢迎 Star ⭐ 和 Fork！

---

**感谢阅读！** 🚀
