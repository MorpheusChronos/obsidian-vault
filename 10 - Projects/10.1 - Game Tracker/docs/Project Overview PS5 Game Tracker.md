```markdown
# Game Tracker — Executive Summary

**Project:** PS5 Game Tracker

**Goal:** To build a personalized application that automates the management and prioritization of a large backlog of PlayStation 5 games.

**Functionality:** The system will use a custom-built, AI-powered sequential ordering algorithm to create a realistic, date-aware game queue. By analyzing upcoming game release dates and estimating available time between them, it will intelligently schedule backlog and in-progress games into appropriate time windows.

**Key Features:**

- **AI-Driven Scheduling**: The application computes an adaptive and optimized game queue that adjusts automatically to changes in your schedule and new game releases.
- **Persistent Knowledge**: Leverages a custom RAG system, grounded in your project's rules and documentation, to provide a contextual AI assistant.
- **Automated Sync**: Integrates with external APIs (like PSN and IGDB) via Cloudflare Workers to keep game data and metadata accurate and up-to-date.
- **Intuitive Interface**: Provides a dashboard on a Vercel frontend for managing your game collection and visualizing your queue.

# Project History

## Excel failure — manual approach — Microsoft Copilot

The sheet approach collapsed under real constraints: no reliable time‑window logic, unstable formatting, and unhelpful Copilot prompts.

- Frequent over‑allocations and noisy recalculations
- User requires a more intuitive platform with more advanced AI capabilities that can intelligently adapt to changing schedules.

## Notion discovered

Seemed perfect at the beginning: great for capturing the tracker concept, structuring properties, and documenting rules. The problems started when “Cumulative Backlog Hours” failed due to Notion limitations—rolling up cross‑record totals and feeding them back into per‑record decisions created circular dependencies and stalled recalculations.

- Rollups + formulas couldn’t maintain a running total across changing release windows
- Conclusion: Notion remains ideal for organization and notes, but the sequencing engine needs a consistent recompute outside formula chains

## Discovering external options

First attempt used a simple deploy flow (Vercel + GitHub) with an external automation via Make.com. The approach failed in practice: Make’s complex scenario sent inconsistent JSON payloads and had a 15‑minute run interval, which made game data updates too infrequent to be practical going forward.

- Outcome: pipelines built on long‑running, multi‑step Make scenarios proved unreliable for deterministic ordering

## Back to square one, but …

User asked AI for a [Make.com](http://Make.com) replacement, and Cloudflare was discussed as the path forward: a short‑lived recompute step with deterministic input/output, low‑latency triggers, and simple logs.

- Goal: replace fragile multi‑step runs with a single pass that recalculates time windows and the play order
- Benefit: faster updates without waiting on long intervals, and fewer moving parts to break

## More Research

Introduced a lightweight Vercel frontend as the single UI surface. Defined two direct paths with no middleman:

- Notion → Vercel: data entry and updates flow directly to the UI
- Cloudflare → Vercel: sequential ordering results feed the UI, with room for multiple workers as future enhancements

Rationale: keep data and sequencing separate but converge in the UI. This dual system avoids brittle intermediaries, shortens feedback loops, and keeps updates fast and predictable.

## Another hurdle - Notion AI New Chat

New chats had no memory of the project, so the user had to re‑introduce the system each time. Verification methods were attempted to keep quality high, but the AI learned to game the process, passing checks without truly following the rules.

- Impact: high friction onboarding per session, false positives from gamed verification
- Result: this approach was abandoned; solutions must assume stateless sessions and use safeguards that cannot be optimized around

## Possible solution?

Use a RAG system so every AI response is grounded in your rules without any user “training” step. Cloudflare Workers retrieve the latest rules, select the relevant ones per request, and inject them into the prompt before generation. This keeps sessions stateless and consistent.

- Retrieval before generation: responses cite the exact rules used
- No memory dependency: each call rebuilds context from the rules index

## Fundamental System changes

### Part 1 — Data platform shift (Supabase)

Notion’s limitations for complex computations and maintaining data integrity (circular rollups, stalled recalcs, brittle formula chains) make it unsuitable as the system of record. Moving data to Supabase resolves this by providing a proper database with reliable constraints and policies.

- Decision: store all game data in Supabase instead of Notion
- Benefits: strong integrity (RLS, constraints), deterministic recompute, clearer inputs and outputs

### Part 2 — Docs platform and cost (Obsidian + token‑based Copilot)

The Notion Business plan isn’t cost‑effective as a documents‑only solution. Obsidian is proposed for docs and rules, and a token‑priced Copilot plugin offers better economics for AI assistance.

- Decision: swap Notion docs for Obsidian; keep data in Supabase
- Benefits: lower doc costs, versioned rules, and AI usage that scales with tokens rather than a flat seat price
```