# Handoff: Homepage Job Search Packaging

## Session Metadata
- Created: 2026-03-19 15:50:27
- Project: /home/louis/LouisLau-art.github.io
- Branch: main
- Session duration: ~45 minutes of site structure review

## Recent Commits (for context)
  - fa401cf docs(blog): add 10-day PR digest and fix podman PR link
  - 9341501 blog: cibuildwheel TOML links
  - e503d92 blog: pip require-virtualenv tests
  - f7d03bd blog: podman-desktop pr-labeler checkout
  - cbc36b9 blog: pip zsh completion compdef not found

## Handoff Chain

- **Continues from**: None (fresh start)
- **Supersedes**: None

> This is the first handoff for this task.

## Current State Summary

The personal homepage is clean and functional, but it is still packaging Louis primarily as an open-source contributor rather than as someone with strong self-directed product/system work. The homepage currently shows a generic “Selected Work” section populated from the `work` collection, but that collection only contains three open-source entries and does not yet surface ScholarFlow or Multi-Cloud Email Sender as flagship projects. This task is to design and implement that packaging layer without touching the in-progress ScholarFlow article body.

## Codebase Understanding

## Architecture Overview

This Astro site uses a `work` collection for portfolio cards and a `blog` collection for articles. The home page uses `src/pages/index.astro` plus `PortfolioPreview.astro` to render selected work, while `src/pages/work.astro` renders the full work archive. That means the most leverage comes from adding or revising content entries in `src/content/work/` and making any small home-page copy adjustments necessary to frame Louis as a job-seeking full-stack / AI-agent-oriented developer.

## Critical Files

| File | Purpose | Relevance |
|------|---------|-----------|
| src/pages/index.astro | Homepage layout and selected-work section | Likely file for top-level packaging adjustments |
| src/pages/work.astro | Full portfolio listing | Needs to stay aligned with any new work entries |
| src/components/PortfolioPreview.astro | Visual card rendering for work items | Important if project cards need richer or clearer presentation |
| src/content.config.ts | Defines the `work` schema | Confirms required fields for new work entries |
| src/content/work/nuxt.md | Example of current work-entry shape | Use as format reference |
| src/content/work/magic-regexp.md | Another work-entry example | Shows tone and structure for short case studies |
| src/content/work/rust-tools.md | Existing work entry focused on contributions | Helps compare current style against desired project packaging |

## Key Patterns Discovered

The site is content-driven, so the easiest way to improve job-search packaging is by changing content entries and small home-page copy, not by redesigning the whole UI. Existing `work` entries are concise and case-study-like, which is a good fit for adding ScholarFlow and Multi-Cloud Email Sender. Also, because the ScholarFlow blog polish is already a separate task, this task should avoid editing `src/content/blog/scholarflow-ai-friendly-development-journey.md` to prevent parallel conflicts.

## Work Completed

## Tasks Finished

- [x] Reviewed how the homepage and work archive are populated
- [x] Confirmed that current `work` entries focus on open-source contributions rather than personal flagship projects
- [x] Verified that the schema and card components are simple enough for content-first packaging work

## Files Modified

| File | Changes | Rationale |
|------|---------|-----------|
| .claude/handoffs/2026-03-19-155027-homepage-job-search-packaging.md | Added this handoff | Enables a separate agent to own the homepage-packaging task |

## Decisions Made

| Decision | Options Considered | Rationale |
|----------|-------------------|-----------|
| Treat this as a content-and-positioning task, not a redesign task | Full visual redesign, content-only packaging, hybrid approach | Content changes create faster job-search value with lower risk |
| Keep homepage packaging separate from ScholarFlow article polishing | One agent does both, two parallel agents, defer packaging | Separation reduces conflict and keeps each task sharply scoped |
| Prioritize adding flagship project entries over adding more generic sections | New CTA blocks, new nav items, richer project entries | Project cards directly improve what interviewers notice first |

## Pending Work

## Immediate Next Steps

1. Add new `work` entries for ScholarFlow and Multi-Cloud Email Sender with job-search-oriented summaries and realistic tags.
2. Update `src/pages/index.astro` copy only if needed so the homepage more clearly signals full-stack projects, AI workflow thinking, and open-source contributions together.
3. Review whether `PortfolioPreview.astro` needs a small enhancement for stronger project differentiation; keep this minimal unless clearly necessary.

## Blockers/Open Questions

- [ ] The site currently does not have dedicated project images for ScholarFlow or Multi-Cloud Email Sender. The next agent must decide whether to reuse stock assets, add new assets, or ship text-first entries for now.
- [ ] There is an existing older blog post about Multi-Cloud Email Sender. The next agent should not rewrite that article as part of this packaging task, but should keep future alignment in mind.

## Deferred Items

- Full homepage redesign is deferred; this task should stay focused on better portfolio packaging.
- Resume download / explicit hiring CTA changes are deferred unless they become obviously necessary after the project cards are in place.

## Context for Resuming Agent

## Important Context

This task is independent enough for a separate agent because it mainly touches homepage structure and `work` content, while the ScholarFlow content task is operating on a blog markdown file. The main strategic goal is to make the homepage tell a clearer hiring story: “open-source contributor” should remain visible, but “built substantial systems like ScholarFlow and Multi-Cloud Email Sender” needs to become much more obvious. The next agent should optimize for fast, credible presentation rather than inventing an over-designed landing page.

## Assumptions Made

- The current Astro structure and content-collection approach will remain unchanged.
- Small, high-signal packaging changes are preferable to a broad UI rewrite.
- Project cards are currently the highest-leverage surface for improving first impressions.

## Potential Gotchas

- Do not edit the ScholarFlow blog draft in this task; that belongs to the separate content handoff.
- `work` entries use `publishDate`, while blog entries use `pubDate`; do not mix those schemas.
- Each `work` entry needs an `img` field, so missing assets may force a temporary placeholder decision.

## Environment State

## Tools/Services Used

- Astro content collections
- Git and shell inspection tools

## Active Processes

- No required long-running processes at handoff time

## Environment Variables

- None required for content packaging work

## Related Resources

- `src/pages/index.astro`
- `src/pages/work.astro`
- `src/components/PortfolioPreview.astro`
- `src/content.config.ts`
- `src/content/work/nuxt.md`
- `src/content/work/magic-regexp.md`
- `src/content/work/rust-tools.md`

---

**Security Reminder**: Before finalizing, run `validate_handoff.py` to check for accidental secret exposure.
