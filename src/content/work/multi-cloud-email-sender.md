---
title: Multi-Cloud Email Sender
publishDate: 2026-03-19 12:00:00
img: /assets/at-work.jpg
img_alt: A resilient internal operations system with handoff and deployment continuity
description: |
  An internal email operations platform for non-technical users, built for cloud takeover, tracking diagnostics, and offline-friendly delivery.
tags:
  - FastAPI
  - React
  - Cloudflare
  - Operations
---

### The Problem

Internal users needed one tool to manage templates, contacts, and scheduled email sends without touching cloud consoles or deployment plumbing. The harder requirement was continuity: the system had to survive handoff and still be operable by the next person.

### The Delivery

I built the workflow around the failure modes that mattered in practice:

*   `Cloudflare` takeover and offboarding docs so ownership could move cleanly.
*   Tracking diagnostics and quick tunnel fallback to verify delivery paths before issues reached users.
*   CSV/Excel cleanup, timezone handling, and single-file packaging so the tool stayed usable on ordinary office machines.

### What It Demonstrates

This project shows that I care about operational resilience as much as feature delivery. I can take a messy internal workflow, make it deterministic, and document it so another person can continue from where I left off.
