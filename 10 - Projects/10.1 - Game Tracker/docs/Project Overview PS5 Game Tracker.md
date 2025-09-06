---
title: Game Tracker — Executive Summary
category: RAG-overview
priority: High
created: 2025-09-05
tags:
  - Overview
  - Project
---
```markdown

# Game Tracker — Executive Summary

**Project:** PS5 Game Tracker

**Goal:** Build a personalised app that automates managing and prioritising a large PS5 backlog.

**Functionality:** A custom, AI-powered sequential ordering algorithm creates a realistic, date‑aware game queue. It analyses upcoming release dates and your available time between them to schedule backlog and in‑progress games into appropriate time windows.

## Key Features

- **AI‑Driven Scheduling:** Adaptive queue that auto‑adjusts to schedule changes and new releases.
- **Persistent Knowledge (RAG):** Contextual AI assistant grounded in your project’s rules and docs.
- **Automated Sync:** Cloudflare Workers integrate PSN and IGDB to keep data accurate and up‑to‑date.
- **Intuitive Interface:** Vercel dashboard to manage the collection and visualise the queue.

# Project History

## Excel failure — manual approach — Microsoft Copilot

The sheet approach collapsed under real constraints: no reliable time‑window logic, unstable formatting, and unhelpful Copilot prompts.

- Frequent over‑allocations and noisy recalculations
- Need for a more intuitive platform with adaptive AI

## Notion discovered

Great for capturing the tracker concept, structuring properties, and documenting rules. Problems began when “Cumulative Backlog Hours” failed due to Notion limitations. Cross‑record rollups feeding back into per‑record decisions caused circular dependencies and stalled recalculations.

- Rollups + formulas couldn’t maintain a running total across changing release windows
- Conclusion: Notion is ideal for organisation and notes, but the sequencing engine must recompute outside formula chains

## Discovering external options

First attempt: Vercel + GitHub + Make.com. It failed in practice: complex scenarios sent inconsistent JSON and ran every 15 minutes, too infrequent for practical updates.

- Outcome: long‑running, multi‑step Make scenarios were unreliable for deterministic ordering

## Back to square one, but …

Cloudflare proposed as the path forward: short‑lived recompute with deterministic I/O, low latency triggers, and simple logs.

- Goal: replace fragile multi‑step runs with a single pass that recalculates time windows and play order
- Benefit: faster updates and fewer moving parts

## More research

A lightweight Vercel frontend became the single UI. Two direct paths, no middleman:

- Notion → Vercel: data entry and updates flow directly to the UI
- Cloudflare → Vercel: sequential ordering results feed the UI; room for more workers later

Rationale: keep data and sequencing separate but converge in the UI. Avoid brittle intermediaries, shorten feedback loops, and keep updates fast and predictable.

## Another hurdle — Notion AI New Chat

New chats had no memory, so the system had to be re‑introduced each time. Verification methods were gamed by the AI, passing checks without following rules.

- Impact: high friction onboarding, false positives
- Result: abandon this; assume stateless sessions and use safeguards that cannot be optimised around

## Possible solution

Use RAG so every AI response is grounded in rules without a training step. Cloudflare Workers retrieve the latest rules, select relevant ones per request, and inject them into the prompt before generation.

- Retrieval before generation: responses cite exact rules used
- No memory dependency: each call rebuilds context from the rules index

## Fundamental system changes

### Part 1 — Data platform shift (Supabase)

Notion’s limits for complex computation and integrity make it unsuitable as the system of record. Supabase provides proper constraints and policies.

- Decision: store all game data in Supabase
- Benefits: strong integrity (RLS, constraints), deterministic recompute, clearer inputs and outputs

### Part 2 — Docs platform and cost (Obsidian + token‑based Copilot)

Notion Business is not cost‑effective for docs only. Obsidian for docs and rules plus a token‑priced Copilot plugin offers better economics.

- Decision: move docs to Obsidian; keep data in Supabase
- Benefits: lower doc costs, versioned rules, AI usage that scales with tokens rather than flat seats
```