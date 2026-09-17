---
name: gepeto
description: "Use when scaffolding new Pinokio apps from templates."
---

# gepeto — Pinokio app scaffolder

From the PMOVES-gepeto fork submodule (v0.5.7, cocktailpeanut, MIT).

## Usage
- `npx gepeto@latest` — interactive scaffold (use fork checkout: `node PMOVES-gepeto/index.js`)
- Docs site: https://gepeto.pinokio.computer (fetch blocked from this network — offline source is the fork)

## Anatomy of a generated app (template 1 — the canonical skeleton)
- `pinokio.json` — `{title, description, icon}` shown in Discover
- `index.js` — `module.exports = { run: [...] }` script chain
- `install.js` / `start.js` / `reset.js` / `update.js` — lifecycle scripts
- `icon.png|svg`

Script methods (see PMOVES-pinokio apps for real examples):
- `shell.run` {message, path, env, on:[{event:regex, done:true}]}
- `fs.read` {path, encoding} → `{{input.trim()}}`
- `local.set` {key: value} → `{{local.key}}`
- `daemon: true` for background services

## PMOVES conventions (mirror pmoves-agent-zero in PMOVES-pinokio/api/)
- Read repo root once via `fs.read repo-root.txt` → `local.repo_root`, then `shell.run` with `path: {{path.resolve(local.repo_root,'pmoves')}}` and make targets
- `on: [{event: "/(http:\\\/\\/[0-9.:]+)/", done:true}]` captures the ready URL
- Ship a SKILL.md with frontmatter (name/description/keywords/version/category/tier/agent_class/agent_id) like pmoves-services does
- NEW apps belong in the PMOVES-pinokio fork (submodule), not the parent repo — pmoves-hermes is the pending one
