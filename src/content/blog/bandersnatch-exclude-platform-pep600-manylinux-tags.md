---
title: "为 bandersnatch 贡献：exclude_platform 支持 PEP 600 manylinux_2_* 平台标签 (PR #2151)"
description: "修复 exclude_platform 在配置 linux 时未能排除 PEP 600 manylinux_2_* wheels 的问题，并补齐回归测试。"
pubDate: "Feb 13 2026"
tags: ["Open Source", "bandersnatch", "Python", "PyPI", "Testing"]
---

这单属于“规则列表跟不上标准演进”的典型问题：PEP 600 引入了 `manylinux_<glibc-major>_<glibc-minor>_<arch>` 这种新平台标签，老的字符串匹配列表如果没覆盖到，就会漏掉本该被排除的 linux wheels。

- Issue: [pypa/bandersnatch#830](https://github.com/pypa/bandersnatch/issues/830)
- PR: [pypa/bandersnatch#2151](https://github.com/pypa/bandersnatch/pull/2151)

## 🔍 分析 (Analyze)

`exclude_platform` 的目标是：当用户在配置里写了 `linux`，就应该把 linux 平台相关的发行包过滤掉。

但 PEP 600 的 manylinux 标签是“数字方案可扩展”的（例如 `manylinux_2_17_x86_64`），如果只靠枚举具体字符串，很容易漏新值。

## 📍 定位 (Locate)

过滤插件实现：
- `src/bandersnatch_filter_plugins/filename_name.py`

对应测试：
- `src/bandersnatch/tests/plugins/test_filename.py`

## 🛠️ 执行 (Execute)

修复策略保持简单、低风险：
1. 在 linux 平台模式下，新增对 `-manylinux_` 前缀的匹配，覆盖 PEP 600 族的 `manylinux_2_*` 标签；
2. 增加回归测试：当配置为 `linux` 时，确保 `manylinux_2_17_*` wheels 会被排除，而 `py3-none-any` wheels 保持不受影响；
3. 顺手在测试基类里重置插件的 class-level 缓存，避免不同 config 的用例相互污染（否则初始化只执行一次）。

验证命令（定向）：
- `.venv/bin/python -m pytest src/bandersnatch/tests/plugins/test_filename.py`

## ✅ 总结 (Summary)

这类改动的关键是“覆盖未来的变体”：
- 用前缀匹配让规则对 PEP 600 的数字方案更稳健；
- 通过回归测试锁住预期行为；
- 变更点集中在一个插件内，维护者 review/merge 成本低。

