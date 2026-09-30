# Skillify

Skillify is a Codex skill for designing deterministic, contract-first
benchmarks that test whether AI coding skills, prompts, agent instructions, and
development workflows produce measurable improvements.

Its governing rule is simple: if a proposed metric depends on opinion, taste,
or an LLM judge, it is not an objective benchmark metric. Skillify replaces
subjective scoring with machine-checkable evidence.

## What it covers

- Published task contracts and hidden tests
- Ground truth from seeded defects, controlled fixtures, golden references, or
  invariants
- Happy-path, hostile-input, and recovery coverage
- Anti-gaming preservation checks
- Versioned metric, category, and overall score breakdowns
- Reproducible performance and pixel comparisons through pinned environments
- 16 frontend measurements and 24 frontend task archetypes
- 18 backend measurements and 12 backend task archetypes

The included references cover frontend benchmarks, backend benchmarks, and
deterministic scoring. Skillify is an authoring skill and methodology; this
repository does not bundle a benchmark runner or BenchForge CLI.

## Next-generation experiment engine specification

The repository now also preserves the v0 design for a local baseline-versus-
treatment experiment engine. The design was drafted under the working name
**BenchForge**; the intended product name is **igotskilz**. Its specification
covers private benchmark suites, hidden acceptance tests, interleaved repeated
runs, objective metrics, confidence intervals, and evidence-based verdicts.

This is design work, not a claim of a finished product. The CLI described in
[`docs/benchforge-v0-spec/SPEC.md`](docs/benchforge-v0-spec/SPEC.md) is not yet
implemented in this repository. The original `benchforge` command examples are
kept unchanged so the archived specification remains internally consistent.

## Install for Codex

Clone the repository, then copy the installable `skillify/` directory into your
Codex skills directory:

```bash
git clone https://github.com/moshexwarren/Skillify.git
cp -R Skillify/skillify ~/.codex/skills/skillify
```

On PowerShell:

```powershell
git clone https://github.com/moshexwarren/Skillify.git
Copy-Item -Recurse .\Skillify\skillify "$HOME\.codex\skills\skillify"
```

Restart Codex after installation so the skill is discovered.

## Verification

The distributed skill is checked with the official Codex skill validator,
local-reference checks, fresh-context benchmark-design scenarios, Git hygiene
checks, and a credential-pattern scan before release.

## License

MIT. See [LICENSE](LICENSE).
