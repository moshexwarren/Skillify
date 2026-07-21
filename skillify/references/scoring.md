# SCORING — Aggregation System & Deliverables

Composite scores are gameable and arguable. Mitigation: every score is derived by FIXED published formulas, always displayed with its full breakdown, and pinned to suite@version + scoring@version. Overall score alone, ever, anywhere = bug.

## Three layers
- **L1 Evidence** (unchanged): per-metric baseline vs treatment, delta, 95% CI, verdict. Statistical truth. Scores never replace this.
- **L2a Metric sub-scores** 0–100: every metric normalized (formulas below), displayed under its category.
- **L2b Category scores** 0–100: weighted mean of its metric sub-scores.
- **L3 Overall score** 0–100: weighted mean of categories. Display format: `Overall 84 · weights v1 · suite frontend-basic@1.2.0`.

## Display hierarchy (mandatory in all reports)
```
Overall 84 (Baseline 61 · Δ+23)
├─ Functional 90        (weight 30)
│  ├─ hidden_test_pass_rate   92   [raw 0.92 · Δ+0.16 · CI +0.09..+0.22 · significant]
│  └─ regression_failures    100   [raw 0 · Δ−2 · significant]
├─ Accessibility 78     (weight 15)
│  ├─ axe_serious             70   [raw 3 · budget 10]
│  └─ keyboard_pass          100
└─ ...
```
Every node shows sub-score + raw value + delta/CI/verdict. No layer hidden.

## Subject → category mapping (skill identity)
Every experiment declares its subject in config; carried into bundle and every report header:
```json
"subject": {
  "name": "anti-slop-frontend",
  "version": "7.0.0",
  "type": "skill|agents_md|claude_md|prompt|mcp|workflow",
  "domain": "frontend|backend|bugfix|code_review|architecture|planning",
  "sha256": "..."
}
```
- `domain` must match the suite's domain — mismatch = run refused.
- Reports/badges always show: name@version · type · domain. Users always see exactly what category a skill belongs to.

## Comparison rules
- Skills are compared ONLY within: same domain + same suite@version + same scoring_version + same model.
- `compare` prints subject blocks of both bundles side-by-side and refuses aggregate comparison on any mismatch (still prints L1 deltas).

## Normalization (deterministic, fixed functions)
| Metric kind | Formula |
|---|---|
| proportion (pass rates) | value × 100 |
| binary | 0 or 100 |
| count, lower better, budget B | 100 × max(0, 1 − count/B) |
| continuous, lower better, budget B (LCP, bundle KB, wall time) | 100 × clamp(B/actual, 0, 1) |
| continuous, higher better, target T | 100 × clamp(actual/T, 0, 1) |

Budgets B and targets T live in `suite.json → scoring` block. No budget defined = metric excluded from scores (still in L1).

## Default categories & weights — frontend domain (v1 — overridable in suite.json, always printed)
| Category | Weight | Metrics |
|---|---|---|
| Functional | 30 | hidden_test_pass_rate, regression_failures, state-machine coverage |
| Accessibility | 15 | axe by severity, contrast, keyboard pass, semantic checks |
| Performance | 15 | LCP, TBT, CLS, INP, bundle_size, build_time_s |
| Resilience | 15 | chaos specs, console_errors, failed_requests, race dupes |
| Code health | 10 | type_errors, lint, complexity, duplication, deps_added (PROXY-flagged) |
| Cost/Speed | 15 | wall_time_s, cost_usd, tokens vs baseline |

## Default categories & weights — backend domain (v1)
| Category | Weight | Metrics |
|---|---|---|
| Correctness | 30 | hidden endpoint pass, contract conformance, seeded fix rate, regressions |
| Data integrity | 15 | DB state asserts, transaction integrity, idempotency, migration checksum |
| Security/Auth | 15 | auth matrix, validation gauntlet, SQLi inert, secrets-in-logs (PROXY) |
| Resilience | 10 | chaos, retry cap, unhandled 500 = 0, timeout handling, queue discipline |
| Performance | 10 | p95/p99, throughput, N+1 query budget |
| Code health | 10 | type/lint/complexity/duplication/deps (PROXY-flagged) |
| Cost/Speed | 10 | wall_time_s, cost_usd, tokens vs baseline |

- Overall = Σ(category_score × weight) / Σ(weights).
- Category with >50% null metrics → N/A, weight redistributed across remaining, flagged in report.
- Both arms scored: report shows Baseline 61 → Treatment 84 (Δ+23), plus per-category deltas.

## Deliverables (every `benchforge run` and `compare`)
1. **CLI stdout scorecard** — overall + category table + top metric deltas.
2. **report.html** — overall banner, category bars, FULL metric table (baseline, treatment, delta, CI, verdict), weights/version footer, links to run logs.
3. **report.md** — same content, markdown.
4. **benchforge.json** — raw bundle (adds `scores` block).
5. **metrics.csv** — one row per metric per arm.
6. **badge.svg** — `BenchForge 84` local file for READMEs.
- `compare` mode: side-by-side scorecards A vs B + per-metric delta table.

## Schema addition (bundle)
```json
"subject": { "name": "anti-slop-frontend", "version": "7.0.0", "type": "skill", "domain": "frontend", "sha256": "..." },
"scores": {
  "scoring_version": "1.0",
  "weights": { "functional": 30, "accessibility": 15, "performance": 15, "resilience": 15, "code_health": 10, "cost_speed": 15 },
  "baseline":  { "overall": 61, "categories": { "functional": { "score": 55, "metrics": { "hidden_test_pass_rate": 55 } } } },
  "treatment": { "overall": 84, "categories": { "functional": { "score": 90, "metrics": { "hidden_test_pass_rate": 92 } } } },
  "delta_overall": 23
}
```

## Guardrails
- Overall never rendered without category + metric breakdown in the same output.
- Normalization formulas are code constants; changing them = scoring_version bump.
- Score ≠ significance: verdicts come from L1 stats only. A +23 overall with inconclusive metrics must say so.
- Scores comparable ONLY within identical suite@version + scoring_version. `compare` blocks cross-version scoring with a warning.
