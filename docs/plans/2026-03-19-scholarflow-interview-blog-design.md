# ScholarFlow Interview Blog Design

**Goal:** Draft a long-form blog post that helps interviewers understand not just what ScholarFlow does, but how building it changed the author's AI-assisted development workflow.

**Primary audience:** Hiring managers and interviewers for full-stack / AI-agent-adjacent engineering roles.

**Primary impression to leave:** This is a developer who can design systems, reason about fuzzy requirements, and iteratively improve the way they collaborate with AI tools.

## Core Narrative

Use a **growth-arc-first** structure instead of a pure architecture writeup.

ScholarFlow should be framed as the project where three things happened at once:

1. a non-trivial full-stack academic workflow system was built
2. product/requirements ambiguity became visible very early
3. the author's AI workflow evolved from IDE-assisted coding to CLI-agent + docs + skills curation

## What Must Be Included

### 1. AI-friendly stack selection

Explain that the stack was chosen partly for implementation fit and partly for AI collaboration fit:

- backend: FastAPI because Python is a strong lane for AI-assisted backend work, NLP, and API orchestration
- frontend: Next.js because TypeScript/React are also strong lanes for AI-assisted UI and full-stack work
- component system: shadcn because the ecosystem, examples, and AI familiarity made it an efficient choice

### 2. Requirements capture and its failure mode

Include the real workflow:

- recorded a long stakeholder handoff
- transcribed it with Tongyi Tingwu
- used AI to summarize, structure, and generate Mermaid artifacts

Then state the deeper lesson:

- the real problem was not transcription quality
- the real problem was that stakeholders often do not know their own requirements well enough at the start

### 3. Shift from “AI writes code” to “AI helps shape the requirement”

Introduce the move to:

- Qwen CLI
- Brainstorm-like requirement clarification workflow

This is the product-thinking turning point of the article.

### 4. Shift from GUI AI tools to CLI agents

Show the progression:

- VS Code-adjacent AI tools and forked IDEs
- then Claude Code / Gemini CLI / Codex / OpenCode / Context7

The key point is not tool fandom. It is that the author learned text-based agents plus docs access plus shell access are often faster and more composable than GUI-heavy workflows.

### 5. Skills overuse and later correction

This is a strong ending section.

The article must include:

- early over-enthusiasm for installing many skills
- later realization that too many skills harm context efficiency
- source-code reading of agent implementations and compress mechanics
- paper-driven correction: focused, procedural, low-conflict skills beat “install everything”

## Suggested Structure

1. Hook: ScholarFlow looked like a full-stack project, but it became a methodology project too
2. What ScholarFlow is and why it was worth building
3. Why the initial stack was chosen for AI-friendliness
4. How requirements were captured at the beginning
5. Why that process still failed
6. How requirement clarification became part of the system-design workflow
7. How the author moved from IDE plugins to CLI agents
8. Why “more skills” was the wrong lesson
9. What ScholarFlow changed in the author's engineering judgment

## Tone

- first person is fine, but avoid self-congratulation
- emphasize concrete decisions and corrected mistakes
- optimize for “this person can think clearly under ambiguity”
- treat architecture as evidence, not the only story

## Deliverables

1. A blog draft in `src/content/blog/`
2. A built-in “60-second interview version” section near the top
3. A conclusion that ties project complexity to growth in engineering judgment
