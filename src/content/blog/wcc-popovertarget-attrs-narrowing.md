---
title: "为 WCC 修复 JSX 类型：限制 popovertarget 仅适用于 button/input（Issue #236）"
description: "记录我在 ProjectEvergreen/wcc 中修复 popovertarget/popovertargetaction JSX 类型过宽的问题，并通过 @ts-expect-error 夹具做回归保护。"
pubDate: "Feb 5 2026"
tags: ["Open Source", "TypeScript", "JSX", "Web Components", "WCC", "Typing"]
---

最近在看一个小而清晰的类型问题：`popovertarget` / `popovertargetaction` 被 WCC 手动加进了 JSX attributes，但范围过宽，导致像 `<span popovertarget="...">` 这种也不会报错。

- Issue: [ProjectEvergreen/wcc#236](https://github.com/ProjectEvergreen/wcc/issues/236)
- PR: [ProjectEvergreen/wcc#237](https://github.com/ProjectEvergreen/wcc/pull/237)

## 🔍 分析 (Analyze)

现状：WCC 的 `src/jsx.d.ts` 会为所有 `HTMLElementTagNameMap` 的 tag 都注入 `popovertarget` / `popovertargetaction`。

目标：根据规范，这两个 attributes 应该只对 `<button>` 和 `<input>` 生效；其它元素（如 `<span>`）应当类型报错。

## 📍 定位 (Locate)

相关代码集中在：
- `src/jsx.d.ts`：`ElementAttributes<>` 抽取 DOM interface 后，再补充少量 TS DOM lib 缺失的 attributes。
- `test/cases/tsx/src/header.tsx`：作为 TSX 夹具，被 `tsc` 类型检查覆盖。

## 🛠️ 执行 (Execute)

1) 把 popover attributes 独立成 `PopoverTargetAttributes`。
2) 在 `IntrinsicElementsFromDom` 的映射里做条件类型：只有 `button` / `input` 才叠加 `PopoverTargetAttributes`。
3) 在 `header.tsx` 里增加一个 `@ts-expect-error` 用例，确保 `<span popovertarget="...">` 必须报错（避免未来回归）。

## ✅ 总结 (Summary)

这是一个典型的“类型边界收紧”改动：
- 让 JSX 类型更贴近规范，减少误用。
- 用 `@ts-expect-error` 做最小回归保护，改动小但很稳。

接下来等 CI 跑完和维护者 review。

