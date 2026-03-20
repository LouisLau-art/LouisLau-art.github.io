# ScholarFlow Git History Breakdown

**Repo:** `/home/louis/scholar-flow`

**Purpose:** turn the blog draft into a repo-grounded narrative instead of a pure reflection piece.

## What Was Actually Reviewed

This pass did **not** read all `829` commits diff-by-diff. That would be a poor use of time.

This pass reviewed:

- the full chronological commit subject line history
- the early foundation commits
- the major milestone commits
- the recent hardening and workflow commits
- representative `git show --stat` output for milestone changes

At the time of review, the repo had:

- `829` commits
- `1357` tracked files

## High-Confidence Evolution Line

### Phase 0: started from an agent/spec-driven template

Representative commit:

- `a35755b` `Initial commit from Specify template`

Evidence:

- the repo began with `.specify/` scaffolding
- Gemini command files for specify/analyze/clarify/plan/tasks were present from day one

Interpretation:

- ScholarFlow did not start as a blank app
- it started inside a specification-first and agent-assisted workflow

### Phase 1: core academic workflow landed extremely fast

Representative commits:

- `892090b` `phase 1 setup complete`
- `ab962a4` `phase 2 foundational infrastructure`
- `3e232ce` `complete US1 (author submission with AI parsing and UI)`
- `493df8e` `complete US2 (editorial dashboard...)`
- `061a7ca` `complete US3 (no-login reviewer access...)`
- `fc0f945` `complete US4 (invoice generator...)`
- `6b0d32e` `complete US5 (AI reviewer recommendation...)`
- `3188a1f` `finalized ScholarFlow core workflow implementation`

Evidence:

- the project moved from scaffold to multi-role workflow in a very compressed window
- the early structure already included author submission, reviewer access, editorial control, finance gate, and AI-based reviewer recommendation

Interpretation:

- the system was never “just a CRUD admin panel”
- from the beginning it had role boundaries and workflow semantics

### Phase 2: quality gates and workflow extensions appeared immediately

Representative commits:

- `f10df9d` `complete 002-plagiarism-check`
- `eaef652` `complete 003-portal-redesign`
- `e138844` `complete 004-content-ecosystem`
- `6ef15b9` `Supabase Auth infrastructure`
- `ec6aae4` `complete 006-quality-assurance-suite`

Evidence:

- plagiarism checking was treated as a first-class workflow, not an afterthought
- public-facing portal work and internal workflow work advanced in parallel
- test infrastructure entered very early through `pytest`, `vitest`, and `run_tests.sh`

Interpretation:

- ScholarFlow evolved as both product surface and process engine
- the repo shows a strong bias toward hardening, not just shipping UI

### Phase 3: the editor workspace became a real operating console

Representative commit:

- `b2780e8` `完成008-editor-command-center全部功能`

Evidence:

- added dedicated editor API
- added `EditorDashboard`, `EditorPipeline`, `ReviewerAssignModal`, `DecisionPanel`
- included backend tests and a dedicated validation script

Interpretation:

- this is a turning point where the project stops looking like a generic submission app and starts looking like an operations system

### Phase 4: testing and UAT became part of the product story

Representative commits:

- `89f405b` `implement comprehensive E2E regression suite and test infrastructure`
- `a62deac` `add staging environment, feedback widget, and seed script`

Evidence:

- Playwright E2E coverage for author, editor, CMS, DOI, and AI matchmaker flows
- test-reset and seed APIs
- staging/UAT environment
- feedback widget and mock data generation
- `.qwen/commands/` material expanded during this period

Interpretation:

- the repo shows the project moving beyond local development toward structured validation with users
- this supports the story that requirements and workflow clarity became a serious concern

### Phase 5: production and deployment concerns became explicit

Representative commits:

- `e945de0` `post-acceptance pipeline (production upload + publish gate)`
- `fe72fd5` `vercel+render setup and prod env`
- `c4470a1` `GitHub Action to sync backend to Hugging Face Spaces`
- `ca333a6` `update deployment architecture in context files (HF + Vercel)`

Evidence:

- production upload and publish gates were formalized
- deployment docs and environment handling were added
- the hybrid deployment story became explicit in the repo

Interpretation:

- this supports the blog’s claim that the system became a serious full-stack workflow, not just a prototype

### Phase 6: hardening, RBAC, performance, and workflow semantics dominated February

Representative commits:

- `a84a651` `financial gate & reviewer privacy`
- `05e9cbc` `complete GAP-P2 DOI integration and plagiarism relaunch`
- `25e6cad` `add journal management and submission journal binding`
- `2ec03b1` `complete editor performance refactor and stabilize test suite`
- `9223c47` `add managing workspace and fix proofreading reupload flow`
- `17bba89` `complete api/detail/modal structural split`

Evidence:

- security and privacy hardening
- role and journal isolation
- API and editor-surface refactors
- performance and modularity work

Interpretation:

- the system entered a maturity phase where workflow correctness and maintainability mattered more than raw feature count

### Phase 7: March shifted heavily toward email-first operations and production SOP

Representative commits:

- `5845cf8` author manual email UIs
- `a654cea` production SOP assignments/artifacts/transitions/apis
- `22cdc2d` redesign production workspace for SOP workflow
- `2a1d2ad` integrate manual email submission precheck lanes
- `08de18c` show corresponding author emails on author page

Evidence:

- many recent commits focus on email envelopes, author contact context, production transitions, and SOP closure
- this is operational complexity, not feature fluff

Interpretation:

- the recent project state is much closer to a real editorial operations platform than an early MVP

### Phase 8: agent context and skill usage became explicit repo concerns

Representative commits:

- `fc0814f` `docs: 在上下文文件中新增 skills 使用约定`
- `8778f26` `docs(context): require skills and context7 lookup across agent docs`
- multiple `handoff` commits in March

Evidence:

- agent context files were actively maintained
- handoff docs became part of day-to-day workflow
- skill usage rules were explicitly written into repo context

Interpretation:

- this does not prove every detail of the author’s personal tooling journey
- but it does prove that agent workflow and skill discipline became part of the repo’s operating model

## What The Blog Can Safely Claim

The blog can safely claim that:

- ScholarFlow grew from a spec-template start into a complex academic workflow system
- the repo history shows repeated investment in workflow correctness, testing, staging, deployment, and operational hardening
- by March, email operations, SOPs, handoffs, and agent-context management were core concerns

The blog should be more careful when claiming:

- the exact personal moment when Qwen CLI became the preferred clarification tool
- the exact timing of reading Codex/Gemini/OpenCode source code
- the exact timing of the skills-paper mindset shift

Those points are still valid as first-person process claims, but they are not directly visible in git history the same way the workflow/product evolution is.

## Recommended Writing Adjustment

Keep the current growth-arc structure, but add one explicit section that anchors the reflection in repo history:

- from Specify-template scaffold
- to core workflow
- to testing/UAT/deploy
- to email-first production SOP hardening

That makes the article feel earned rather than purely retrospective.
