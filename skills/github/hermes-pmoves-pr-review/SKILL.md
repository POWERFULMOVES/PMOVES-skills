---
name: hermes-pmoves-pr-review
description: "PMOVES.AI Hermes Agent PR review workflow — review, fix, and merge PRs with Hermes-native capabilities."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [github, code-review, pull-requests, pmoves, hermes]
    related_skills: [github-code-review, github-pr-workflow]
---

# PMOVES Hermes PR Review Workflow

End-to-end PR review for the PMOVES.AI repo, combining the github-code-review
and github-pr-workflow skills with Hermes-specific capabilities (subagent
delegation, MCP tools, persistent memory, CHIT trail signing).

## Review Ecosystem

Multiple automated reviewers operate on PMOVES.AI PRs. Know who they are:

| Reviewer | What it checks | When |
|----------|---------------|------|
| **Codex** (`chatgpt-codex-connector[bot]`) | Logic, correctness, API usage, edge cases | On PR open + `@codex review` |
| **CodeRabbit** (`coderabbitai[bot]`) | Diff quality, best practices, test coverage | On PR open (slow, ~2 min) |
| **Claude Code** (CI `claude-review`) | Repo-specific conventions, security, patterns | CI pipeline |
| **CI checks** | `python-tests`, `hardening-validation`, `merge-gate`, `submodule-gitlink-gate`, `verify`, `village-gate` | PR push |
| **Hermes Agent** (this skill) | Fleet context, CHIT compliance, cross-PR patterns, NATS/MCP wiring, profile/bootstrap correctness | Manual or delegated |

### Hermes's Unique Value Add

Hermes reviews things the bots can't:
- **Cross-PR patterns**: Has memory of what previous PRs did, catches duplication
- **Fleet bootstrap correctness**: Knows the crush-pmoves / hermes-pmoves pattern
- **CHIT trail compliance**: Can verify signing patterns match fleet conventions
- **MCP wiring**: Understands Docker MCP, Cipher MCP, NATS bridge configs
- **Profile/config consistency**: Knows HERMES_HOME resolution, Windows vs Linux paths
- **Submodule integrity**: Can run `git submodule status` and verify gitlinks
- **Session history**: Can `session_search` for past decisions on similar code

## Workflow

### Phase 1: Gather Context (batch these calls)

```bash
# PR metadata
gh pr view <N> --repo POWERFULMOVES/PMOVES.AI --json title,body,author,headRefName,baseRefName,additions,deletions,changedFiles,mergeable,mergeStateStatus,reviewDecision

# Changed files with line counts
gh api repos/POWERFULMOVES/PMOVES.AI/pulls/<N>/files --jq '.[] | "\(.status) +\(.additions) -\(.deletions) \(.filename)"'

# CI status
gh pr checks <N> --repo POWERFULMOVES/PMOVES.AI

# Existing bot reviews
gh api repos/POWERFULMOVES/PMOVES.AI/pulls/<N>/reviews --jq '.[] | "\(.user.login): [\(.state)]"'
gh api repos/POWERFULMOVES/PMOVES.AI/pulls/<N>/comments --jq '.[] | "\(.user.login) @ \(.path):\(.line // .original_line)\n\(.body[:200])"'
```

### Phase 2: Review the Diff

```bash
# Full diff
gh pr diff <N> --repo POWERFULMOVES/PMOVES.AI

# Or checkout locally for deeper analysis
gh pr checkout <N> --repo POWERFULMOVES/PMOVES.AI
git diff main...HEAD --stat
```

Apply the review checklist (see github-code-review skill):
1. **Correctness** — edge cases, error paths, null handling
2. **Security** — no secrets, input validation, auth checks
3. **PMOVES conventions** — conventional commits, submodule workflow, CHIT signing
4. **Code quality** — naming, DRY, complexity
5. **Testing** — new code paths tested?
6. **Fleet impact** — does this affect other nodes? Profile configs? MCP wiring?

### Phase 3: Cross-Reference

Use Hermes capabilities the bots don't have:
- `session_search` — "Have we seen this pattern before? Was there a decision?"
- `search_files` — "Is this duplicated elsewhere in the repo?"
- Memory — "Did we agree on a convention for this?"

### Phase 4: Fix Issues (if any)

If the review finds fixable issues, fix them directly on the PR branch:

```bash
gh pr checkout <N> --repo POWERFULMOVES/PMOVES.AI
# Apply fixes with patch/write_file
git add <files>
git commit -m "fix(<scope>): address review feedback — <what>"
git push
```

### Phase 5: Submit Review

Post a structured review comment using the template format:

```bash
gh pr comment <N> --repo POWERFULMOVES/PMOVES.AI --body "$(cat <<'EOF'
## Hermes Agent Review

**Verdict: [Approved ✅ | Changes Requested 🔴 | Reviewed 💬]**

### Findings
...

### Cross-References
- Related: #[PR numbers]
- Memory: [relevant convention or decision]

### Fleet Impact
- [Node/profile/MCP implications if any]

---
*Reviewed by Hermes Agent (pmoves-hermes-elder)*
EOF
)"
```

### Phase 6: Merge

Prerequisites (branch protection requires all):
1. All CI checks pass (merge-gate, python-tests, hardening-validation, verify, submodule-gitlink-gate)
2. 1 approving review (or admin merge)

```bash
# Check mergeable state
gh pr view <N> --json mergeable,mergeStateStatus

# Squash merge (cleanest for feature branches)
gh pr merge <N> --repo POWERFULMOVES/PMOVES.AI --squash --delete-branch

# Admin merge if review requirement blocks but all CI passes
gh api repos/POWERFULMOVES/PMOVES.AI/pulls/<N>/merge --method PUT -f merge_method=squash

# Switch back to main
git checkout main && git pull --no-recurse-submodules origin main
```

## Common Merge Blockers

| Blocker | Cause | Fix |
|---------|-------|-----|
| `DIRTY` | Merge conflict with main | `git merge origin/main`, resolve, push |
| `BLOCKED` | Missing review or failing CI | Fix CI or admin-merge if all green |
| `BEHIND` | Main moved ahead | `git merge origin/main`, push |
| `submodule-gitlink-gate` fail | Submodule pointer mismatch | Run `git submodule update --init`, verify pointers |

## Hermes-Native Review Capabilities

### Delegate Deep Analysis

For large PRs, spawn a subagent to review specific files:

```
delegate_task(
  goal="Review src/auth/ changes in PR #N for security issues",
  context="PR #N adds JWT auth. Focus on token validation and injection risks. The diff is at: gh pr diff N"
)
```

### MCP-Powered Verification

Use Docker MCP and Cipher MCP tools to verify claims in the PR:
- Check if referenced Docker images exist
- Query Neo4j/Cipher for related context
- Verify Hostinger VPS configs

### Session History Integration

Before reviewing, check if similar work was done before:

```
session_search(query="auth JWT token PR", limit=3)
```

This catches patterns like "we already tried this approach and reverted it" that bots miss.

## PMOVES-Specific Review Rules

1. **Secrets**: Check for `YOUR_*_TOKEN_HERE` placeholders that should be real tokens
2. **Submodule pointers**: Verify `.gitmodules` branch matches actual submodule branch
3. **CHIT signing**: New scripts should resolve CHIT passphrase from secrets funnel
4. **Docker context**: Scripts using Docker should handle `DOCKER_HOST` on Windows
5. **HERMES_HOME**: Scripts must handle the profile-dir vs base-dir ambiguity
6. **Make targets**: New scripts should have a corresponding `make` target in `mcp-toolkit.mk`
7. **Conventional commits**: `feat(scope):`, `fix(scope):`, `docs(scope):` — no exceptions
8. **Cross-fleet**: Consider impact on other nodes (Spark, 5090, z890, B850, KVMs)
