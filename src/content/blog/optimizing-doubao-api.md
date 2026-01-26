---
title: '榨干豆包 API 的每一滴性能：我是如何开发 Doubao Batch Translator 的'
description: 'Deep Dive: 架构设计、快慢双车道并发策略与人机协作工作流'
pubDate: 'Dec 15 2025'
tags: ["Archive"]
---

最近，我终于完成了一个心心念念的项目——**Doubao Batch Translator**。

这不仅仅是一个翻译脚本，更是一个为了极致挖掘**豆包（Doubao）**免费 API 额度而生的**高性能异步翻译工具**。从最初的简单 JSON 翻译，到后来支持 EPUB 电子书、HTML、Markdown，再到适配“沉浸式翻译”插件的 HTTP Server，这个项目的演进过程充满了技术挑战和架构优化的乐趣。

今天就来聊聊这个项目的开发历程、架构设计，以及我是如何通过“快慢双车道”策略将并发性能拉满的。

> **项目地址**: [Doubao Batch Translator on GitHub](https://github.com/LouisLau-art/doubao-batch-translator)

## 🎯 为什么要造这个轮子？

豆包的 API 非常良心，尤其是 `doubao-seed-translation` 模型，提供了一定的免费额度。但是，它的并发限制（RPM/TPM）和 API 格式让简单的脚本很难高效利用它。

如果不加控制，请求太快会被 429 限流；请求太慢则浪费了时间。我的目标很明确：**在不触发限流的前提下，把并发跑满，实现最快的翻译速度。**

## 🏗️ "一核多壳" 的架构设计

为了支持多种文件格式（JSON, HTML, EPUB, MD），我采用了一个 **"One Core, Multiple Shells" (一核多壳)** 的架构：

*   **Core (`AsyncTranslator`)**: 这是一个基于 `asyncio` 的核心翻译引擎。它只负责一件事情：高效、稳定地把文本发给 API 并拿回结果。它处理了所有的重试逻辑、频率限制（Rate Limiting）和并发控制。
*   **Shells (Processors)**: 针对不同格式的文件，我写了不同的处理器。
    *   `json_worker`: 处理 RenPy 游戏翻译文件。
    *   `html_worker`: 解析 HTML 结构。
    *   `epub_worker`: 处理复杂的电子书结构（这个最头疼，后面细说）。
    *   `md_worker`: 最新加入的 Markdown 支持，能智能识别代码块和 Frontmatter，只翻译该翻的地方。

## 🚀 核心黑科技：快慢双车道并发策略

这是本项目最让我自豪的优化点。

豆包的不同模型有不同的限流策略。比如 `doubao-seed` 是“慢车道”，RPM（每分钟请求数）限制在 5000 左右；而像 DeepSeek 或 Doubao Pro 这种高性能模型是“快车道”，支持更高的并发。

为了榨干性能，我设计了一套**智能分流系统**：

1.  **🐢 慢车道 (Seed 模型)**: 系统会自动识别如果你用的是免费的 Seed 模型，它会将并发严格限制在 **80** (接近 5000 RPM 的理论上限)。实测下来，吞吐量能稳定在 80 req/s，利用率高达 **96%**。
2.  **🏎️ 快车道 (Pro/DeepSeek)**: 如果你切换到付费或高性能模型，系统会瞬间解锁限制，将并发飙升至 **500**。

这种灵活的策略，让我在省钱（用免费额度）和赶时间（用高性能模型）之间可以无缝切换。

## 📖 EPUB 翻译与“人机协作”工作流

翻译电子书（EPUB）是最复杂的场景。EPUB 本质上是一堆 HTML 的压缩包，处理起来坑非常多（比如 XML 声明重复、循环引用等 bug）。

除了解决这些底层 bug，我更关注**翻译质量**。AI 翻译难免有漏译或错译。为了解决这个问题，我开发了一套完整的 **Human-in-the-loop (人机协作)** 工作流：

1.  **批量粗翻**: 直接把几十本 EPUB 丢进去跑，程序会自动由 AI 处理。
2.  **自动质检**: 程序会检测漏译片段，生成 `漏译报告.txt` 和一个**可编辑的 `人工翻译.json`**。
3.  **人工精修**: 你只需要编辑这个 JSON 文件，填入你认为正确的译文。
4.  **一键回填**: 运行 `apply-fix` 命令，程序会自动把你的译文精准地“缝合”回 EPUB 电子书里，不需要你手动去解压打包。

这个功能让它不仅仅是一个玩具，而是真正可用于生产高质量电子书的工具。

## 🔌 适配“沉浸式翻译”插件

作为一个重度阅读者，我离不开浏览器里的“沉浸式翻译”插件。既然我已经把豆包的 API 调教得这么好了，为什么不让浏览器也用上呢？

于是我写了一个 **FastAPI Server**，完全兼容 OpenAI 的接口格式。

你只需要在本地运行 `python main.py server`，然后在沉浸式翻译插件里把 API 地址填成 `http://localhost:8000/v1/chat/completions`，就能在浏览网页时，享受到为了各种模型专门优化的极速翻译体验。

## 📝 结语

从最初的随手一写，到现在的模块化、高并发、多格式支持，开发 Doubao Batch Translator 的过程让我对 `asyncio` 编程和 API 性能调优有了更深的理解。

目前项目已经开源，如果你也有大量文件翻译的需求，或者想把豆包的免费羊毛薅到极致，欢迎来试用和 Star！

Github: [https://github.com/LouisLau-art/doubao-batch-translator](https://github.com/LouisLau-art/doubao-batch-translator)
