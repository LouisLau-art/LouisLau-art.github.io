---
title: "从零学 Nuxt 全栈(0)：给 Java 后端的前端世界观"
pubDate: 'Dec 16 2025'
tags: ["Archive"]
description: "本系列面向 Java/Spring Boot 后端开发者，通过 Atidraw 项目系统学习 Vue 和 Nuxt。本篇介绍前端基础：HTML/CSS/JS、Node.js、npm，并与你熟悉的 Java 生态做类比。"
---

如果你和我一样，是从 Java 后端入坑编程的，那么初次接触前端时一定会感到困惑：

> "为什么前端项目也有 `package.json`？这和 `pom.xml` 是一回事吗？"
> "JavaScript 和 Java 到底什么关系？"
> "`npm run dev` 之后发生了什么？"

别担心，这篇文章就是为你准备的。我们将用你熟悉的 Spring Boot 生态作为参照，快速建立对前端开发的基本认知。

## Web 三剑客：HTML, CSS, JavaScript

在深入框架之前，先回归本质。**所有**前端最终产物都是这三种文件：

| 技术             | 职责       | Spring 类比                        |
| -------------- | -------- | -------------------------------- |
| **HTML**       | 页面结构（骨架） | Thymeleaf 模板的 `<div>`, `<p>` 等标签 |
| **CSS**        | 页面样式（皮肤） | 决定颜色、字体、布局                       |
| **JavaScript** | 页面行为（大脑） | 处理用户交互、发请求、操作 DOM                |

在传统 Spring MVC + Thymeleaf 项目中，你可能写过这样的代码：

```html
<!-- Thymeleaf 模板 (HTML) -->
<div th:text="${user.name}">用户名</div>
<button onclick="alert('点击了!')">点我</button>
```

这里，`th:text` 是服务端渲染（SSR）——数据在服务器上就填好了，浏览器拿到的是完整的 HTML。

而**现代前端框架（如 Vue）的核心思想是：把"渲染"这件事从服务器搬到浏览器**。服务器只返回一个"空壳" HTML + 一堆 JavaScript，然后由浏览器执行 JS 来生成页面内容。这就是**客户端渲染 (CSR)** 或 **单页应用 (SPA)**。

## JavaScript ≠ Java

虽然名字里都有 "Java"，但它们**完全不同**：

| 特性   | Java      | JavaScript       |
| ---- | --------- | ---------------- |
| 类型系统 | 静态强类型     | 动态弱类型            |
| 运行环境 | JVM       | 浏览器 / Node.js    |
| 编译   | 编译成字节码    | 解释执行 (JIT)       |
| 主要用途 | 后端服务、桌面应用 | 浏览器交互、Node.js 后端 |

**TypeScript** 是 JavaScript 的超集，给它加上了**静态类型**，写起来更像 Java。Atidraw 项目全程使用 TypeScript。

## Node.js：让 JS 脱离浏览器

在很长一段时间里，JavaScript 只能在浏览器里跑。2009 年，**Node.js** 诞生了，它把 Chrome 的 V8 引擎抠出来，让 JS 可以像 Java 一样在服务器上运行。

这意味着：

1. **前端工具链**（打包、编译、开发服务器）可以用 JS 写了。
2. **全栈开发**成为可能——前后端用同一门语言。

Nuxt 的后端 API 就是跑在 Node.js（或兼容的 Cloudflare Workers）上的。

## npm / pnpm / bun：前端的 Maven

Java 有 Maven Central 和 `pom.xml`，前端有 **npm (Node Package Manager)** 和 `package.json`。

| Maven                 | npm/pnpm/bun                                   |
| --------------------- | ---------------------------------------------- |
| `pom.xml`             | `package.json`                                 |
| `mvn install`         | `npm install` / `pnpm install` / `bun install` |
| `mvn spring-boot:run` | `npm run dev`                                  |
| `~/.m2/repository`    | `node_modules/`                                |
| Maven Central         | npmjs.com                                      |

`pnpm` 和 `bun` 是 `npm` 的替代品，速度更快。Atidraw 项目使用 `pnpm`，但你也可以直接用 `bun`（你提到已经安装了）。

打开 Atidraw 的 `package.json`，你会看到类似这样的结构：

```json
{
  "name": "atidraw",
  "scripts": {
    "dev": "nuxt dev",      // ≈ mvn spring-boot:run
    "build": "nuxt build",  // ≈ mvn package
    "preview": "nuxt preview"
  },
  "dependencies": {
    "nuxt": "^4.1.3",       // ≈ spring-boot-starter-web
    "@nuxt/ui": "^4.0.1",   // ≈ 某个 UI 组件库依赖
    // ...
  },
  "devDependencies": {
    "typescript": "...",    // 开发时依赖，不打包到生产
    // ...
  }
}
```

## ES Modules (ESM)：现代模块化

Java 用 `import com.example.MyClass;` 来引入类。JavaScript 也有类似的机制，叫 **ES Modules (ESM)**：

```javascript
// Java
import com.example.service.UserService;

// JavaScript (ESM)
import { ref, onMounted } from 'vue'
import SignaturePad from 'signature_pad'
```

区别在于：

- Java 按**类名**引入，路径由包结构决定。
- JS 按**文件路径**或**包名**引入，`'vue'` 会去 `node_modules/vue` 里找。

你还会看到 `export`：

```javascript
// 导出一个函数，供其他文件 import
export function formatDate(date) {
  return date.toLocaleDateString()
}
```

## 首次运行 Atidraw

现在，让我们把项目跑起来！

```bash
cd /home/louis/atidraw

# 安装依赖 (相当于 mvn install)
bun install

# 启动开发服务器 (相当于 mvn spring-boot:run)
bun dev
```

如果一切顺利，你会看到类似这样的输出：

```
Nuxt 4.1.3 with Nitro 2.x.x
  ➜ Local:    http://localhost:3000/
  ➜ Network:  http://192.168.x.x:3000/
```

打开浏览器访问 `http://localhost:3000`，你应该能看到 Atidraw 的首页——一个画作画廊。

**这背后发生了什么？**

1. `bun dev` 执行了 `package.json` 里 `scripts.dev` 定义的命令：`nuxt dev`。
2. Nuxt 启动了一个**开发服务器**（类似 Spring Boot 的内嵌 Tomcat）。
3. 它**实时编译** `.vue` 文件和 TypeScript，支持**热更新 (HMR)**——你改代码，浏览器自动刷新。

## 小结

| 概念   | 前端                           | Spring Boot 类比        |
| ---- | ---------------------------- | --------------------- |
| 包管理  | npm / pnpm / bun             | Maven                 |
| 依赖描述 | `package.json`               | `pom.xml`             |
| 依赖存储 | `node_modules/`              | `~/.m2/repository`    |
| 入口命令 | `npm run dev`                | `mvn spring-boot:run` |
| 模块化  | ES Modules (`import/export`) | Java `import`         |
| 运行时  | Node.js / 浏览器                | JVM                   |

下一篇，我们将正式进入 **Vue 3** 的世界，学习它最核心的概念：**响应式 (Reactivity)** 和 **单文件组件 (.vue)**。

敬请期待！
