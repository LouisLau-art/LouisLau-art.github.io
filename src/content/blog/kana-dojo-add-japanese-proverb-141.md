---
title: "超快合并型贡献：为 Kana Dojo 添加一条日语谚语（Issue #3694）"
description: "在 lingdojo/kana-dojo 中按贡献指南向 JSON 内容库追加一条日语谚语（ことわざ），属于 1 分钟级别的 good first issue。"
pubDate: "Feb 5 2026"
tags: ["Open Source", "GitHub", "PR", "Content", "Japanese", "DX"]
---

有些开源贡献不需要改一行业务逻辑：只要按项目的贡献规范补齐内容，就能快速交付一个低风险、易 review、很容易被合并的 PR。

这次我在 [Kana Dojo](https://github.com/lingdojo/kana-dojo) 里完成了一个 `good first issue`：向社区内容库追加一条日语谚语（ことわざ）。

- Issue: [lingdojo/kana-dojo#3694](https://github.com/lingdojo/kana-dojo/issues/3694)
- PR: [lingdojo/kana-dojo#3698](https://github.com/lingdojo/kana-dojo/pull/3698)

## 🔍 分析 (Analyze)

这类内容型 PR 的合并概率很高：
- 改动范围极小（单文件 JSON 增量）。\n
- 不影响核心逻辑，几乎没有回归风险。\n
- 维护者 review 成本低，通常很快就能合并。

## 📍 定位 (Locate)

Issue 已经明确指定了目标文件：
- `data/community-content/japanese-proverbs.json`

## 🛠️ 执行 (Execute)

按 issue 步骤操作即可：
1) 在 JSON 数组尾部追加一个 proverb 对象。\n
2) 校验 JSON 格式合法（避免漏逗号/括号）。\n
3) 提交 PR，并在描述中 `Closes #3694` 自动关联关闭。

## ✅ 总结 (Summary)

这种 “内容增量 + 明确指引” 的任务非常适合刷贡献节奏：
- 快、稳、低风险。\n
- 还能顺手积累一个可展示的开源足迹。\n
- 后续如果想“每天 2~3 个 PR”，这类任务可以作为稳定底座。

