---
title: "Harper：扩展 time unit 场景下 few vs a few 的纠错（Issue #1114）"
description: "在 harper-core 中将既有的 few time unit 规则从仅匹配 “few <unit> ago” 扩展到更多常见上下文（After few minutes / Few minutes after 等），并补齐测试用例。"
pubDate: "Feb 5 2026"
tags: ["Open Source", "Rust", "harper", "NLP", "Linting", "GitHub"]
---

- Issue: [Automattic/harper#1114](https://github.com/Automattic/harper/issues/1114)
- PR: [Automattic/harper#2660](https://github.com/Automattic/harper/pull/2660)

## 🔍 分析 (Analyze)

`few` 和 `a few` 的语义差异在英文里非常明显：`few` 往往带有“太少/不够”的负面意味，而表达“一点点/几分钟之后”时通常应该用 `a few`。\n
Issue 里提到的真实例子非常常见，例如：
- “After **few** minutes ...”\n
- “**Few** minutes after ...”\n

harper-core 已经有一个规则能纠正 `few <time unit> ago`，但覆盖面偏窄（只看 `ago`），并且句首场景的测试被 `ignore` 掉了。

## 📍 定位 (Locate)

目标规则与测试都在：`harper-core/src/linting/few_units_of_time_ago.rs`。

## 🛠️ 执行 (Execute)

保持改动尽量小、同时尽量降低误报：
1) 将表达式从只匹配 `few <time unit> ago` 扩展为匹配 `few <time unit>`。\n
2) 在 `match_to_lint_with_context` 里结合前后文决定是否触发：\n
   - 后面接 `ago/after/before/later`，或前面接 `after/before/in/within/for/since` 时触发。\n
   - 已经有 `a/an/the` 的情况直接跳过。\n
   - 遇到明显的负面用法（如 `too/very/so/quite/how few`）跳过，避免产生 “too a few ...” 这类错误修复。\n
3) 补充测试：新增 `after few minutes` / `few minutes after`，并重新启用句首相关用例。

本地验证：`cargo fmt` + `cargo test -p harper-core`。

## ✅ 总结 (Summary)

这类规则最怕“泛化过度导致误报”，所以这次采用了“time unit + 邻近上下文关键词”的保守策略：\n
它能覆盖 Issue 里最高频的时间表达错误，同时通过少量反例测试约束行为，更容易被维护者快速 review 并合并。

