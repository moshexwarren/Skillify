# BenchForge CLI v0 — SPEC

Working name only. Local-only experimentation engine. Private suites. No server, no publish, no signing, no leaderboards.

## Purpose
Answer: "Did this change (skill/file injection) improve outcomes?" via baseline-vs-treatment, N runs each, objective metrics, delta + 95% CI, verdict.

## Commands
- `benchforge init` — scaffold `benchforge.config.json` + `suites/example/`. Fails if config exists.
- `benchforge run` — execute experiment per config. Output: `results/<experiment-id>/benchforge.json` + per-run logs.
- `benchforge compare <bundleA> <bundleB>` — stats diff table between two bundles (e.g., v6 vs v7 treatments).
- `benchforge report <bundle>` — render local `report.html` + `report.md` from bundle. No network.

## Experiment model (v0 template: injection on/off)
- **control**: workspace as-is, no injected files.
- **treatment**: workspace + injected files (skill / CLAUDE.md / prompt file).
- N runs per arm (default 10, configurable).
- Arm-level model override allowed in schema (enables model A/B later); v0 CLI only implements injection on/off.

## Suite (private) definition — `suites/<name>/suite.json`
```json
{
  "name": "string",
  "version": "semver",
  "workspace": { "type": "path|git", "ref": "path or git ref" },
  "task_prompt_file": "task.md",
  "hidden_tests": { "dir": "hidden/", "command": "npx vitest run --dir hidden" },
  "regression_tests": { "command": "npm test" },
  "build_command": "npm run build",
  "lint_command": "npx eslint .",
  "typecheck_command": "npx tsc --noEmit",
  "timeout_s": 1800
}
```
Rule: `hidden/` is NEVER present in the workspace given to the agent. Harness copies it in post-run.

## Run flow (per run)
1. Materialize fresh workspace (copy or git worktree). Hash it.
2. Apply arm: inject files (treatment) or nothing (control).
3. Invoke agent adapter with task prompt. Capture transcript, usage, turns, tool calls. Enforce timeout.
4. Snapshot `git diff` (LOC, files touched, edit churn from transcript).
5. Copy `hidden/` in. Run collectors: hidden tests, regression tests, build (timed), typecheck, lint, deps diff, complexity, duplication.
6. Write run record. Destroy workspace.

Runs are sequential v0. No cross-run state. Interleave arms (C,T,C,T,...) to spread drift.

## Agent adapter interface
```ts
interface AgentAdapter {
  run(workspacePath: string, prompt: string, cfg: ArmConfig): Promise<{
    transcript: string; turns: number; toolCalls: number;
    tokensIn: number; tokensOut: number; costUsd: number; exit: "done"|"timeout"|"error";
  }>;
}
```
Implementations: `mock` (for tests, scripted edits), `claude-code` (headless `claude -p`, `--output-format json` for usage). Adapter selected in config.

## Stats
- Per metric: control mean + 95% CI, treatment mean + 95% CI, delta, delta 95% CI.
- Continuous metrics: Welch's t-test.
- Binary/rate metrics (build_success, fix_rate): Wilson CI, two-proportion z (Fisher exact if any cell < 5).
- Verdict: `significant` if p < 0.05 else `inconclusive`.
- Scoring layer on top of stats per SCORING.md: normalized category scores + overall, always with breakdown, pinned to suite@version + scoring_version.

## Out of scope v0
Public suites, publish/upload, signing, browser metrics (Playwright/axe/Lighthouse), pixel diff, RTL dual-dir, code review suite runner, parallel runs, Docker pinning.
