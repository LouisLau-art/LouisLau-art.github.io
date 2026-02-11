---
title: "为 Biome 贡献：修复 hr + role=presentation 的误报 (PR #9028)"
description: "Biome 规则 noInteractiveElementToNoninteractiveRole 会把 `<hr role=\"presentation\">` 误判为错误。本文记录定位、修复与测试验证。"
pubDate: "Feb 11 2026"
tags: ["Open Source", "Biome", "Rust", "Linter", "Accessibility", "AI"]
---

这次继续在 [Biome](https://github.com/biomejs/biome) 做一个小而明确的 a11y 规则修复。

- Issue: [biomejs/biome#9024](https://github.com/biomejs/biome/issues/9024)
- PR: [biomejs/biome#9028](https://github.com/biomejs/biome/pull/9028)

## 🔍 分析 (Analyze)

`lint/a11y/noInteractiveElementToNoninteractiveRole` 的目标是阻止“交互元素被赋予非交互 role”。

但在当前实现中，`<hr role="presentation" />` 会被误判。`hr` 的隐式语义是 `separator`，而 `presentation/none` 在这里是合法的语义抹平方式，不应报错。

## 📍 定位 (Locate)

核心规则文件：
- `crates/biome_js_analyze/src/lint/a11y/no_interactive_element_to_noninteractive_role.rs`

对应测试：
- `crates/biome_js_analyze/tests/specs/a11y/noInteractiveElementToNoninteractiveRole/valid.jsx`
- `crates/biome_js_analyze/tests/specs/a11y/noInteractiveElementToNoninteractiveRole/valid.jsx.snap`

## 🛠️ 执行 (Execute)

我做了最小行为修复：
1. 在规则里加入特判：当元素是 `hr` 且 `role` 为 `presentation` 或 `none` 时直接放行；
2. 在 `valid.jsx` 增加用例：
   - `<hr role="presentation" />`
   - `<hr role="none" />`
3. 更新 snapshot 并跑定向测试：
   - `cargo test -p biome_js_analyze --test spec_tests specs::a11y::no_interactive_element_to_noninteractive_role`

## ✅ 总结 (Summary)

这次改动范围小、风险低，目标是修正明确误报并保持其它场景行为不变。

修复后，`hr` 在 `presentation/none` 场景下不会再被错误拦截，同时测试覆盖也补齐，后续回归风险更低。
