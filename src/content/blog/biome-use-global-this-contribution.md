---
pubDate: "Jan 24 2026"
title: "为 Biome 贡献：添加 useGlobalThis 规则"
description: "介绍我如何为 Biome 添加新的 useGlobalThis lint 规则，检测 window、self 和 global 的使用并建议替换为 globalThis"
tags: ["open-source", "contribution", "rust", "biome", "linting", "javascript"]
---

# 为 Biome 贡献：添加 useGlobalThis 规则

今天我向 [Biome](https://github.com/biomejs/biome) 项目做出了另一项贡献，添加了一个新的 lint 规则 `useGlobalThis`，用于检测 `window`、`self` 和 `global` 的使用并建议替换为 `globalThis`。这个 PR 已经提交，可以在 [biomejs/biome#8852](https://github.com/biomejs/biome/pull/8852) 查看。

## 问题

JavaScript 中有多个全局对象引用方式：`window`（浏览器）、`self`（Web Workers）、`global`（Node.js）。然而，这些对象在不同环境中有所不同，导致代码可移植性问题。ES2020 引入了 `globalThis`，提供了一个标准化的全局对象访问方式，适用于所有环境。

## 解决方案

我实现了一个新的 lint 规则，检测对 `window`、`self` 和 `global` 的引用，并提供自动修复功能，将它们替换为 `globalThis`：

1. **检测全局对象引用**：规则识别对 `window`、`self` 和 `global` 的引用
2. **避免误报**：检查这些标识符是否被局部变量遮蔽
3. **提供修复**：自动将全局对象引用替换为 `globalThis`

## 技术实现

规则使用 Biome 的语义分析功能来区分全局和局部标识符：

```rust
fn run(ctx: &RuleContext<Self>) -> Self::Signals {
    let node = ctx.query();
    let name = node.name().ok()?;

    match name.value_token().ok()?.text() {
        "window" | "self" | "global" => {
            // 检查这是否是对全局对象的引用
            // 确保它不是被局部变量遮蔽的
            let model = ctx.model();
            let reference = node.name().ok()?;
            
            // 如果标识符绑定到局部声明，则不标记
            if model.binding(&reference).is_some() {
                return None;
            }

            // 检查是否是我们想要替换的全局对象之一
            match name.value_token().ok()?.text() {
                "window" => Some(GlobalObject::Window),
                "self" => Some(GlobalObject::Self_),
                "global" => Some(GlobalObject::Global),
                _ => None,
            }
        }
        _ => None,
    }
}
```

## 主要优势

1. **跨环境兼容性**：`globalThis` 在所有环境中提供一致的全局对象访问
2. **代码可移植性**：提高代码在不同 JavaScript 环境中的可移植性
3. **现代化**：鼓励使用 ES2020 标准功能
4. **自动修复**：提供安全的自动修复功能

## 结论

此贡献增强了 Biome 的 JavaScript linting 功能，帮助开发者编写更可移植和现代化的代码。`useGlobalThis` 规则检测旧式的全局对象引用并建议使用标准化的 `globalThis`，同时避免对同名局部变量的误报。

Pull Request 已提交至 [biomejs/biome#8852](https://github.com/biomejs/biome/pull/8852)，目前正在审查中。