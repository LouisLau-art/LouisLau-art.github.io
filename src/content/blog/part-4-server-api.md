---
title: "从零学 Nuxt 全栈(4)：Nuxt Server API - 用 TypeScript 写后端"
pubDate: 'Dec 16 2025'
tags: ["Archive"]
description: "本篇介绍 Nuxt 的内置后端引擎 Nitro：基于文件的 API 路由、事件处理器 eventHandler、请求参数读取，并逐行解析 upload.post.ts 文件上传接口。"
---

如果你是 Java 后端开发者，这一篇会让你感到格外亲切。

在 Spring Boot 中，你习惯了这样写 API：

```java
@RestController
@RequestMapping("/api")
public class DrawingController {
    
    @PostMapping("/upload")
    public Drawing upload(@RequestParam("file") MultipartFile file) {
        // 处理上传
    }
    
    @GetMapping("/drawings")
    public List<Drawing> list() {
        // 返回列表
    }
}
```

在 Nuxt 中，**你不需要任何注解**，只需要创建文件，路由自动生效！

## Nitro：Nuxt 的后端引擎

Nuxt 内置了一个名为 **Nitro** 的服务端引擎。它负责：
-   处理 API 请求
-   服务端渲染 (SSR)
-   构建优化

Nitro 可以部署到几乎任何环境：Node.js、Cloudflare Workers、Vercel、Deno 等。

## 基于文件的 API 路由

就像 `pages/` 目录用于前端路由，`server/api/` 目录用于后端 API 路由。

### 文件命名约定

| 文件名 | HTTP 方法 | URL 路径 |
|--------|-----------|----------|
| `server/api/hello.ts` | GET (默认) | `/api/hello` |
| `server/api/hello.get.ts` | GET | `/api/hello` |
| `server/api/hello.post.ts` | POST | `/api/hello` |
| `server/api/user/[id].get.ts` | GET | `/api/user/:id` |

**规则总结**：
-   `xxx.get.ts` → GET 请求
-   `xxx.post.ts` → POST 请求
-   `[param]` → 动态路由参数

### 类比 Spring

| Nuxt 文件 | Spring 注解 |
|-----------|-------------|
| `drawings.get.ts` | `@GetMapping("/api/drawings")` |
| `upload.post.ts` | `@PostMapping("/api/upload")` |
| `user/[id].get.ts` | `@GetMapping("/api/user/{id}")` |

## `eventHandler`：处理请求

每个 API 文件需要导出一个 `eventHandler`，类似 Spring 的 Controller 方法：

```typescript
// server/api/hello.get.ts
export default eventHandler(async (event) => {
  return { message: 'Hello World!' }
})
```

-   `event` 参数包含请求的所有信息（类似 `HttpServletRequest`）
-   直接 `return` 对象，Nitro 自动序列化为 JSON（类似 `@ResponseBody`）

### 常用工具函数

Nitro 提供了一系列辅助函数：

| 函数 | 用途 | Spring 类比 |
|------|------|-------------|
| `getQuery(event)` | 获取 URL 查询参数 | `@RequestParam` |
| `readBody(event)` | 读取 JSON 请求体 | `@RequestBody` |
| `readFormData(event)` | 读取 FormData | `@RequestParam MultipartFile` |
| `getRouterParam(event, 'id')` | 获取路由参数 | `@PathVariable` |
| `setHeader(event, 'key', 'val')` | 设置响应头 | `response.setHeader()` |
| `createError({ statusCode, message })` | 抛出 HTTP 错误 | `throw new ResponseStatusException()` |

## 实战：解读 drawings.get.ts

让我们看看 Atidraw 如何获取画作列表：

```typescript
// server/api/drawings.get.ts
import { blob } from 'hub:blob'  // NuxtHub 提供的 Blob 存储

export default eventHandler(async (event) => {
  // 1. 获取查询参数 cursor (用于分页)
  const { cursor } = await getQuery<{ cursor?: string }>(event)

  // 2. 从 R2 存储列出文件
  const res = await blob.list({
    limit: 20,           // 每页 20 条
    cursor,              // 分页游标
    prefix: 'drawings/', // 只查 drawings 文件夹
  })

  // 3. 返回结果 (自动转 JSON)
  return res
})
```

**逐行解读**：
1.  `getQuery(event)` 获取 URL 参数，`?cursor=abc` → `{ cursor: 'abc' }`
2.  `blob.list()` 调用 Cloudflare R2 存储 API（下篇详细讲）
3.  直接 `return` 对象，Nuxt 自动处理 JSON 序列化

## 实战：解读 upload.post.ts

这是一个更复杂的例子——文件上传处理：

```typescript
// server/api/upload.post.ts
import { blob, ensureBlob } from 'hub:blob'

export default eventHandler(async (event) => {
  // 1. 鉴权：确保用户已登录
  const { user } = await requireUserSession(event)
  
  // 2. 读取上传的文件
  const form = await readFormData(event)
  const drawing = form.get('drawing') as File
  
  // 3. 校验文件
  ensureBlob(drawing, {
    maxSize: '1MB',
    types: ['image/jpeg'],
  })
  
  // 4. 生成文件名 (时间戳倒序，方便排序)
  const name = `${new Date('2050-01-01').getTime() - Date.now()}`
  
  // 5. 存储到 R2
  return blob.put(`${name}.jpg`, drawing, {
    prefix: 'drawings/',
    addRandomSuffix: true,
    customMetadata: {
      userId: user.id,
      userName: user.name,
      // ... 更多元数据
    },
  })
})
```

### 与 Spring 对比

```java
// Spring Boot 版本
@PostMapping("/api/upload")
public Drawing upload(
    @AuthenticationPrincipal User user,    // 对应 requireUserSession
    @RequestParam("drawing") MultipartFile file  // 对应 readFormData
) {
    // 校验
    if (file.getSize() > 1024 * 1024) {
        throw new ResponseStatusException(HttpStatus.BAD_REQUEST, "File too large");
    }
    
    // 存储
    String name = String.valueOf(System.currentTimeMillis());
    s3Client.putObject(...);  // 对应 blob.put
    
    return new Drawing(...);
}
```

你会发现，**Nuxt/Nitro 更简洁**：
-   不需要 `@PostMapping` 注解，文件名就是路由
-   不需要 `@RequestParam`，用 `readFormData()` 函数
-   不需要手动构造 `ResponseEntity`，直接 `return`

## 错误处理

抛出 HTTP 错误：

```typescript
export default eventHandler(async (event) => {
  const { id } = getRouterParams(event)
  
  const drawing = await findDrawing(id)
  if (!drawing) {
    throw createError({
      statusCode: 404,
      message: 'Drawing not found',
    })
  }
  
  return drawing
})
```

这相当于 Spring 的：

```java
throw new ResponseStatusException(HttpStatus.NOT_FOUND, "Drawing not found");
```

## 中间件：requireUserSession

你可能注意到了 `requireUserSession(event)` 这个调用。这是 `nuxt-auth-utils` 模块提供的认证工具。

```typescript
// 如果用户未登录，自动抛出 401 错误
const { user } = await requireUserSession(event)
```

类似 Spring Security 的 `@PreAuthorize`。

## 小结

| 概念 | Nuxt/Nitro | Spring Boot |
|------|------------|-------------|
| API 文件 | `server/api/xxx.post.ts` | `@PostMapping` |
| 请求处理 | `eventHandler(event => {})` | Controller 方法 |
| 查询参数 | `getQuery(event)` | `@RequestParam` |
| 请求体 | `readBody(event)` | `@RequestBody` |
| 文件上传 | `readFormData(event)` | `@RequestParam MultipartFile` |
| 路由参数 | `getRouterParam(event, 'id')` | `@PathVariable` |
| 错误处理 | `createError({ statusCode })` | `ResponseStatusException` |
| 认证 | `requireUserSession(event)` | `@PreAuthorize` |

下一篇，我们将学习**前端如何调用这些 API**：`useFetch`、`$fetch`，以及状态管理 `useState`。
