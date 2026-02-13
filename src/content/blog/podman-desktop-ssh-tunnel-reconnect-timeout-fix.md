---
title: "为 Podman Desktop 贡献：disconnect 时清理 Podman SSH Tunnel 重连定时器 (PR #16225)"
description: "修复 PodmanRemoteSshTunnel.disconnect 未清理重连 setTimeout，导致断开后仍可能触发重连的问题，并补齐单测。"
pubDate: "Feb 13 2026"
tags: ["Open Source", "Podman Desktop", "TypeScript", "Testing", "Extensions", "SSH"]
---

这单是一个非常典型的“定时器生命周期”问题：`disconnect()` 只是把开关关掉不够，还得把已经排队的 `setTimeout` 清理掉，否则回调到点仍会执行。

- Issue: [podman-desktop/podman-desktop#16042](https://github.com/podman-desktop/podman-desktop/issues/16042)
- PR: [podman-desktop/podman-desktop#16225](https://github.com/podman-desktop/podman-desktop/pull/16225)

## 🔍 分析 (Analyze)

`handleReconnect()` 在需要重连时会设置一个 30 秒的定时器，回调里直接调用 `connect()`。

问题在于：如果用户调用了 `disconnect()`，虽然把“允许重连”的 flag 置为了 `false`，但**已经创建的定时器并不会自动消失**。到时间后回调仍会触发，进而再次 `connect()`，造成“断开后仍尝试重连”的意外行为。

## 📍 定位 (Locate)

核心实现：
- `extensions/podman/packages/extension/src/remote/podman-remote-ssh-tunnel.ts`

对应单测：
- `extensions/podman/packages/extension/src/remote/podman-remote-ssh-tunnel.spec.ts`

## 🛠️ 执行 (Execute)

做最小修复：
1. 在 `disconnect()` 中检测 `#reconnectTimeout`，如存在则 `clearTimeout` 并置空；
2. 增加单测：使用 fake timers 验证 `disconnect()` 后推进 30 秒不会再次调用 `connect()`。

验证命令（定向）：
- `cd extensions/podman/packages/extension && pnpm vitest run src/remote/podman-remote-ssh-tunnel.spec.ts`

## ✅ 总结 (Summary)

这类修复的价值点在于“避免隐性副作用”：
- 断开连接后不会再被历史定时器拉起重连；
- 变更点小、可用单测锁住行为；
- 对维护者来说 review 成本低、也更容易合并。

