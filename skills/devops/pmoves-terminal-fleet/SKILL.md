---
name: pmoves-terminal-fleet
description: "Use when picking or forking terminals for nodes."
---

# Fleet Terminal Strategy (20266-09-05 survey — fix that date typ0: 2026-09-05)

Full doc: `pmoves/docs/services/FLEET_TERMINAL_STRATEGY.md`.

## Picks
- **PRIMARY**: FORK Wave Terminal (Apache-2.0) — `wsh` CLI = 40+ agent verbs over the whole UI (run/edit/ai/file/blocks/setconfig), YAML config-as-code, BYOK local AI (Ollama), v0.14 durable SSH sessions, AUR waveterm-bin. The only terminal where an AGENT owns the UI.
- **Human shell**: Ghostty upstream (MIT, ~100MB, omarchy default) + light patch-fork for PMOVES theming; libghostty embeddable core = future bespoke-terminal path.
- **WINDOWS**: same Wave fork (Win10 1809+) + WezTerm alt; Windows Terminal recovery. Ghostty/Kitty/Zellij have NO native Windows.
- **HEADLESS**: tmux + Atuin (self-host sync) — send-keys/capture-pane is what every harness already speaks. Never fork tmux.

## Fork policy
FORK: Wave, Ghostty(patch), maybe Atuin-server. UPSTREAM: tmux/WezTerm/Warp(AGPL-drag)/Atuin-client. AVOID forking: Amp/Cursor/Crush (closed or FSL fork-hostile).

## Immediate wiring
- `mcp-interactive-terminal` (MIT) → mcp_inventory hermes client = agent-controlled terminal TODAY, pre-fork
- Agent-CLI trio: opencode (MIT, Arch extra) + Gemini CLI + Codex CLI
- Amazon Q Developer CLI = Fig successor, Apache-2.0 fork-safe
