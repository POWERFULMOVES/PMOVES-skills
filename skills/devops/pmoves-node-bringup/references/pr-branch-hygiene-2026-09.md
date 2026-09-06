# PR-branch git-hygiene — recovery sequences that worked (2026-09-05)

From the #2905 merge battle and the #2904/#2907 review-fix passes on Elder-Melchor (Windows, git-bash, concurrent cron main-sync watcher).

## 1. BEHIND that refuses to clear — stale LOCAL branch

Symptom: `gh pr view <N> --json mergeStateStatus` says `BEHIND`, but `git merge origin/main` says "Already up to date" and `git push` says "Everything up-to-date".
Cause: the local checkout of the PR branch is itself stale (often pointing at an old main). `gh pr checkout` reuses the existing local branch without fast-forwarding.

```bash
git fetch origin <pr-branch>
git rev-parse HEAD origin/<pr-branch>   # WILL differ — that's the confirmation
git reset --hard origin/<pr-branch>     # local = remote truth
git merge origin/main -m "merge main (BEHIND resolve)"
git push
```

## 2. mergeable=UNKNOWN lag after push

GitHub recomputes mergeability asynchronously after a branch update; polls read `UNKNOWN/UNKNOWN` for 1–3+ minutes. Not a failure.

- `gh pr merge <N> --squash --auto` exits **rc=0** but silently does NOT enable auto-merge while UNKNOWN — re-run once the state computes.
- Verify auto-merge actually engaged: `gh pr view <N> --json autoMergeRequest --jq '.autoMergeRequest.mergeMethod'` (null = not enabled).
- In this session the state never computed in-window, but a later plain `gh pr merge --squash --auto` attempt went through — the PR merged. Treat UNKNOWN as retry-later, never blocked-forever.

## 3. git add -A swept workspace scratch into a PR commit

`.hermes-audit-ws/` (15k+ lines of audit JSON), `terminal_survey/` etc. live untracked in the repo tree. Even `git add -A ':!*.lock'` swept them into a docs PR commit.

Recovery (worked first try):
```bash
git reset --soft HEAD~1 && git reset -q
git add pmoves/docs/services/<file>          # explicit paths ONLY
git commit -m "..." && git push --force-with-lease
```
Confirm with `git show --stat HEAD` before pushing when in doubt.

## 4. Verify a fix commit actually carried the fix

#2907's fix commit (`c33dbe5ee`) claimed to fix "Firefly-only via up-external" — line 17 of firefly-iii/README.md still carried the original phrasing, and the seeder parenthetical still named the nonexistent `firefly_seed_sample.py` make wrapper.

After ANY fix commit:
```bash
git grep '<the exact pattern the reviewer quoted>' -- '<fixed file>'
```
Empty = carried. Non-empty = a second occurrence survived; fix it too. (Reviewers WILL re-check.)

## 5. .git/index.lock contention (cron main-sync watcher)

A background `main-sync-drift-watch` cron keeps git.exe processes alive; the lock recreates mid-commit.

Recovery that works: `tasklist | grep '^git\.exe'` → `sleep 30–45` → `rm -f .git/index.lock` → retry. Do NOT kill git blind; if a kill is needed remember `taskkill //F //IM` FAILS in this MSYS shell — use `taskkill /F /IM` or PowerShell `Stop-Process`.

## 6. Register RELEASE-pairing contract (backfill releases)

Codex P2 "Have <owner> close its own claim": the register's co-owner contract says RELEASE pairing stays with the signing owner — a co-owner has no authority to close another owner's claim, and a foreign-authored RELEASE corrupts `open_claims_in()`.

Correct backfill shape (accepted on #2960):
- Row authored as `<FILING-AGENT> RELEASE` acknowledging the delivering PR (cite its body text + merged_at + releases-after-claim count, e.g. "181 releases after the claim, none matching handoff").
- Original CLAIM annotated closed-by-evidence.
- ACK signed under the FILING agent's card, explicitly noting "no <owner> authority exercised".
- The formal owner-authored RELEASE stays owed — record it as lane remainder.

Corollary: before assuming a lane is open, grep MERGED PRs (`gh pr list --state merged --search "<scope keywords>"`) for the claimed scope — the Mavis handoff lane sat delivered-but-unreleased for 16 days.
