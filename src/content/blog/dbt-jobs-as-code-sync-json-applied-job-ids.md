---
title: "为 dbt-jobs-as-code 补齐 sync --json：输出实际创建的 Job ID（Issue #182）"
description: "记录我在 dbt-labs/dbt-jobs-as-code 中改进 sync --json 输出：在 apply 后返回 applied 结果，并包含 job_id，方便 CI/CD 自动触发新建 Job。"
pubDate: "Feb 5 2026"
tags: ["Open Source", "Python", "CLI", "dbt", "CI/CD", "GitHub"]
---

有些团队的发布流程不是靠分支，而是靠“创建/更新 Job + 立刻触发”。在这种模式下，`dbt-jobs-as-code` 的 `sync` 就不只是把配置同步上去，还需要把“同步后真实存在的 Job ID”返回给流水线使用。

- Issue: [dbt-labs/dbt-jobs-as-code#182](https://github.com/dbt-labs/dbt-jobs-as-code/issues/182)
- PR: [dbt-labs/dbt-jobs-as-code#183](https://github.com/dbt-labs/dbt-jobs-as-code/pull/183)

## 🔍 分析 (Analyze)

现状：`sync --json` 只输出 apply 前的 *plan*（变更集），没有 “apply 后实际执行了什么/新建 Job 的 job_id 是多少”。

这会让自动化很尴尬：你能看到要创建哪些 job，却拿不到创建成功后的 job_id，无法继续做“触发 job 运行”。

## 📍 定位 (Locate)

关键路径在：
- `src/dbt_jobs_as_code/main.py`：`sync` 命令的 JSON 输出时机在 apply 之前。
- `cloud_yaml_mapping/change_set.py`：`ChangeSet.apply()` 调用 client 方法，但不会保留返回值（比如 `create_job()` 返回的 `JobDefinition.id`）。

## 🛠️ 执行 (Execute)

这次改动保持输出为**单个 JSON 对象**，并尽量不引入额外噪音：

1) 在 `Change.apply()` 里返回 sync_function 的返回值。\n
2) 在 `ChangeSet.apply()` 过程中记录 `applied_changes`，并为 job 类型提取 `job_id`。\n
3) `sync --json` 在 apply 后输出：\n
   - 原有 plan keys：`job_changes` / `env_var_overwrite_changes`\n
   - 新增 `applied`：真正执行过的动作（含 `job_id`）\n
   - 新增 `apply_success`：同步是否成功

## ✅ 总结 (Summary)

这个改动能直接提升 CI/CD 友好度：
- `sync --json` 能拿到 apply 后的 `job_id`，后续“立刻触发新建 job”变得可自动化。
- 保持 plan 输出兼容（原 key 不变），只是追加 applied 结果。

后续就等维护者 review / 合并。

