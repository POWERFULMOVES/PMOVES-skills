# channel-monitor bring-up on Elder-Melchor — 2026-09-03 worked example

Goal: stand up the YouTube ingestion lane (channel-monitor :8097 → pmoves-yt :8077 → NATS) on the Windows laptop, using documented targets only.

## Failure chain (each step a real discovery)

1. `docker compose --profile channel-monitor up -d` → "no configuration file provided" — run from repo root, not pmoves/.
2. From pmoves/: interpolation demanded WGER_DB_PASSWORD etc. — the WHOLE compose tree interpolates; unrelated services' required vars block any bring-up. Fix path: stub the named vars in the shell, iterate (script harvests `required variable [A-Z_0-9]+` and re-exports).
3. `env.tier-media not found` → `touch env.tier-media`; later: seed ALL tiers from examples (`for ex in env.*.example; do cp $ex ${ex%.example}; done`).
4. Missing `env.tier-supabase` (NO example committed — undocumented requirement of channel_monitor_up.sh:25). Created from PMOVES-supabase/docker/.env.example key names.
5. `make channel-monitor-up` (after installing WSL make): `env.tier-ui` missing → seeded; then SUPABASE_JWT_SECRET STILL 'missing' despite being in env.shared:139 — ROOT CAUSE: `env.tier-api:21` carried `SUPABASE_JWT_SECRET=` EMPTY; empty in any chained --env-file fails `${VAR:?}` and later files can't recover. Confirmed `--env-file` DOES satisfy `:?` with a minimal repro (t.env/t.yml test) — so the file chain was the variable, not compose behavior.
6. ensure_env_shared.py rewrote env.shared between runs and dropped manual stub lines (only POSTGRES_PASSWORD survived) — lesson: stubs go in tier files only.
7. Build contexts: channel-monitor/pmoves-yt images BUILT fine once PMOVES.YT submodule was initialized (its pmoves_yt_service/ is the dockerfile context).
8. `pmoves_external` network missing → `docker network create pmoves_external`.
9. Legacy `pmoves-nats` (exited 7 weeks) conflicted with fresh `pmoves-nats-1`; docker daemon degraded under concurrent compose (ps/inspect hung). Recovery: kill compose job, rm stale containers, serial `docker start`.

## End state at session close
Containers reached Created; final `docker start` sequence still running under daemon contention. Config was fully valid — remaining work was purely daemon/latency. Health checks: `curl localhost:8097/health`, `:8077/health`; then `POST /api/monitor/check-now` and verify Cole Medin (@ColeMedin, channel_monitor.json:403, priority 2, tag archon) ingests through the pipeline with Discord approval (auto_process=false).

## Transfers to next bring-up
- Stub-unrelated-required-vars loop only unblocks interpolation; the file-based fix (fill empties in tier files) is the durable one.
- The make target's own preflight (`ensure-env-shared`) is the best error oracle — its failures name exactly which file it wants next.
- Budget 20+ min for first compose up on Starlink (image pulls); prefer serial starts once images exist.
