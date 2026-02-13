---
title: "为 Poetry 贡献：发布到 legacy endpoint 时自动补全尾部斜杠 (PR #10732)"
description: "修复自定义仓库 upload URL 以 /legacy 结尾但缺少 / 时，可能触发重定向导致上传静默失败的问题，并补齐单测。"
pubDate: "Feb 13 2026"
tags: ["Open Source", "Poetry", "Python", "Publishing", "Testing"]
---

这单的核心是一个很常见但很“阴”的坑：URL 少一个尾部 `/`，服务端可能返回重定向，而重定向对 `POST` 的处理在不同实现里可能会导致上传失败但不容易被感知。

- Issue: [python-poetry/poetry#6687](https://github.com/python-poetry/poetry/issues/6687)
- PR: [python-poetry/poetry#10732](https://github.com/python-poetry/poetry/pull/10732)

## 🔍 分析 (Analyze)

用户配置仓库时常会写成：

- `https://test.pypi.org/legacy`（少了尾部 `/`）

而正确的 legacy upload endpoint 通常是：

- `https://test.pypi.org/legacy/`

当 URL 缺少尾部 `/` 时，部分服务端会返回重定向；在上传场景下，这可能带来“请求被重定向但行为改变”的问题，最终表现为“看似没有报错，但包没有被上传上去”。

## 📍 定位 (Locate)

发布逻辑入口：
- `src/poetry/publishing/publisher.py`

相关测试：
- `tests/publishing/test_publisher.py`

## 🛠️ 执行 (Execute)

做最小修复：仅对 legacy upload URL 做规范化处理：
- 当 `path` 以 `/legacy` 结尾且没有 `/legacy/` 时，自动补一个尾部 `/`。

并新增单测覆盖：配置 `repositories.testpypi.url = https://test.pypi.org/legacy` 时，实际上传应使用 `https://test.pypi.org/legacy/`。

验证命令（定向）：
- `python -m pytest -o addopts='' tests/publishing/test_publisher.py`

## ✅ 总结 (Summary)

这是一类“用户容易踩坑、维护者容易 review”的修复：
- 不改变非 legacy 仓库 URL 的行为；
- 修复点非常聚焦，风险可控；
- 单测直接锁住期望行为，合并成本低。

