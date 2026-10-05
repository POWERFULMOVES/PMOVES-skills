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

## Pinokio 8 reconciliation (2026-10-05, upstream pinokiocomputer/pterm README)

- Cross-platform: pterm ships via `npm install -g pterm` (axios sibling dep). Verify with `pterm version terminal` (also pinokiod, pinokio, script).
- Refs: `pinokio://<host>:<port>/<scope>/<id>` (scope `api` = installed app under PINOKIO_HOME/api). pterm talks to the LOCAL control plane, which resolves/forwards to the target node — apps are never addressed by their ready_url.
- Peer ops: `pterm open <url> [--peer <host|host:port|name>] [--surface browser|popup] [--preset center-small|center-medium|center-large|fullscreen]` — with --peer, URLs open from the peer node's point of view (127.0.0.1 = peer loopback).
- Fleet: nodes WITH a local Pinokio control plane reach peers natively via refs/--peer. Containers WITHOUT a control plane (e.g. A0 sidecar) must delegate to a Pinokio-node agent instead.
- Never pass secrets through clipboard on shared nodes.
