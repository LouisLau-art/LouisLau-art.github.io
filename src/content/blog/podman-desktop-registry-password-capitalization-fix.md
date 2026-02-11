---
title: "为 Podman Desktop 贡献：修复 Registry 设置中的 Password 大小写 (PR #16169)"
description: "在 Settings > Registry 场景中，密码字段文案是小写 password。本文记录我如何做最小改动修复并补齐测试。"
pubDate: "Feb 11 2026"
tags: ["Open Source", "Podman Desktop", "TypeScript", "Svelte", "UI", "Testing"]
---

这次接的是一个小而明确的 UI 文案一致性问题，目标是快速修复、低风险合并。

- Issue: [podman-desktop/podman-desktop#14747](https://github.com/podman-desktop/podman-desktop/issues/14747)
- PR: [podman-desktop/podman-desktop#16169](https://github.com/podman-desktop/podman-desktop/pull/16169)

## 🔍 分析 (Analyze)

问题点很直接：Registry 相关输入里，用户名文案是 `Username`，但密码文案是小写 `password`，视觉与术语风格不一致。

这种问题虽然小，但属于高频可见路径（设置页），而且修复面可控，适合做成“快速合并型”贡献。

## 📍 定位 (Locate)

核心组件：
- `packages/renderer/src/lib/ui/PasswordInput.svelte`

受影响测试：
- `packages/renderer/src/lib/ui/PasswordInput.spec.ts`
- `packages/renderer/src/lib/preferences/PreferencesRegistriesEditing.spec.ts`

## 🛠️ 执行 (Execute)

改动保持最小化：
1. 将 `PasswordInput` 的 `placeholder` 从 `password` 改为 `Password`；
2. 将 `aria-label` 从 `password {id}` 改为 `Password {id}`；
3. 同步更新两处 renderer 单测中的断言文本。

验证命令（定向）：
- `pnpm run build:preload:types`
- `pnpm run build:ui`
- `pnpm vitest --run --project=renderer packages/renderer/src/lib/ui/PasswordInput.spec.ts packages/renderer/src/lib/preferences/PreferencesRegistriesEditing.spec.ts`

## ✅ 总结 (Summary)

这单属于典型“文案一致性 + 测试对齐”任务：
- 变更范围小、回归风险低；
- 对用户可见；
- 维护者评审成本低。

继续按这个节奏，优先找“未认领 + 无冲突 PR + 可定向验证”的小单，能稳定提高合并率。
