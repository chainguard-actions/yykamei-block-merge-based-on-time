<!-- markdownlint-disable -->

# Hardening Report: yykamei--block-merge-based-on-time/v5.0.25

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **yykamei--block-merge-based-on-time/v5.0.25** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two workflow files reference actions using mutable branch names (`@main`) instead of pinned full-length commit SHAs, making them vulnerable to supply-chain attacks if the referenced repository is compromised.

- `.github/workflows/block-merge-based-on-time.yaml` line 15: `uses: yykamei/block-merge-based-on-time@main`
- `.github/workflows/metrics.yml` line 12: `uses: yykamei/github-workflows-metrics@main`

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:15`
- `.github/workflows/metrics.yml:12`

### script-injection (severity: high)

Sub-rule (a): A `run:` block in `.github/workflows/block-merge-based-on-time.yaml` directly interpolates a `${{ ... }}` expression into the shell command string. The offending line is:

  `run: echo pr-blocked=${{ steps.block.outputs.pr-blocked }}`

The expression `${{ steps.block.outputs.pr-blocked }}` is substituted into the shell command before the shell ever sees it, allowing any attacker-controlled value in that output to inject arbitrary shell commands. It should be passed via an `env:` variable and double-quoted instead.

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:21`

### missing-permissions (severity: medium)

Two workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs, meaning they run with the default (potentially broad) repository permissions:

- `.github/workflows/ci.yml`: No top-level or job-level permissions defined across four jobs (ci, dependency-review, auto-build-trusted, auto-build-untrusted).
- `.github/workflows/fetch-holidays.yml`: No top-level or job-level permissions defined for the fetch-holidays job.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/fetch-holidays.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings:
1. unpinned-uses: Pinned yykamei/block-merge-based-on-time@main → @9457d7741e55ec4c3ad76085251c8868eea29b23 in block-merge-based-on-time.yaml, and yykamei/github-workflows-metrics@main → @778d9db33ef12a32c03416e9ccd2e3f1d9af6b22 in metrics.yml.
2. script-injection: Moved ${{ steps.block.outputs.pr-blocked }} out of the run: shell string into an env: block (PR_BLOCKED) and referenced it as "$PR_BLOCKED" in the echo command.
3. missing-permissions: Added top-level `permissions: {}` and job-level minimal permissions to ci.yml (contents: read for ci/auto-build-untrusted; contents: read + pull-requests: write for dependency-review; contents: write + pull-requests: read for auto-build-trusted) and fetch-holidays.yml (contents: write + pull-requests: write for fetch-holidays).

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

In .github/workflows/release.yml, added sanitization of the `next` variable before writing to $GITHUB_OUTPUT. The fix introduces `safe_next=$(printf '%s' "$next" | tr -d '\n\r')` and then writes `echo "next=${safe_next}" >> "$GITHUB_OUTPUT"` instead of writing `$next` directly. This strips any embedded newlines or carriage returns from the user-controlled input before it reaches the special environment file, preventing header injection attacks.

