---
pubDate: "Jan 24 2026"
title: "为 Rustrak 贡献：添加全面的贡献指南"
description: "介绍我如何通过创建详细的 CONTRIBUTING.md 指南为 Rustrak 错误跟踪系统做贡献"
tags: ["open-source", "contribution", "documentation", "rust", "rustrak", "community"]
---

今天我向 [Rustrak](https://github.com/AbianS/rustrak) 项目做了另一次开源贡献，这是一个超轻量级、自托管的错误跟踪系统，与 Sentry SDK 兼容。我创建了一份全面的 `CONTRIBUTING.md` 文件，如 [issue #7](https://github.com/AbianS/rustrak/issues/7) 中所要求的。

## 问题

Rustrak 项目在 README.md 文件中引用了 "贡献指南"，但实际的 `CONTRIBUTING.md` 文件不存在。这为想要了解如何参与项目的潜在贡献者造成了障碍。

该问题要求创建一个贡献指南，包括：

- 入门说明和先决条件
- 开发工作流程和分支命名约定
- 项目结构概述
- Rust 和 TypeScript 组件的代码标准
- 提交更改的流程
- 社区准则

## 解决方案

我创建了一份全面的 `CONTRIBUTING.md` 文件，涵盖了问题中提到的所有要求。指南包括：

1. **入门部分**：详细的先决条件说明（Rust、Node.js、pnpm、Docker）和本地开发设置，带逐步命令。

2. **开发工作流程**：分支命名约定和提交消息格式的清晰指南。

3. **项目结构**：monorepo 结构的概述，使用 Turborepo 和 pnpm 管理，解释了不同组件（服务器、UI、客户端包）。

4. **代码标准**：Rust（服务器）和 TypeScript/JavaScript（UI）组件的具体指南，包括格式化和测试要求。

5. **提交更改**：创建问题和提交拉取请求的清晰流程。

6. **社区准则**：行为准则引用和获取帮助的指导。

## 技术实现

贡献指南精心制作，以匹配我在存储库中观察到的现有项目结构和开发实践：

- 引用了现有的 CLAUDE.md 文件以获取组件特定的上下文
- 包含了运行测试、linting 和格式化的正确命令
- 提供了开发设置的 Docker 特定说明
- 使用了与现有文档相同的术语

## 主要优势

1. **降低入门门槛**：新贡献者现在有清晰的说明，了解如何设置他们的开发环境。

2. **一致的贡献**：标准化的工作流程和代码标准有助于保持贡献的一致性。

3. **更好的质量**：清晰的测试和审查要求有助于保持高质量的代码。

4. **社区增长**：完善的贡献流程文档鼓励更多社区参与。

## 结论

此贡献填补了 Rustrak 项目文档的重要空白，使新贡献者更容易参与项目。全面的指南涵盖了参与项目的所有方面，从初始设置到提交更改。

拉取请求已提交，可在 [AbianS/rustrak#17](https://github.com/AbianS/rustrak/pull/17) 查看。这解决了 [AbianS/rustrak#7](https://github.com/AbianS/rustrak/issues/7) 问题，并显著改进了项目的贡献体验。