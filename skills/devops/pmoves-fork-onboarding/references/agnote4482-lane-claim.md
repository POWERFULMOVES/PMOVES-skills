# AGNOTE4482 lane claiming — row grammar and gates

Register: `pmoves/docs/AGENTS/AGNOTE4482PHI.t1.md` (append-only, 600+ rows). Gateway doc: `pmoves/docs/AGENTS/AGNOTE4482.md`. Related: `AGNOTE4482_SIGNOFF_CHECKLIST.md`, `KRISS_KROSS_ACCORD.md` (rail strategy), `THREE_BODY_DOCTRINE.md`.

## Row grammar (enforced by parsers)

```
- `<ISO_TIMESTAMP>` <KIND> `<OWNER-ID>` [branch: `<branch>`] [· **TTL 72h (expires <ts>)**] [· co-owners: `<ID>` (<contribution>), ...] · scope: <prose>
```

- `<KIND>`: CLAIM, RELEASE, CLAIM+RELEASE, UPDATE, REVIEW, HANDOFF
- Exactly ONE backticked owner; identity must resolve in `pmoves/config/identity_vocabulary.yaml` (aliases fold, undeclared = verify failure)
- `co-owners:` requires the `:` — backticks delimit ID, parenthetical after is contribution; folds through the vocabulary
- Timestamp MUST be ≤ commit author time (read the clock at COMMIT time, not draft time — drafting always postdates)
- Footer HTML comment: `<!-- GRAPHITI_MARK: <IDENTITY>::<LANE>-CLAIM::<date> -->`

## Gates to run before committing
1. `PYTHONPATH=. python pmoves/tools/identity_lineage.py --verify` → `identity lineage: clean`, exit 0
2. Collision gate (reads JSON payload on stdin):
   ```bash
   cat > /tmp/claim-gate-test.json <<'JSON'
   {"tool_name": "Write", "tool_input": {"file_path": "pmoves/docs/AGENTS/AGNOTE4482PHI.t1.md", "content": "- `<ts>` CLAIM `<ID>` branch: `<branch>` · scope: ..."}}
   JSON
   python .claude/hooks/governance/claim-collision-pre.py < /tmp/claim-gate-test.json; echo exit=$?
   ```
   exit 0 = clear; exit 2 = undeclared squat (blocked); exit 0 + `permissionDecision: "ask"` = unilateral co-owner declaration.

## Three-Body requirement
Each row names delivery (signer), control (reviewer/operator), memory (trail/CHIT). Sign the CHIT trail too:
`PYTHONPATH=. python pmoves/tools/sign_trail.py --agent-id hermes-agent --summary "..." --phase "..."` (this laptop has no `make` — call the tool directly; card 00000000-0000-4000-8000-000000000037).

## Release
Append a RELEASE row on the same branch when the lane closes; pairing is keyed on the signing owner (co-owners cannot close someone else's claim).

## PR landing pitfalls learned from this register (2026-09)

- **Merge conflict = silent CI bypass**: `mergeable_state: dirty` prevents GitHub computing `refs/pull/N/merge`, so NO pull_request-triggered workflow dispatches. Required contexts are ABSENT, not failing — a "is anything red?" scan reads clean on a stuck PR. Recorded by B850-CLAUDE 2026-09-02 against #2858.
- **Conflicted register PRs merge as UNION, never pick-a-side**: the register is append-only; taking one side deletes another node's claim rows. Keep main's block contiguous-first, append the lane's rows after; prove zero deletions via insertions/deletions against all three refs (merge-base, main, head).
- **Register file grep**: ripgrep treats it as binary (very long lines) — use `grep -a` or python when scanning rows.
- **Pushes race on this repo**: many nodes push concurrently. On push rejection: `git fetch origin`, `git pull --rebase`, then RE-VERIFY submodule gitlinks and `.gitmodules` registrations survived the rebase (they can silently drop to unregistered state).
