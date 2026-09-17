---
name: pmoves-n8n
description: "Use when working with self-hosted n8n workflows or API."
---

# PMOVES n8n — self-hosted automation fabric

**Three layers, don't confuse them:**
1. `PMOVES-N8N-Auto` submodule — the n8n SOURCE fork (n8n-io/n8n mirror synced to upstream master `44ddbbd2`, Hardened branch). Only needed for source patches; ~546MB repo, shallow only.
2. `PMOVES-n8n` submodule — workflows-as-code OVERLAY repo (55 files: hardened workflow definitions, `compose/n8n/Dockerfile`, `docker-compose.pmoves.yml`, CHIT wiring). NOT a code fork — merging upstream n8n into it fails 'unrelated histories' BY DESIGN.
3. Runtime — `pmoves/docker-compose.n8n.yml` (+ `.n8n.postgres.yml`): image `pmoves/n8n:2.1.5-runtime` built from the overlay, container `pmoves-n8n`, port 5678.

## Operations
- Up: `docker compose -f pmoves/docker-compose.n8n.yml -f pmoves/docker-compose.n8n.postgres.yml up -d` (needs `N8N_DB_*` env from secrets funnel — env.tier-worker)
- Health: `curl -sf http://localhost:5678/healthz`
- API: public API enabled (`N8N_PUBLIC_API_ENABLED`, `N8N_API_AUTH_ACTIVE`) — key via env, workflows CRUD at `/api/v1/*`
- Workflows live as code in `PMOVES-n8n` (PR #4 hardened-flows set: side-effect routing, signature-failure paths); sync via the repo's own sync flows
- Webhooks need `WEBHOOK_URL` env; runners default internal mode (`N8N_RUNNERS_MODE=internal` unless token set)

## Fleet context
- Companion automation: activepieces (skill `pmoves-activepieces`) — n8n is primary, AP is the low-code companion
- MCP: n8n MCP bridge sketched on the stale Integrations rail (`pmoves/config/mcp/n8n.yaml` — 'adaptor todo'); needs porting to mcp_inventory.json if wanted
- Secrets flow env.shared → env.tier-worker (DISCORD_WEBHOOK_URL, GH_WEBHOOK_SECRET, AGENT_ZERO_EVENTS_TOKEN referenced by workflows)
