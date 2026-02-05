---
title: "修复 React 组件 setTimeout 未清理导致的资源泄漏（Issue #859）"
description: "在 xiaozhi-client 前端组件中将复制提示的 setTimeout 存入 ref，并在组件卸载时清理，避免内存泄漏与卸载后 setState 警告。"
pubDate: "Feb 5 2026"
tags: ["Open Source", "React", "TypeScript", "Frontend", "Code Quality", "GitHub"]
---

这次是一个很常见但容易被忽略的问题：组件里 `setTimeout` 没有保存 handle，也没有在卸载时 `clearTimeout`。结果就是：
- 组件卸载后定时器仍可能触发，导致 “Can't perform a React state update on an unmounted component” 的警告。\n
- 定时器闭包持有状态引用，造成不必要的资源占用（典型的小型内存泄漏）。

- Issue: [shenjingnan/xiaozhi-client#859](https://github.com/shenjingnan/xiaozhi-client/issues/859)
- PR: [shenjingnan/xiaozhi-client#860](https://github.com/shenjingnan/xiaozhi-client/pull/860)

## 🔍 分析 (Analyze)

“复制成功提示 2 秒后消失”这类 UX 很常见，往往用 `setTimeout(() => setCopied(false), 2000)` 实现。\n
关键点在于：组件生命周期内可能触发多次复制，也可能在 2 秒内卸载，所以需要：
- 复用/覆盖旧 timer（避免叠加多个 timer）。\n
- 卸载时清理 timer（避免卸载后 setState）。

## 📍 定位 (Locate)

问题出现在两个组件：
- `apps/frontend/src/components/version-display.tsx`\n
- `apps/frontend/src/components/tool-debug-dialog.tsx`

## 🛠️ 执行 (Execute)

实现方式保持最小改动：
1) 用 `useRef<ReturnType<typeof setTimeout> | null>` 保存 timer。\n
2) 每次设置新 timer 前先清理旧 timer。\n
3) 通过 `useEffect(() => () => clearTimeout(...), [])` 在卸载时清理。

并在本地跑了前端的 lint 与 typecheck，确保改动不引入格式/类型问题。

## ✅ 总结 (Summary)

这是典型的 “小改动 + 高收益”：
- 避免卸载后状态更新警告。\n
- 避免 timer 累积与闭包残留。\n
- 代码更符合 React 组件生命周期的最佳实践，也更容易被维护者快速合并。

