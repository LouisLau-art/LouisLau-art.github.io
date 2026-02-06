---
title: "Issue/PR Digest：已完成提交与待合并进展（2026-02-06）"
description: "记录本轮已完成的 issue 处理与 PR 提交（含待 review 项），不只统计 merged，方便持续跟踪与复盘。"
pubDate: "Feb 6 2026"
tags: ["Open Source", "GitHub", "PR Digest", "Issue Tracking", "AI"]
---

这次开始把“已经做了但还没 merge”的 issue/PR 也纳入博客追踪，避免只在合并后记录导致过程不可见。

## 🔍 分析 (Analyze)

只统计 merged 会遗漏大量真实产出：
- 问题定位、修复实现、测试补充都已经完成，但在等待 review/CI；
- 这些工作同样可验证、可追溯，也决定了后续合并效率。

因此这次改成双轨记录：**Merged** + **In Progress（已提交待合并）**。

## 📍 定位 (Locate)

本次整理基于 `gh`：
- 个人近期 PR 列表（按更新时间）
- 对应 issue 的评论/关联状态
- 本地已完成提交与新开 PR 链接

## 🛠️ 执行 (Execute)

### 已 merge（近期）

- 2026-02-05 [storybookjs/storybook#33776 Logger: Honor --loglevel for npmlog output](https://github.com/storybookjs/storybook/pull/33776)
- 2026-02-05 [shenjingnan/xiaozhi-client#860 fix(frontend): cleanup copy timeout on unmount](https://github.com/shenjingnan/xiaozhi-client/pull/860)

### 已完成并提交（待 review / 待合并）

- [cli/cli#12622](https://github.com/cli/cli/pull/12622) `gh pr create` 支持 `--json/--jq` 输出，补充互斥校验与测试
- [cli/cli#12623](https://github.com/cli/cli/pull/12623) `gh issue list -S 'is:pr'` 直接报错并引导 `gh pr list`
- [RoboSats/robosats#2418](https://github.com/RoboSats/robosats/pull/2418) `/api/book` 无订单时从 `404` 改为 `200 + []`，补空簿测试
- [kubernetes-sigs/cluster-api-provider-openstack#3000](https://github.com/kubernetes-sigs/cluster-api-provider-openstack/pull/3000) 清理重复 `nolint`，统一 lint 规则
- [pulumi/esc#617](https://github.com/pulumi/esc/pull/617) 2 段环境引用歧义提示
- [python-poetry/poetry#10715](https://github.com/python-poetry/poetry/pull/10715) `poetry self` 锁文件提示命令修正
- [python-poetry/poetry#10716](https://github.com/python-poetry/poetry/pull/10716) Windows 上 `bash` 执行 `poetry env activate` 时按 shell 输出 `source ...`，并补回归测试（关联 [#10395](https://github.com/python-poetry/poetry/issues/10395)）
- [python-poetry/poetry#10717](https://github.com/python-poetry/poetry/pull/10717) 修复仓库名包含 `.` 时配置路径被错误拆分的问题（关联 [#1328](https://github.com/python-poetry/poetry/issues/1328)），统一处理 `repositories/http-basic/pypi-token/certificates` 的键路径转义
- [astral-sh/uv#17888](https://github.com/astral-sh/uv/pull/17888) 修复 `uv python find --project` 场景下未优先解析目标项目 `.venv` 的问题（关联 [#11990](https://github.com/astral-sh/uv/issues/11990)）
- [delta-io/delta-kernel-rs#1783](https://github.com/delta-io/delta-kernel-rs/pull/1783) 为 `Transaction` 增加 blind append 支持，写入 `CommitInfo.isBlindAppend=true` 并补充语义校验与测试（关联 [#1771](https://github.com/delta-io/delta-kernel-rs/issues/1771)）
- [pytest-dev/pytest-reportlog#100](https://github.com/pytest-dev/pytest-reportlog/pull/100) 去除 report message 中 ANSI 转义
- [vueuse/vueuse#5227](https://github.com/vueuse/vueuse/pull/5227) 跟进 reviewer 评论，确认清理重复 SSR 测试并改为更聚焦的回归用例

## ✅ 总结 (Summary)

从今天开始，贡献记录从“结果导向（只记 merged）”升级为“过程 + 结果”：
- 结果层：看已合并产出
- 过程层：看已完成但待合并队列

这样可以更清楚地看到真实交付节奏，也方便后续做周报、复盘和优先级调整。
