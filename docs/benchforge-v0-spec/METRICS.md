# METRICS — v0

Every metric = deterministic tool + defined computation. No LLM judges anywhere.

## v0 collectors (implement now)

| Metric | Tool / source | Type | Computation |
|---|---|---|---|
| build_success | build_command exit code | proportion | exit 0 = pass |
| build_time_s | timed build_command | continuous | wall clock |
| hidden_test_pass_rate | hidden_tests.command (vitest/pytest JSON reporter) | continuous | passed/total per run |
| regression_failures | regression_tests.command | continuous | failing count |
| type_errors | typecheck_command (tsc --noEmit) | continuous | error count parsed |
| lint_errors / lint_warnings | lint_command (eslint --format json) | continuous | counts |
| deps_added | lockfile diff (package-lock/pnpm-lock) | continuous | new top-level deps |
| complexity_mean / complexity_max | typhonjs-escomplex (JS/TS) | continuous | cyclomatic over changed files |
| duplication_pct | jscpd --reporters json | continuous | % duplicated lines, changed files |
| diff_loc | git diff --numstat | continuous | added+removed |
| files_touched | git diff --name-only | continuous | count |
| edit_churn | transcript parse | continuous | files edited >1× |
| wall_time_s | harness timer | continuous | agent start→done |
| tokens_in / tokens_out / cost_usd | adapter usage | continuous | from API response |
| turns / tool_calls | transcript parse | continuous | counts |
| completion_rate | run exit status | proportion | done / n_runs per arm |

## Collector rules
- Each collector runs in the post-run workspace (after hidden/ copy-in), isolated, deterministic.
- Collector failure ≠ run failure: record metric as null, log, continue.
- Parse structured output only (JSON reporters). Never regex human-readable logs when a JSON reporter exists.

## Deferred (v0.1+, do NOT build)
Playwright hidden specs, responsive 3-viewport, console errors, axe violations, Lighthouse (pinned Docker, median-of-5: perf/a11y/BP/LCP/FCP/TBT/INP), RTL dual-dir mirrored-geometry, RTL risk flags, hardcoded-value count, pixel diff (reproduction only), code review suite scoring (detection rate / FN / unverified flags).

## Presentation rules
- L1 evidence always shown: raw per-metric deltas + CI + verdict.
- Category + overall scores per SCORING.md — never displayed without the metric breakdown.
- complexity/duplication labeled "proxy metrics" in report output.
