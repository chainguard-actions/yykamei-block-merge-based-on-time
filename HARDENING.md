<!-- markdownlint-disable -->

# Hardening Report: yykamei--block-merge-based-on-time/v5.0.37

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **yykamei--block-merge-based-on-time/v5.0.37** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A ${{ }} expression is interpolated directly inside a run: shell command. The step `run: echo pr-blocked=${{ steps.block.outputs.pr-blocked }}` embeds the `steps.block.outputs.pr-blocked` context value directly into the shell command string before the shell ever sees it. An attacker who can influence that output value could inject arbitrary shell commands. The value should be passed via an env: variable and then referenced as a quoted shell variable (e.g., `echo "pr-blocked=$PR_BLOCKED"`).

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:18`

### github-env-injection (severity: high)

The 'Decide the next version' step in release.yml writes the variable `next` (derived from `$INPUT_VERSION`, which is set from `${{ inputs.version }}` — a user-controlled workflow_dispatch input) to `$GITHUB_OUTPUT` via `echo "next=${next}" >> "$GITHUB_OUTPUT"` without first sanitizing newline characters. An attacker supplying a crafted version string containing newlines could inject arbitrary key=value pairs into GITHUB_OUTPUT. The fix is to sanitize: `safe=$(printf '%s' "$next" | tr -d '\n\r')` and then `echo "next=${safe}" >> "$GITHUB_OUTPUT"`.

Locations:

- `.github/workflows/release.yml:36`

### unpinned-uses (severity: high)

Two workflow files reference actions by mutable branch names instead of pinned 40-character commit SHAs, making them vulnerable to supply-chain attacks if the referenced branch is compromised:
- `.github/workflows/block-merge-based-on-time.yaml`: `uses: yykamei/block-merge-based-on-time@main`
- `.github/workflows/metrics.yml`: `uses: yykamei/github-workflows-metrics@main`
These should be pinned to a full SHA, e.g. `uses: yykamei/block-merge-based-on-time@<40-char-sha> # main`.

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:13`
- `.github/workflows/metrics.yml:9`

### missing-permissions (severity: medium)

Two workflow files have no top-level `permissions:` block and none of their jobs define job-level `permissions:` blocks. Without explicit permissions, GitHub Actions defaults to the repository's default token permissions (which may be broad). Both files should declare a top-level `permissions: {}` and grant only the minimal scopes required by each job.
- `ci.yml`: 4 jobs (ci, dependency-review, auto-build-trusted, auto-build-untrusted) — none have permissions.
- `fetch-holidays.yml`: 1 job (fetch-holidays) — no permissions declared.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/fetch-holidays.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all four findings: (1) script-injection in block-merge-based-on-time.yaml: moved steps.block.outputs.pr-blocked into env var PR_BLOCKED; (2) github-env-injection in release.yml: sanitized 'next' variable with tr -d '\n\r' before writing to GITHUB_OUTPUT; (3) unpinned-uses: pinned yykamei/block-merge-based-on-time@main to SHA 481754706c6f2ee8ba588d603cc347a08037bee1 and yykamei/github-workflows-metrics@main to SHA 4ff4a6bba410696d7d450c7b7e4f5eea367a0514; (4) missing-permissions: added top-level permissions: {} and minimal job-level permissions blocks to ci.yml (contents:read for ci/auto-build-trusted/auto-build-untrusted, contents:read + pull-requests:write for dependency-review) and fetch-holidays.yml (contents:read for fetch-holidays job).

