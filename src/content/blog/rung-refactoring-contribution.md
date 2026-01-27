---
pubDate: "Jan 24 2026"
title: "为 Rung 贡献：重构 CLI 参数以提高可维护性"
description: "介绍我如何通过将函数参数重构为结构化选项对象为 Rung CLI 工具做贡献"
tags: ["open-source", "contribution", "rust", "refactoring", "rung", "cli"]
---

今天我向 [Rung](https://github.com/auswm85/rung) 项目做了另一次开源贡献，这是一个用于 PR 堆叠的 CLI 工具和 VS Code 插件。我执行了一项重构任务，将多个函数参数合并到一个结构体中，如 [issue #89](https://github.com/auswm85/rung/issues/89) 中所要求的。

## 问题

Rung 代码库中的 `restack::run()` 函数累积了 8 个布尔值/选项参数，需要 clippy 抑制 `too_many_arguments` 和 `fn_params_excessive_bools`。建议将这些参数合并到 `RestackOptions` 结构体中，以提高可维护性。

原始函数签名：
```rust
pub fn run(
    json: bool,
    branch: Option<&str>,
    onto: Option<&str>,
    dry_run: bool,
    continue_: bool,
    abort: bool,
    include_children: bool,
    force: bool,
) -> Result<()>
```

## 解决方案

我实施了重构，将 8 个参数合并到 `RestackOptions` 结构体中：

1. **创建 `RestackOptions` 结构体** 以持有所有参数：

```rust
pub struct RestackOptions<'a> {
    pub json: bool,
    pub branch: Option<&'a str>,
    pub onto: Option<&'a str>,
    pub dry_run: bool,
    pub continue_: bool,
    pub abort: bool,
    pub include_children: bool,
    pub force: bool,
}
```

2. **更新函数签名** 以接受结构体而不是单独参数

3. **移除 clippy 抑制** 不再需要

4. **更新调用点** 在 `main.rs` 中传递结构体

5. **更新所有内部引用** 使用 `opts.field_name` 语法

## 技术实现

重构很简单但需要注意细节：

- 保留了所有现有功能，同时改进了代码结构
- 更新了函数内部的所有参数引用以使用 `opts.field_name` 语法
- 修改了 `main.rs` 中的调用以在调用函数前构造 `RestackOptions` 结构体
- 确保所有测试仍能通过（通过 `cargo check` 验证）

## 主要优势

1. **改进可读性**：函数签名现在更简洁，只有一个参数而不是 8 个。

2. **更好的可维护性**：将来添加新选项只需更新结构体，无需更改函数签名。

3. **自我文档化**：结构体清楚地表明哪些选项属于 restack 命令。

4. **减少 Clippy 抑制**：消除了 `#[allow(clippy::fn_params_excessive_bools, clippy::too_many_arguments)]` 的需要。

5. **一致的模式**：遵循 Rust CLI 中组织命令选项的常见模式。

## 结论

此重构显著改进了 restack 命令的可维护性，同时保留了所有现有功能。更改将 8 个参数整合到一个结构化选项对象中，使代码更清洁，便于未来增强。

拉取请求已提交，可在 [auswm85/rung#90](https://github.com/auswm85/rung/pull/90) 查看。这解决了 [auswm85/rung#89](https://github.com/auswm85/rung/issues/89) 问题，并增强了 Rung 代码库的长期可维护性。目前该 PR 已经被项目维护者合并。