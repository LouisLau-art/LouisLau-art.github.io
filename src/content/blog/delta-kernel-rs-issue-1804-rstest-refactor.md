---
title: "Delta Kernel Rust 贡献记录：#1804 用 rstest 重构 table feature 校验测试"
description: "记录我在 delta-io/delta-kernel-rs 上对 issue #1804 的第一轮实现：把 actions 模块高重复校验测试改为 rstest 参数化用例。"
pubDate: "Feb 11 2026"
tags: ["Open Source", "Rust", "Delta Lake", "Testing", "rstest", "GitHub"]
---

这次是一个典型“重构测试、提升维护性”的任务：不改业务逻辑，只改测试组织方式。

- Issue: [delta-io/delta-kernel-rs#1804](https://github.com/delta-io/delta-kernel-rs/issues/1804)
- PR: [delta-io/delta-kernel-rs#1825](https://github.com/delta-io/delta-kernel-rs/pull/1825)

## Analyze

`actions/mod.rs` 里 table feature 校验测试存在明显重复：

- 同一套协议校验逻辑被多段循环/多个单测重复覆盖；
- 增加新 case 时，需要手动复制结构；
- case 意图（哪组输入对应哪种预期）不够直观。

`rstest` 已在仓库其他位置使用，适合作为统一风格推进。

## Locate

本轮聚焦一个高重复区域，避免大范围改动：

- `kernel/src/actions/mod.rs`

主要重构以下测试组：

- `test_validate_table_features_invalid`
- `test_validate_table_features_valid`
- legacy table-feature 校验相关的多个独立测试

## Execute

第一轮实现内容：

- 将 invalid / valid 两组 table-feature 校验改为 `#[rstest]` 参数化 case；
- 将 legacy 成功/失败场景合并为单个参数化测试矩阵；
- 保持断言语义不变，仅降低重复并提升可读性。

验证命令：

- `cargo fmt --all`
- `cargo test -p delta_kernel validate_table_features`
- `cargo test -p delta_kernel test_validate_legacy_table_features`

## Summary

这次改动的价值在于“可维护性增益”：

- 同类测试 case 更集中，新增场景更容易；
- 失败时能更快定位到具体 case；
- 保持行为不变，review 风险低、易合并。

如果 maintainer 认可这轮范围，我会按 issue 列表继续扩展到其它高重复测试文件。

## Follow-up

在 PR 评论反馈前，我先做了第二轮小步扩展（commit: `ea02b5a`）：

- 新增参数化范围：`kernel/src/table_configuration.rs`
  - `test_is_feature_info_supported_writer`
  - `test_is_feature_info_supported_reader_writer`
- 同样保持“仅重构测试组织，不改行为逻辑”。

第二轮验证命令：

- `cargo fmt --all`
- `cargo test -p delta_kernel is_feature_info_supported_writer`
- `cargo test -p delta_kernel is_feature_info_supported_reader_writer`

PR 跟进评论：<https://github.com/delta-io/delta-kernel-rs/pull/1825#issuecomment-3881655148>

## Follow-up 2

我又补了第三轮（commit: `1b15674`），继续沿着 issue 里提到的 `create_table` 测试组做参数化：

- 重构文件：`kernel/tests/create_table.rs`
- 合并两个成对测试：
  - `test_clustering_stats_columns_within_limit`
  - `test_clustering_stats_columns_beyond_limit`
- 新增参数化测试：`test_clustering_stats_columns`
  - case 1：`10` 列 + `col5`
  - case 2：`40` 列 + `col35` + 期望 stats 列数 `33`

第三轮验证命令：

- `cargo fmt --all`
- `cargo test -p delta_kernel test_clustering_stats_columns`

PR 跟进评论：<https://github.com/delta-io/delta-kernel-rs/pull/1825#issuecomment-3881662224>
