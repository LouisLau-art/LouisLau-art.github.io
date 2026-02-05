---
title: "补齐脚本使用文档：为 Azure NVIDIA Robotics Reference Architecture 添加 CLI 参数表（Issue #75）"
description: "给 Azure-Samples/azure-nvidia-robotics-reference-architecture 的 4 个 submit 脚本补齐 CLI 参数表（默认值/来源）与示例命令，降低上手成本并满足基础文档要求。"
pubDate: "Feb 5 2026"
tags: ["Open Source", "Documentation", "Bash", "Azure", "DevOps", "GitHub"]
---

这次贡献是典型的“低风险高收益”文档补齐：项目有多个提交训练/验证作业的脚本，但 README 里缺少完整的 CLI 参数说明。对于新用户来说，这会直接增加试错成本，也容易导致脚本能力被低估。

- Issue: [Azure-Samples/azure-nvidia-robotics-reference-architecture#75](https://github.com/Azure-Samples/azure-nvidia-robotics-reference-architecture/issues/75)
- PR: [Azure-Samples/azure-nvidia-robotics-reference-architecture#123](https://github.com/Azure-Samples/azure-nvidia-robotics-reference-architecture/pull/123)

## 🔍 分析 (Analyze)

脚本类项目的“可用性”很大一部分取决于文档质量：
- 参数默认值和覆盖优先级不清晰时，用户很难判断“到底哪里生效”。\n
- 没有示例命令时，用户需要读脚本源代码才能开始用。\n

这类改动基本不影响运行逻辑，却能显著降低上手门槛，通常也更容易被维护者接受。

## 📍 定位 (Locate)

目标文件与范围都很明确：
- 文档：`scripts/README.md`\n
- 涉及脚本：`submit-azureml-training.sh`、`submit-azureml-validation.sh`、`submit-osmo-training.sh`、`submit-osmo-dataset-training.sh`

## 🛠️ 执行 (Execute)

我按脚本的 `--help` / 参数解析逻辑整理出每个脚本的：
1) **CLI 参数表**（Option / Default / Description / Source）。\n
2) **最小可运行示例**（至少包含一个必需参数 + 一个可选参数）。\n
3) **依赖/前置条件**（如 `az`/`osmo`/`zip`/`rsync` 等）。

## ✅ 总结 (Summary)

这次属于“提升 DX 的文档补齐”：
- 让四个提交脚本的参数一目了然（默认值与来源更清楚）。\n
- 给出可复制的命令示例，减少读源码成本。\n
- 不改动业务逻辑，风险低、合并速度通常更快。

