---
name: pmoves-agent-zero
description: "Use when operating Agent Zero: compose, MCP, smokes."
---

# PMOVES Agent Zero — flagship orchestrator

Closest service to the full plugin pattern: submodule `PMOVES-Agent-Zero` (Hardened, path= set) + service dir `pmoves/services/agent-zero` (60 files) + compose always-on + Pinokio launcher `PMOVES-pinokio/api/pmoves-agent-zero` (the reference launcher: pinokio.json/install/start/status/update/reset).

## Architecture
- Ports: **8080** (UI) / **8081** (API+MCP, SSE at `/mcp/t-${AGENT_ZERO_MCP_TOKEN}/sse`)
- Compose: docker-compose.yml agent-zero stanza (no profile = always-on) + agents.images/agents.integrations overlays + arm64/vps overrides + darkxside/spark sidecars
- Env: env.shared + env.tier-agent via funnel; CAPTURE_OUTPUT=false per compose:3249; EXTRA_ARGS carries `--dockerized=true` (compose:3334)
- Legacy duplicate: `services/agent_zero` (4 files) is dead — the hyphenated dir is canonical

## Known Roads
- Bring-up: Pinokio launcher (one-click) or compose via agents profile targets
- MCP seed: `make a0-mcp-seed`; UI at http://localhost:8080
- **BROKEN (audit 2026-09-03)**: `health-agent-zero` target does NOT exist — `agents-headless-smoke` (Makefile:3708) calls it and breaks. Same for a0-mcp-smoke / a0-mcp-exec-smoke cited in docs. Use direct healthz curl until the define-or-delete PR lands: `curl -sS http://localhost:8080/healthz`

## Fleet context
- A2A router exists in-tree: services/agent-zero/python/features/a2a/server.py (create_a2a_router) — the dsh-agent dispatch surface
- E2B Danger-Room is its code-exec backend when configured
- Hermes MCP inventory: `agent-zero` entry (SSE, endpoint_prefix agent_zero)
