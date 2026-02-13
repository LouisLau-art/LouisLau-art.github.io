---
title: "为 pytest-reportlog 贡献：去掉报告日志里的 ANSI 转义序列 (PR #102)"
description: "在写入 JSONL 之前递归剥离字符串中的 ANSI escape sequences，避免机器可读的 report log 被颜色码污染，并补齐回归测试。"
pubDate: "Feb 13 2026"
tags: ["Open Source", "pytest", "Python", "Testing", "CLI"]
---

`pytest-reportlog` 的输出是 line-based JSON（JSONL），本意是给工具/系统做机器解析用的。但 pytest 近版本开始给一些 diff 加颜色后，错误信息里会混入 ANSI 转义序列，导致日志里出现 `\u001b[...m` 这类控制字符，影响下游消费。

- Issue: [pytest-dev/pytest-reportlog#83](https://github.com/pytest-dev/pytest-reportlog/issues/83)
- PR: [pytest-dev/pytest-reportlog#102](https://github.com/pytest-dev/pytest-reportlog/pull/102)

## 🔍 分析 (Analyze)

问题不是“JSON 不能存”，而是“机器可读性被破坏”：ANSI 颜色码对终端有意义，对解析器没有意义。更糟的是，这类控制字符往往出现在断言 diff / traceback 文本里，下游做搜索、归档、或生成报告时会遇到乱码、匹配失败或额外清洗成本。

因此更合理的默认行为是：在写入 JSONL 前，把所有字符串里的 ANSI escape sequences 去掉。

## 📍 定位 (Locate)

- 写入 JSONL 的入口：`src/pytest_reportlog/plugin.py`（`ReportLogPlugin._write_json_data`）
- 回归测试：`tests/test_reportlog.py`

## 🛠️ 执行 (Execute)

- 增加一个递归清洗函数：对 dict/list/tuple 里的所有字符串执行正则剥离（常见 ANSI CSI 序列）；
- 在 `json.dumps` 前清洗一次；若触发 `cleanup_unserializable`，再清洗一次（避免 `str(value)` 生成带颜色码的文本）；
- 单测不依赖 pytest 是否真的生成彩色 diff：通过自定义 `pytest_report_to_serializable` hook 注入带 ANSI 的字段，断言日志里最终只有纯文本。

## ✅ 总结 (Summary)

这类改动对用户可见行为影响很小（只影响 reportlog 的文本字段），但对稳定消费日志很关键：把“终端展示层的控制码”隔离在 JSONL 之外，能显著降低后续处理链路的复杂度，也更符合 report log 的定位。

