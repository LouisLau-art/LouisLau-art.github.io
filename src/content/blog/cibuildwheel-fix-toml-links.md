---
title: "为 cibuildwheel 贡献：修复 options 文档里 TOML 链接 (PR #2740)"
description: "修复 docs/options.md 中缺失的 TOML reference definition，避免站点渲染出破损的 [TOML] 链接。"
pubDate: "Feb 13 2026"
tags: ["Open Source", "cibuildwheel", "Python", "Docs"]
---

这单是一个很典型的“reference link 写法 + 定义缺失”导致的文档渲染问题：页面上出现了 `[TOML][]` 这种看起来像链接、实际点不了的文本。

- Issue: [pypa/cibuildwheel#2725](https://github.com/pypa/cibuildwheel/issues/2725)
- PR: [pypa/cibuildwheel#2740](https://github.com/pypa/cibuildwheel/pull/2740)

## 🔍 分析 (Analyze)

`docs/options.md` 里用的是 Markdown reference-style 链接（例如 `[uv][]` + `[uv]: ...`）。但 `TOML` 这处只有引用，没有定义，导致静态站点构建后呈现为破损链接。

这类问题修复思路很直接：要么补齐 reference definition，要么改成 inline 链接。项目整体风格偏 reference-style，所以补齐定义更一致。

## 📍 定位 (Locate)

- 文档文件：`docs/options.md`
- 触发点：两处 `[TOML][]`（table/list 示例段落）

## 🛠️ 执行 (Execute)

- 将 `[TOML][]` 改为 `[TOML]`（使用 shortcut reference link 形式）；
- 增加缺失的定义：`[TOML]: https://toml.io/en/`；
- 保持其它链接定义风格一致，不引入额外格式改动。

## ✅ 总结 (Summary)

这类“细小但显眼”的文档修复很适合作为快速贡献：改动小、review 成本低，但能直接改善用户阅读体验，也能减少后续重复提问/报错。

