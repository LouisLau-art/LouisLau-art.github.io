---
title: "Delta Kernel Rust：集中管理 TableProperties 默认值（Issue #1821）"
description: "为 delta-kernel-rs 添加 TableProperties 统一 accessor，集中处理协议默认值，并迁移现有调用点以减少 unwrap_or 分散逻辑。"
pubDate: "Feb 11 2026"
tags: ["Open Source", "Rust", "Delta Lake", "Refactor", "Issue Tracking", "GitHub"]
---

- Issue: [delta-io/delta-kernel-rs#1821](https://github.com/delta-io/delta-kernel-rs/issues/1821)
- PR: [delta-io/delta-kernel-rs#1823](https://github.com/delta-io/delta-kernel-rs/pull/1823)

## 🔍 分析 (Analyze)

这个 issue 的核心不是“新增能力”，而是“收敛默认值语义”。

之前有些默认值散落在调用点，例如 `unwrap_or(false)`、`unwrap_or_default()`。短期看没问题，但长期会造成两个风险：
- 新调用点容易忘记协议默认值；
- 同一属性在不同模块里可能被写成不同默认行为。

## 📍 定位 (Locate)

主要涉及这些位置：
- `kernel/src/table_properties.rs`
- `kernel/src/scan/data_skipping/stats_schema/column_filter.rs`
- `kernel/src/table_configuration.rs`
- `kernel/src/transaction/mod.rs`
- `kernel/src/table_features/mod.rs`

## 🛠️ 执行 (Execute)

这次采用“在 `TableProperties` 提供 resolved accessor”的方案：

- 新增 accessor：
  - `data_skipping_num_indexed_cols()`
  - `is_change_data_feed_enabled()`
  - `is_row_tracking_enabled()`
  - `is_row_tracking_suspended()`
- 在上述调用点替换 ad-hoc `unwrap_or` 逻辑，统一走 accessor。
- 补齐单测，验证：
  - 默认值行为正确；
  - 显式配置覆盖默认值时行为正确。

本地验证：
- `cargo fmt --all`
- `cargo test -p delta_kernel`

## ✅ 总结 (Summary)

这类重构的价值在于“减少未来错误”，不是立刻增加功能。

把默认值解释权收口到 `TableProperties` 后，调用方只消费业务语义，不再重复携带协议细节。对后续新增属性和重构来说，review 成本和回归风险都会更低。
