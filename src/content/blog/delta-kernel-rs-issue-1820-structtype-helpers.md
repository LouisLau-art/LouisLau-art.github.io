---
title: "Delta Kernel Rust 贡献记录：#1820 StructType 字段插入/删除/替换辅助方法"
description: "记录我在 delta-io/delta-kernel-rs 完成 issue #1820 的实现过程：新增 StructType helper API、补测试、重构事务 schema 插入逻辑并提交 PR。"
pubDate: "Feb 11 2026"
tags: ["Open Source", "Rust", "Delta Lake", "Schema", "Testing", "GitHub"]
---

这篇记录一次“已完成并已提交 PR”的 issue 贡献，不等 merge 再写，方便后续追踪 review 进展。

- Issue: [delta-io/delta-kernel-rs#1820](https://github.com/delta-io/delta-kernel-rs/issues/1820)
- PR: [delta-io/delta-kernel-rs#1824](https://github.com/delta-io/delta-kernel-rs/pull/1824)

## Analyze

这个 issue 的核心诉求是把 `StructType` 的常见字段变更操作标准化，避免业务侧重复写“找位置 + 手工改 Vec”的易错逻辑。目标是：

- API 清晰：插入、删除、替换都由 `StructType` 自身提供；
- 错误可预期：缺失字段、重复字段名等情况返回明确错误；
- 落地验证：用现有事务 schema 逻辑做一次真实替换，证明 helper 可用。

## Locate

本次改动集中在两个文件：

- `kernel/src/schema/mod.rs`
- `kernel/src/transaction/mod.rs`

其中 `schema/mod.rs` 负责新增 helper 与单测，`transaction/mod.rs` 负责把原本手工插入 `dataChange` 字段的逻辑改为调用新 helper。

## Execute

实现内容：

- 新增 `with_field_inserted(after, field)`；
- 新增 `with_field_removed(name)`；
- 新增 `with_field_replaced(name, field)`；
- 补齐成功路径与错误路径测试（missing anchor / missing field / duplicate name）；
- 将 `ADD_FILES_SCHEMA_WITH_DATA_CHANGE` 改为调用 `with_field_inserted("modificationTime", ...)`，去掉手工索引计算。

本地验证：

- `cargo fmt --all`
- `cargo test -p delta_kernel`

## Summary

这次改动属于“低风险、可复用”的基础能力补齐：

- 减少 schema 组装时的重复样板代码；
- 降低手工插入字段时的越界/错位风险；
- 通过真实调用点替换证明 helper 设计可直接用于生产路径。

后续按 reviewer 反馈继续迭代，优先保持改动小、测试完整、可快速评审。
