---
name: pmoves-fork-onboarding
description: "Use when syncing or wiring POWERFULMOVES fork submodules."
---

# PMOVES Fork Onboarding

Repeatable pattern for bringing a POWERFULMOVES fork (e.g. PMOVES-composio, pmoves-composio-mcp-plugin) into the PMOVES.AI monorepo: **sync the fork → create the Hardened branch → wire the submodule → claim the lane**. Operator directive: a fork must be synced against its upstream BEFORE any wiring work.

## Step 1 — Sync fork to upstream (zero data transfer)

Forks share git objects with upstream on GitHub, so move branch REFS via the API — never clone a big repo over slow WAN:

```bash
UP=$(gh api repos/<UPSTREAM_ORG>/<REPO>/git/ref/heads/<UPSTREAM_BRANCH> --jq '.object.sha')
# hard-move an existing branch (safe when all 'ahead' commits are upstream-authored — verify first!)
gh api -X PATCH repos/POWERFULMOVES/<FORK>/git/refs/heads/<BRANCH> -f sha="$UP" -F force=true --jq '.object.sha'
# create the Hardened branch
gh api -X POST repos/POWERFULMOVES/<FORK>/git/refs -f ref=refs/heads/PMOVES.AI-Edition-Hardened -f sha="$UP" --jq '.object.sha'
```

Pitfalls:
- `gh repo sync` / `merge-upstream` API returns **409 merge conflicts** on real divergence; the refs PATCH above is the fallback when the fork has no original work (check `gh api repos/<FORK>/commits?per_page=5` — if all authors are upstream, hard reset is safe).
- High-velocity upstreams (composio pushes daily) drift within hours — re-verify SHA equality right before committing the gitlink.

## Step 2 — Wire the submodule (THE path= KEY TRAP)

`git submodule add` clones the full repo (minutes-to-timeouts over Starlink). Manual wiring is faster, but on git 2.52-windows a `.gitmodules` section WITHOUT an explicit `path = <dir>` key fails lookup: `git submodule status` says "no submodule mapping found" / "No url found for submodule path" even though `git config --file .gitmodules --get submodule.<name>.url` returns the URL fine. Every pre-existing section in PMOVES.AI carries `path =`; section name alone does NOT resolve. Full manual sequence:

```
# .gitmodules: [submodule "<name>"] with path=, url=, branch=PMOVES.AI-Edition-Hardened, shallow=true
# stage the gitlink without cloning:
git update-index --add --cacheinfo 160000,<SHA>,<path>
git add .gitmodules
# then shallow-clone the working tree:
git clone --depth 1 --branch PMOVES.AI-Edition-Hardened <url> <path>
git submodule absorbgitdirs <path>   # migrate .git dir into .git/modules/
git submodule init <path>            # register in .git/config
```

Diagnosis script for the mapping failure: compare sections lacking `path =` — see `references/submodule-path-key-pitfall.md`.

Other submodule notes:
- A killed clone leaves `.git/index` + `index.lock` sentinels; delete BOTH (`rm -f <sub>/.git/index*`) then `git -C <sub> checkout -f HEAD`.
- `.gitmodules` edits are read from the INDEX in some codepaths — `git add .gitmodules` before `git submodule init`.
- CRLF churn in `.gitmodules` is cosmetic; the path= key is the real breaker.

## Step 3 — Claim the lane in AGNOTE4482PHI.t1.md

All integration lanes must be claimed in the register (`pmoves/docs/AGENTS/AGNOTE4482PHI.t1.md`, append-only). Row grammar and gate checks: see `references/agnote4482-lane-claim.md`. Short form:

```
- `<UTC ts ≤ commit time>` CLAIM `<identity>` branch: `<branch>` · **TTL 72h (expires <ts>)** · scope: ...

<!-- GRAPHITI_MARK: <IDENTITY>::<LANE-NAME>-CLAIM::<date> -->
```

- Owner must resolve in `pmoves/config/identity_vocabulary.yaml` (HERMES-AGENT is declared).
- Verify: `PYTHONPATH=. python pmoves/tools/identity_lineage.py --verify` → `identity lineage: clean`.
- Collision gate: `python .claude/hooks/governance/claim-collision-pre.py < payload.json` (JSON on stdin with tool_name/tool_input) — exit 0 = clear.
- Release with a RELEASE row on the same lane when done; keep timestamp discipline (read clock at commit time).

## Step 4 — Branch/rail strategy (KRISS_KROSS)

Docs/protocol-only → `PMOVES.AI-Edition-Hardened` or main; runtime/container → `PMOVES.AI-Edition-Hardened-Integrations` rail. **BUT measure rail freshness first**: as of 2026-09-03 the Integrations rail is stale (last commit 2026-08-18, 4675/2197 commits diverged from main, predates `pmoves/config/mcp_inventory.json` entirely). All MCP inventory work has been landing on main directly. If the target file doesn't exist on the rail, don't rebase there — record the finding and stay on main.

## Checkpoint before pushing

`git submodule status <paths>` should show ` <SHA> <path> (origin/PMOVES.AI-Edition-Hardened)` (no leading `-`), then `git push`. On remote-moved rejections: `git pull --rebase origin main`, re-verify gitlinks survived, re-run submodule init if the config registration was lost.
