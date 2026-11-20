---
name: dependency-security-remediation
description: "Use when triaging and remediating Dependabot/npm-audit alerts across multiple manifests — wave planning, override strategy with API-compatibility checks, and per-manifest verification loops."
---

# dependency-security-remediation — wave-based alert triage

Proven on a 292-alert triage (13 critical) across 4 npm manifests + 2 pip manifests (PMOVES-ToKenism-Multi PR #71).

## Wave planning

1. Pull ALL alerts via API (Link-header pagination; the `page` param is unsupported on the Dependabot endpoint)
2. Group by package, extract `first_patched_version` from `security_advisory.vulnerabilities[]` (top-level fields are empty)
3. Rank: critical → high → count; the top 8 packages often cover ~67% of alerts
4. Wave 1 = direct-dep bumps (P0 criticals); Wave 2 = transitive overrides (P1 highs); Wave 3 = hygiene (stale files, no-patch risk acceptance)

## Override strategy — the API-compatibility trap

**An override can clear an advisory but break a consumer.** Real case: `undici ^7.30.1` override cleared the advisories but broke hardhat's compiler downloader — 7.x removed the `maxRedirections` API hardhat's fetcher uses (`InvalidArgumentError: maxRedirections is not supported`).

Correct resolution: pin to the **lowest patched line compatible with consumers** — `undici ^6.29.1` (patched ≥6.29.1, keeps hardhat's API). Always:

1. Check `first_patched_version` across ALL version lines (e.g. patches exist in both 6.x and 7.x lines)
2. Pick the line matching the consumer's API expectations
3. Run the consumer's real tests after install — not just `npm ls`

If no compatible patched line exists: remove the override, document as blocked on upstream (like `extract-zip`, `elliptic`), and rely on CI's advisory tracking.

## Hygiene wave: stale manifest files carry alerts

A duplicate `requirements-updated.txt` carried 21 pip alerts on its own. Deleting stale manifests/duplicates is often the cheapest wave — check for committed logs, generated artifacts, and old reports too.

## Verification loop (per manifest, before push)

```bash
# pmoves-nextjs: Jest + production build
npm ci --no-audit --no-fund && npx jest --silent && npm run build
# integrations: Jest
npm ci --no-audit --no-fund && npx jest --silent
# contracts/solidity: Hardhat compile + test (NOT Jest)
npm ci --no-audit --no-fund && npx hardhat compile && npx hardhat test
```

One manifest at a time; one clean jest launch at a time (duplicate pools corrupt each other and hang). Expect `npm install` output variations — verify installed versions in `package-lock.json`, not install-log summaries.

## Single-run discipline

Parallel/duplicate jest launches (two parents, 60+ workers) starve CPU and corrupt results. Kill with `pkill -9 -f processChild` + `node.*jest`, verify `remaining: 0`, then launch exactly one run per suite set.
