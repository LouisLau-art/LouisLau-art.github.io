---
title: '挑战高性能 Rust 生态：为 Ruff 贡献代码'
description: '修复 Ruff 中的 FURB180 规则误删注释的 Bug，体验 Rust 高性能 Linter 的开发流程。'
pubDate: 'Jan 23 2026'
tags: ["Rust", "Open Source", "Linter"]
---


在完成了 Vue 生态的贡献后，今天我决定挑战一下更硬核的领域 —— 高性能 Rust 生态。我为目前最快的 Python Linter [Ruff](https://github.com/astral-sh/ruff) 提交了一个 Bug 修复。

## PR 详情

- **仓库**: astral-sh/ruff
- **PR 编号**: #22824
- **标题**: refactor(refurb): mark MetaClassABCMeta fix as unsafe if comments would be deleted
- **链接**: [https://github.com/astral-sh/ruff/pull/22824](https://github.com/astral-sh/ruff/pull/22824)

## 发现问题

我在研究 `FURB180` 规则时发现了一个问题。该规则旨在将 `class C(metaclass=abc.ABCMeta):` 优化为 `class C(abc.ABC):`。

然而，如果代码中包含注释，例如：

```python
class C(
    other_kwarg=1,
    # 这是一个重要的注释
    metaclass=abc.ABCMeta,
):
    pass
```

Ruff 的自动修复（Safe Fix）会直接删除这个注释，并把代码压缩成一行。这显然是不符合预期的。

## 修复方案

在 Rust 编写的 Ruff 源码中，我通过以下逻辑进行了修正：

1.  **检测注释范围**：利用 `checker.comment_ranges()` 检查待删除或替换的代码范围内是否包含注释。
2.  **调整修复安全性**：如果范围内存在注释，则将该修复的 `Applicability` 从 `Safe` 降级为 `Unsafe`。
3.  **文档更新**：同步更新了规则的文档，向用户说明在存在注释或多个基类时，该修复可能是不安全的。

```rust
if applicability == Applicability::Safe {
    let range = if position > 0 {
        TextRange::new(class_def.keywords()[position - 1].end(), keyword.end())
    } else {
        keyword.range()
    };
    if checker.comment_ranges().intersects(range) {
        applicability = Applicability::Unsafe;
    }
}
```

## 技术心得

Ruff 的开发体验非常出色。尽管代码库很庞大，但 Rust 的强类型系统和完善的集成测试框架（基于 `insta` 的快照测试）让重构变得非常有信心。

这次贡献让我对 AST（抽象语法树）的操作有了更深的理解，同时也感受到了 Rust 在处理这类大规模逻辑时的严谨性。

---

**保持好奇，继续探索更高性能的世界！** 🦀
