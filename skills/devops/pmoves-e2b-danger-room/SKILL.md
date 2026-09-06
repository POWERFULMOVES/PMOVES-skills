---
name: pmoves-e2b-danger-room
description: "Use when running E2B sandboxes for code execution."
---

# PMOVES E2B Danger Room — sandboxed code execution

Fork of e2b-dev/E2B (submodule `PMOVES-E2B-Danger-Room`): secure cloud sandboxes for agent code execution. Synced 2026-09-03 via merge-upstream (base e2b-dev:main) — preserves the 1 PMOVES commit (sync PR #2, 2026-06-10).

## SDK usage
- JS: `npm i e2b` · Python: `pip install e2b`
- `E2B_API_KEY=e2b_***` from dashboard.e2b.dev
- Python: `from e2b import Sandbox; s = Sandbox.create(); s.commands.run('...')`
- Code Interpreter: `pip install e2b-code-interpreter` for `run_code()` rich outputs

## Self-hosting (the PMOVES path)
- Infra repo: e2b-dev/infra, Terraform, AWS/GCP supported
- Danger-Room pattern: untrusted agent code runs in firewapped microVMs, never on fleet nodes
- Siblings: PMOVES-E2B-Danger-Room-Desktop (UI), pmoves-e2b-mcp-server (MCP bridge), PMOVES-E2b-Spells

## Fleet context
- Agent Zero code-exec tools route here when configured
- Templates: `templates/base`, `skills/stripe-projects`
