---
title: "为 Documenso 贡献：修复长描述导致模板操作按钮不可见 (PR #2491)"
description: "在 public profile 模板列表中为标题/描述增加可收缩与换行处理，避免超长/无空格内容把操作菜单或 Sign 按钮挤出可视区域。"
pubDate: "Feb 13 2026"
tags: ["Open Source", "Documenso", "TypeScript", "Remix", "Tailwind", "UI"]
---

这单是典型的 UI 布局健壮性问题：当用户把描述填到上限（尤其是无空格的长串）时，flex 布局可能把右侧操作区域“挤出去”，再叠加 `overflow-hidden` 就会直接把按钮裁掉，导致无法操作。

- Issue: [documenso/documenso#2472](https://github.com/documenso/documenso/issues/2472)
- PR: [documenso/documenso#2491](https://github.com/documenso/documenso/pull/2491)

## 🔍 分析 (Analyze)

问题本质不是“字符数太多”，而是“可换行/可收缩能力不足”：
- flex 子项默认 `min-width: auto`，遇到超长 token（无空格）时不愿意收缩；
- 右侧操作按钮/菜单被挤到容器外侧；
- 容器上又有 `overflow-hidden`，结果按钮被裁切，变成不可见/不可点。

## 📍 定位 (Locate)

涉及两个展示模板标题/描述并带操作区的列表：
- `apps/remix/app/components/tables/settings-public-profile-templates-table.tsx`
- `apps/remix/app/routes/_profile+/p.$url.tsx`

## 🛠️ 执行 (Execute)

采用最小侵入的 Tailwind 调整，目标是“文本可收缩 + 可断词 + 操作区不被挤压”：
- 给文本所在的 flex 容器加 `min-w-0`，允许在 flex 布局下收缩；
- 标题用 `truncate`，描述用 `line-clamp-3` + `break-words`，避免长串撑爆布局；
- 给操作触发器加 `flex-shrink-0`，确保右侧操作区始终保留可见空间。

## ✅ 总结 (Summary)

这类修复的关键是把 UI 的“最坏输入”当成常态：任何可被用户输入的字段都应该能在长文本/无空格/异常字符下保持布局稳定，至少不能把关键操作裁掉。

