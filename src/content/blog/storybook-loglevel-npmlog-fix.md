---
title: "修复 Storybook CLI：让 --loglevel / --debug 真正生效（Issue #22061）"
description: "记录我在 Storybook 中修复日志级别参数不生效的问题：统一 new logger 与 legacy npmlog 的 level，并补齐 CLI 参数校验。"
pubDate: "Feb 5 2026"
tags: ["Open Source", "Storybook", "CLI", "TypeScript", "Logging", "GitHub"]
---

最近在使用 Storybook CLI 时遇到一个体验问题：传了 `--loglevel` 但输出并没有明显变化（尤其是一些仍然走 legacy `npmlog` 的日志）。我顺手把这个坑填了，并提交了 PR。

- Issue: [storybookjs/storybook#22061](https://github.com/storybookjs/storybook/issues/22061)
- PR: [storybookjs/storybook#33776](https://github.com/storybookjs/storybook/pull/33776)

## 🔍 分析 (Analyze)

Storybook 的日志体系处于“新旧并存”的阶段：
- 部分日志走新的 `node-logger`（支持 `trace/debug/info/...`）。
- 仍有代码路径使用 legacy `npmlog`（它有自己的 level 系统，比如 `verbose/silly`）。

结果就是：CLI 在启动时调用了 `logger.setLogLevel(...)`，但这个方法只影响新 logger，不会同步到 `npmlog.level`，导致 `--loglevel` 对一部分输出“看起来没用”。

## 📍 定位 (Locate)

关键点在两处：
- `code/core/src/bin/core.ts`：解析 `--loglevel` / `--debug` 并在 `preAction` 设置日志级别。
- `code/core/src/node-logger/index.ts`：`logger.setLevel` 会同步 `npmlog`，但 `logger.setLogLevel`（来自 spread 的 new logger）不会。

## 🛠️ 执行 (Execute)

这次修复做了三件事：
1) 让 `logger.setLogLevel` 同步设置 `npmlog.level`（并做 level 映射：`trace -> silly`，`debug -> verbose`）。
2) 让 `--debug` 真正生效：在 CLI 层强制覆盖 `loglevel=debug`。
3) 为 `--loglevel` 增加 commander choices 校验，避免传入未知值。

同时补了 `node-logger` 的单测，覆盖 `setLogLevel` / `setLevel` 的一致性。

## ✅ 总结 (Summary)

这个改动属于“低风险但高收益”的体验修复：
- `--loglevel`/`--debug` 对所有日志路径都一致生效。
- 新旧 logger 行为对齐，减少排查时的困惑。
- CLI 参数更可控，降低误用成本。

后续就等 CI 跑完、维护者 review。

