---
name: pmoves-fleet-console
description: "Use when enhancing the Pinokio fleet console or IDE configs."
---

# PMOVES Fleet Console — IDE + Pinokio integration

Plan: `pmoves/docs/services/IDE_PINOKIO_FLEET_CONSOLE_PLAN.md` (PR #2916). Audit found: 9 VS Code gaps (zero MCP clients despite inventory SSE endpoints; compose overlays schema-less; 12/878 targets in tasks.json; Windows venv bin/Scripts split), 6 harness gaps (Known-Roads hooks Claude-only → KiloCode runs raw docker ungated; kilo.json fail-open via render_kilocode mcp_config_generator.py:234-257).

## The autonetwork mechanism (fleet-sentinel design)
- Announce listener: reuses `ServiceAnnouncementListener` (nats_service_listener.py:63-131, queue group dedupes) on `services.announce.v1`
- Health poller: 30s parallel HTTP over registry health_check_urls (flight_check_retro.py pattern); staleness = 2× announcer interval
- Self-heal: 3 consecutive failures → Known-Road recovery (`make secrets-funnel && make up-<svc>`) via with-env.sh subprocess; 1 restart/service/10min; jsonl action trail
- HTTP: GET /registry.json (Pinokio consumes — launchers need NO NATS client), /healthz, /actions

## Pinokio console (PMOVES-pinokio fork)
- Registry-driven menu: sentinel /registry.json → one row per announced service (tier icons from ServiceTier enum)
- Sibling apps: pmoves-fleet (scanner + fleet-status), pmoves-dispatch (REUSES pinokio_bridge :8130 / nats_event_bus :8131 from make up-pinokio)
- Self-heal in status.js: health curl → red = Known-Road recovery
- Launcher discipline: PINOKIO_LAUNCHER_GUIDE.md pattern, URL capture on:[{event,done:true}]

## Transformation loop
Service announces → registry row → Pinokio menu row + health chip → self-heal owns it → fleet load/resource sharing optimizes around REAL capability, not static config.
