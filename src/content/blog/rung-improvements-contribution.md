---
title: '开源贡献：优化 Rung CLI 的创建与变基流程 (PR #106, #90)'
description: '为 Rust 开发的 CLI 工具 Rung 改进易用性，包括支持更灵活的创建参数以及重构变基参数结构。'
pubDate: 'Jan 28 2026'
tags: ["Open Source", "Rust", "CLI", "AI"]
---

在参与 [Rung](https://github.com/auswm85/rung) 项目的过程中，我提交并成功合并了两个关于改进 CLI 体验的 PR。Rung 是一个用 Rust 编写的高效开发工具，而我的任务是让它的命令交互变得更加人性化和结构化。

## 🔍 分析 (Analyze)

在第一个问题（#106）中，用户发现 `rung create` 命令在处理分支名称和提交消息时存在冲突：如果同时提供分支名和 `-m` 消息，工具会报错或行为不符合预期。

在第二个问题（#90）中，随着 `restack` 命令功能的增加，函数参数列表变得冗长且难以维护。这是一种典型的代码异味（Code Smell），需要通过结构化重构来解决。

## 📍 定位 (Locate)

*   **#106**: 逻辑位于 `crates/rung_cli/src/commands/create.rs`，负责解析用户输入的参数。
*   **#90**: 逻辑位于 `crates/rung_cli/src/commands/restack.rs` 及其关联的 service 层，涉及大量直接传递的布尔值和字符串参数。

## 🛠️ 执行 (Execute)

### 1. 灵活的创建参数 (#106)
我重写了参数校验逻辑，允许用户在不冲突的情况下灵活组合参数。现在的逻辑优先解析分支名称，并能正确处理与之关联的消息描述。

### 2. 参数结构化重构 (#90)
我引入了 `RestackOptions` 结构体，将原本散落在函数签名中的多个参数（如 `force`, `no_verify`, `target_branch` 等）封装起来。这不仅缩短了函数定义，也为未来的功能扩展打下了基础。

```rust
// 重构后的参数结构示例
pub struct RestackOptions {
    pub force: bool,
    pub no_verify: bool,
    pub target: Option<String>,
}
```

## 🚀 总结

通过这两次贡献，Rung 的 CLI 代码变得更加整洁，用户体验也得到了提升。这展示了开源贡献中**易用性改进 (UX)** 和 **代码重构 (Refactoring)** 的重要性。

- PR #106: `fix: allow both branch name and -m message in rung create`
- PR #90: `refactor(cli): consolidate restack parameters into RestackOptions struct`