---
name: pmoves-mcp-gateway
description: "Use when building or debugging the Docker MCP gateway fork."
---

# PMOVES mcp-gateway — Docker MCP Toolkit fork

Fork of docker/mcp-gateway (submodule `PMOVES-mcp-gateway`): the `docker-mcp` CLI plugin + MCP Gateway powering the fleet's Docker MCP usage (the very tool serving this session's Hostinger/VPS tools). Synced 2026-09-03 via merge-upstream fast-forward to docker:main.

## What it is
- Gateway pattern: `AI client → MCP Gateway → MCP servers (containers)`
- Secrets via Docker Desktop secret management (not env vars)
- OAuth flows for token-service connections; dynamic tool/resource/prompt discovery
- Catalog: Docker Hub MCP catalog (`docker mcp catalog ...`)

## Fleet usage
- Elder-Melchor serves Docker MCP via `docker mcp gateway run --servers ...` (profile config `docker-mcp` server)
- PMOVES profiles: pmoves_5090_web etc. via `--profile` flags
- When the gateway CLI misbehaves, THIS repo is the source; Go 1.24+ to build

## Ops
- Sync (real GitHub fork, zero PMOVES commits): `gh api -X POST repos/POWERFULMOVES/PMOVES-mcp-gateway/merge-upstream -f branch=main` → fast-forward
- Submodule branch: main (independent release cadence per .gitmodules strategy — NOT on Hardened)
