---
title: "Headscale 贡献记录：修复 OIDC 场景下 ephemeral 节点未生效（#2719）"
description: "记录我在 juanfont/headscale 的一次 Go 修复：让 `tailscaled --state=mem:` 在 interactive/OIDC 注册路径下也能正确标记并持久化为 ephemeral 节点。"
pubDate: "Feb 11 2026"
tags: ["Open Source", "Go", "Headscale", "OIDC", "Auth", "GitHub"]
---

这次是新单，从 issue 到修复、测试、开 PR 一次走完。

- Issue: [juanfont/headscale#2719](https://github.com/juanfont/headscale/issues/2719)
- PR: [juanfont/headscale#3078](https://github.com/juanfont/headscale/pull/3078)
- Commit: `a14d4601`

## Analyze

问题现象很明确：客户端使用 `tailscaled --state=mem:` 并走 OIDC 登录后，节点在服务端没有被当作 ephemeral。

根因是服务端历史上主要通过 `PreAuthKey.Ephemeral` 判断 ephemeral，而 interactive/OIDC 路径本身不依赖 pre-auth key，导致该路径丢失了 ephemeral 语义。

## Locate

关键链路在两段：

- 注册入口：`hscontrol/auth.go`
- 节点落库与更新：`hscontrol/state/state.go`

另外，数据库层和统计/筛选逻辑也依赖旧判断：

- `hscontrol/db/node.go`
- `hscontrol/state/debug.go`
- `hscontrol/db/schema.sql` 与 `hscontrol/db/db.go`（迁移）

## Execute

本次修复做了三件事：

- 给 `nodes` 增加持久字段 `ephemeral`，并在 migration 中从已有 `pre_auth_keys.ephemeral` 回填历史数据。
- interactive 注册路径把 `RegisterRequest.Ephemeral` 传到注册缓存并最终写入节点。
- 统一 ephemeral 判定与查询逻辑，改为节点级为主（同时兼容 pre-auth key 语义）。

同时补了两类测试：

- `TestAuthenticationFlows` 新增 interactive + `Ephemeral: true` 场景；
- `TestListEphemeralNodes` 新增“无 pre-auth key 但节点 ephemeral=true”的数据库场景。

## Summary

这次修复的价值在于把 ephemeral 从“认证方式附属属性”提升为“节点自身属性”，让 OIDC/web 注册路径与 pre-auth key 路径行为一致。

结果是：`--state=mem:` 的节点在 interactive 场景下也能被正确识别、统计和生命周期处理。
