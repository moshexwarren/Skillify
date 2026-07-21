---
name: skillify
description: Use when someone needs to create or review deterministic benchmarks, benchmark suites, hidden tests, seeded bugs, scoring systems, or measurable evaluations for AI coding skills, prompts, AGENTS.md, CLAUDE.md, or frontend and backend workflows, especially when ground truth, gameability, subjective metrics, or proof of improvement are concerns.
---

# Skillify

Authors benchmark suites where every metric is provable. Core law: **if a metric requires opinion, taste, or an LLM judge — it is not a metric. Delete it.**

## Non-negotiables
1. **Contract-first.** Open-ended tasks are unmeasurable. Task prompt publishes an interface contract (routes/testids/behaviors for frontend; endpoints/schemas/invariants for backend). Hidden tests verify the contract. We measure conformance, never "quality."
2. **Hidden tests.** Suite author's tests live outside the workspace given to the agent; harness copies them in post-run. Agent-written tests never count.
3. **Ground truth via one of four mechanisms:**
   - Seeding — we broke it, so we know (bug counts, violation counts)
   - Fixtures — we control the data, so we know edge cases (incl. hostile payloads)
   - Golden reference — we built the answer, so we can diff
   - Invariants — machine-checkable rules that must always hold (DB constraints, request counts)
4. **3-state meta-rule.** Every behavior tested in: happy path, hostile/edge input, recovery after failure. Happy-path-only suites are rejected.
5. **Anti-gaming pairing.** Every budget metric (LCP, latency, bundle) is paired with content/behavior-preservation specs — can't win by deleting the feature.
6. **No composite without breakdown.** Scores follow the fixed hierarchy: metric sub-score → category → overall, always shown together, pinned to suite@version + scoring_version.

## Workflow
1. Ask/derive: domain (frontend | backend), archetype, difficulty.
2. Read the matching reference file (below). Pick archetype(s).
3. Write task.md with contract block. No contract = stop.
4. Build golden workspace or seed manifest (`{id, file, lines, type, severity, hidden_test_id}`).
5. Write hidden specs covering 100% of contract + 3-state rule.
6. Define scoring block in suite.json: budgets/targets per metric, category weights (see references/scoring.md).
7. Validation gate before shipping the suite: golden passes ALL hidden specs; seeded version fails EXACTLY the mapped ones; nothing else fails.
8. Pin environment (Docker digest, Chromium, throttles) for any perf/pixel metric.

## References (read the one you need)
- `references/frontend.md` — 16 deterministic frontend measurables + 24 task archetypes (repair, state-machine, fixture torture, chaos, forms, auth, permissions, optimistic rollback, routing, upload, timezone, security, migration, skeleton...).
- `references/backend.md` — backend measurables + 12 archetypes (idempotency, race, auth matrix, transactions, migrations, N+1, chaos, load budgets...).
- `references/scoring.md` — normalization formulas, category weights per domain, score hierarchy, subject→domain mapping, comparison rules.

## Refuse-list (never author these as metrics)
Visual quality, design taste, "slop", readability, originality, suggestion quality beyond seeded ground truth, anything scored by an LLM or human opinion.
