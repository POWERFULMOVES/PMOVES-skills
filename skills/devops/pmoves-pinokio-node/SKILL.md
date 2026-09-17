---
name: pmoves-pinokio-node
description: "Use when bringing up or reviewing Pinokio on a PMOVES node."
---

# Pinokio on PMOVES nodes — install, control plane, fork PRs, voice engines

Sister skills (pterm, gepeto, pmoves-pinokio-fork, pmoves-fleet-console, hermes-pmoves-pr-review, pmoves-voice-fabric) are currently USER-OWNED (created_by=None — autonomous patches refused); this skill carries the 2026-09-05 session truth until they are adopted via `hermes curator adopt <name>`.

## Local control plane (elder-melchor, verified 2026-09-05)
- PINOKIO_HOME: `C:\pinokio` (confirm: `pterm home`). Upstream install: `%LOCALAPPDATA%/Programs/Pinokio/Pinokio.exe` — launch the exe to start the :42000 control plane; it is NOT auto-running.
- Probe: `curl -fsS http://localhost:42000/` returns the Pinokio HTML shell (`/version` is 404); `pterm version pinokiod` is the reliable version probe (8.0.118 then; user began 8.2 update 2026-09-05).
- pterm Axios ETIMEDOUT on a LAN IP (e.g. 192.168.1.163:42000) = pointed at a PEER control plane that's down — check `pterm home`, probe localhost then tailnet nodes (:42000 on 100.x) before debugging pterm.
- Local api/: PMOVES.AI full clone, DramaBox-TTS-Pinokio.git (Gradio/LTX-2.3 cloning TTS, NVIDIA GPU, GRADIO_SERVER_PORT={{port}}), Qwen3-TTS-Pinokio.git, Customokio.git, hermes-agent.pinokio.git, hermes-mod.git. App dirs carry `.git` suffix; `api/PMOVES.AI/20` is an empty-file artifact, not a dir.
- Upstream auto-update drifts off the fork build (app-update.yml owner) — after update prompts, re-verify + reinstall from fork release.

## Fork PR review state (PMOVES-pinokio, 2026-09-05)
- **PR #12** (pmoves-fleet + pmoves-hermes apps): rebased onto fresh main after stale-branch leak (107→9 files). New-app pattern: pinokio.json + install.js (repo-root.txt detect; PATH tools to ~/.local/bin) + start.js (shell.run chain, on:[{event,done:true}] URL capture, daemon:true) + SKILL.md frontmatter. pinokio.js menu: `menu: async (kernel, info) =>` with `info.running('status.js')` start/live flip; href may be local script OR http URL.
- **PR #11** (7.0.0→8.2.0 sync, branch sync/hardened-upstream-8.2.0) — TWO P1s verified, fixes owed: (1) `release-pmoves.yml` ARM64 (~L274/L330) installs `@parcel/watcher-linux-arm64-glibc` but NOT `@koromix/koffi-linux-arm64@3.0.2` (build.yml:340 has it) — 8.2 pinokiod's Koffi native dep makes ARM64 releases broken-yet-green; mirror install + pick_file/file-validate the koffi payload. (2) `full.js`: pinokio.co trusted-host exemption loads public pages in privileged renderers (contextIsolation:false, nodeIntegrationInSubFrames:true, full electronAPI, grant-all permission handler) — keep public URLs on the sandboxed community window or origin-gate preload/permissions.
- **Branch hygiene**: fork branches from fresh `origin/main` only — stale local main carries absorbed fork state (99 stray files on PR #12). Fix: fetch → `git checkout -B <b> origin/main` → cherry-pick → push -f; verify `gh pr view N --json changedFiles` after opening.
- PR #5 CONFLICTING (stale), PR #2 mergeable docs.

## Voice-on-node via Pinokio engines (flute architecture, compose :4969+)
- NATIVE-first: `ULTIMATE_TTS_URL=http://host.docker.internal:7860` — UTS engines run IN PINOKIO on the host; the GHCR container is UI/MPC shell WITHOUT engine packages. Same pattern OmniVoice :8002, VibeVoice :7860. `VOICE_HOST_AFFINITY=1` + `VOICE_FLEET_NODES` swaps engine URLs to fleet Tailscale hostnames.
- Flute up REQUIRES the full env chain (wger `:?` interpolation guards): `docker compose --env-file env.shared --env-file env.tier-data --env-file env.tier-api --env-file env.tier-llm --env-file env.tier-worker --env-file env.tier-media --env-file env.tier-agent --env-file env.tier-supabase -f docker-compose.yml --profile orchestration up -d flute-gateway`. Pulls over Starlink exceed 5min — background it.
- Node TTS switch: hermes tts default is edge provider (en-US-AriaNeural); the upgrade = point hermes tts at flute :8055, engines stay Pinokio-side. DramaBox/Qwen3 are Gradio → candidates for `pmoves/config/voice/engines/*.yaml` manifests.
- llamanet (pinokiocomputer/llamanet, MIT): local llama.cpp OpenAI-replacement — STALE since 2024-06; role now covered by Ollama/cloud. Don't wire without a revival.

## PR/register workflow rules (session-hardened, generalizes beyond pinokio)
- **Register RELEASE pairing**: a RELEASE row belongs to the CLAIM's signing owner — any other agent files an acknowledgement row (filed-by + closed-by-evidence), never a closure under the claimant's name; `open_claims_in()` keys on canonical identity and misfiled rows corrupt ownership (Codex P2 on #2960).
- **Lane hygiene sweep**: before claiming, grep AGNOTE4482PHI.t1.md for old CLAIMs whose work merged (PR body 'Closes...') with no RELEASE — backfill with evidence + honest remainder under YOUR identity.
- **index.lock contention** from concurrent cron git processes: wait ~30s, `rm -f .git/index.lock`, retry.
- **Stage explicit paths** in PR branches, never `git add -A` — workspace scratch (.hermes-audit-ws/, terminal_survey/) leaks in; force-push the fix if it lands.
- **mergeStateStatus UNKNOWN after push** = GitHub recompute lag — poll later / enable --auto rather than re-pushing.
- **BEHIND PR with current-looking local branch** = stale LOCAL branch — `git fetch` + `git reset --hard origin/<branch>` before merging main.

## Sentinel registry console (live)
pmoves-fleet app consumes fleet-sentinel :8116 `/registry.json` (health chips, 10s refresh). pmoves-hermes app bootstraps profile+MCP+CHIT → gateway :7700.
