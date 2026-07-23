<!-- markdownlint-disable -->

# Hardening Report: yykamei--block-merge-based-on-time/v5.0.23

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **yykamei--block-merge-based-on-time/v5.0.23** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A GitHub Actions expression `${{ steps.block.outputs.pr-blocked }}` is directly interpolated inside a `run:` shell command string. The offending line is: `run: echo pr-blocked=${{ steps.block.outputs.pr-blocked }}`. This allows the value of the step output to be parsed by the shell before quoting, enabling potential command injection.

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:21`

### unpinned-uses (severity: high)

Two workflow files reference actions pinned to the mutable branch ref `@main` instead of a full 40-character commit SHA, making them vulnerable to supply-chain attacks: (1) `uses: yykamei/block-merge-based-on-time@main` in block-merge-based-on-time.yaml; (2) `uses: yykamei/github-workflows-metrics@main` in metrics.yml.

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:15`
- `.github/workflows/metrics.yml:13`

### missing-permissions (severity: medium)

Two workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs, meaning they run with the default (potentially broad) token permissions: (1) `ci.yml` — jobs `ci`, `dependency-review`, `auto-build-trusted`, and `auto-build-untrusted` all lack permissions; (2) `fetch-holidays.yml` — job `fetch-holidays` lacks permissions.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/fetch-holidays.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings: (1) script-injection in block-merge-based-on-time.yaml: moved `${{ steps.block.outputs.pr-blocked }}` to an env block as PR_BLOCKED and referenced it as $PR_BLOCKED in the run command; (2) unpinned-uses: pinned yykamei/block-merge-based-on-time@main to SHA 9457d7741e55ec4c3ad76085251c8868eea29b23 and yykamei/github-workflows-metrics@main to SHA 778d9db33ef12a32c03416e9ccd2e3f1d9af6b22; (3) missing-permissions: added job-level permissions blocks to all four jobs in ci.yml (ci: contents:read; dependency-review: contents:read + pull-requests:write; auto-build-trusted: contents:write; auto-build-untrusted: contents:read) and to the fetch-holidays job in fetch-holidays.yml (contents:read).

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in .github/workflows/release.yml at line 35. Added sanitization of the `next` variable before writing to $GITHUB_OUTPUT: `safe_next=$(printf '%s' "$next" | tr -d '\n\r')` followed by `echo "next=${safe_next}" >> "$GITHUB_OUTPUT"`. This strips any embedded newline/carriage-return characters from the untrusted workflow_dispatch `inputs.version` value before it is written to the special GitHub output file, preventing newline injection attacks.

