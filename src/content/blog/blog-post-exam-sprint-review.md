---
title: "ExamSprint 开发复盘：Nuxt 4、SQLite 与那些踩过的坑"
pubDate: 'Dec 15 2025'
tags: ["Archive"]
description: "一个全栈备考管理应用的开发实录。从 LocalStorage 到 SQLite，从 ApexCharts 到 nuxt-charts 再回滚，深度复盘 SSR 状态管理与 UI 细节打磨。"
---

在过去的几天里，我开发了 **ExamSprint** —— 一个为考试冲刺设计的备考管理系统。这不仅仅是一个简单的 Todo List，而是一个集成了任务管理、目标追踪、番茄钟专注和数据统计的全栈应用。

这个项目采用了相当激进（尝鲜）的技术栈：**Nuxt 4 (Beta)**、**Tailwind CSS v4**、**Nuxt UI v4**，以及 **Bun** 作为运行时。

在这篇文章中，我想跳过那些枯燥的"如何创建项目"，直接聊聊在开发过程中遇到的**真实挑战（坑）**，那些**技术取舍**，以及为了极致体验所做的**细节打磨**。

## 🛠️ 技术栈一览

*   **框架**: Nuxt 4 + Vue 3 (Composition API)
*   **UI 库**: Nuxt UI v4 (基于 Radix Vue，设计感极佳)
*   **数据库**: SQLite (`better-sqlite3`)
*   **运行时**: Bun (快，真的快)
*   **状态管理**: Pinia (配合 SSR 的特殊处理)
*   **图表**: ApexCharts
*   **部署**: SSR 全栈

---

## 💥 踩坑实录：那些让我们头秃的时刻

### 1. 服务端导入的路径迷宫 (ENOENT Error)

**背景**：
为了实现数据持久化，我们决定从 `LocalStorage` 迁移到服务端的 `SQLite`。我创建了一个 `server/utils/db.ts` 来初始化数据库连接，试图在 `server/api/tasks/index.ts` 中引入它。

**问题**：
最开始，我像写常规 Node.js 代码一样使用相对路径导入：
```typescript
import db from '../../utils/db' // 或者 ~/server/utils/db
```
在开发环境一切正常，但在某些构建场景下，Nuxt (Nitro) 抛出了 `ENOENT` 错误。因为 Nitro 在打包服务端代码时，文件结构会发生变化，导致硬编码的相对路径失效。

**解决方案（真·魔法）**：
**利用 Nitro 的自动导入 (Auto-import)**。
不需要显式 `import`！只要文件位于 `server/utils` 目录下，Nitro 会自动扫描并导出它。
我们删除了所有的 `import db from ...`，直接在 API 处理函数中使用全局可用的 `db` 对象。问题瞬间解决，代码还更干净了。

### 2. 日期选择器的三生三世

**第一阶段：消失的组件**
我们试图封装一个 `DatePicker.vue`。但在引用时，Vue 居然报 `Failed to resolve component`。排查了半天 Nuxt 的自动导入配置，甚至尝试手动引入，依然不稳定。这直接导致用户反馈："添加任务不能选日期了"。

**第二阶段：回归原生**
为了优先保证功能（MVP 思维），我果断把所有日期选择器降级可以工作的 `<UInput type="date" />`。丑是丑了点，但能用。

**第三阶段：UCalendar 的救赎**
功能跑通后，我们不能容忍原生控件在不同浏览器下的丑陋和不一致。最终，我们利用 `UPopover` 包裹 `UCalendar`，手写了一个 `CalendarPicker.vue` 组件，实现了既美观又统一的交互体验。

**教训**：先让它跑起来 (Make it work)，再让它变好 (Make it better)。不要在项目初期死磕一个 UI 细节。

### 3. 图表库的反复横跳

统计页面需要展示"专注时长趋势"。

*   **Round 1**: 使用 **ApexCharts**。老牌库，功能强大但包体积略大。工作正常。
*   **Round 2**: 看到 **nuxt-charts** (基于 unovis) 发布了 v2.0，心想用原生 Nuxt 模块肯定更好且更轻量。于是大刀阔斧地重构了代码，卸载了 ApexCharts。
*   **Round 3**: 启动报错。`yAxis is required`，但由于文档缺失，按照直觉配置 `y-label` 无效。去翻源码才发现属性名是驼峰 `yLabel`。修复后又遇到 Vue 渲染周期的报错。
*   **Round 4 (结局)**：果断回滚。

**教训**：
对于生产力工具，**稳定性 > 新颖性**。`nuxt-charts` 很有潜力，但对于一个需要快速交付的项目，成熟的 ApexCharts 虽然"重"一点，但它稳。我们浪费在折腾新库上的时间，本可以用来开发两个新功能。

---

##  进阶技巧与细节打磨

除了填坑，我们也在一些技术细节上做了深耕：

### 1. SSR 下的 Pinia 水合问题 (`skipHydrate`)

在从 LocalStorage 迁移到 SSR 的过程中，我们遇到了一个经典问题：**水合不匹配 (Hydration Mismatch)**。
服务端渲染出的 HTML 初始状态，和客户端从 Pinia 读取到的状态（尤其是涉及到时间倒计时、复杂对象时）不一致。

我们使用了 Pinia 的 `skipHydrate` 辅助函数，明确告知 Nuxt 哪些状态不需要在客户端重新水合，或者如何在客户端接管状态。这保证了在页面刷新时，倒计时组件不会出现"闪烁"或时间跳变的 bug。

### 2. 番茄钟的即时反馈

为了让番茄钟真的"好用"，我们把状态直接做在 Pinia 里，利用 `Web Audio API`（或者简单的 Audio 对象）播放白噪音。
更重要的是，我们设计了**目标进度圈**的动态颜色逻辑。当你进度落后时，它是醒目的颜色；当你完成目标时，它会自动变成绿色。这不是简单的 CSS，而是基于 TS 计算属性的动态样式绑定。

### 3. 深色模式的极致适配

ExamSprint 的深色模式不是无脑反色。
*   我们使用了 `slate-50` 到 `slate-900` 的色阶，而不是纯黑。
*   图表库的 Tooltip 在深色模式下往往会看不清文字。我们专门为 ApexCharts 编写了 `tooltip.theme: 'dark'` 的配置，并手动调整了 CSS，确保在任何主题下，数据都清晰可读。

### 4. 为什么是 Bun？

在整个开发过程中，**Bun** 给了我极大的惊喜：
*   **依赖安装**：`npm install` 需要 1 分多钟，`bun install` 只需要 20 秒。在反复尝试不同图表库时，这个速度差异感知极强。
*   **启动速度**：Nuxt 的冷启动速度体感提升了 30% 以上。
*   **兼容性**：除了偶尔遇到生僻包的解析问题，它对 Node 生态的兼容性已经完全足够支撑这样一个全栈项目。

---

## 🚀 架构升级：从 LocalStorage 到 SQLite

最初版 ExamSprint 数据存在浏览器里。这有个致命问题：我在电脑上制定的复习计划，躺在床上用手机看时就没了。

迁移到 Nuxt Server API + SQLite 是一个质的飞跃：
1.  **架构分层**：以前 Store 直接操作 state 并同步到 LocalStorage；现在 Store 变成了 API 的消费者。这种**关注点分离**让代码逻辑更清晰。
2.  **better-sqlite3 的选择**：为什么不用 Drizzle ORM？
    *   因为项目只有 5 张表。
    *   因为我想让代码对初学者更透明（直接看 SQL 就能懂）。
    *   在小规模项目中，手写 SQL 的开发速度往往快于配置 ORM。

---

## 📝 总结

ExamSprint 的开发过程完美诠释了"敏捷开发"的精髓：

1.  **拥抱变化**：从简单的 UI 到复杂的全栈架构。
2.  **适度折腾**：尝试了 Nuxt 4 和 Bun，体验极佳；但在图表库上折腾过度，及时止损。
3.  **用户体验至上**：哪怕是后台逻辑再复杂，前台的加载状态、空状态（Empty State）、深色模式适配（Dark Mode）一样都不能少。

现在的 ExamSprint 已经是一个可以在本地稳定运行、数据持久化的全栈应用了。

**Next Step?** 也许是 Docker 化部署，也许是添加用户认证系统。技术的探索永无止境！

---

*本文源码已开源至 GitHub: [ExamSprint](https://github.com/LouisLau-art/exam-sprint)*
