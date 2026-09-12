# `sources/` — PMOVES skill sources

Skill **sources** the `skills` CLI installs from. Each is a PMOVES fork, pinned
as a submodule.

This directory exists only on **`PMOVES.AI-Edition-Hardened`**. That is the whole
point of it, and the rule below is load-bearing.

## The branch contract

| branch | contents | who writes it |
|---|---|---|
| `main` | byte-identical to `vercel-labs/skills` | upstream only, via fork-sync |
| `PMOVES.AI-Edition-Hardened` | `main` + this overlay | PMOVES |

**Never commit PMOVES changes to `main`.** It exists to track upstream cleanly so
`fork-sync` is always a fast-forward. The moment PMOVES content lands there, every
future sync becomes a merge with conflicts to resolve — which is how
`PMOVES.YT` ended up 206 commits behind its hardened branch, the cautionary case
`fork_registry_ratchet` cites in its own docstring.

`sources/` was chosen because upstream owns `skills/` (it ships `find-skills`
there) and has no `sources/` directory, so this overlay cannot collide with an
upstream addition.

## What is here

| source | upstream | branch tracked |
|---|---|---|
| `Pmoves-Claude-skills` | `anthropics/skills` | `PMOVES.AI-Edition-Hardened` |
| `Pmoves-Minimax-skills` | `MiniMax-AI/skills` (MIT) | `PMOVES.AI-Edition-Hardened` |
| `PMOVES-agent-sandbox-skill` | `disler/agent-sandbox-skill` | `main` |
| `pmoves-fork-repository-skill` | `disler/fork-repository-skill` | `main` |
| `PMOVES-awesome-agent-skills` | `heilcheng/awesome-agent-skills` | `main` |
| `Pmoves-claude-d3js-skill` | `chrisvoncsefalvay/claude-d3js-skill` | `main` |

The four `main`-tracked forks have no hardened branch yet; they are pinned at the
same commits `PMOVES.AI` pinned before this move, so nothing changed but location.

## Why sources rather than vendored copies

The `skills` CLI resolves a source to a set of skills and installs them into
whichever harness directory the target agent uses — `.claude/skills/`,
`.agents/skills/`, `.minimax/skills/`, `.codex/`, and 70+ more. So:

- a **source** is the thing under version control here
- an **installed skill** is a build artifact in a harness directory

Keeping that distinction is what makes a skill transferable across harnesses
instead of pinned to whichever one it was authored for.

## Skills must satisfy the Agent Skills spec

<https://agentskills.io/specification>. The parts that actually reject a skill:

- `SKILL.md` is **required** at the skill root
- `name`: 1-64 chars, lowercase `a-z0-9` and `-`, no leading/trailing hyphen, no
  `--`, and **must match the parent directory name**
- `description`: 1-1024 chars, non-empty — it is loaded at startup for *every*
  skill, so it is what makes a skill discoverable at all

Validate with `skills-ref validate ./<skill>`.
