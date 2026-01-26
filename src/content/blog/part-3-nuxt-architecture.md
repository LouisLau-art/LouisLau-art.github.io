---
title: "从零学 Nuxt 全栈(3)：Nuxt 架构 - SSR、路由与目录约定"
pubDate: 'Dec 16 2025'
tags: ["Archive"]
description: "本篇介绍 Nuxt 在 Vue 之上的增强：SSR 服务端渲染、基于文件的自动路由、Nuxt 4 的目录结构，以及 nuxt.config.ts 配置解析。"
---

到目前为止，你已经学会了 Vue 3 的核心：响应式、组件、通信。

但如果你直接用 Vue CLI 或 Vite 创建一个 Vue 项目，你需要自己处理很多事情：
-   路由配置 (vue-router)
-   服务端渲染 (如果需要)
-   构建优化
-   API 代理

**Nuxt** 就是来解决这些问题的。它是一个基于 Vue 的**全栈框架**，提供了开箱即用的：
-   📁 **基于文件的路由**
-   🖥️ **服务端渲染 (SSR)**
-   🔌 **内置后端 API (Nitro)**
-   📦 **自动导入**

## 渲染模式：SSR vs CSR vs SSG

在深入 Nuxt 之前，我们需要理解三种渲染模式。

### CSR：客户端渲染

传统的 Vue SPA (单页应用) 使用 CSR：
1.  服务器返回一个几乎空的 HTML（只有 `<div id="app">`）。
2.  浏览器下载 JavaScript。
3.  JavaScript 在浏览器中运行，生成 HTML 内容。

**优点**：交互流畅，页面切换不刷新。
**缺点**：首屏加载慢，不利于 SEO（搜索引擎看到的是空白页）。

### SSR：服务端渲染

Spring Boot + Thymeleaf 就是 SSR：
1.  服务器执行模板引擎，生成完整 HTML。
2.  浏览器直接显示 HTML。
3.  JavaScript 加载后，页面变得"可交互"。

**Nuxt 的 SSR** 结合了两者的优点：
-   首次请求：服务器返回完整 HTML（有利于 SEO 和首屏速度）。
-   后续导航：客户端接管，像 SPA 一样流畅。

### SSG：静态站点生成

在构建时就把所有页面生成为静态 HTML 文件，适合博客、文档等内容不常变的网站。

| 模式 | 首屏速度 | SEO | 动态数据 | 适用场景 |
|------|----------|-----|----------|----------|
| CSR | 慢 | 差 | 实时 | 后台管理系统 |
| SSR | 快 | 好 | 实时 | 电商、社交 |
| SSG | 最快 | 好 | 构建时 | 博客、文档 |

Atidraw 使用 **SSR** 模式，因为画作列表需要实时从 R2 获取。

## 基于文件的路由

在传统 Vue 项目中，你需要手动配置路由：

```javascript
// router/index.js (vue-router)
const routes = [
  { path: '/', component: Home },
  { path: '/about', component: About },
  { path: '/user/:id', component: User },
]
```

在 Nuxt 中，**路由由文件结构自动生成**。

### 类比 Spring

这和 Spring 的 `@RequestMapping` 类似，但是是基于文件名：

| Nuxt 文件路径 | 对应 URL | Spring 类比 |
|---------------|----------|-------------|
| `pages/index.vue` | `/` | `@GetMapping("/")` |
| `pages/about.vue` | `/about` | `@GetMapping("/about")` |
| `pages/user/[id].vue` | `/user/123` | `@GetMapping("/user/{id}")` |

### Atidraw 的路由

```
app/pages/
├── index.vue    →  /        (首页画廊)
└── draw.vue     →  /draw    (画板页面)
```

只需要创建文件，路由自动生效！

### 动态路由参数

如果你需要动态 URL（如 `/user/123`），使用方括号命名：

```
pages/
└── user/
    └── [id].vue   →  /user/:id
```

在组件中获取参数：

```vue
<script setup>
const route = useRoute()
console.log(route.params.id)  // '123'
</script>
```

## Nuxt 4 目录结构

Nuxt 4 引入了新的目录结构，把**前端**和**后端**代码分开：

```
atidraw/
├── app/                  # 🎨 前端代码
│   ├── components/       # Vue 组件
│   ├── pages/            # 页面路由
│   ├── plugins/          # Vue 插件
│   ├── assets/           # 静态资源 (CSS, 图片)
│   └── app.vue           # 根组件
├── server/               # ⚙️  端代码 (Nitro)
│   ├── api/              # REST API
│   └── routes/           # 其他服务端路由
├── nuxt.config.ts        # 📝 项目配置
└── package.json
```

### `app/` vs `server/`

| 目录 | 运行环境 | 类比 |
|------|----------|------|
| `app/` | 浏览器 + 服务端 (SSR 时) | Vue 前端 |
| `server/` | 仅服务端 (Node.js/Workers) | Spring Controller |

### `app.vue`：根组件

`app/app.vue` 是整个应用的入口，类似 `index.html` 的 `<body>`：

```vue
<template>
  <UApp>
    <AppHeader />
    <UContainer>
      <main>
        <NuxtPage />  <!-- 这里渲染当前路由对应的页面 -->
      </main>
    </UContainer>
    <AppFooter />
  </UApp>
</template>
```

`<NuxtPage />` 是一个特殊组件，Nuxt 会根据当前 URL 自动替换为对应的页面组件。

## `nuxt.config.ts`：项目配置

这个文件类似 Spring 的 `application.yml`，用于配置 Nuxt 的各种行为。

看看 Atidraw 的配置：

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  // 1. 启用的模块 (类似 starter 依赖)
  modules: [
    '@nuxthub/core',    // Cloudflare 集成
    '@nuxt/ui',         // UI 组件库
    '@nuxt/eslint',     // 代码检查
    'nuxt-auth-utils',  // 身份认证
  ],
  
  // 2. 启用开发工具
  devtools: { enabled: true },
  
  // 3. 全局 CSS
  css: ['~/assets/main.css'],
  
  // 4. 路由规则
  routeRules: {
    '/drawings/**': { isr: true },  // 增量静态再生
  },
  
  // 5. 启用 Nuxt 4 兼容模式
  future: { compatibilityVersion: 4 },
  compatibilityDate: '2025-10-14',
  
  // 6. NuxtHub 配置：启用 Blob 存储
  hub: {
    blob: true,
  },
})
```

### 关键配置解读

| 配置 | 说明 |
|------|------|
| `modules` | 扩展 Nuxt 功能的插件，类似 Spring 的 starter |
| `css` | 全局 CSS 文件，`~` 表示项目根目录 |
| `routeRules` | 按路由定制渲染策略 |
| `hub.blob` | 启用 NuxtHub 的对象存储 (Cloudflare R2) |

## 自动导入

Nuxt 的另一个神奇特性：**你不需要手动 import 常用函数**。

在 Vue 项目中，你需要这样写：

```vue
<script setup>
import { ref, onMounted } from 'vue'
import { useRoute } from 'vue-router'

const count = ref(0)
</script>
```

在 Nuxt 中，直接用：

```vue
<script setup>
// 不需要 import！Nuxt 自动导入
const count = ref(0)
const route = useRoute()
</script>
```

以下函数/组件会被自动导入：
-   Vue 的 `ref`, `reactive`, `computed`, `watch`, `onMounted` 等
-   Nuxt 的 `useFetch`, `useRoute`, `navigateTo` 等
-   `components/` 目录下的所有组件

## 小结

| 概念 | Nuxt | Spring 类比 |
|------|------|-------------|
| 路由 | 基于文件 (`pages/`) | `@RequestMapping` |
| 根组件 | `app.vue` | `index.html` |
| 配置文件 | `nuxt.config.ts` | `application.yml` |
| 后端目录 | `server/` | `@RestController` |
| 自动导入 | 内置支持 | - |

下一篇，我们将深入 `server/` 目录，学习如何用 TypeScript 写后端 API。你会发现，Nuxt 的 Server API 写起来比 Spring Controller 还要简洁！
