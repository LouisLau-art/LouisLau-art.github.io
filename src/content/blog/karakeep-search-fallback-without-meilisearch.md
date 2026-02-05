---
title: "为 Karakeep 增加搜索回退：Meilisearch 不可用时用标题 LIKE（Issue #1187）"
description: "记录我在 karakeep-app/karakeep 中为书签搜索增加轻量回退：当 Meilisearch 未配置或临时不可用时，回退到数据库 title-only 搜索，保证最小安装也能用。"
pubDate: "Feb 5 2026"
tags: ["Open Source", "Karakeep", "Search", "TypeScript", "tRPC", "DX", "GitHub"]
---

Karakeep 的搜索体验很依赖 Meilisearch：它带来全文检索能力，但在“最小安装”或 Meilisearch 临时不可用时，用户几乎没法检索，只能滚动列表翻找。

这次我补了一个轻量回退：当搜索引擎未配置或查询时报错时，退化到数据库的 **title-only LIKE** 搜索，至少能用“标题关键字”把书签找出来。

- Issue: [karakeep-app/karakeep#1187](https://github.com/karakeep-app/karakeep/issues/1187)
- PR: [karakeep-app/karakeep#2456](https://github.com/karakeep-app/karakeep/pull/2456)

## 🔍 分析 (Analyze)

目标不是替代全文检索，而是保证可用性：
- Meilisearch 不存在/没启动时，用户仍能搜索标题（最基础能力）。
- Meilisearch 临时挂了时，不要直接 500，让用户“至少还能搜到标题匹配”。

## 📍 定位 (Locate)

核心入口是 `bookmarks.searchBookmarks`（tRPC）：
- 原逻辑：必须拿到 `getSearchClient()`，否则直接报 “Search functionality is not configured”。
- 影响：最小安装（未配 Meili）完全无法搜索。

## 🛠️ 执行 (Execute)

实现策略（尽量小改动）：
1) 保留原有流程：优先用 search client 做搜索（Meilisearch 正常时完全不变）。\n
2) 当 client 不存在或 `client.search()` 报错时：回退到 `bookmarks.title LIKE %query%`。\n
3) 不引入额外日志噪音；回退只影响结果能力（title-only），不会影响 qualifier/matcher 的过滤逻辑。\n
4) 增加单测覆盖“未配置搜索引擎也能搜标题”的路径。

## ✅ 总结 (Summary)

这个回退不是“最强搜索”，但能显著提升实际可用性：
- 最小部署也能通过标题关键字检索。\n
- Meilisearch 临时不可用时仍能提供降级体验。\n
- 改动范围可控、测试可验证，便于合并。

后续就等 CI 跑完与维护者 review。

