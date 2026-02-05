---
title: "给 Kubetail 更新 AGENTS/CLAUDE：让 AI 贡献指引更贴近现状（Issue #706）"
description: "记录我在 kubetail-org/kubetail 中更新 AGENTS.md 与 CLAUDE.md：对齐 Make targets 与 dashboard-ui 脚本，修正 Jotai/Recoil 描述，并补充常用 CI 命令入口。"
pubDate: "Feb 5 2026"
tags: ["Open Source", "Docs", "Kubetail", "Monorepo", "AI", "GitHub"]
---

偶尔最容易被忽略、但又很影响贡献体验的东西，就是 repo 里的“AI/贡献助手文档”。Kubetail 的 `AGENTS.md` / `CLAUDE.md` 已经有一段时间没更新了，一些命令和描述（比如测试命令、状态管理库）开始和现状不一致。

- Issue: [kubetail-org/kubetail#706](https://github.com/kubetail-org/kubetail/issues/706)
- PR: [kubetail-org/kubetail#946](https://github.com/kubetail-org/kubetail/pull/946)

## 🔍 分析 (Analyze)

文档类 PR 的目标是“降低上手成本 + 避免踩坑”：
- 命令要和 `package.json` / `Makefile` 对齐（不然新贡献者/agent 很容易跑错）。
- 架构描述要跟实际技术栈一致（避免误导）。
- 最常用的入口最好能一眼看到（例如 `make ci-checks`）。

## 📍 定位 (Locate)

本次主要更新两个文件：
- `AGENTS.md`：面向通用 agent 的快速指引。
- `CLAUDE.md`：面向 Claude Code 的项目结构与常用命令。

## 🛠️ 执行 (Execute)

做了几处小而关键的对齐：
1) dashboard UI 测试命令对齐为 `pnpm test run`（与脚本/Makefile一致）。\n
2) 在 `AGENTS.md` 增加 “Common Build / CI Commands” 段落，指向 `make ci-checks`。\n
3) 修正前端状态管理描述：Jotai（而不是 Recoil）。\n
4) 说明 Rust 检查通常围绕 `crates/rgkl`（与 Make targets 的使用方式一致）。

## ✅ 总结 (Summary)

这是一个“零业务改动，但提升贡献成功率”的 PR：
- 降低新贡献者/agent 的沟通成本。\n
- 减少因为命令不一致导致的无效 debug。\n
- 帮维护者省掉反复解释“应该怎么跑”的时间。

后续等 CI 通过与维护者 review。

