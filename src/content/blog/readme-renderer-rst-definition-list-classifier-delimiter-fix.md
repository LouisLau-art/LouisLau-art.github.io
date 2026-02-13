---
title: "为 readme_renderer 贡献：修复 RST 定义列表 classifier 分隔符丢失 (PR #353)"
description: "为 docutils 的 HTML5 输出补回 definition list classifier 的 ` : ` 分隔符，避免 PyPI 上出现 termclassifier 这类黏连文本，并补齐回归测试。"
pubDate: "Feb 13 2026"
tags: ["Open Source", "PyPI", "Python", "RST", "docutils", "Testing"]
---

这单的问题很“细”，但用户感知很强：在 README.rst 里写了标准的 definition list classifier（例如 `term : classifier`），PyPI 页面上却变成了 `termclassifier`，冒号和空格都不见了。

- Issue: [pypa/readme_renderer#317](https://github.com/pypa/readme_renderer/issues/317)
- PR: [pypa/readme_renderer#353](https://github.com/pypa/readme_renderer/pull/353)

## 🔍 分析 (Analyze)

根因不在清洗器（nh3）或 PyPI 的 HTML 展示层，而在 docutils 的 html5 writer：它会把 classifier 包在 `<span class="classifier">...</span>` 里，但不会输出分隔符（`:`）及两侧空格，导致 term 和 classifier 在纯文本层面直接拼接。

在 PyPI 这种“内容为主”的场景，分隔符是语义的一部分，缺失会直接影响可读性与复制/搜索。

## 📍 定位 (Locate)

- RST 渲染入口：`readme_renderer/rst.py`（自定义 `ReadMeHTMLTranslator`）
- 测试基于 fixtures：`tests/test_rst.py` + `tests/fixtures/test_*.rst/.html`

## 🛠️ 执行 (Execute)

- 在 `ReadMeHTMLTranslator` 里覆盖 `visit_classifier`：
  - 在输出 `<span class="classifier">` 之前插入 ` <span class="classifier-delimiter">:</span> `；
  - 行为对齐 docutils 的 html4 writer，确保 term 与 classifier 之间不会黏连。
- 增加一个最小 fixture 回归用例，锁定输出包含 delimiter。

## ✅ 总结 (Summary)

这类“渲染语义细节”的问题，最重要的是：
- 输出应当在**纯文本层面**也保持可读（不依赖 CSS 才能恢复语义）；
- 用一个最小 fixture 把行为固定住，避免 docutils/渲染链路升级时回归。

