---
name: pmoves-pinokio-fork
description: "Use when installing/updating the PMOVES Pinokio fork build."
---

# PMOVES Pinokio fork

Upstream Pinokio auto-updates from `pinokiocomputer/pinokio` (see app-update.yml owner/repo) — an upstream install will drift. Use the fork build to stay PMOVES-branded.

## Install (fork prebuilt)
1. `gh release list --repo POWERFULMOVES/PMOVES-pinokio --limit 5` — v7.0.0 Latest; v8.0.40 draft (pinokio8)
2. `gh release download <tag> --repo POWERFULMOVES/PMOVES-pinokio --pattern 'Pinokio.exe' --pattern 'latest.yml' --dir <dir>`
3. Install Pinokio.exe over the old install (`%LOCALAPPDATA%/Programs/pinokio`). Close running Pinokio first (`tasklist | grep -i pinokio`).
4. Verify app-update.yml owner is `POWERFULMOVES` — CAUTION: fork's package.json `build.publish` still points at pinokiocomputer/pinokio, so fork-built updaters may STILL pull upstream. Disable auto-update in settings or re-install from fork after any update prompt until fixed.

## Fork layout (submodule PMOVES-pinokio, branch main)
- `api/` — 12 PMOVES apps (pmoves-agent-zero is the reference implementation; pmoves-hermes NOT YET created — mirror claude/crush launchers using pmoves-agent-zero pattern + hermes-pmoves script)
- Build: electron-builder; `npm run mw` (mac+win), `dist2` (all). CI dispatch: `.github/workflows` publish-binaries (PR #3)
- Releases carry .exe/.AppImage/.deb/.rpm + latest.yml per platform

## PMOVES hermes app (pending)
Pattern: pinokio.json + install.js (write repo-root.txt) + start.js → `hermes-pmoves` (pmoves/scripts/hermes-pmoves) via shell.run on the repo; ready URL from `hermes gateway` port 7700. Bootstrap chain: hermes-pmoves → make -C pmoves hermes-bootstrap → hermes-fleet-bootstrap.sh (profile pmoves-hermes-elder).
