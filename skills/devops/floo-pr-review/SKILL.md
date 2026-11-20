---
name: floo-pr-review
description: "Use when reviewing open PRs for merge readiness, addressing bot review threads, or running the strict merge gate — REST-native sweep, thread reply+resolve, admin merge with local verification."
---

# floo-pr-review — REST-native PR review chain

Sanctioned chain: **validate → resolve → run-dry → chit-packet → strict** (AGNOTE4482.FlOO$ / pr-monitor skill). Runs REST-native via `gh api` when the full `make pr-monitor` toolchain isn't available locally.

## Setup

- `gh` CLI from GitHub releases (apt package is stale/missing): extract `bin/gh` from the tarball to `/usr/local/bin`
- `gh auth login --with-token` with a PAT (fine-grained: Contents + Pull Requests read/write)

## Chain

1. **Validate**: `gh pr view <n> --json state,mergeStateStatus,mergeable,statusCheckRollup,reviews,comments` per PR; `gh run list --branch <branch>` for CI. Compare failing checks against the base branch's own runs — failures identical on docs-only PRs are pre-existing base debt, not introduced.
2. **Resolve**: fetch review threads (`gh api repos/{o}/{r}/pulls/{n}/comments`), classify per pr-monitor taxonomy (missed-signal / fix-pattern / wrong-suggestion / already-addressed), verify each against actual code/data before acting.
3. **Run-dry**: fix verified issues locally in worktrees (`git worktree add .wt/<name> <branch>`; symlink `node_modules` from the main checkout), run the full suite, THEN push.
4. **Chit-packet**: encode learnings as a CGP packet (`chit.cgp.v0.2`, one constellation node per chain stage).
5. **Strict**: admin merge in dependency order (stacked PRs first: #66 → #67), `gh pr merge <n> --admin --squash`. Strict RED on remaining lanes' actionables is correct doctrine — the gate reports them; it is not yours to force green.

## Thread reply + resolve via GraphQL

Use `--input` with a JSON payload file (NOT `-f` args — multiline bodies break arg splitting):

```python
payload = {'query': 'mutation($t: ID!, $b: String!) { addPullRequestReviewThreadReply(input: {pullRequestReviewThreadId: $t, body: $b}) { comment { id } } }', 'variables': {'t': tid, 'b': body}}
subprocess.run(['gh', 'api', 'graphql', '--input', path], ...)
# then resolvePullRequestReviewThread(input: {threadId: $t})
```

Reply bodies cite the exact fix commit and local-verification evidence per thread.

## Known traps

- Stacked PRs conflict after the base merges: `git merge origin/base`, resolve, re-verify, push, then merge
- Session gaps kill background runs — always relaunch detached (`nohup setsid ... & disown`) and never run two jest parents at once (60+ workers starve CPU and corrupt results)
- Reviewers are usually right: verify their claim against code/data before dismissing — this flow's 13-thread sweep had zero wrong-suggestions
