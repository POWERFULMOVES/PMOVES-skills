---
name: pmoves-hirag
description: "Use when operating Hi-RAG retrieval gateways (v1/v2, GPU)."
---

# PMOVES Hi-RAG — retrieval backbone

Fork: `PMOVES-HiRAG` @ PMOVES.AI-Edition-Hardened (standalone, last push 2026-03-02 — upstream relationship manual). Compose: v1 + v2 service dirs coexist; canonical entries UNPROFILED (always-on).

## Surface
- v1 HTTP: **8086**; GPU variant host port **8187** (HIRAG_V1_GPU_HOST_PORT default — old docs said 8090, fixed #2907)
- v2: `hi-rag-gateway-v2` — the canonical lane; fleet services point HIRAG_URL at it (pmoves-yt does)
- In-tree services: `pmoves/services/hi-rag-gateway/` (v1) + v2; submodules `pmoves-hirag-mcp` (MCP bridge)

## Ops
- Bring-up: core stack always-on; explicit GPU variant via HIRAG_V1_GPU_HOST_PORT
- MCP: pmoves-hirag-mcp submodule registers the bridge
- Collapse decision (2026-09-03 audit): v2 canonical; v1 either profile-gated `legacy` or deleted once this skill covers ops — tracked in docs/services/RUNTIME_REINTEGRATION_QUEUE.md
