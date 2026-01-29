---
title: "为 GitHub CLI 贡献后续：修复 gofmt 与 lint 失败 (PR #12527)"
description: "记录我在 GitHub CLI PR #12527 中根据评审意见运行 gofmt，修复缩进问题并恢复 CI 通过。"
pubDate: "Jan 29 2026"
tags: ["Open Source", "Go", "GitHub CLI", "CI", "Gofmt"]
---

在推进 [GitHub CLI](https://github.com/cli/cli) 的 PR [#12527](https://github.com/cli/cli/pull/12527) 时，维护者指出代码缩进问题并提示需要运行 `gofmt`。这是典型的格式化失败导致 CI 不通过，需要尽快修复。

## 🔍 分析 (Analyze)

- PR 功能已完成，但 Go 格式化未通过，影响合并。
- 评审明确要求运行 `gofmt`，属于“必须修复”的基础问题。

## 📍 定位 (Locate)

- 修改集中在 `pkg/cmd/repo/clone/clone.go` 与 `pkg/cmd/repo/clone/clone_test.go`。
- 这些文件的缩进不符合 Go 规范，导致 lint 失败。

## 🛠️ 执行 (Execute)

1) 在分支 `feat-repo-clone-no-upstream` 上运行 `gofmt`。
2) 检查 diff，确认仅为格式化变动。
3) 提交并推送更新到 PR 分支。
4) 在 PR 中回复维护者说明修复完成。

## ✅ 总结 (Summary)

这次更新没有改变功能逻辑，只是修复 Go 代码格式问题，但它对合并流程非常关键：
- CI/lint 通过，PR 可以继续评审。
- 尊重维护者反馈，减少重复沟通。

以后提交 Go 代码前，我会优先运行 `gofmt`，避免类似阻塞。
