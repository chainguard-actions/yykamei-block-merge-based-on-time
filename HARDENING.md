<!-- markdownlint-disable -->

# Hardening Report: yykamei--block-merge-based-on-time/v5.0.22

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **yykamei--block-merge-based-on-time/v5.0.22** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A ${{ }} expression is directly interpolated inside a run: shell command. The line `echo pr-blocked=${{ steps.block.outputs.pr-blocked }}` injects the step output value directly into the shell command string before the shell ever sees it, allowing an attacker who can influence that output to inject arbitrary shell commands. Fix by routing through an env: variable and double-quoting: `env: { PR_BLOCKED: "${{ steps.block.outputs.pr-blocked }}" }` then `echo "pr-blocked=$PR_BLOCKED"`.

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:22`

### unpinned-uses (severity: high)

Two workflow files reference actions by mutable branch name (@main) rather than a full 40-character commit SHA. This means the action code can change at any time without notice, enabling supply-chain attacks. Failing references:
- `.github/workflows/block-merge-based-on-time.yaml`: `yykamei/block-merge-based-on-time@main`
- `.github/workflows/metrics.yml`: `yykamei/github-workflows-metrics@main`
Pin each to a specific commit SHA, e.g. `uses: yykamei/block-merge-based-on-time@<40-char-sha> # main`.

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:16`
- `.github/workflows/metrics.yml:13`

### missing-permissions (severity: medium)

Two workflow files have no top-level `permissions:` key and no job-level `permissions:` keys on any of their jobs. Without an explicit permissions block, GitHub grants the default token permissions (which may include write access to contents, pull-requests, etc.), violating the principle of least privilege. Add a top-level `permissions: {}` block and grant only the specific scopes each job requires.
- `ci.yml`: jobs `ci`, `dependency-review`, `auto-build-trusted`, `auto-build-untrusted` all lack permissions.
- `fetch-holidays.yml`: job `fetch-holidays` lacks permissions.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/fetch-holidays.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings: (1) script-injection in block-merge-based-on-time.yaml by moving ${{ steps.block.outputs.pr-blocked }} into an env: variable PR_BLOCKED and referencing it as "$PR_BLOCKED" in the run: command; (2) unpinned-uses by pinning yykamei/block-merge-based-on-time@main to SHA 6f9af9cb2267234702a351ac04edb35abc4fb9c8 and yykamei/github-workflows-metrics@main to SHA 778d9db33ef12a32c03416e9ccd2e3f1d9af6b22; (3) missing-permissions by adding top-level permissions: {} and job-level permissions blocks to ci.yml (contents: read for ci/auto-build-trusted/auto-build-untrusted, contents: read + pull-requests: write for dependency-review) and fetch-holidays.yml (contents: read for fetch-holidays job).

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in .github/workflows/release.yml at line 35. In the 'Decide the next version' step, added sanitization before writing to $GITHUB_OUTPUT: `safe_next=$(printf '%s' "$next" | tr -d '\n\r')` and then `echo "next=${safe_next}" >> "$GITHUB_OUTPUT"`. This strips any embedded newline or carriage-return characters from the user-controlled INPUT_VERSION value before it is written to the GitHub output context, preventing injection of arbitrary key-value pairs.

