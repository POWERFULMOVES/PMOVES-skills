---
name: gepeto-wrapper
description: >
  PMOVES-side mirror of Pinokio 8 built-in gepeto. Operates on the
  pmoves/configs/pinokio-apps/{curated,user}/ registry (creator-collab slice 4):
  list/inspect/scaffold/validate/promote entries; reconcile via mesh_exposure.
  NOT for launcher authoring itself.
version: 1.0.1
lane: creator-collab
slice: 4
origin: pmoves/skills/gepeto-wrapper-skill/SKILL.md
---

# gepeto-wrapper — PMOVES registry surface for the Pinokio apps lane

Boundary: contract layer between Pinokio and PMOVES. Reads the registry, calls
pmoves/services/mesh_exposure to reconcile the live fleet, scaffolds entries.
Launchers stay in the Pinokio ecosystem (built-in gepeto).

## When to use

- List curated apps + network_exposure contracts
- Show one app entry (runtime, endpoints, network_exposure)
- Scaffold user/<slug>.yaml; validate against pinokio-app.v1.schema.json
- Promote user/ to curated/ after operator review; reconcile vs headscale ACL + cloudflared + DNS

## Pinokio 8 reconciliation (2026-10-05)

- Upstream gepeto: npx gepeto@latest — docs gepeto.pinokio.computer.
- v8 launcher patterns changed: API + template features (long-prompt agent passing,
  HF login/upload handling, native desktop tool opening, footer visibility, GPU-routed PyTorch).
- Registry impact: entries SHOULD record v8 template/api usage + GPU routing assumption
  alongside runtime/endpoints/network_exposure. Schema pinokio-app.v1 needs a v8-fields
  bump — flag to schema owner, do not silently extend.

## Relationship to the built-in gepeto (boundary)

- This wrapper is REGISTRY-side: pmoves/configs/pinokio-apps/{curated,user}/ + mesh_exposure reconciliation.
- Launcher scaffolding itself is the separate `software-development/gepeto` skill (PMOVES-gepeto fork, npx gepeto@latest). Do not cross the boundary.
- v8 fields: entries SHOULD record template/api-pattern usage + GPU routing assumption (schema bump pending — flag to schema owner, do not silently extend).
