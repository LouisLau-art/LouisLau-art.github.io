---
title: 'Rust 极简翻译器：从 Go 迁移到低内存 + systemd 自启'
description: '记录一个极简翻译器的 Rust 重写：低内存、无前端框架、离线依赖、systemd 静默自启。'
pubDate: 'Jan 29 2026'
tags: ["Rust", "Backend", "Performance", "Systemd"]
---

这次把原本的 Go 翻译器重写成 Rust 版本，目标非常明确：**极简、快、低内存、能静默自启**，并且保留 Markdown + LaTeX 渲染与历史记录。

## 🔍 分析 (Analyze)

痛点集中在两类：
- 运行时要轻：长时间后台运行时占用低、稳定；
- 部署要快：新机器上 clone 后直接跑起来，不依赖 CDN。

## 📍 定位 (Locate)

重写的关键点：
- 后端改为 Rust + axum，尽量减少依赖与内存占用
- 前端不用框架，纯原生 JS
- marked + MathJax 走本地静态文件（`static/libs`）
- UI 允许拖拽调整左右面板，支持一键“专注输出”
- 翻译历史改为底部抽屉，不挤占主翻译区域
- systemd 服务静默启动、开机自启

## 🛠️ 执行 (Execute)

- **后端**：实现 `/api/translate`、`/api/languages`、`/api/health`，并加入 LRU 缓存与简单限流。
- **前端**：输入后防抖翻译，输出 Markdown 渲染并触发 MathJax typeset；历史记录保存在 localStorage。
- **离线依赖**：从旧项目直接复制 `static/libs/`，避免 CDN 造成的速度瓶颈。
- **systemd**：安装脚本将二进制与静态资源复制到 `/opt/doubao-translator-rust`，服务静默运行，自动重启。

关键仓库：
- `https://github.com/LouisLau-art/doubao-translator-rust`

## 🚀 总结

这次迁移带来的最大收益是**启动与部署更可控、运行成本更低**。Rust 的编译速度确实慢一些，但一旦完成首次构建，后续增量和运行表现非常稳定。

如果后续要进一步压榨性能，优先方向会是：减少依赖、收紧日志输出、优化请求路径。
