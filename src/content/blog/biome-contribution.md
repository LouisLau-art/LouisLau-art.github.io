---
title: "为 Biome 贡献：修复 @supports 查询中的 CSS Lint 规则"
pubDate: "Jan 24 2026"
description: "介绍我如何通过修复 @supports 查询中的 CSS linting 问题为 Biome JavaScript 工具链做贡献"
tags: ["open-source", "contribution", "rust", "css", "biome", "linting"]
---

今天我向 [Biome](https://github.com/biomejs/biome) 项目做了另一次开源贡献，这是一个用 Rust 编写的快速 JavaScript 工具链。这一次，我解决了一个与 `@supports` 查询中 CSS linting 相关的问题。

## 问题

问题出现在 Biome 的 CSS linter 规则 `useGenericFontNames` 中。此规则强制要求 CSS font-family 声明包含通用字体系列（如 `sans-serif`、`serif`、`monospace` 等）作为后备。然而，它在 `@supports` 查询中错误地触发，而 `@supports` 是用于功能检测的条件 CSS 块。

例如，以下 CSS 被错误地标记：

```css
@supports (font: -apple-system-body) {
  a { font-family: Arial; }
}
```

该问题报告为 [biomejs/biome#8845](https://github.com/biomejs/biome/issues/8845)。

## 解决方案

我实现了一个修复，检测 CSS 属性何时位于 `@supports` 规则内，并在这些上下文中跳过 linting 检查。修复涉及：

1. 添加辅助函数 `is_in_supports_at_rule()` 来检测 CSS 属性是否在 `@supports` 规则内
2. 修改主要规则逻辑，在 `@supports` 块内跳过验证
3. 添加测试用例以确保修复正确工作

## 技术实现

实现需要理解 Biome 的 CSS 解析的 AST（抽象语法树）结构。我利用了用于检测 `@font-face` 规则的现有模式，并扩展它以处理 `@supports` 规则。

```rust
fn is_in_supports_at_rule(node: &CssGenericProperty) -> bool {
    node.syntax()
        .ancestors()
        .find(|n| n.kind() == CssSyntaxKind::CSS_AT_RULE)
        .and_then(|n| n.cast::<CssAtRule>())
        .and_then(|n| n.rule().ok())
        .is_some_and(|n| matches!(n, AnyCssAtRule::CssSupportsAtRule(_)))
}
```

## 关键要点

1. **理解 AST 遏历**：使用 Biome 的语法树帮助我理解代码解析器在内部如何工作
2. **上下文感知 linting**：linter 需要考虑代码出现的上下文，而不仅仅是语法本身
3. **测试至关重要**：添加全面的测试用例确保修复正确工作，而不会破坏现有功能

## 结论

此贡献通过使 Biome 了解 CSS 功能查询，提高了其 CSS linting 的准确性。修复确保开发人员可以使用 `@supports` 规则而不会从 linter 获得误报，同时在常规 CSS 上下文中仍保持规则的预期功能。

拉取请求已提交，可在 [biomejs/biome#8848](https://github.com/biomejs/biome/pull/8848) 查看。