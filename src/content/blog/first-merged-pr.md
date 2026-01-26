---
title: '终于！我的第一个被合并的 PR'
description: '在 vueuse 这样重量级的开源项目中贡献代码，PR 成功合并！'
pubDate: 'Jan 23 2026'
tags: ["Archive"]
---


今天是个值得纪念的日子 —— 我在 [vueuse](https://github.com/vueuse/vueuse) 仓库的第一个 PR 被合并了！

## PR 详情

- **仓库**: vueuse/vueuse
- **PR 编号**: #5225
- **标题**: fix(useMagicKeys): handle undefined key in keyboard events
- **链接**: [https://github.com/vueuse/vueuse/pull/5225](https://github.com/vueuse/vueuse/pull/5225)

## 修复内容

这个 PR 修复了 `useMagicKeys` 组合式函数中的一个边界情况问题。在处理键盘事件时，如果 key 为 undefined，会导致运行时错误。修复很简单，就是添加了一个安全检查：

```typescript
// 修复前
const key = e.key.toLowerCase()

// 修复后
const key = e.key?.toLowerCase() ?? ''
```

## 感受

vueuse 是 Vue 生态系统中非常核心的工具库，被成千上万的项目使用。能在这样一个重量级的开源项目中贡献代码，感觉真的很棒！

虽然只是一个小小的 bug 修复，但这是我开源贡献之路的第一步。希望未来能为开源社区做出更多贡献。

## 其他 PR

除了这个已合并的 PR，我还有几个 PR 正在等待 review：

- naive-ui: 2 个 PR（date-picker 相关功能）
- una-ui: 3 个 PR（主题、颜色变量、checkbox 修复）
- vueuse: 2 个 PR（Nuxt 指令注册、SSR 兼容性）

期待它们也能顺利合并！

---

**开源之路，始于足下！** 🚀
---
