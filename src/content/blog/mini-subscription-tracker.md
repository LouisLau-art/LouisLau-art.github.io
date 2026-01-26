---
title: '从零到一：构建一个极简订阅管理工具'
description: '想试试超级轻量级的应用'
pubDate: 'Dec 16 2025'
tags: ["Archive"]
---

最近我有个想法：用最小化的技术栈，快速开发一个实用的个人工具。经过一周的迭代，我成功打造了一个**订阅管理器** —— 一个帮助追踪所有自动扣费服务的极简应用。

这篇文章将记录整个开发过程，从需求分析到最终上线，希望能给你一些启发。

## 背景与需求

### 问题的发现

你有没有这样的经历：每个月收到银行账单时，才发现自己订阅了N个根本不用的服务？Netflix、Spotify、云存储、在线工具……这些小额订阅加起来，一年竟然花了几千块。

我意识到我需要一个工具来管理这些订阅：

- 清楚地看到每月要花多少钱
- 快速找到想取消的服务
- 知道下一次扣款是什么时候

### 为什么不用现有方案？

市面上有很多记账应用（如 Notion、钉钉等），但它们要么功能过剩、要么隐私风险高。我想要的是：

- **开源且可自部署** —— 数据掌握在自己手中
- **极简且快速** —— 不需要复杂的配置
- **单文件应用** —— 没有繁琐的构建流程

## 技术方案选择

### 核心决策

我选择了一个"轻量级但不小众"的技术栈：

| 需求   | 选择                     | 原因            |
| ---- | ---------------------- | ------------- |
| 后端   | PocketBase             | 开源、自部署、无需复杂配置 |
| 前端框架 | Vue 3                  | 轻量级、学习成本低     |
| UI框架 | DaisyUI + Tailwind CSS | 开箱即用、暗色主题支持好  |
| 打包   | 单文件 HTML               | 无需构建工具，CDN 引入 |
| 部署   | 本地或云服务                 | 灵活选择          |

### 为什么不选择 Spring Boot 或 Django？

我的目标是**快速迭代**，而不是构建企业级系统。Spring Boot 和 Django 都太重了：

- 需要安装 Java/Python 环境
- 需要配置数据库
- 需要构建和部署流程
- 对于一个简单工具来说，杀鸡用牛刀

FastAPI 是个选择，但我发现 PocketBase 更适合这个场景 —— 它自带数据库、认证、API，一个二进制文件就能运行。

## 开发过程

### 第一阶段：核心功能（第1-2天）

#### 1. 数据模型设计

首先，我在 PocketBase 中创建了 `subscriptions` 集合，定义了这些字段：

```
name (Text)           - 服务名称
amount (Number)       - 金额
billing_cycle (Select) - 月付或年付
next_date (Date)      - 下次扣款日期
category (Select)     - 分类（娱乐/工作/生活）
emoji (Text)          - 表情符号
user (Relation)       - 关联用户
```

关键设计决策：

- 使用 `next_date` 而不是 `billing_date`，这样���清楚看到什么时候会被扣费
- `billing_cycle` 分为月付和年付，便于计算月度和年度支出
- `user` 字段用于用户隔离，每个用户只能看到自己的订阅

#### 2. 界面规划

我画了一个简单的原型：

```
┌─────────────────────────┐
│   我的订阅  [退出]      │
├─────────────────────────┤
│ 本月支出: ¥500          │
│ 年度预估: ¥6000         │
├─────────────────────────┤
│ [搜索框]                │
│ [全部] [娱乐] [工作]... │
├─────────────────────────┤
│ 🎬 Netflix              │
│    ¥12.99  [明天]      │
│    [编辑] [删除]        │
├─────────────────────────┤
│              [+ 按钮]   │
└─────────────────────────┘
```

#### 3. 实现基础功能

使用 Vue 3 的 Composition API，我快速实现了：

- 登录/登出
- 获取订阅列表
- 添加新订阅
- 删除订阅

代码结构很简单：

```javascript
const { createApp, ref, computed, onMounted } = Vue;
const pb = new PocketBase('http://127.0.0.1:8090');

createApp({
    setup() {
        const subscriptions = ref([]);
        const isLoggedIn = ref(false);

        const monthlyTotal = computed(() => {
            return subscriptions.value.reduce((total, sub) => {
                if (sub.billing_cycle === 'yearly') {
                    return total + (sub.amount / 12);
                }
                return total + sub.amount;
            }, 0);
        });

        // ... 其他逻辑
    }
}).mount('#app');
```

### 第二阶段：编辑功能与暗色主题（第3-4天）

#### 1. 添加编辑功能

发现初版只有"增删查"，缺少"改"。我添加了编辑模态框，统一处理添加和编辑：

```javascript
const openAddModal = () => {
    currentSubscription.value = { /* 初始值 */ };
    isEditing.value = false;
    showModal.value = true;
};

const openEditModal = (subscription) => {
    currentSubscription.value = { ...subscription };
    isEditing.value = true;
    showModal.value = true;
};

const saveSubscription = async () => {
    if (isEditing.value) {
        // 更新现有
        await pb.collection('subscriptions').update(
            currentSubscription.value.id,
            currentSubscription.value
        );
    } else {
        // 创建新的
        await pb.collection('subscriptions').create({
            ...currentSubscription.value,
            user: pb.authStore.model.id
        });
    }
};
```

#### 2. 实现暗色主题

我尝试了两种方案：

**方案 A：Tailwind 原生暗色模式**

```html
<html class="dark">
<body class="bg-gray-50 dark:bg-gray-900">
```

**方案 B：DaisyUI 主题系统**

```html
<html data-theme="dark">
```

最终选择了 **DaisyUI**，因为：

- 提供了完整的组件库（卡片、模态框、按钮等）
- 暗色主题支持更完善
- 无需自己编写大量 CSS

### 第三阶段：Bug 修复与优化（第5-7天）

#### 1. 金额输入框的 undefined 错误

这是一个隐蔽的 bug。当用户点击金额输入框的上下箭头时（而不是直接输入数字），会出现：

```
Uncaught TypeError: Cannot read properties of undefined (reading 'length')
```

根本原因是 `v-model.number` 在转换过程中，`ref` 的值还是 `undefined`。

解决方案是添加安全转换函数：

```javascript
const safeNumber = (val) => {
    const num = Number(val);
    return isNaN(num) ? 0 : num;
};

const monthlyTotal = computed(() => {
    return subscriptions.value.reduce((total, sub) => {
        const amt = safeNumber(sub.amount);  // 安全转换
        if (sub.billing_cycle === 'yearly') {
            return total + (amt / 12);
        }
        return total + amt;
    }, 0);
});
```

#### 2. 搜索和筛选功能

添加了按名称搜索和按分类筛选：

```javascript
const filteredSubscriptions = computed(() => {
    return subscriptions.value.filter(sub => {
        const matchesSearch = searchQuery.value === '' ||
            sub.name.toLowerCase().includes(searchQuery.value.toLowerCase());

        const matchesCategory = filterCategory.value === '' ||
            sub.category === filterCategory.value;

        return matchesSearch && matchesCategory;
    });
});
```

#### 3. 智能日期显示

实现了像 Slack 一样的友好日期显示：

```javascript
const formatDate = (dateString) => {
    const date = new Date(dateString);
    const today = new Date();
    const diffDays = Math.ceil((date - today) / (1000 * 60 * 60 * 24));

    if (diffDays < 0) return '已过期';
    if (diffDays === 0) return '今天';
    if (diffDays === 1) return '明天';
    if (diffDays <= 7) return `${diffDays}天后`;

    return date.toLocaleDateString('zh-CN');
};
```

#### 4. API Rules 配置

这是安全性的关键。在 PocketBase 中设置规则，确保用户只能看到自己的数据：

```
List rule: user = @request.auth.id
View rule: user = @request.auth.id
Create rule: user = @request.auth.id
Update rule: user = @request.auth.id
Delete rule: user = @request.auth.id
```

## 开发过程中的关键决策

### 1. 为什么选择 CDN 而不是 npm？

一开始我想用 Vite 来构建，但后来意识到这违背了"极简"的初衷。改用 CDN 后：

- 减少了构建步骤
- 用户可以直接打开 HTML 文件
- 没有 node_modules，项目更轻

### 2. 为什么用 PocketBase 而不是传统数据库？

PocketBase 的优势：

- 开箱即用的 REST API
- 内置用户认证系统
- 支持权限控制（API Rules）
- 单个二进制文件，部署简单
- 有网页管理后台

对比传统方案（Django + PostgreSQL），节省了至少 80% 的配置时间。

### 3. 搜索 vs 全文搜索

我一开始想用 Meilisearch 来做全文搜索，后来意识到对于这个小工具来说，前端过滤足够了。YAGNI（You Aren't Gonna Need It）原则告诉我，不要过度设计。

## 遇到的问题与解决方案

| 问题                 | 症状                                       | 解决方案                      |
| ------------------ | ---------------------------------------- | ------------------------- |
| DaisyUI require 错误 | `ReferenceError: require is not defined` | 只引入 CSS，不用 npm 插件         |
| 金额输入 undefined     | 点击上下箭头时报错                                | 添加 `safeNumber` 安全转换      |
| 用户隔离失效             | 用户能看到别人的数据                               | 配置 API Rules              |
| 日期计算错误             | 时区问题导致日期偏差                               | 使用 `setHours(0,0,0,0)` 归零 |

## 最终产品演示

### 核心功能清单

- ✅ 用户认证（登录/登出）
- ✅ CRUD 操作（增删改查）
- ✅ 财务统计（本月支出、年度预估）
- ✅ 搜索和筛选
- ✅ 智能日期显示
- ✅ 暗色主题
- ✅ 移动响应式
- ✅ 用户数据隔离

### 代码行数

- HTML: ~50 行
- CSS：0 行（全用 Tailwind）
- JavaScript: ~400 行

总代码量不到 500 行，却实现了一个完整的功能应用。

## 技术栈总结

```
Frontend:     Vue 3 (CDN)
UI:           DaisyUI + Tailwind CSS (CDN)
Backend:      PocketBase
Database:     SQLite (内置)
Auth:         PocketBase Built-in
Deployment:   Single HTML file
```

## 性能与优化

- **初加载时间**：< 1s（大多数 CDN 资源都被缓存）
- **交互响应时间**：< 100ms（数据量小，无需虚拟滚动）
- **文件大小**：HTML 文件 ~20KB，加上 CDN 资源约 500KB
- **浏览器兼容性**：Chrome、Firefox、Safari、Edge（需要 ES6 支持）

## 上线与开源

### GitHub 上线步骤

```bash
# 初始化
git init
git add .
git commit -m "Initial commit"

# 创建远程仓库
gh repo create subscription-tracker --public --push

# 后续更新
git add .
git commit -m "描述更改"
git push origin main
```

### 项目结构

```
subscription-tracker/
├── index.html       # 唯一的应用文件
├── README.md        # 项目文档
└── .gitignore       # Git 配置
```

## 如何部署和使用

### 本地运行

1. 启动 PocketBase：
   
   ```bash
   ./pocketbase serve
   ```

2. 在浏览器打开 `index.html`

3. 使用 PocketBase 中的用户账号登录

### 云端部署选项

- **PocketBase 部署**：Fly.io、Railway 等
- **前端部署**：GitHub Pages、Vercel、Netlify
- **一体化部署**：Docker 容器

## 学到的经验

### 1. 轻量级优于全能

一开始我想加入数据可视化、AI 分析、多币种支持等功能。但意识到这些都不是 MVP（最小可用产品）所需的。选择"少但精"的功能，反而让用户体验更好。

### 2. 开源工具生态强大

PocketBase、DaisyUI、Vue 这些开源项目，让我能以极低的成本实现高质量的应用。这就是开源的力量。

### 3. 单文件应用的价值

不需要 npm、Webpack、构建流程，用户直接打开 HTML 就能用。这降低了使用门槛，也简化了部署。

### 4. 用户需求决定技术选择

最开始我想用"最新最酷"的技术。但问题是，用户不关心你用什么框架，只关心是否好用。这让我更现实地评估技术选择。

## 未来方向

### 可能的增强功能

- 📱 PWA 支持（可安装到桌面）
- 📊 数据可视化（图表展示消费分布）
- 🔔 通知提醒（扣款前提醒）
- 📤 数据导出（CSV/JSON）
- 🌐 多语言支持
- 🤖 AI 分析建议（哪些订阅可以取消）

### 为什么现在不加这些？

因为它们都不是核心功能。我会根据用户反馈来决定优先级。这就是 MVP 的哲学：快速交付，持续迭代。

## 总结

这个项目从零到一只花了一周时间，却展现了一个完整的开发周期：需求分析 → 技术选择 → 编码实现 → 问题修复 → 开源上线。

关键成功因素：

1. **明确的需求** —— 知道自己要做什么
2. **合适的技术** —— 选择了轻量级方案
3. **快速迭代** —— 边做边改，不过度设计
4. **充分测试** —— 发现并修复了多个 bug
5. **及时开源** —— 分享给社区

如果你也想快速打造一个个人工具，这个技术栈是个不错的参考。不必完美，但要实用；不必全能，但要专注。

---

**项目链接**: https://github.com/LouisLau-art/subscription-tracker

**下一步**: 欢迎 Star、Fork、反馈问题和建议！