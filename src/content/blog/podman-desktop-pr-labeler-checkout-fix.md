---
title: "为 Podman Desktop 贡献：修复 PR Labeler workflow 未 checkout 导致 NOP (PR #16227)"
description: "在 pr-labeler GitHub Actions workflow 中增加 actions/checkout（checkout base SHA），让 actions/labeler 能读取 .github/labeler.yml 正常打标签。"
pubDate: "Feb 13 2026"
tags: ["Open Source", "Podman Desktop", "GitHub Actions", "CI"]
---

这单属于“看起来跑了，其实没干活”的 CI 小坑：workflow 里调用了 `actions/labeler`，但没有 `checkout` 代码，导致 `configuration-path: .github/labeler.yml` 在工作目录里根本不存在，最终相当于 NOP。

- Issue: [podman-desktop/podman-desktop#14443](https://github.com/podman-desktop/podman-desktop/issues/14443)
- PR: [podman-desktop/podman-desktop#16227](https://github.com/podman-desktop/podman-desktop/pull/16227)

## 🔍 分析 (Analyze)

`actions/labeler` 需要读取仓库里的配置文件（这里是 `.github/labeler.yml`）。如果 job 没有先 `checkout`，runner 的 workspace 里没有任何 repo 文件，action 就无法找到配置，自然也无法按规则打标签。

此外，这个 workflow 用的是 `pull_request_target`（为了用 bot token 写 PR labels）。在这种触发器下，**checkout 的 ref 需要格外谨慎**，避免把 PR 的不可信代码拉到带 secrets 的上下文里执行。

## 📍 定位 (Locate)

- workflow：`.github/workflows/pr-labeler.yaml`

## 🛠️ 执行 (Execute)

- 在 Labeler step 前增加一个 pinned 的 `actions/checkout`；
- 显式 checkout PR 的 base SHA：`ref: ${{ github.event.pull_request.base.sha }}`，确保只拿到基准分支的可信内容（主要是 labeler 配置文件）。

## ✅ 总结 (Summary)

这是典型的“CI plumbing”修复：改动很小，但能把一个实际无效的 workflow 变成真正工作，同时也保持 `pull_request_target` 的安全边界（只 checkout base 代码）。

