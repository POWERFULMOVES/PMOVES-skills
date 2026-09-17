---
name: pmoves-node-bringup
description: "Use when bringing up PMOVES services on any fleet node."
---

# PMOVES node bring-up — env funnel, compose chains, Known Roads

The recurring class: standing up a PMOVES service on a node (any topology — Windows laptop, Linux workstation, VPS). Every bring-up hits the same funnel, the same compose chains, and the same Windows/WSL toolchain questions. Hard-won 2026-09-03 on Elder-Melchor bringing up channel-monitor + pmoves-yt.

## The env funnel model
- Scripts chain N `--env-file` args (e.g. `scripts/channel_monitor_up.sh` loads 9: env.shared, tier-data, tier-supabase, tier-api, tier-llm, tier-worker, tier-media, tier-agent, tier-ui).
- **Empty value = missing**: `${VAR:?msg}` interpolation FAILS on `KEY=` (empty) in ANY chained file, even if a later file sets it. `env.tier-api:21` had `SUPABASE_JWT_SECRET=` empty and poisoned the whole chain while env.shared carried a valid value — the error message names the wrong file to check. Fix: fill or delete every EMPTY `KEY=` line in every file the script loads.
- **`ensure_env_shared.py` regenerates env.shared** ("Branded defaults applied") and DROPS manual additions — never hand-edit env.shared for stubs/overrides; use env.tier-* files.
- Tier files without committed examples exist (env.tier-supabase had none) — when a script requires one, create it; then contribute the example upstream.
- Seeding: `cp env.<tier>.example env.<tier>` for all missing, then fill empties. Real values come from the secrets funnel / PMOVES-supabase (source of truth: `PMOVES-supabase/docker/.env.example` key names).

## Known Roads doctrine — use the documented make targets, never raw docker compose
- The governance hook BLOCKS raw `docker compose up` in agent sessions.
- Make targets carry real wiring the raw command lacks — proven example: `make up-yt` adds the yt-cookies overlay; without it downloads run cookie-less and bot-gate.
- `make channel-monitor-up` runs `scripts/channel_monitor_up.sh` (runtime-aware Supabase URL wiring).
- Before invoking any bring-up, `grep -n '<service>' pmoves/Makefile` for the target; dry-run it first: `make -n <target>`.

## Windows toolchain (Elder-Melchor class nodes)
- **No usable Windows-native make**: GnuWin32 3.81 and ezwinports 4.4 both crash silently (exit 127). Fix: WSL make — `wsl -d Ubuntu-24.04 -- bash -lc 'cd /mnt/c/<repo>/pmoves && make <target>'` (dry-run verified). Install make passwordlessly via `wsl -d Ubuntu-24.04 -u root -- bash -lc 'apt-get install -y make'` (sidesteps the sudo prompt).
- WSL docker controls the same Docker Desktop daemon — compose from WSL acts on the node's containers.
- Windows-native compose from git-bash works too (scripts export MSYS_NO_PATHCONV=1 for path safety).
- Docker Desktop under load: concurrent compose pulls + stale half-created containers degrade the daemon (ps/inspect hang). Recover: kill the compose process, wait, `docker rm -f` stale entries, then serial `docker start` per container.
- Windows file checkout: repos with >260-char paths need `git config core.longpaths true` (parent + submodule) before checkout succeeds.
- **`.git/index.lock` contention**: a concurrent watcher (cron main-sync) or lingering git.exe keeps recreating the lock mid-commit. Recovery that works: `tasklist | grep '^git\.exe'` to see them → `sleep 30-45` → `rm -f .git/index.lock` → retry the git op. Do NOT kill git blind. If a kill is needed, `taskkill //F //IM` FAILS in this shell (MSYS arg-conversion disabled — `//F` passes through literally); use `taskkill /F /IM` or PowerShell `Stop-Process`.

## PR-branch hygiene (2026-09-05, #2905/#2904/#2907 passes)
- **BEHIND that won't clear = stale LOCAL branch**: `git merge origin/main` says "Already up to date" while GitHub says BEHIND → the local PR-branch checkout is stale; `gh pr checkout` reuses local branches without fast-forwarding. Fix: `git rev-parse HEAD origin/<branch>` (they differ), `git reset --hard origin/<branch>`, re-merge main, push.
- **`mergeable: UNKNOWN` after push is async lag** (1–3+ min) — poll, don't conclude. `gh pr merge --squash --auto` exits 0 but silently does NOT enable auto-merge while UNKNOWN; verify with `gh pr view <N> --json autoMergeRequest --jq '.autoMergeRequest.mergeMethod'` and re-run once computed.
- **Never `git add -A`/`git add .` on a PR branch** — workspace scratch (`.hermes-audit-ws/`, `terminal_survey/`, delegation caches) lives untracked in the tree and gets swept in. Explicit paths only. If swept: `git reset --soft HEAD~1 && git reset -q && git add <paths> && git commit && git push --force-with-lease`.
- **Verify fix commits carried the fix**: `git grep '<reviewer-quoted pattern>' -- <file>` after committing — #2907's fix commit claimed a fix a second occurrence survived.

## Register (AGNOTE4482) RELEASE pairing
- An agent filing a RELEASE for a CLAIM owned by ANOTHER agent violates the co-owner contract (closure authority stays with the signing owner; foreign RELEASEs corrupt `open_claims_in()`). Codex flags it P2. Correct backfill shape (accepted on #2960): file under the FILING agent's identity as an acknowledgement citing the delivering PR's body + merged_at + releases-after-claim count, annotate the original CLAIM closed-by-evidence, sign the ACK under the filing agent's card noting "no owner authority exercised", and record the owner's formal RELEASE as lane remainder.
- **Check merged PRs before assuming a lane is open** — the Mavis handoff lane sat delivered-but-unreleased for 16 days because nobody grepped merged PRs for the claimed scope.

## Submodule wiring rules
- `.gitmodules` entries NEED an explicit `path = <name>` key on this fleet's git build — without it `git submodule status` reports "no url found / no submodule mapping" even when the section otherwise parses. Symptom chain looks like a CRLF bug; it isn't.
- Wire order that works: config-file entries (`path`+`url`+`branch`+`shallow`) → `git add .gitmodules` → `git update-index --add --cacheinfo 160000,<sha>,<path>` → `git submodule init/update --depth 1 <path>`.

## Fork sync (fleet repos)
- Real GitHub forks: `gh api -X POST repos/<org>/<repo>/merge-upstream -f branch=<b>` (fast-forward when no PMOVES commits; merge preserves them).
- Standalone mirrors (is_fork=false): refs API zero-transfer — `gh api -X PATCH repos/<org>/<repo>/git/refs/heads/<b> -f sha=<upstream-sha> -F force=true`. ONLY safe when the 'ahead' commits are all upstream-authored (verify authors first — PMOVES-n8n's hardened-flows work made force-reset wrong there; its 'unrelated histories' merge failure was CORRECT: it's a workflows overlay, not a code fork).
- Hardened branch creation: `gh api -X POST .../git/refs -f ref=refs/heads/PMOVES.AI-Edition-Hardened -f sha=<sha>`.

## Delegated audits (the pattern that worked twice)
- Dispatch docs-vs-reality / integration-drift audits as `delegate_task` subagents with a strict `output_schema` — 13+34 services audited this way in two rounds (PRs #2904/#2907), each finding file:line-cited gaps.
- `delegation.model` in profile config.yaml must be a CONCRETE model code (glm-5.2), not a provider alias (zai-coding) — aliases 400 with 'modelCode: does not exist' in subagent calls while working as the session default.
- Verify subagent findings against source before editing (one audit claim had the right line numbers but different wording than the doc).

## Worked example
- `references/channel-monitor-bringup-2026-09.md` — the full Elder-Melchor battle (env chain, docker recovery, what remained open).
- `references/pr-branch-hygiene-2026-09.md` — PR merge-state recovery (BEHIND/UNKNOWN), add -A scratch sweeps, fix-commit verification, register RELEASE pairing.
