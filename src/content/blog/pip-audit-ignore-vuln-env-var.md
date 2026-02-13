---
title: "为 pip-audit 贡献：新增忽略漏洞的环境变量 PIP_AUDIT_IGNORE_VULN (PR #1001)"
description: "增加 PIP_AUDIT_IGNORE_VULN（空格分隔的漏洞 ID 列表）作为 --ignore-vuln 的环境变量等价物，并补齐文档与单测。"
pubDate: "Feb 13 2026"
tags: ["Open Source", "pip-audit", "Python", "CLI", "CI", "Testing"]
---

CI 场景里漏洞库一更新就可能把流水线打红。命令行 `--ignore-vuln` 能解决，但改 CI 配置有时不方便；用环境变量更灵活，尤其适合临时/分支级别的豁免。

- Issue: [pypa/pip-audit#948](https://github.com/pypa/pip-audit/issues/948)
- PR: [pypa/pip-audit#1001](https://github.com/pypa/pip-audit/pull/1001)

## 🔍 分析 (Analyze)

`pip-audit` 已经支持用环境变量配置部分 CLI 选项（如输出格式、服务端选择等）。把“忽略漏洞 ID”也补齐成环境变量，可以让 CI 不必频繁改命令行参数，同时保留 `--ignore-vuln` 的显式覆盖能力。

## 📍 定位 (Locate)

- CLI 参数定义：`pip_audit/_cli.py`（`--ignore-vuln`）
- 环境变量文档：`README.md`
- CLI 单测：`test/test_cli.py`

## 🛠️ 执行 (Execute)

- 新增 `PIP_AUDIT_IGNORE_VULN`，内容为“空格分隔的漏洞 ID 列表”；
- 解析结果作为 `--ignore-vuln` 的默认值，因此会与显式传入的 `--ignore-vuln` 组合；
- 更新 README 环境变量表格，并补齐单测覆盖默认值与组合行为。

## ✅ 总结 (Summary)

这类“把已有 CLI 行为暴露成 env var”的改动通常 merge 成本低：
- 不改变默认行为（未设置 env var 时仍是空列表）；
- 对 CI/自动化更友好；
- 单测把组合语义锁住，避免后续调整时悄悄变更行为。

