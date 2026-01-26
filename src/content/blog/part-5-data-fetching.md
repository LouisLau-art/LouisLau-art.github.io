---
title: "从零学 Nuxt 全栈(5)：数据获取与状态管理 - useFetch 与 useState"
pubDate: 'Dec 16 2025'
tags: ["Archive"]
description: "本篇讲解 Nuxt 的数据获取方式：$fetch 与 useFetch 的区别、SSR 数据水合原理，以及使用 useState 管理全局状态。通过首页画廊的无限滚动实现加深理解。"
---

后端 API 写好了，前端怎么调用？

在 Java 世界，你可能用过 `RestTemplate` 或 `WebClient` 来请求其他服务的 API。在前端，我们有 `fetch` API 和 Nuxt 封装的 `$fetch` / `useFetch`。

## `$fetch`：基础请求方式

`$fetch` 是 Nuxt 基于 `ofetch` 库封装的请求方法：

```typescript
// 发送 GET 请求
const data = await $fetch('/api/drawings')

// 发送 POST 请求
const result = await $fetch('/api/upload', {
  method: 'POST',
  body: formData,
})
```

### 类比 Spring

| Nuxt | Java |
|------|------|
| `$fetch('/api/xxx')` | `restTemplate.getForObject("/api/xxx", ...)` |
| `$fetch('/api/xxx', { method: 'POST', body })` | `restTemplate.postForObject(...)` |

### 特点

-   自动处理 JSON 序列化/反序列化
-   自动处理错误（抛出异常）
-   支持拦截器
-   **但**：在 SSR 时有问题（下面会讲）

## `useFetch`：SSR 友好的数据获取

在 Nuxt 中，**不推荐**在组件顶层直接用 `await $fetch()`，而是用 `useFetch`：

```vue
<script setup>
// ✅ 推荐
const { data, pending, error } = await useFetch('/api/drawings')

// ❌ 不推荐 (在 SSR 时会有问题)
const data = await $fetch('/api/drawings')
</script>
```

### 为什么？理解 SSR 数据水合

还记得 SSR 吗？页面先在服务器渲染，然后发送 HTML 给浏览器。

如果你在服务端和客户端都用 `$fetch` 请求一次，会导致：
1.  服务端请求一次 API
2.  HTML 发送给浏览器
3.  客户端 JavaScript 执行，**又请求一次 API**

这不仅浪费资源，还可能导致数据不一致（flickering）。

`useFetch` 解决了这个问题：
1.  服务端请求 API，拿到数据
2.  数据被"嵌入"到 HTML 中
3.  客户端 JavaScript 执行时，**直接使用嵌入的数据**，不再请求

这个过程叫 **数据水合 (Hydration)**。

### `useFetch` 返回值

```typescript
const { 
  data,      // 响应数据 (Ref)
  pending,   // 是否加载中 (Ref<boolean>)
  error,     // 错误信息 (Ref)
  refresh,   // 手动重新请求
  execute,   // 手动执行 (配合 lazy 选项)
} = await useFetch('/api/drawings')
```

### 常用选项

```typescript
// 带查询参数
const { data } = await useFetch('/api/drawings', {
  query: { cursor: 'abc123' },
})
// 相当于请求 /api/drawings?cursor=abc123

// 懒加载 (不阻塞渲染)
const { data, execute } = await useFetch('/api/drawings', {
  lazy: true,
})
// 需要时手动调用 execute()

// 只在客户端执行
const { data } = await useFetch('/api/drawings', {
  server: false,
})
```

## 实战：首页画廊的无限滚动

看看 `pages/index.vue` 如何加载画作列表：

```vue
<script setup>
// 1. 初始加载
const { data } = await useFetch('/api/drawings', {
  deep: true,  // 返回深层响应式对象
})

// 2. 无限滚动加载更多
const loading = ref(false)

async function loadMore() {
  if (loading.value || !data.value?.hasMore) return
  loading.value = true

  // 用 $fetch 加载更多 (因为不是首次渲染，不需要 SSR 友好)
  const more = await $fetch('/api/drawings', {
    query: { cursor: data.value.cursor },
  })
  
  // 追加数据
  data.value.blobs.push(...more.blobs)
  data.value.cursor = more.cursor
  data.value.hasMore = more.hasMore
  
  loading.value = false
}
</script>

<template>
  <div v-infinite-scroll="[loadMore, { distance: 10 }]">
    <div v-for="drawing in data?.blobs" :key="drawing.pathname">
      <!-- 渲染画作 -->
    </div>
  </div>
</template>
```

**要点**：
-   首次加载用 `useFetch`（SSR 友好）
-   加载更多用 `$fetch`（用户交互触发，无需 SSR）
-   `v-infinite-scroll` 来自 VueUse，滚动到底部自动触发 `loadMore`

## `useState`：全局状态管理

有时候，你需要在多个组件之间共享状态。比如：
-   当前用户信息
-   主题设置 (暗黑/明亮)
-   OAuth 提供商列表

### 类比 Spring

类似于 Spring 的 `@SessionScoped` Bean，在一次会话中共享状态。

### 基本用法

```typescript
// composables/useTheme.ts (可选，如果想封装成 Composable)
const theme = useState('theme', () => 'light')  // 默认值为 'light'

// 任何组件中
<script setup>
const theme = useState('theme')
theme.value = 'dark'  // 修改会全局生效
</script>
```

### Atidraw 中的 useState

在 `pages/draw.vue` 中：

```vue
<script setup>
// 从服务端获取的 OAuth 提供商信息
const authProviders = useState<{ google: boolean, github: boolean }>('authProviders')
</script>

<template>
  <UButton v-if="authProviders.google" to="/auth/google">
    Sign-in with Google
  </UButton>
  <UButton v-if="authProviders.github" to="/auth/github">
    Sign-in with GitHub
  </UButton>
</template>
```

这里 `authProviders` 是在服务端设置的全局状态，客户端可以直接访问。

### `useState` vs `ref`

| | `ref` | `useState` |
|--|-------|-----------|
| 作用域 | 当前组件 | 全局 (跨组件) |
| SSR 水合 | ❌ 需要手动处理 | ✅ 自动水合 |
| 持久化 | ❌ 页面刷新丢失 | ❌ 页面刷新丢失 |

如果需要持久化（如用户偏好），可以配合 `localStorage` 或用 VueUse 的 `useLocalStorage`。

## 什么时候用 Pinia？

对于更复杂的状态管理（如购物车、多步骤表单），可以使用 **Pinia**（Vue 官方状态管理库）。

Atidraw 项目比较简单，没有用到 Pinia，所以这里只简单提一下：

```typescript
// stores/cart.ts
import { defineStore } from 'pinia'

export const useCartStore = defineStore('cart', {
  state: () => ({
    items: [],
  }),
  actions: {
    addItem(item) {
      this.items.push(item)
    },
  },
})
```

## 小结

| 概念 | Nuxt | Spring 类比 |
|------|------|-------------|
| 基础请求 | `$fetch` | `RestTemplate` |
| SSR 友好请求 | `useFetch` | - |
| 数据水合 | 自动 | - |
| 全局状态 | `useState` | `@SessionScoped` |
| 复杂状态 | Pinia | Spring Service |

下一篇是系列的最后一篇，我们将学习 **NuxtHub** 如何让我们"一行代码"对接 Cloudflare 的各种服务，实现真正的 Serverless 全栈部署。
