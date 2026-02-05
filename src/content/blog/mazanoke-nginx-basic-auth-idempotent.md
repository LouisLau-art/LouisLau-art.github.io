---
title: "修复 Mazanoke Nginx Basic Auth 重启重复注入：让 basicauth.sh 幂等（Issue #49）"
description: "Mazanoke 在启用 Basic Auth 时，容器重启会重复写入 auth_basic 指令导致 Nginx 报错并 crash loop。本次通过让 basicauth.sh 幂等化并在启动时始终执行脚本，解决重复注入问题。"
pubDate: "Feb 5 2026"
tags: ["Open Source", "Docker", "Nginx", "Mazanoke", "DevOps", "Shell", "GitHub"]
---

最近在使用 [Mazanoke](https://github.com/civilblur/mazanoke) 时遇到一个很“典型的容器问题”：开启 Nginx Basic Auth 后，容器重启会把 `auth_basic` 相关配置**再次注入**到 Nginx 配置里，导致 Nginx 启动时报 “directive is duplicate” 并进入 crash loop。

- Issue: [civilblur/mazanoke#49](https://github.com/civilblur/mazanoke/issues/49)
- PR: [civilblur/mazanoke#50](https://github.com/civilblur/mazanoke/pull/50)

## 🔍 分析 (Analyze)

这类问题的本质是：启动脚本“只会追加、不会清理”，导致配置文件在多次启动后逐步变脏。容器环境下重启是常态，因此 **启动脚本必须尽量幂等（idempotent）**。

## 📍 定位 (Locate)

关键在 `scripts/basicauth.sh`：
- 当设置了 `USERNAME` / `PASSWORD` 时，它会向 Nginx 配置里插入 `auth_basic` 与 `auth_basic_user_file`。
- 但脚本不会检查/清理已有的同类配置，重启后就会重复插入。

## 🛠️ 执行 (Execute)

我做了两点改动，尽量保持实现简单、可预测：

1) **让脚本幂等化**：在插入前先清理已有的 `auth_basic` / `auth_basic_user_file` 行，确保反复执行仍然只有一份配置。\n
2) **启动时始终执行 `basicauth.sh`**：即使没设置账号密码，也让脚本有机会移除旧的 Basic Auth 配置，避免“上一次开过、这一次关掉却残留”的脏状态。

## ✅ 总结 (Summary)

这次修复的价值是把“容器重启”变成安全操作：
- 反复重启不会重复注入配置。\n
- 开/关 Basic Auth 都能得到可预期的最终 Nginx 配置。\n
- 改动集中在启动脚本与 Docker 启动流程，风险可控、易于 review。

