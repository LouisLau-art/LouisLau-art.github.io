---
title: '从零开始：用 Astro + Tailwind CSS 搭建个人技术博客'
description: '手把手教你使用 Astro、Tailwind CSS 和 GitHub Pages 搭建一个现代化的个人技术博客，支持暗色模式和评论系统。'
pubDate: 'Dec 15 2025'
tags: ["Archive"]
---

作为一名开发者，拥有一个个人技术博客是非常有价值的。它不仅能记录你的学习成长，还能帮助他人解决问题。今天，我将分享如何从零开始搭建一个现代化的技术博客。

## 为什么选择 Astro？

在众多静态站点生成器中，我选择了 [Astro](https://astro.build)，原因如下：

- **零 JavaScript 开销**：默认情况下，Astro 不向浏览器发送任何 JavaScript
- **内容优先**：专为博客、文档等内容型网站设计
- **组件岛屿架构**：只在需要的地方添加交互性
- **Markdown/MDX 支持**：原生支持 Markdown 写作
- **优秀的 DX**：开发体验非常流畅

## 技术栈

本博客采用以下技术栈：

| 技术 | 用途 |
|------|------|
| Astro | 静态站点生成器 |
| Tailwind CSS v4 | 原子化 CSS 框架 |
| TypeScript | 类型安全 |
| Bun | 包管理器（比 npm 快很多） |
| GitHub Pages | 免费托管 |
| GitHub Actions | 自动部署 |
| Giscus | 评论系统（基于 GitHub Discussions） |

## 第一步：初始化项目

首先，使用 Astro 的 Blog Starter 模板创建项目：

```bash
# 使用 Bun 创建项目
bunx create-astro@latest tech-blog --template blog --typescript strict --install --no-git

cd tech-blog
```

## 第二步：安装 Tailwind CSS

Tailwind CSS v4 需要通过 PostCSS 配置：

```bash
# 安装依赖
bun add tailwindcss @tailwindcss/postcss
```

创建 `postcss.config.mjs`：

```javascript
export default {
  plugins: {
    '@tailwindcss/postcss': {},
  },
};
```

创建 `tailwind.config.mjs`：

```javascript
export default {
  darkMode: 'class',  // 启用暗色模式
  content: ['./src/**/*.{astro,html,js,jsx,md,mdx,ts,tsx}'],
  theme: {
    extend: {},
  },
  plugins: [],
};
```

在 `src/styles/global.css` 顶部添加：

```css
@import "tailwindcss";
```

## 第三步：配置 GitHub Pages

修改 `astro.config.mjs`：

```javascript
import mdx from '@astrojs/mdx';
import sitemap from '@astrojs/sitemap';
import { defineConfig } from 'astro/config';

export default defineConfig({
  site: 'https://YOUR_USERNAME.github.io',
  base: '/tech-blog',  // 仓库名
  integrations: [mdx(), sitemap()],
});
```

创建 `.github/workflows/deploy.yml`：

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: oven-sh/setup-bun@v2
        with:
          bun-version: latest
      - run: bun install
      - run: bun run build
      - uses: actions/upload-pages-artifact@v3
        with:
          path: ./dist

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - uses: actions/deploy-pages@v4
        id: deployment
```

## 第四步：实现暗色模式

创建 `src/components/ThemeToggle.astro`：

```astro
---
// ThemeToggle 组件
---

<button id="theme-toggle" type="button" aria-label="Toggle dark mode">
  <!-- Sun Icon -->
  <svg id="sun-icon" class="hidden" fill="none" viewBox="0 0 24 24" stroke="currentColor">
    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" 
      d="M12 3v1m0 16v1m9-9h-1M4 12H3m15.364 6.364l-.707-.707M6.343 6.343l-.707-.707m12.728 0l-.707.707M6.343 17.657l-.707.707M16 12a4 4 0 11-8 0 4 4 0 018 0z" />
  </svg>
  <!-- Moon Icon -->
  <svg id="moon-icon" fill="none" viewBox="0 0 24 24" stroke="currentColor">
    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" 
      d="M20.354 15.354A9 9 0 018.646 3.646 9.003 9.003 0 0012 21a9.003 9.003 0 008.354-5.646z" />
  </svg>
</button>

<script is:inline>
  function getThemePreference() {
    if (localStorage.getItem('theme')) {
      return localStorage.getItem('theme');
    }
    return window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light';
  }

  function updateTheme(theme) {
    document.documentElement.classList.toggle('dark', theme === 'dark');
    document.getElementById('sun-icon').classList.toggle('hidden', theme !== 'dark');
    document.getElementById('moon-icon').classList.toggle('hidden', theme === 'dark');
  }

  // 初始化
  updateTheme(getThemePreference());

  // 切换事件
  document.getElementById('theme-toggle').addEventListener('click', () => {
    const isDark = document.documentElement.classList.contains('dark');
    const newTheme = isDark ? 'light' : 'dark';
    localStorage.setItem('theme', newTheme);
    updateTheme(newTheme);
  });
</script>
```

在 `global.css` 中添加暗色模式变量：

```css
.dark {
  --accent: #6366f1;
  --black: 229, 231, 235;
  --gray-dark: 229, 231, 235;
  --gray-gradient: rgba(26, 26, 46, 50%), #1a1a2e;
}
```

## 第五步：添加评论系统

我们使用 [Giscus](https://giscus.app)，它基于 GitHub Discussions，无需数据库。

1. 在 GitHub 仓库设置中启用 Discussions
2. 访问 https://giscus.app 获取配置
3. 创建 `src/components/Giscus.astro` 组件

```astro
---
const giscusConfig = {
  repo: 'YOUR_USERNAME/tech-blog',
  repoId: 'YOUR_REPO_ID',
  category: 'Announcements',
  categoryId: 'YOUR_CATEGORY_ID',
};
---

<section class="giscus-container">
  <script
    src="https://giscus.app/client.js"
    data-repo={giscusConfig.repo}
    data-repo-id={giscusConfig.repoId}
    data-category={giscusConfig.category}
    data-category-id={giscusConfig.categoryId}
    data-mapping="pathname"
    data-theme="preferred_color_scheme"
    data-lang="zh-CN"
    crossorigin="anonymous"
    async
  ></script>
</section>
```

## 第六步：发布到 GitHub

```bash
# 初始化 Git
git init
git add -A
git commit -m "feat: initial blog setup"

# 使用 GitHub CLI 创建并推送
gh repo create tech-blog --public --source . --remote origin --push
```

然后在 GitHub 仓库的 **Settings → Pages** 中，将 Source 设置为 **GitHub Actions**。

几分钟后，你的博客就会上线！

## 写作流程

以后写博客就很简单了：

1. 在 `src/content/blog/` 目录下创建新的 `.md` 文件
2. 添加 frontmatter（标题、描述、日期等）
3. 用 Markdown 写作
4. 提交并推送到 GitHub
5. GitHub Actions 自动部署

```markdown
---
title: '我的新文章'
description: '文章描述'
pubDate: 'Dec 15 2024'
tags: ["Archive"]
---

文章内容...
```

## 总结

恭喜你！现在你拥有了一个：

- ✅ 现代化的静态博客
- ✅ 响应式设计
- ✅ 暗色模式支持
- ✅ 评论系统
- ✅ 自动部署
- ✅ 完全免费托管

整个搭建过程大约只需要 30 分钟。赶紧开始你的技术写作之旅吧！

---

**有问题？** 欢迎在下方评论区留言讨论！
