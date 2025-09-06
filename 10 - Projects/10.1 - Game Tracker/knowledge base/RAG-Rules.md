---
title: RAG-Rules
category: RAG-policy
priority: High
created: 2025-09-06
tags:
  - rules
  - rag
---
```markdown
## 1. Contract-first tool inputs
* **Priority**: High
* **Rule**: Validate tool inputs against JSON Schemas. If invalid or missing, ask one concise clarification.
* **Reason**: Prevents bad writes and ensures consistency.
* **Example**: Schema requires {title, platform}. If title is missing → “Which game title should I add?”

## 2. Destructive intent requires explicit confirmation
* **Priority**: High
* **Rule**: For irreversible actions, return a proposal with confirmation_required instead of executing.
* **Reason**: Keeps safety and undo in the UI layer.
* **Example**: “Delete game X” → return {confirmation_required: true, undo_payload: …}.

## 3. Idempotent upserts and request IDs
* **Priority**: High
* **Rule**: Use upserts and a request_id ledger to dedupe retries.
* **Reason**: Prevents duplicate side effects during network issues.
* **Example**: Upsert on unique platform_game_id; replay with same request_id creates no new row.

## 4. Idempotent writes with request IDs
* **Priority**: High
* **Rule**: Include a unique request_id on all mutating requests.
* **Reason**: Ensures repeatable, safe writes under retries.
* **Example**: request_id=add_game_2025-09-03T01:35Z_abc123 → replays ignored.

## 5. No self‑grading of compliance
* **Priority**: High
* **Rule**: Do not assert pass or fail; return result, evidence, and calibrated confidence.
* **Reason**: Evaluation is handled by the app or ops, not the model.
* **Example**: Return {result, evidence, confidence} instead of status:"passed"/"failed".

## 6. Prioritise automation over manual workflows
* **Priority**: High
* **Rule**: Prefer automation for repetitive tasks when net time savings are significant.
* **Reason**: Targets 95%+ automation and reduces ongoing effort.
* **Example**: Use algorithmic ordering instead of manual priority updates.

## 7. Privacy and secret hygiene
* **Priority**: High
* **Rule**: Never expose tokens, PII, internal IDs, or stack traces to users.
* **Reason**: Protects security and privacy.
* **Example**: Replace “OpenAI key invalid sk-…” with “Upstream authentication failed” code=UPSTREAM_AUTH.

## 8. RLS-first access pattern
* **Priority**: High
* **Rule**: Client reads must respect RLS; privileged ops run in Workers.
* **Reason**: Prevents data leaks and enforces least privilege.
* **Example**: Frontend uses user JWT; Worker uses service_role in a secrets store.

## 9. Secrets hygiene and rotation
* **Priority**: High
* **Rule**: Keep secrets only in Workers’ secret store; rotate on incidents; never log values.
* **Reason**: Minimises exposure and blast radius.
* **Example**: Store SUPABASE_SERVICE_ROLE in Cloudflare Secrets; log only hash or last 4 chars.

## 10. Single mutation path via Workers
* **Priority**: High
* **Rule**: Perform all INSERT/UPDATE/DELETE in Workers, not the browser.
* **Reason**: Centralises auth, validation, retries, and logging.
* **Example**: “Add game” → frontend emits action → Worker writes → returns normalised result.

## 11. Single-attempt per tool action
* **Priority**: High
* **Rule**: On first failure of any external call, stop and return a structured error with options.
* **Reason**: Avoids cascading failures and hidden partial writes.
* **Example**: “Sync trophies” times out → options: retry once, run later, open incident.

## 12. Stop and report instead of trial-and-error
* **Priority**: High
* **Rule**: On first tool/API/DB error, disclose and pause. Include standard error block and remediations.
* **Reason**: Prevents double writes and hard-to-trace side effects.
* **Example**: Schema change causes read error → return {operation, correlation_id, code, message, retryable, impact, remediations}.

## 13. Build automation for repetitive gaming tasks
* **Priority**: Medium
* **Rule**: Invest in structured automation to replace manual processes.
* **Reason**: Improves reliability and long‑term efficiency.
* **Example**: Hourly PSN sync worker instead of manual updates.

## 14. Deterministic response schemas
* **Priority**: Medium
* **Rule**: Each intent returns a strict JSON schema.
* **Reason**: Enables robust UI handling and logging.
* **Example**: Intent `recommend_next_game` → {"version":"1.0","item":{"id":"psn:RE2-2019","confidence":0.82}}

## 15. Enforce timeouts and budgets
* **Priority**: Medium
* **Rule**: Set per-tool timeouts and an overall orchestration budget.
* **Reason**: Prevents lockups and runaway costs.
* **Example**: Tool timeout=8s, total budget=12s; exceed → return partial with resume_token.

## 16. API automation for external gaming data
* **Priority**: Medium
* **Rule**: Automate PSN, IGDB, and price integrations.
* **Reason**: Reduces human error and ensures consistent updates.
* **Example**: PSN Worker hourly; IGDB fetch on new game.

## 17. Observability with correlation IDs
* **Priority**: Medium
* **Rule**: Attach correlation_id to every call and persist in logs.
* **Reason**: Enables cross‑service tracing.
* **Example**: correlation_id=vercel-01JABC… sent in headers and stored in audit logs.

## 18. Prefer Workers for side effects
* **Priority**: Medium
* **Rule**: The browser plans; Workers execute state changes and external integrations.
* **Reason**: Centralises auth, logging, and retries.
* **Example**: Emit action to ai-processing-hub instead of client calling third‑party APIs.

## 19. Least privilege in tool use
* **Priority**: Medium
* **Rule**: Invoke only tools strictly required by the intent.
* **Reason**: Reduces risk, latency, and cost.
* **Example**: Queue preview → read‑only path; avoid write endpoints.

## 20. Respect provider rate limits
* **Priority**: Medium
* **Rule**: Back off near quotas; queue non‑urgent work and inform the user.
* **Reason**: Prevents throttling and failures.
* **Example**: “Approaching IGDB limit. Queued price updates for 00:00 UTC reset.”

## 21. Schema-validated payloads
* **Priority**: Medium
* **Rule**: Validate inputs with JSON Schema or Zod before DB ops.
* **Reason**: Avoids bad writes and inconsistent records.
* **Example**: Reject {title:null} with 400 → prompt “Which game title should I add?”

## 22. Standardised error model
* **Priority**: Medium
* **Rule**: Normalise errors to {code, message, retryable, remediation}.
* **Reason**: Consistent UI and logs.
* **Example**: IGDB_RATE_LIMIT → retryable=true, remediation="Try after 60s or reduce batch size."

## 23. Trace and correlate all tool calls
* **Priority**: Medium
* **Rule**: Include correlation_id on every request across services.
* **Reason**: Unified observability.
* **Example**: correlation_id=ux-CHAT-2025-09-03T01:35Z-42 on all Worker requests.

## 24. Transactional integrity for multi-step changes
* **Priority**: Medium
* **Rule**: Group related writes in one RPC or DB transaction.
* **Reason**: Maintains consistency on failure.
* **Example**: Insert game, relations, audit log inside one Postgres function; rollback on error.

## 25. Valid state transitions only
* **Priority**: Medium
* **Rule**: Enforce a defined state machine; reject illegal transitions.
* **Reason**: Preserves data integrity.
* **Example**: Wishlist → Completed is invalid; suggest adding Backlog first.

## 26. Calibrated confidence with rationale
* **Priority**: Low
* **Rule**: Provide confidence with cited evidence; high confidence requires specific sources.
* **Reason**: Improves trust and debuggability.
* **Example**: “Confidence 0.82 based on IGDB length, PSN hours, release window fit.”

## 27. Compact upfront, details on demand
* **Priority**: Low
* **Rule**: Lead with results; put reasoning in collapsible or secondary sections.
* **Reason**: Keeps UX fast and readable.
* **Example**: “Added Elden Ring.” Details include sources, fields, timing.

## 28. Consistent progress calculations
* **Priority**: Low
* **Rule**: When given percent or hours, compute both; flag discrepancies.
* **Reason**: Prevents drift over time.
* **Example**: 75% of 20h → 15h; if stored=12h, add discrepancy_note.

## 29. Consistency and cache invalidation
* **Priority**: Low
* **Rule**: Use SWR or stale‑while‑revalidate; explicitly revalidate after writes.
* **Reason**: Keeps the UI fresh and consistent.
* **Example**: After add‑game, revalidate tag "games-list".

## 30. Evidence‑cited recommendations
* **Priority**: Low
* **Rule**: Ground suggestions in concrete data or clearly mark estimates.
* **Reason**: Transparency and reliability.
* **Example**: “Next: RE2 remake, 12h left; fits before Oct 15. Source: release calendar API.”
```