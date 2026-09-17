---
name: pmoves-skills-cli
description: "Use when installing agent skill packs from the ecosystem."
---

# PMOVES-skills — open agent skills ecosystem CLI (vercel-labs fork)

Submodule `skills/PMOVES-skills` @ PMOVES.AI-Edition-Hardened (upstream: vercel-labs/skills, v1.5.22). The `skills` CLI installs skill packs across 76+ agent harnesses (Hermes, Claude Code, Codex, Cursor, OpenCode…).

## Core
- `npx skills add <owner>/<repo>` — install a skill pack (from the skills.sh registry or git)
- `npx skills add vercel-labs/agent-skills` — canonical example
- `npx skills exec` — generate a prompt for one skill without installing / interactive
- Fork adds PMOVES packs: `Pmoves-Claude-skills`, `Pmoves-Minimax-skills`, `PMOVES-skills` on POWERFULMOVES

## When to use
- User asks for a capability skill that exists in an ecosystem (bash, code-review, testing, docs…)
- Fleet pattern: install once per node profile (Hermes profile skills dir), mirror across nodes
- PMOVES convention: prefer PMOVES forks of packs when they exist (grep POWERFULMOVES repo list first)

## Install into THIS Hermes profile
Skills live at `~/AppData/Local/hermes/profiles/pmoves-hermes-elder/skills/` (Windows). Either `skill_manage` tool or the CLI pointed at that dir.
