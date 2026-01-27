---
pubDate: "Jan 27 2026"
title: "持续为 Rung 贡献：修复 CLI 参数冲突与增强状态显示"
description: "记录我在 Rung 项目中的进一步贡献，包括修复 create 命令的参数冲突 bug 以及为 status 命令添加 PR 状态颜色显示。"
tags: ["open-source", "contribution", "rust", "rung", "cli", "ai-assisted"]
---

继上一篇关于重构 Rung 代码库的博文之后，我又发现了这个优秀的 CLI 工具还有进一步打磨的空间。Rung 是一个用于管理堆叠 PR (Stacked PRs) 的工具，能够极大地提升开发效率。今天，我为它提交了两个新的 Pull Request，分别解决了一个影响体验的 Bug 和增加了一个期待已久的功能。

当然，按照惯例，这些代码的实现细节和 Git 操作都是在 AI 智能体的协助下完成的。

## 贡献一：修复 CLI 参数冲突 (Fix Argument Conflict)

### 问题描述
在使用 `rung create` 创建新分支时，我习惯于同时指定分支名称和提交信息，例如：
```bash
rung create feat/new-feature -m "feat: add new feature"
```
然而，这个命令会报错，提示 `[NAME]` 参数不能与 `--message` 同时使用。这显然不符合直觉，因为同时指定这两个参数是完全合理的场景。

这个问题的原因在于 `clap` (Rust 的命令行参数解析库) 的配置。在 Rung 的代码中，这两个参数被放入了一个 `ArgGroup`，但默认情况下 Group 内的参数是互斥的。

### 解决方案
修复方案非常简单且直接。我只需在 `crates/rung-cli/src/commands/mod.rs` 的参数组定义中添加 `.multiple(true)` 属性：

```rust
#[command(group(
    clap::ArgGroup::new("create_input")
        .required(true)
        .multiple(true) // <--- 添加这一行
        .args(["name", "message"])
))]
```

这个改动提交在 [PR #106](https://github.com/auswm85/rung/pull/106)。

## 贡献二：增强状态可视化 (Color-coded Status)

### 目标
`rung status` 是使用频率最高的命令之一，它以树状结构显示当前的 PR 堆栈。但目前所有的 PR 引用（如 `#123`）都显示为相同的颜色，无法一眼区分哪些是已合并的 (Merged)、哪些还是草稿 (Draft)。

社区提出了 [Issue #76](https://github.com/auswm85/rung/issues/76)，希望为不同状态的 PR 添加颜色区分。

### 技术实现
这是一个涉及多层修改的功能增强：

1.  **API 数据获取**：首先，我完善了 `rung status --fetch` 的逻辑。利用项目已有的 `rung_github` crate，我使用了批量 GraphQL 查询 (`get_prs_batch`) 来一次性获取堆栈中所有 PR 的最新状态，这比循环发送 REST 请求要高效得多。

2.  **状态映射**：为了解耦显示逻辑，我在 `output` 模块中引入了一个新的枚举 `PrStatus`：
    ```rust
    pub enum PrStatus {
        Open,
        Draft,
        Merged,
        Closed,
    }
    ```

3.  **颜色渲染**：更新了 `pr_ref` 函数，根据状态应用不同的颜色：
    *   **Merged**: 绿色 (Green) —— 代表已完成
    *   **Draft**: 黄色 (Yellow) —— 代表进行中/未就绪
    *   **Closed**: 红色 (Red) —— 代表已关闭但未合并
    *   **Open**: 默认颜色 —— 代表正常开启状态

现在，运行 `rung status --fetch` 后，输出的堆栈图变得色彩丰富且信息量大增，用户可以立刻识别出哪些分支已经合入主干，可以安全删除了。

这个功能的实现提交在 [PR #107](https://github.com/auswm85/rung/pull/107)。

## 总结

通过这两个 PR，我们不仅修复了一个恼人的 CLI 交互 bug，还显著提升了工具的视觉体验。这也是“AI 驱动开发”模式的一次成功实践：我负责发现需求和审查方案，AI 负责快速定位代码位置、编写 Rust 代码并处理繁琐的 Git 流程。

希望这些改进能被合并，让 Rung 变得更好用！
