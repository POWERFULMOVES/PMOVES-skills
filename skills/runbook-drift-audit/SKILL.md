---
name: runbook-drift-audit
description: ORGSIM-MIRROR deterministic runbook drift auditor — verify our runbooks' claims against reality with ZERO LLM calls. Use when reviewing docs touching plans/, auditing runbook accuracy before or after merges, wiring docs-drift CI, or answering 'does the doc match reality?'. Works on any node from any clone of PMOVES.AI; stdlib-only, no keys, no GPU, no Docker.
---

# Runbook Drift Audit (ORGSIM-MIRROR)

Deterministic auditor that checks whether our runbooks' checkable claims still hold against reality. Mirrors the orgsim-model-refiner pattern (peer: Richard Aragon / MMVFIRM — see `POWERFULMOVES/orgsim-model-refiner` fork): read-only oracle probing, fail-closed verdicts, sealed hash-chained reports.

## When to use

- Before/after editing anything in `plans/` or `pmoves/docs/` — did the doc's claims survive?
- After merge waves — did the merge make any runbook claim false?
- Before following a runbook literally — will it 404 on you mid-lane?
- CI advisory on docs PRs (`docs-drift-check.yml` pattern).

## How to run (any node, any AGInTZ)

```bash
cd <your-pmoves-clone>
git fetch origin main                      # keep the cross-check reference fresh
python3 scripts/runbook_drift.py \
  --globs 'plans/*.md' \
  --out research/runbook_drift_report_$(date -u +%Y-%m-%d).json
```

Optional local decision-model classification (frozen-weight, still no API keys):

```bash
python3 scripts/runbook_drift.py --globs 'plans/*.md' --jev --out research/runbook_drift_report_jev.json
```

## Reading the verdicts

- **VERIFIED** — claim probed and holds.
- **DRIFTED** — claim probed and false (file/target gone everywhere including origin/main, endpoint 4xx/5xx).
- **UNDECIDED** — probe cannot decide (unreachable net, stale local checkout, ambiguous basename). Never treat UNDECIDED as drift. Fail-closed.

## Rules

1. **The audit itself is ZERO LLM** — intelligence in the harness, not the model. `--jev` is optional frozen-weight classification (AgentJev-0.6B ONNX), never required, silent fallback to heuristics.
2. **Read-only against repo and network** — `make -n` dry-runs, file existence, GET-only curls with 6s cap. Never writes, never POSTs.
3. **Never call DRIFTED without the origin/main cross-check** — v2's lesson: stale checkouts lie. Exists-on-main => UNDECIDED.
4. **Historical lines are records, not claims** — destroyed/superseded/deprecated language is filtered (v3).
5. **Sealed reports** — per-record sha256 chain, chain tip = run digest. Compare digests across runs to detect tampering or drift movement.
6. **Circuit breaker** — if the auditor crashes twice for the same reason, stop, log, report once. Never a third blind retry.
7. **Dep lockfiles: deliberately NOT used** — floor-pins are PMOVES policy for fleet portability (SPARK GB10, Knuckles gfx1201, Jetsons ARM, 5090 Windows). Do not 'fix' this.

## Weekly watch

A scheduled task (`runbook-drift-weekly`, sidecar) runs this every Monday 09:00 ET, CHIT-signs the delta, and notifies only on DRIFTED movement. Nodes can replicate with their own scheduler; keep report files out of commits except when a lane explicitly ships them (see PR #3375 for the shipped-artifact pattern).

## Scorer history (each version exists because the previous failed on something real)

- v1: bare `make` from repo root — the AGENTS.md Known-Road trap (targets live in `pmoves/Makefile` + `pmoves/mk/*.mk`).
- v2: canonical make, but stale-checkout false positives (DRIFTED 24, true count 1).
- v3: origin/main cross-checks + history/annotation filters + DRAFT intent tags + dated `--out` + `--globs`.

Corrections live in the tool docstring and `research/analyses/2026-10-13_ORGSIM_MIRROR_RUNBOOK_DRIFT.md`. The corrections ARE the feature.
