# ACCEPTANCE — LOCKED

These tests are the ONLY valid exit condition. Green gate = done. Agent consensus, self-review, or "looks correct" are NOT exit conditions. Do not modify, weaken, skip, or reinterpret these tests. If a test seems wrong, STOP and report — do not edit it.

All ATs must run with the `mock` adapter (no API key, no network).

- **AT-01 init**: `benchforge init` in empty dir creates `benchforge.config.json` + `suites/example/` (suite.json, task.md, workspace/, hidden/). Second `init` exits non-zero, changes nothing.
- **AT-02 run produces valid bundle**: `benchforge run` on fixture suite + mock adapter (N=3/arm) writes `benchforge.json` that passes zod schema validation.
- **AT-03 isolation**: each run gets fresh workspace; `workspace_hash_pre` identical across all runs; a file written by mock agent in run 1 does not exist in run 2's workspace.
- **AT-04 hidden tests never leak**: workspace handed to adapter contains no `hidden/` path; post-run collector sees hidden tests and they execute.
- **AT-05 collector ground truth**: fixture workspace with known state (exactly 2 lint errors, 1 type error, 3/5 hidden tests passing, build passing, 1 dep added) → run record metrics equal exactly those values.
- **AT-06 injection**: treatment runs contain injected file at dest with matching sha256; control runs do not contain it.
- **AT-07 stats correctness**: given synthetic run arrays with known values, Welch t / two-prop z outputs match precomputed reference values (±1e-6 for means/CI, ±1e-4 for p). Reference values checked into fixtures.
- **AT-08 verdict logic**: p=0.049 → significant; p=0.051 → inconclusive.
- **AT-09 timeout handling**: mock adapter configured to hang → run recorded exit=timeout, excluded from metric means, completion_rate reflects it, CLI exits 0.
- **AT-10 compare**: two fixture bundles → table of per-metric deltas; mismatched suite hash → warning emitted, comparison still printed.
- **AT-11 report deliverables**: `benchforge report` on fixture bundle emits report.html, report.md, metrics.csv, badge.svg; zero network calls; overall score never appears without category + metric breakdown in same file; weights + scoring_version + suite@version printed.
- **AT-12 schema rejection**: `compare`/`report` on malformed bundle exit non-zero with validation error path listed.
- **AT-13 arm interleaving**: run order in bundle alternates C,T,C,T,...
- **AT-14 collector failure isolation**: collector command that exits non-zero (e.g., jscpd missing) → metric null, run still recorded, other metrics present.
- **AT-15 scoring determinism**: fixture runs with known metric values + known budgets → metric sub-scores, category scores, and overall equal precomputed reference values exactly; recompute on `report` matches stored `scores` block; mismatch → non-zero exit; report output contains full hierarchy (overall → category → metric sub-score → raw+CI+verdict) and subject block (name@version · type · domain).
- **AT-16 scoring edge cases**: metric without budget excluded from scores but present in L1 table; category >50% null → N/A + weight redistribution matches reference; cross-scoring_version `compare` refuses score comparison with warning, still prints L1 deltas; subject.domain ≠ suite domain → `run` refuses with non-zero exit.

Gate = AT-01..16 green + `tsc --noEmit` clean + eslint clean.
