# BACKEND-SUITE — Measurable Benchmarks

Same law as frontend: contract-first, hidden tests, no opinions. Backend contract = endpoints + request/response schemas + invariants (published in task.md, e.g., as OpenAPI + invariant list).

## Measurables (all deterministic)

| # | Metric | Tool | Assertion / value |
|---|---|---|---|
| 1 | Contract conformance | schemathesis/dredd vs published OpenAPI | violations count |
| 2 | Hidden endpoint tests | supertest/pytest hidden specs | passed/total |
| 3 | Seeded-bug fix rate | reverted-fix or injected bugs + hidden tests | fixed/N, false-fix (≥2 tests/bug) |
| 4 | Regression failures | existing suite | count |
| 5 | DB state asserts | post-run SQL checks vs expected fixtures | mismatches count |
| 6 | Idempotency | duplicate POST / webhook replay | side-effect rows == 1 |
| 7 | Race safety | k concurrent requests | invariant holds (e.g., stock ≥ 0, unique constraint), duplicate rows = 0 |
| 8 | Auth matrix | role × endpoint grid | expected status codes (200/403/401) exact match |
| 9 | Validation gauntlet | hostile payload fixtures (SQLi strings, oversize, wrong types) | 4xx returned, zero state change, injection stored as text |
| 10 | Transaction integrity | kill/fault mid-operation (fault injection) | no partial writes (SQL assert) |
| 11 | Migration mission | migrate up → down → up | all green + data checksum preserved |
| 12 | Chaos | dependency mock returns 500/timeout | graceful 5xx→mapped error, retries ≤ cap (request count), no crash |
| 13 | p95/p99 latency + throughput | autocannon, pinned container, fixed conns/duration | ms / req-s vs budget |
| 14 | N+1 / query budget | query counter middleware per endpoint | queries/request ≤ budget |
| 15 | Queue/job behavior | seeded failing job | retry count == policy, dead-letter after cap, side-effect exactly once |
| 16 | Error taxonomy | chaos runs | unhandled 500 count = 0, error body matches schema |
| 17 | Secrets/PII in logs | grep log output for fixture secrets | count — PROXY |
| 18 | Timeout handling | slow dependency mock | request completes with mapped timeout error ≤ deadline |

## Archetypes

| # | Archetype | Ground truth | Measures |
|---|---|---|---|
| B1 | Seeded-bug repair | Seed manifest (revert-fix method preferred) | Fix rate, regressions, false-fix |
| B2 | Contract implementation | Published OpenAPI + hidden supertest | Conformance violations, hidden pass rate |
| B3 | Idempotency/double-fire | Row-count invariant | Duplicate POST/replay → 1 side-effect |
| B4 | Concurrency/race | Invariant asserts | k-parallel writes, constraint holds, dupes = 0 |
| B5 | Auth/permission matrix | Role×endpoint table | Status-code grid exact match, forbidden mutation rows = 0 |
| B6 | Validation gauntlet | Hostile fixtures | 4xx matrix, state unchanged, SQLi inert |
| B7 | Transaction integrity | Fault injection points | Zero partial writes |
| B8 | Migration mission | Checksums | up/down/up green, data preserved |
| B9 | Chaos/resilience | Dependency mocks | Mapped errors, retry cap, zero unhandled 500 |
| B10 | Performance rescue | Pinned load + content specs | p95 under budget, behavior specs stay green |
| B11 | N+1 hunt | Query counter | queries/request ≤ budget, responses unchanged |
| B12 | Queue/job discipline | Seeded failing job | Retry policy exact, dead-letter, exactly-once |

3-state meta-rule applies: happy / hostile / recovery per behavior.

## Environment
DB + app in pinned Docker compose (digests). Load tests: fixed connections, duration, warmup; report median of 3 load runs. Fault injection via toxiproxy or route mocks — scripted, never manual.
