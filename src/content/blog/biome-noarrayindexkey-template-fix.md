---
title: "为 Biome 贡献：修复 noArrayIndexKey 在模板字符串中的漏报 (PR #8968)"
description: "当 key 使用模板字符串/拼接表达式时，Biome 的 noArrayIndexKey 只看最后一个标识符，导致 `${index}-${item}` 等场景漏报。本文记录修复与测试补充。"
pubDate: "Feb 5 2026"
tags: ["Open Source", "Biome", "Rust", "Linter", "Testing", "AI"]
---

这次我继续给 [Biome](https://github.com/biomejs/biome) 做一个小而明确的修复：`lint/suspicious/noArrayIndexKey` 在 `key` 使用 **模板字符串** 或 **二元拼接表达式** 时，会出现漏报。

对应 issue：[#8812](https://github.com/biomejs/biome/issues/8812)  
对应 PR：[#8968](https://github.com/biomejs/biome/pull/8968)

## 🔍 分析 (Analyze)

规则的目标是：**不要用数组的 `index` 作为 React key**（即便混在模板字符串/拼接里也不建议）。

但原实现有两个问题：
1) **只追踪“最后一个 identifier”**：例如 `key={`${index}-${item}`}` 会把最后的 `item` 当成候选，导致漏报。
2) **模板字符串里出现非 identifier 就提前退出**：例如 `key={`${item.title}-${index}`}` 第一个表达式是 `item.title`（member expression），会直接导致整条规则返回 `None`。

## 📍 定位 (Locate)

规则实现位于：
- `crates/biome_js_analyze/src/lint/suspicious/no_array_index_key.rs`

核心逻辑是从 `key={...}` 的表达式里“抓”出一个引用，然后沿着语义模型回溯到 `map/forEach/...` 的回调参数，判断它是不是数组 index 参数。

## 🛠️ 执行 (Execute)

我的改动是把“只抓一个引用”改成“收集所有可能的引用”：
- 遍历 `key` 表达式（模板字符串 / 二元表达式 / 括号表达式）
- 收集其中所有 `identifier` 引用
- 逐个尝试解析为回调参数，并判断是否是数组 index 参数

同时补充了测试用例覆盖三种典型漏报：
- `key={`${index}-${item}`}`
- `key={`${item.title}-${index}`}`
- `key={index + item}`

## ✅ 总结 (Summary)

这次修复属于典型的“规则实现细节导致漏报”：
- 修复后，无论 `index` 在模板字符串/拼接表达式的哪个位置，都会被正确识别并报告。
- 测试补齐后，可以降低未来回归风险。

PR：[#8968](https://github.com/biomejs/biome/pull/8968)

