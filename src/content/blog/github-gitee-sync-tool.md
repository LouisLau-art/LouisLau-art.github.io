---
title: "GitHub 到 Gitee 同步工具：实现仓库自动镜像"
description: "如何构建一个自动将 GitHub 仓库同步到 Gitee 的工具，让国内用户也能顺畅访问"
pubDate: "Jan 24 2026"
tags: ["github", "gitee", "自动化", "bash", "git", "效率工具"]
---

# GitHub 到 Gitee 同步工具：实现仓库自动镜像

今天我开发了一个实用的工具，解决了一个常见问题：如何让 GitHub 仓库在国内也能顺畅访问。Gitee 是国内流行的代码托管平台，但手动维护两个相同的仓库既繁琐又容易出错。

## 问题背景

许多开发者都面临这样的困境：GitHub 是全球最大的代码托管平台，但国内用户访问时常遇到网络问题。虽然 Gitee 是一个很好的替代方案，但手动同步两个平台上的仓库非常耗时且容易出错。

## 解决方案

我创建了一个功能完善的工具，支持两种将 GitHub 仓库同步到 Gitee 的方式：

### 1. 本地镜像同步（一次性）

使用 Git 的镜像功能进行一次性同步：

```bash
./sync_github_to_gitee.sh LouisLau-art louis-shawn local
```

这种方式：
- 自动获取指定用户的所有公开仓库
- 为每个仓库创建镜像克隆
- 将镜像推送到对应的 Gitee 仓库
- 保留所有分支、标签和提交历史

### 2. GitHub Actions 自动同步

生成 GitHub Actions 配置文件实现自动同步：

```bash
./sync_github_to_gitee.sh LouisLau-art louis-shawn actions
```

生成的配置文件会：
- 在推送到 main/master 分支时触发
- 支持手动触发同步（workflow_dispatch）
- 处理仓库删除事件
- 自动将变更镜像到 Gitee

## 技术实现

工具是一个 Bash 脚本，充分利用了 Git 强大的 `--mirror` 功能。主要特性包括：

### 依赖检查
脚本启动时会自动检查必要的依赖（curl、git），如果缺少会提示使用 `apt` 安装。`jq` 是可选的，如果没有安装会使用备用解析方法。

### 分页处理
GitHub API 默认每页只返回 100 个结果，脚本会自动处理分页，确保获取所有仓库，不会遗漏。

### JSON 解析
优先使用 `jq` 进行 JSON 解析，如果不可用则使用 `grep + sed` 的备用方案。

### 错误处理
- 使用 `set -e` 确保遇到错误立即退出
- 每个仓库同步都有独立的错误处理
- 提供镜像推送失败时的备用方法

### 资源清理
- 使用 `trap` 捕获中断信号（Ctrl+C）
- 退出时自动清理临时目录
- 避免残留临时文件

### 统计输出
同步完成后会显示统计信息：
- 成功数量
- 跳过数量（如 GitHub Pages 仓库）
- 失败数量

## GitHub Actions 集成

生成的 GitHub Actions 工作流使用了官方的 `actions/checkout@v4`，相比第三方 action 更加可靠：

```yaml
name: Sync to Gitee

on:
  push:
    branches: [ "main", "master" ]
  delete:
  workflow_dispatch:

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Setup SSH
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.GITEE_PRIVATE_KEY }}" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          ssh-keyscan gitee.com >> ~/.ssh/known_hosts

      - name: Sync to Gitee
        run: |
          git remote add gitee git@gitee.com:YOUR_GITEE_USERNAME/$(basename ${{ github.repository }}).git
          git push gitee --all --force
          git push gitee --tags --force
```

工作流特点：
- 支持推送、删除和手动触发
- 使用 SSH 进行安全推送
- 自动同步所有分支和标签

## 工具优势

这个工具提供了多个优势：

1. **可访问性**：让国内开发者也能顺畅访问代码
2. **自动化**：消除手动同步的繁琐工作
3. **一致性**：确保两个平台保持同步
4. **灵活性**：支持一次性同步和持续集成两种方式
5. **安全性**：使用加密的 Secrets 管理凭据
6. **健壮性**：完善的错误处理和资源清理

## 使用说明

工具使用非常简单：

1. **本地同步**：适合一次性迁移所有仓库
2. **GitHub Actions**：适合需要持续自动同步的场景

README 中提供了详细的配置说明，包括 SSH 密钥配置和密钥管理。

## 总结

这个项目展示了简单的 Bash 脚本如何解决开发者社区中的实际问题。通过结合 Git 强大的镜像功能和 GitHub Actions，我们可以创建可靠的跨平台仓库管理解决方案。

工具已在 [GitHub](https://github.com/LouisLau-art/github-gitee-sync-tool) 上开源，使用宽松的许可证，鼓励大家使用和改进。

这是开源工具如何弥合开发生态系统差距、提高全球开发者可访问性的又一个例子。
