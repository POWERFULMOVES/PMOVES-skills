---
name: pmoves-supabase
description: "Use when operating the PMOVES Supabase fleet tier."
---

# PMOVES Supabase — foundational fleet tier

Submodule `PMOVES-supabase` @ PMOVES.AI-Edition-Hardened (source of truth for real values: `docker/.env.example` carries the canonical key names — JWT_SECRET, ANON_KEY, SERVICE_ROLE_KEY, SECRET_KEY_BASE, VAULT_ENC_KEY, ANON_KEY_ASYMMETRIC). Auth/db backbone for ~13 services incl channel-monitor persistence and pmoves-yt ingest.

## Compose topology
- Main entry: docker-compose.core.yml (supabase stack: kong, gotrue, postgrest, realtime, pooler, edge-functions, storage, logflare) + compose supabase-* entries with `--profile supabase` gating
- REST via **supabase-kong:8000** (fleet services use `http://supabase-kong:8000/rest/v1` — NOT postgrest:3000, that default in old docs is wrong)
- Env funnel: env.tier-supabase (NO example file committed — known gap; the file is REQUIRED by scripts/channel_monitor_up.sh:25)

## Gotchas learned live (2026-09-03)
- `${VAR:?}` interpolation: an EMPTY value in any of the 9 chained --env-file files triggers the required-variable error even when a later file sets it — env.tier-api had `SUPABASE_JWT_SECRET=` empty and poisoned the whole chain
- ensure_env_shared.py regenerates env.shared and drops manual additions — put stubs/overrides in env.tier-* files, never env.shared
- Local bring-up without real secrets: seed tier files from examples, then fill any EMPTY `KEY=` lines (empty = missing to compose)

## Ops
- `make supabase-up` / `supabase-stop` / `supabase-clean`
- Kong is the fleet entrypoint; service URLs route through it
- Real values: PMOVES-supabase/docker/.env (from its example) — funnel via secrets sync, never commit
