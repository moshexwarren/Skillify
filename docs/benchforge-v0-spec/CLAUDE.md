# CLAUDE.md — BenchForge CLI v0

## Stack
- Node 20+, TypeScript strict, ESM, single package. Runs on WSL2 and Windows (use path.join, no hardcoded `/`).
- Test runner: vitest.

## Dependency allowlist (nothing else without explicit approval)
commander, zod, execa, simple-statistics, jscpd, typhonjs-escomplex, uuid, vitest, eslint, typescript.

## Hard rules
- Read SPEC.md, SCHEMA.md, METRICS.md, SCORING.md, ACCEPTANCE.md before writing code.
- ACCEPTANCE.md tests are locked. Never edit them. Green gate is the only exit.
- No LLM-judge code paths. Scores only via SCORING.md fixed formulas; overall score never rendered without breakdown; normalization change = scoring_version bump.
- All agent interaction behind AgentAdapter interface. `mock` adapter first; `claude-code` adapter second, kept thin.
- KISS/YAGNI: no plugin system, no server, no upload, no signing, no config options not in SPEC. If tempted to generalize — don't.
- DRY last and conservatively.
- Deterministic collectors: JSON reporters only, parse structured output.
- Errors: fail loud with actionable message; never swallow.

## Build order
1. Schema (zod) + fixtures
2. Stats module + AT-07/08
3. Workspace materializer + isolation + AT-03/04/06
4. Mock adapter + run loop + AT-02/09/13
5. Collectors + AT-05/14
6. Scoring module + AT-15/16
7. compare/report deliverables (html/md/csv/badge) + AT-10/11/12
8. init + AT-01
9. claude-code adapter (manual smoke test only, not in AT gate)

## Definition of done
AT-01..16 green, tsc clean, eslint clean. Nothing else counts.
