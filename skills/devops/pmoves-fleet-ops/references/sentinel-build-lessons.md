# Sentinel build lessons — PR #2934 review receipts (2026-09-04)

Full findings → fix mapping from the codex-connector review of the fleet-sentinel.
Cited so future announce-driven consumers start already knowing these.

| # | Finding | Fix landed in 86e689e25 |
|---|---------|------------------------|
| P1 | Build context `./services/fleet_sentinel` can't COPY sibling `services/common/` | Dockerfile self-contained + raw-NATS subscriber fallback (`_start_raw_listener`) when common absent |
| P1 | Listener dies: nats_service_listener → service_registry → `from supabase import Client` (module level, service_registry.py:35) not installed | Full dep set in image (fastapi uvicorn nats-py httpx supabase) — lean-image call was wrong |
| P1 | No `networks:` → implicit default; `nats` hostname unresolvable (nats on pmoves_bus/pmoves_external) | networks: pmoves_app + pmoves_bus + pmoves_external |
| P1 | Fixed 120s staleness kills one-shot announcers (announce_service publishes once, closes conn — no heartbeat in producers) | Staleness gated on optional `metadata.announce_interval_s`; one-shot announces never expire |
| P1 | `/srv/pmoves` layout: with-env.sh at `/srv/pmoves/pmoves/scripts` not `/srv/pmoves/scripts`; slim image lacks make/docker anyway | Path detection (`PMOVES_DIR/pmoves/Makefile` exists check) + honest limitation: no docker socket, host-side runner required, exit codes recorded |
| P1 | **RCE**: NATS slug interpolated unquoted into `bash -c` — announcement + failing health URL = arbitrary exec at threshold | `SLUG_RE = ^[a-z0-9][a-z0-9-]{0,63}$` before ANY use; `create_subprocess_exec` fixed argv, no shell |
| P2 | `make up-sentinel` referenced but didn't exist | Target added (infra.mk), dry-run verified |
| P2 | `str(ServiceTier.API)` → `"ServiceTier.API"` in registry.json | Serialize `tier.value` |

## Transferable patterns
1. **Untrusted-bus-input rule**: anything off NATS/MQ/webhooks that reaches exec/paths gets allowlisted + fixed-argv. The 'it's internal' assumption is the vulnerability.
2. **Silent-task failure mode**: an unawaited startup task that raises on import leaves /healthz green and the feature empty — health checks must assert FEATURE state (listener running, registry count), not process liveness.
3. **One-shot vs heartbeat contracts**: never build age-based staleness against a producer contract you haven't verified emits heartbeats. Gate on an explicit interval declaration.
4. **Review against CURRENT head**: 4 of 8 findings on the sibling a0-smoke PR targeted an already-superseded commit — verify each finding's base before fixing; reply with the pointer when superseded.
