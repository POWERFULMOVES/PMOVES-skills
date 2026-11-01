---
name: acp-registry-publish
description: Publish an Agent Zero (or any Python agent) to the Agent Client Protocol registry, and validate A0 plugin manifests against official A0 docs. Use when exposing an agent via ACP stdio, preparing an agent.json for the ACP registry (agentclientprotocol.com), fixing registry CI authMethods/distribution rejections, or auditing plugin.yaml fields like always_enabled.
---

# ACP Registry Publish (A0 edition)

Battle-tested path for exposing Agent Zero as an ACP stdio agent and listing it
in the official registry — including the two failure modes we hit and fixed.

## Registry contract (what CI enforces)

- Manifest = `<id>/agent.json` in the registry repo. Required: `id`, `name`,
  `version`, `description`, `distribution`.
- `distribution` types the CI accepts: **`binary` / `npx` / `uvx` ONLY**.
  - `binary` must ship darwin + linux + windows archives or CI rejects it.
  - For a Python agent, use **`uvx`**. A git URL is a valid `package` value
    (`git+https://github.com/OWNER/REPO.git`) — PyPI publication is NOT required
    (trade-off: registry hourly auto-update tracks PyPI/npm/releases only, so a
    git-distribution entry updates via manual PRs).
  - **`stdio` is NOT a distribution type** — a manifest using it fails the
    registry schema. (We found a bundled adapter manifest using `stdio`; the fix
    was a uvx manifest pointing at the packaged console script.)
- CI verifies agents "return valid `authMethods` in the ACP handshake":
  - Advertise an env-var authMethod when a secret is set (e.g. `env_var`
    `agent-zero-api-key` iff `AGENT_ZERO_API_KEY` present); validate the bearer
    on `authenticate`; without the key return `authMethods=[]` (permissive local
    editor use, backwards compatible).

## A0 plugin manifest rules (official docs)

From the A0 plugin implementation contract (`a0-create-plugin` reference):

- `always_enabled: true` is **reserved for framework use** — bundled system
  plugins (`_browser`, `_code_execution`, `_kokoro_tts`, ... living in the
  framework's own plugins dir) may set it. A third-party/standalone plugin
  setting it locks itself permanently ON and violates the contract. **Drop it**
  and rely on the default (activation defaults to ON when no toggle rule
  exists) or use `per_project_config` / `per_agent_config` for scope control.
- Lifecycle belongs in plugin-root `hooks.py` (`install`/`pre_update`/
  `uninstall`); never in `execute.py`; `execute.py` is omitted unless there is
  an explicitly requested manual action unrelated to lifecycle.
- Plugin-local imports use `usr.plugins.<name>...`; no `sys.path` hacks.

## PMOVES proven steps

1. Package the adapter: `pyproject.toml` + console script entrypoint
   (`acp_adapter.py`), proven with
   `uvx --from git+https://github.com/OWNER/REPO.git@branch <script> --version`.
2. Author the registry manifest (`uvx` distribution, repository + MIT license,
   optional authors). Validated draft:
   `plans/ACP_REGISTRY_MANIFEST_DRAFT_2026-10-07.md` in PMOVES.AI.
3. Fork POWERFULMOVES/PMOVES-registry, add `pmoves-agent-zero/agent.json`, PR
   per its CONTRIBUTING.md. CI runs the authMethods handshake — satisfied by
   the env-var contract above.
4. Optional later: PyPI publish so the registry's hourly auto-update tracks it.

## Gotchas we hit

- Bundled adapter manifest (`acp_registry/agent.json`) and the registry
  submission are DIFFERENT artifacts: the bundled one serves the local plugin;
  only the uvx-shaped one goes to the registry.
- Fork SDKs (protocol repo + TS/Python SDKs) are validation harnesses — use the
  Python SDK's `echo_agent.py` pattern to handshake-test before opening the
  registry PR.
