# PMOVES-pinokio fork — open review detail (2026-09-05 snapshot)

## PR #11 (sync/hardened-upstream-8.2.0): the two P1s, exact locations

### P1-1: release-pmoves.yml ARM64 Koffi gap
- `build.yml:274` area: `npm install --no-save @parcel/watcher-linux-arm64-glibc@2.5.1` (with npm_config_os=linux cpu=arm64 libc=glibc force=true)
- `build.yml:340`: `npm install --no-save @koromix/koffi-linux-arm64@3.0.2` — the step release-pmoves.yml LACKS
- Validation block ~L330-360: pick_file over `.../node_modules/@parcel/watcher-linux-arm64-glibc/watcher.node` (root + pinokiod nested paths), then `file | grep -qi aarch64` checks for pty/watcher/watcher-platform — koffi absent from both install and validation
- package.json:47-48 packs `node_modules/koffi/**` + `node_modules/@koromix/koffi-*/**` → 8.2 pinokiod depends on Koffi natively
- Fix shape: add the koffi install step (mirror npm_config_* env), add koffi payload pick_file + aarch64 `file` check beside the watcher checks

### P1-2: full.js privileged-renderer exposure
- Trusted hosts (pinokio.co + subdomains + others) in the openNonPinokioHttpsInBrowser exemption
- Affected windows: mainWindow/loadNewWindow — preload.js with contextIsolation:false, nodeIntegrationInSubFrames:true, window.electronAPI exposed, session handler grants EVERY permission request
- Contrast: community window is explicitly sandboxed
- Fix shape: keep public URLs on the sandboxed surface, or origin-gate preload exposure + the permission handler

## PR #12 branch leak postmortem
- Symptom: `gh pr view 12 --json changedFiles` = 107 (expected 9)
- Cause: `feat/fleet-console-apps` cut from stale local main carrying 2 extra commits of absorbed fork state (release-pmoves.yml, .gitmodules, api/pmoves-agent-zero/* ...)
- Fix applied: `git checkout main && git pull && git checkout -B feat/fleet-console-apps origin/main && git cherry-pick 918e4eb && git push -f` → 9 files
- Rule: verify changedFiles matches intent EVERY time a PR opens from a fork checkout

## Local node facts (elder-melchor)
- Upstream Pinokio 8.0.118 at %LOCALAPPDATA%/Programs/Pinokio; control plane :42000 only while the exe runs; /version 404s — probe `pterm version pinokiod`
- `api/PMOVES.AI/` is a FULL repo clone (76 entries, has pmoves/); `20` inside it is an empty FILE not a dir
- DramaBox-TTS start.js: GRADIO_SERVER_PORT={{port}}, PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True, darwin refused, URL capture via `/(http:\\/\\/[0-9.:]+)/`
- Qwen3-TTS app/app.py is Gradio (import gradio as gr) — no FastAPI route surface
- peer control plane 192.168.1.163:42000 was DOWN; tailnet candidates 100.73.74.3 / 100.122.182.3 / 100.87.181.8 all refused :42000 too
- flute-gateway compose block: docker-compose.yml:4969-5010 (env contract, host-affinity routing #2305, CHIT attribution subjects)
- Hermes tts default (this profile): edge provider en-US-AriaNeural — the switch target is flute :8055 once up
