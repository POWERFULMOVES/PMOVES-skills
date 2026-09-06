---
name: pmoves-lane-register
description: "Use when claiming, releasing, or auditing PMOVES.AI lanes."
---

# PMOVES.AI Lane Register protocol (AGNOTE4482PHI.t1.md)

Lanes are governed by CLAIM/RELEASE rows in `pmoves/docs/AGENTS/AGNOTE4482PHI.t1.md`.
**A merged PR does NOT auto-close its claim — only a RELEASE row does.** An open CLAIM with no release blocks the lane against every collision check.

## Claiming a lane
1. Append CLAIM row: `- \`<UTC-timestamp>\` CLAIM \`<AGENT>\` branch: \`<branch>\` · TTL <N>h (expires <ts>) · scope: **<one-sentence scope>**` (72h default TTL).
2. Verify BEFORE pushing: `PYTHONPATH=. python pmoves/tools/identity_lineage.py --verify` (must print clean) + pipe a synthetic Write-event through `python .claude/hooks/governance/claim-collision-pre.py` (exit 0 = no collision).
3. Commit the register row in the same PR series as the work (atomic docs commit is fine).

## Releasing a lane
- RELEASE row cites: delivering PR number + merged_at + what shipped, then the honest remainder explicitly ("delivered X, NOT Y — Y is a new lane if taken"). Never claim more than the PR body supports.
- Add the advisory signature line (`agent_signature: ACK::...`) and a `<!-- GRAPHITI_MARK: ... -->` marker if the repo convention shows them nearby.

## Backfill audits (the Mavis lesson, #2960)
- Before treating an old open CLAIM as abandoned, search merged PRs for the claimed scope: `gh pr list --state all --search "<scope keywords>"`, then read PR bodies for "Closes the ... half of PR #N" scope-match language. The 2026-08-20 Mavis claim was delivered by #2651 on 8/21 but sat open 16 days because nobody wrote the RELEASE.
- BACKFILL RELEASE format: evidence (quote PR body scope-match), measurement (e.g. count of releases after the claim, none matching), who directed the backfill. State the co-owner as the auditing agent.

## Windows/MSYS gotchas when editing the register
- The register file may contain NULs/extended chars (git treats it as binary in grep) — use `python - <<PY` readers with `errors='replace'` for scripted analysis, not grep.
- `.git/index.lock` contention: concurrent cron main-sync recreates the lock. `sleep 30-45` → `rm -f .git/index.lock` → retry; don't kill git blind. (`taskkill //F` fails under this shell — MSYS arg conversion is off; use `/F` form.)
