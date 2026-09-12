# PMOVES-skills fork — sync agent skills into the repo

The fork of vercel-labs/skills is our fleet skill distribution channel. The
`skills/` dir it ships upstream is nearly empty (find-skills only); our
PMOVES + hermes + github skills live in the Hermes profile and must be
mirrored into the fork so any harness (Claude, Codex, Cursor, +72) installs
them via `npx skills add POWERFULMOVES/PMOVES-skills`.

## One-time copy (from profile → fork)
Profile root is PER NODE — resolve it for the machine you mirror from; never
copy another node's absolute path (an earlier draft hardcoded Elder Melchor's,
which is wrong on every other host):

| Node class | Profile root |
|---|---|
| Elder Melchor (win) | `%LOCALAPPDATA%\hermes\profiles\pmoves-hermes-elder\skills` |
| any POSIX node | `$HERMES_PROFILE_ROOT/skills` (default `~/.hermes/profiles/pmoves-hermes-elder/skills`) |

Fork skills dir: `skills/<category>/<name>/SKILL.md`

Priority set to mirror first:
- `autonomous-ai-agents/hermes-agent` — the Hermes playbook
- `github/*` — all 8 (auth, code-review, issues, issue-to-pr, pr-workflow,
  repo-management, hermes-pmoves-pr-review, pmoves-lane-register)
- `devops/pmoves-*` — all 17 (activepieces, agent-zero, e2b, fleet-console,
  fleet-ops, fork-onboarding, hirag, mcp-gateway, n8n, node-bringup,
  pinokio-fork, pinokio-node, skills-cli, supabase, terminal-fleet, …)
- `media/pmoves-voice-fabric`, `productivity/pterm`, `software-development/gepeto`

## Rules
- Keep DESCRIPTION.md per category where it exists (skills.sh renders it)
- Strip absolute local paths from SKILL.md before committing (portability)
- Sync direction: profile is source of truth → fork is distribution;
  re-run after every new skill batch
