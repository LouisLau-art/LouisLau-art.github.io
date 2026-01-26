---
title: 'Vibe Coding 实战录：从“屎山”堆积到 Google Antigravity 的救赎 —— CSIT883 项目复盘'
description: 'CSIT883 (JI125) System Analysis and Project Management 项目复盘。分享如何使用 Google Antigravity 和 AI-Driven 选型策略，将一个混乱的项目拯救回来。'
pubDate: 'Dec 15 2025'
tags: ["Archive"]
---

CSIT883 (JI125) System Analysis and Project Management 的大作业终于要交差了。回看这几个月的开发历程，简直是一部血泪史。我们的项目是 **Intelligent Campus Lost & Found Platform**，听起来高大上，实际上...一言难尽。

但这篇文章不全是吐槽，我想分享的是在经历了无数次“反复横跳”后，我悟出的 **AI 时代编程（Vibe Coding）的核心心法**。

## Phase 1. 工具的选择：为什么一开始就要用最好的？

项目初期，我像神农尝百草一样试遍了市面上的 IDE：Cursor, Windsurf, Qoder, Trae... 代码质量参差不齐，导致项目早期就埋下了无数颗雷，后期 debug 的时间比写代码还长。

**直到我遇到了 Google Antigravity。**

这是 Google 推出的对标 Cursor 的大杀器。最重要的是，它对学生极其友好（白嫖一年的 Gemini Pro 会员！）。有了会员，我能在 IDE 里直接调用 **Gemini 3.0 Pro**，甚至还有 **Claude Sonnet 4.5** 和 **Opus 4.5**。

**Vibe Coding 第一定律：**
> **要做 AI Driven 的项目，必须在一开始就选择最强的模型。**

别为了省钱用 GPT-3.5 或者是弱智模型，它们产出的低质量代码（逻辑漏洞、幻觉库、过时语法）是**负资产**。低质量代码不仅消耗你的时间，还会指数级增加工期。只有 Sonnet 4.5 或 Gemini 3.0 这种级别的模型，才能保证“一次做对”，这才是真正的效率。

## Phase 2. 架构的迷思：什么才是“高成功率”的方案？

在开发过程中，我一直在思考：**对于 AI 编程来说，到底什么是最佳架构？**

我们经历过无数次动摇：
*   **全栈 Next.js/Nuxt？** 前后端一把梭，心智负担小。
*   **前后端分离？** 经典的 Python 后端 + 前端框架。

结合本项目，我总结了一套**“AI 友好型”选型指南**：

### 1. 后端：Python 是不可替代的 (FastAPI)
有些同学问，Next.js 后端不够用吗？
对于简单的 CRUD，够用。但对于我们的 **“智能匹配系统”**，Node.js 生态就是灾难。
看看我们的核心算法需求：
*   **TF-IDF + 余弦相似度 (权重40%)**：依赖 `scikit-learn`。
*   **图像标签处理**：依赖 Python 强大的数据处理能力。
*   **Levenshtein 距离**：Python 库极其成熟。

让 AI 用 Node.js 写这些算法，它会给你找一堆冷门的 npm 包，或者让你手写数学公式，出错率极高。而让 AI 写 Python 的 `sklearn`，它简直是信手拈来。
**结论：涉及算法/AI逻辑，后端必须是 Python (FastAPI)。**

### 2. 前端：React (Next.js) > Vue
这是我最痛的领悟。虽然 Vue 上手简单，Nuxt 开发体验好，但在 AI 眼里：
**React 是亲儿子，Vue 是干儿子。**
Github 上 React 的语料库是 Vue 的数倍。用 React 时，AI 写出的组件逻辑严密、样式规范；用 Vue 时，AI 经常混淆 Vue2/3 的语法，或者在 Props 传递上犯蠢。
**结论：为了让 AI 发挥最大效能，请顺从 AI 的“偏科”，选择 React 生态。**

### 3. 数据库：拒绝“过度工程化”
我们曾在这个问题上浪费了太多时间：PostgreSQL -> MySQL -> PostgreSQL...
最后顿悟：**老师根本不 Care 你用什么数据库！**
Presentation 只有 7 分钟，演示能跑通才是王道。
**结论：SQLite 才是永远的神。** 无需配置 Docker，无需安装服务，单文件直接跑，迁移零成本。

## Phase 3. 核心创新：如何在“糊弄”中寻找亮点？

为了满足 Marking Criteria 里的 Innovation，我们必须得有 AI 元素。

**起初的想法：** 部署 YOLOv5 做目标检测。
**现实的打击：** 依赖包几个 G，模型权重几百兆，部署极其麻烦。

**最终的“高成功率”方案：**
**ONNX Runtime + MobileNetV2**。
*   **轻量级：** 模型仅十几兆，CPU 也能跑得飞起。
*   **足够“智能”：** 能识别出 `["backpack", "water bottle"]` 等标签。
*   **部署简单：** 一个 Python 库搞定。

**这一招“降维打击”完美解决了演示需求：既有了 AI 噱头，又没有引入沉重的运维负担。**

## Phase 4. 算法实现的细节（AI 填空的艺术）

有了 Antigravity 加持，我们将复杂的匹配逻辑拆解给 AI，它完成得非常出色。这是我们最终实现的**多模态加权匹配算法**：

1.  **文本相似度 (40%)**：AI 帮我写了基于 `scikit-learn` 的 TfidfVectorizer，甚至还贴心地加上了中文分词预处理。
2.  **类别匹配 (20%)**：硬逻辑，同类满分，不同类零分。
3.  **位置接近度 (15%)**：计算 Levenshtein 编辑距离（AI 甚至帮我写了归一化公式）。
4.  **时间接近度 (15%)**：引入了**线性衰减**机制（7天内分数递减），这是 AI 自己补充的亮点。
5.  **视觉标签相似度 (10%)**：计算 Jaccard 相似度（交集/并集），让图片识别不仅仅是个摆设。

## 总结：AI 时代的生存法则

如果这个项目能重来，我会这样开局：
1.  **IDE**: Google Antigravity (Gemini 3.0 Pro / Sonnet 4.5)。
2.  **Stack**: FastAPI (为了 Python 算法库) + Next.js (为了 AI 的 React 熟练度) + SQLite (为了不折腾)。
3.  **Visual**: ONNX Runtime (为了轻量化部署)。
4.  **Mindset**: **Don't Over-engineer.** 甚至不要自己写 CSS，直接让 AI 用 Tailwind CSS 一把梭。

**Vibe Coding 不是乱写，而是学会如何“驾驭”最强的模型，避开 AI 的弱点，用最小的代价换取最高的 Presentation 效果。**

祝大家的 Final Presentation 都能顺利过关！

---
> **Tags:** #CSIT883 #SystemAnalysis #Antigravity #GeminiPro #FastAPI #React #VibeCoding #AI编程 #项目复盘
