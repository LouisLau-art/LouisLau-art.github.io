---
title: "为 pip-audit 贡献：用 communicate() 修复 subprocess 输出读取 (PR #1000)"
description: "用 Popen.communicate() 替换自写的 poll/read 循环，可靠读取完整 stdout/stderr，避免 UTF-8 边界被截断/污染，并补齐单测。"
pubDate: "Feb 13 2026"
tags: ["Open Source", "pip-audit", "Python", "Subprocess", "Testing"]
---

这单属于“看起来能跑，但细节会咬人”的 subprocess 细节：如果读取 stdout/stderr 的方式不严谨，可能把本来合法的 UTF-8 流读成不完整片段，导致解码异常或出现替换字符，进而影响日志展示与错误诊断。

- Issue: [pypa/pip-audit#574](https://github.com/pypa/pip-audit/issues/574)
- PR: [pypa/pip-audit#1000](https://github.com/pypa/pip-audit/pull/1000)

## 🔍 分析 (Analyze)

原实现自己写了一个循环来 `poll()` 进程并读取 pipe。类似逻辑常见坑是：
- 进程已退出但 pipe 里仍有剩余字节没被读完；
- 读取固定长度时，UTF-8 多字节字符可能恰好被切在边界上；
- 最终 decode 时会出现 `UnicodeDecodeError` 或被替换成 `�`（即使原始输出整体是合法 UTF-8）。

因此更稳妥的做法是交给 `Popen.communicate()`：它会负责把 stdout/stderr drain 到 EOF，再返回完整字节串。

## 📍 定位 (Locate)

- subprocess 封装：`pip_audit/_subprocess.py`
- 相关单测：`test/test_subprocess.py`

## 🛠️ 执行 (Execute)

- 用 `Popen.communicate()` 替换原先的 poll/read 逻辑，确保 stdout/stderr 一次性完整读取；
- 补齐单测：
  - `run()` 正常返回 stdout；
  - 构造 4096 边界的 UTF-8 两字节字符，确保输出不出现替换字符。

本地验证（定向 + 全量）：
- `.venv/bin/python -m pytest -q test/test_subprocess.py`
- `.venv/bin/python -m pytest -q`

## ✅ 总结 (Summary)

subprocess 这类基础设施代码，优先选用标准库提供的“正确默认实现”，比自己写轮询/缓冲更不容易踩边界条件；同时用一个贴近真实故障模式的回归用例把行为锁住，后续重构才不会悄悄退化。

