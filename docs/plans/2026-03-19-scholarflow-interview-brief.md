# ScholarFlow Interview Brief

## 30-second version

`ScholarFlow` 是一个学术出版工作流系统。我用 `FastAPI + Next.js + Supabase + Vercel + Hugging Face Spaces` 做了一套覆盖稿件提交、审稿、修回、通知和 AI reviewer matchmaking 的全栈系统。这个项目对我最大的价值，不只是做出了功能，而是让我形成了一套更成熟的 AI 协作开发方式，包括 AI-friendly 技术选型、需求澄清前移、CLI agent 工作流，以及对 skills 的系统筛选。

## 60-second version

`ScholarFlow` 一开始是一个偏全栈工程的项目，但做着做着，我发现它其实也是一个方法论项目。  
我最初围绕 “AI-friendly” 做技术选型，所以后端选了 `FastAPI`，前端选了 `Next.js`，组件系统选了 `shadcn`。原因不是追热点，而是这些栈在 Python / TypeScript 生态里更适合和 AI 长时间协作开发。  

项目早期我通过录音、转写、AI 整理和 Mermaid 图来接住甲方需求，但后来发现真正的问题不是整理速度，而是提需求的人自己也未必想清楚。所以我后来开始把“需求澄清”本身交给 CLI agent 和 brainstorm 式 workflow 去做。  

也是在这个项目里，我从 IDE 型 AI 工具逐步转向 `Claude Code / Gemini CLI / Codex / Context7` 这类更可组合的工作流，并意识到 skills 不是越多越好，而是要按 focused、procedural、低冲突的原则筛选。  

所以这个项目最能代表我的地方，不只是全栈能力，而是我在复杂需求和 AI 工具之间做判断的能力。

## Interviewer-facing highlights

- 能讲系统设计：角色、状态流转、邮件与审稿工作流、AI 匹配、部署架构
- 能讲产品判断：为什么需求整理不等于需求澄清
- 能讲工程成长：从 GUI AI 工具迁移到 CLI agent 工作流
- 能讲 AI 方法论：从“多装 skills”转向“按论文原则做 curation”
- 能用 git 历史做证据：从 Specify 模板起步，逐步长出核心 workflow、QA、staging、deployment 和 production SOP

## Likely follow-up questions

1. 为什么是 `FastAPI + Next.js`，而不是别的组合？
2. 你怎么处理学术工作流里那些一开始说不清楚的需求？
3. 你如何看待 AI 在真实项目中的位置，是写代码工具还是流程工具？
4. 你说 skills 不能太多，具体是怎么判断的？
5. 你能从仓库历史里讲出哪几个关键里程碑？
