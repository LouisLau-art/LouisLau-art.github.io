---
title: "为 pytest-reportlog 贡献：避免 user_properties 因 set 值被整体字符串化 (PR #101)"
description: "修复 cleanup_unserializable 只要遇到一个不可 JSON 序列化值就把整段 user_properties 变成字符串的问题，并补齐回归测试。"
pubDate: "Feb 13 2026"
tags: ["Open Source", "Pytest", "Python", "Testing", "JSON"]
---

这单属于“输出格式一致性”问题：测试报告写入 JSONL 时，结构一旦变成字符串就很难再被下游工具可靠解析。

- Issue: [pytest-dev/pytest-reportlog#12](https://github.com/pytest-dev/pytest-reportlog/issues/12)
- PR: [pytest-dev/pytest-reportlog#101](https://github.com/pytest-dev/pytest-reportlog/pull/101)

## 🔍 分析 (Analyze)

`record_property` 的 `user_properties` 本质是一个键值对列表。如果某个 value 是 `set` 之类的不可 JSON 序列化类型，原实现会直接把整个 `user_properties`（甚至整个字段值）转换成字符串，导致：

- 同一字段在不同用例里有时是结构化 JSON，有时是字符串；
- 下游消费者需要做大量兼容处理，且容易出错。

目标是：只清洗不可序列化的“那一小块”，而不是把整段结构打散成字符串。

## 📍 定位 (Locate)

核心逻辑：
- `src/pytest_reportlog/plugin.py` (`cleanup_unserializable`)

测试覆盖：
- `tests/test_reportlog.py`

## 🛠️ 执行 (Execute)

改动策略：
1. 将 `cleanup_unserializable` 从“只检查顶层 key/value”升级为递归清洗（dict / list / tuple / set）；
2. `set` 统一转换为稳定排序后的 `list`（避免输出顺序不稳定）；
3. 增加回归测试：确保 `user_properties` 内部某个值不可序列化时，不会把整个字段字符串化。

验证命令：
- `python -m pytest`

## ✅ 总结 (Summary)

这类修复的价值在于“结构稳定”：
- JSONL 输出字段类型更一致，便于下游消费；
- 变更集中在序列化清洗函数，可控且容易 review；
- 有回归测试覆盖，避免以后再退化成字符串化。

