---
name: pmoves-activepieces
description: "Use when running ActivePieces self-hosted (compose, flows)."
---

# PMOVES ActivePieces — low-code automation companion

Self-hosted on PMOVES.AI as companion to the n8n fabric. Source fork: `PMOVES-activepieces` submodule (synced to upstream main `ecd58d73`, Hardened branch). Runtime: `pmoves/activepieces/docker-compose.yml` (standalone stack — app+worker+postgres+redis, image `ghcr.io/activepieces/activepieces:0.86.3`, opt-in, not baked into fleet compose).

## Operations
- Bring up: `docker compose --env-file pmoves/activepieces/env.activepieces -p pmoves-activepieces up -d` (seed env from `env.activepieces.example` via secrets pipeline — never hand-generate)
- UI: `http://localhost:8087` (AP_APP_PORT default)
- Flows-as-code: `pmoves/activepieces/flows/` — Git Sync points BOTH the hosted cloud project (cataclysmstudios@gmail.com) and self-host at this dir (mirrors n8n convention). Git Sync is ENTERPRISE: needs `AP_EDITION=ee` + `AP_LICENSE_KEY`; Community (`ce`, default) uses export→commit→import
- Flow search: `.github/workflows/activepieces-flow-search.yml` indexes flows for CI search

## Fleet context
- n8n is PRIMARY (skill `pmoves-n8n`); AP handles low-code/piece flows (~400 MCP-server pieces upstream)
- Phase 2 (tracked follow-up): fold into main fleet compose
- Source fork synced via refs API (standalone mirror, 60k commits — always shallow)
