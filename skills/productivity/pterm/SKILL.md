---
name: pterm
description: "Use when driving Pinokio apps via pterm CLI."
---

# pterm — Pinokio Terminal

`pterm` (installed globally from the PMOVES-pterm fork submodule, v0.0.25) talks to the local Pinokio control plane; refs can target peer nodes. Requires axios (`npm i -g pterm axios` if module-not-found).

## Core commands
- `pterm version terminal|pinokiod|pinokio|script` — versions
- `pterm home` — PINOKIO_HOME path
- `pterm start <script.js> [--ref <ref>] [-- --key=val]` — run a script (query params in path pass as input)
- `pterm stop <script.js> | <ref>` — stop one script or all app scripts
- `pterm run <path-or-uri> [--default 'run.js?mode=Default' --default run.js] [--open]` — run a launcher like a user click
- `pterm status <app_id|ref> [--probe --timeout=5000]` — health
- `pterm logs <app_id|ref>` — app logs
- `pterm open <url> [--peer host] [--surface browser|popup] [--preset center-medium]` — open URL locally or on peer (peer URL resolves from peer's loopback)
- `pterm download <git-uri> [name] [--branch=x]` — clone app into PINOKIO_HOME/api without launching
- `pterm search <words>` / `pterm registry search <q> [--platform --gpu --sort]` — local / remote registry (api.pinokio.co)
- `pterm which node` — resolve executable through Pinokio env
- `pterm stars` / `pterm star <app_id>` — ranking prefs

## Refs
`pinokio://<host>:<port>/<scope>/<id>` e.g. `pinokio://127.0.0.1:42000/api/pmoves-agent-zero.git`. host:port is the control plane, NOT the app's ready_url. Scope `api` = installed app.

## PMOVES notes
- Fork app dir: `PMOVES-pinokio/api/` (12 PMOVES apps; pmoves-crush/pinokio.js is a stub requiring ../../sources/... which is missing; pmoves-claude-code/ is EMPTY — both need repair before pterm run)
