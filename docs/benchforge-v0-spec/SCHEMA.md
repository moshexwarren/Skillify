# SCHEMA — benchforge.json (schema_version 0.1)

Validate with zod. Malformed bundle = hard reject in `compare`/`report`.

```json
{
  "schema_version": "0.1",
  "experiment": {
    "id": "uuid",
    "created": "ISO8601",
    "hypothesis": "string",
    "subject": { "name": "string", "version": "semver", "type": "skill|agents_md|claude_md|prompt|mcp|workflow", "domain": "frontend|bugfix|code_review|architecture|planning", "sha256": "..." },
    "suite": { "name": "string", "version": "semver", "hash": "sha256 of suite dir", "domain": "frontend|bugfix|code_review|architecture|planning" },
    "model": { "provider": "anthropic", "id": "claude-sonnet-4-6", "version": "string|null" },
    "n_runs_per_arm": 10,
    "arms": {
      "control": { "inject": [], "model_override": null },
      "treatment": { "inject": [{ "src": "path", "dest": "path", "sha256": "..." }], "model_override": null }
    }
  },
  "environment": {
    "os": "string", "node": "string", "cli_version": "string",
    "agent_adapter": "mock|claude-code", "agent_version": "string|null"
  },
  "runs": [
    {
      "run_id": "uuid",
      "arm": "control|treatment",
      "index": 0,
      "workspace_hash_pre": "sha256",
      "started": "ISO8601",
      "exit": "done|timeout|error",
      "process": {
        "wall_time_s": 0, "turns": 0, "tool_calls": 0,
        "tokens_in": 0, "tokens_out": 0, "cost_usd": 0
      },
      "diff": { "loc_added": 0, "loc_removed": 0, "files_touched": 0, "edit_churn": 0 },
      "metrics": {
        "build_success": true,
        "build_time_s": 0,
        "hidden_tests_passed": 0,
        "hidden_tests_total": 0,
        "regression_failures": 0,
        "type_errors": 0,
        "lint_errors": 0,
        "lint_warnings": 0,
        "deps_added": 0,
        "complexity_mean": 0,
        "complexity_max": 0,
        "duplication_pct": 0
      },
      "logs_path": "relative path"
    }
  ],
  "stats": [
    {
      "metric": "hidden_test_pass_rate",
      "kind": "continuous|proportion",
      "control": { "mean": 0, "ci95": [0, 0], "n": 10 },
      "treatment": { "mean": 0, "ci95": [0, 0], "n": 10 },
      "delta": 0,
      "delta_ci95": [0, 0],
      "test": "welch_t|two_prop_z|fisher_exact",
      "p": 0,
      "verdict": "significant|inconclusive"
    }
  ]
}
```

Rules:
- All results pinned to suite@version+hash and model@id+version. Bundles with mismatched suite hash are not comparable — `compare` must warn.
- Timeout/error runs are recorded, excluded from metric means, counted in a `completion_rate` stat per arm.
- `hidden_test_pass_rate` = passed/total per run, then aggregated.
- Bundle includes `scores` block per SCORING.md (scoring_version, weights, per-arm category + overall, delta_overall). Scores are derived data — recomputable from runs; `report`/`compare` must recompute and error on mismatch.
