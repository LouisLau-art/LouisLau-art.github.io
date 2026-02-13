---
title: "为 Podman Desktop 贡献：修复容器详情页 TTY 标签大小写 (PR #16224)"
description: "将容器详情页的 Tty 标签与面包屑统一为 TTY（缩写一致性），并做定向测试验证。"
pubDate: "Feb 13 2026"
tags: ["Open Source", "Podman Desktop", "TypeScript", "Svelte", "UI", "Testing"]
---

这单是典型的 UI 文案一致性修复：把缩写从 `Tty` 统一成 `TTY`，降低用户阅读成本、也让界面更专业。

- Issue: [podman-desktop/podman-desktop#15663](https://github.com/podman-desktop/podman-desktop/issues/15663)
- PR: [podman-desktop/podman-desktop#16224](https://github.com/podman-desktop/podman-desktop/pull/16224)

## 🔍 分析 (Analyze)

`TTY` 是常见缩写（teletypewriter / terminal 相关语境），在 UI 上用 `Tty` 会显得不一致、也不符合常见写法。

这类修复虽然小，但变更面极窄、风险低，维护者也容易快速 review / merge。

## 📍 定位 (Locate)

容器详情页的 tab 与路由面包屑在这里定义：
- `packages/renderer/src/lib/container/ContainerDetails.svelte`

TTY 终端页的空态提示在这里：
- `packages/renderer/src/lib/container/ContainerDetailsTtyTerminal.svelte`

## 🛠️ 执行 (Execute)

做最小改动：
1. 将 tab 标题与 breadcrumb 从 `Tty` 改为 `TTY`；
2. 将空态文案 `Tty has stopped` 改为 `TTY has stopped`。

验证命令（定向）：
- `pnpm run build:core-api`
- `pnpm vitest --run --project=renderer packages/renderer/src/lib/container/ContainerDetails.spec.ts`

## ✅ 总结 (Summary)

这是一个“低成本、高可见”的 UI polish：
- 用户界面的缩写更符合习惯；
- 变更范围小，回归风险低；
- 有定向测试覆盖，便于维护者快速合并。

