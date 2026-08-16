<!-- markdownlint-disable -->

# Hardening Report: yykamei--block-merge-based-on-time/v5.0.31

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **yykamei--block-merge-based-on-time/v5.0.31** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A ${{ }} expression is directly interpolated inside a run: shell command. Line 22 of block-merge-based-on-time.yaml contains: `run: echo pr-blocked=${{ steps.block.outputs.pr-blocked }}`. The steps.*.outputs.* context is workflow-controllable and flows through YAML template substitution before the shell sees it, making this a script injection risk. The value should be passed via an env: variable and then referenced as a quoted shell variable instead.

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:22`

### unpinned-uses (severity: high)

Two workflow files reference actions by mutable branch names rather than full 40-character commit SHAs, making them vulnerable to supply-chain attacks if the referenced branch is compromised:
- `.github/workflows/block-merge-based-on-time.yaml` line 16: `uses: yykamei/block-merge-based-on-time@main` (branch ref)
- `.github/workflows/metrics.yml` line 13: `uses: yykamei/github-workflows-metrics@main` (branch ref)
These should be pinned to a full SHA digest, e.g. `uses: yykamei/block-merge-based-on-time@<40-char-sha> # main`.

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:16`
- `.github/workflows/metrics.yml:13`

### missing-permissions (severity: medium)

Two workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs, meaning the GITHUB_TOKEN is granted the default (potentially broad) permissions:
- `.github/workflows/ci.yml`: contains 4 jobs (ci, dependency-review, auto-build-trusted, auto-build-untrusted) with no permissions declared at any level.
- `.github/workflows/fetch-holidays.yml`: contains 1 job (fetch-holidays) with no permissions declared at any level.
Each file should declare a top-level `permissions:` block with the minimal required scopes, or add per-job `permissions:` blocks.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/fetch-holidays.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings:
1. script-injection (block-merge-based-on-time.yaml line 22): Moved `${{ steps.block.outputs.pr-blocked }}` into an `env:` block as `PR_BLOCKED` and referenced it as `$PR_BLOCKED` in the shell command.
2. unpinned-uses: Pinned `yykamei/block-merge-based-on-time@main` → SHA `e6f658cae691da8bb36181835b7c669c024a487f` and `yykamei/github-workflows-metrics@main` → SHA `44ee33cf23b5ff6e90dcf3f886376b402dee220c`.
3. missing-permissions: Added `permissions: {}` at top level and per-job minimal permissions to ci.yml (contents: read for ci/auto-build-trusted/auto-build-untrusted; contents: read + pull-requests: read for dependency-review) and fetch-holidays.yml (contents: read for fetch-holidays job; the actual write operations use a GitHub App token, not GITHUB_TOKEN).

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in .github/workflows/release.yml at line 44. Added sanitization of the `next` variable before writing to $GITHUB_OUTPUT: `safe_next=$(printf '%s' "$next" | tr -d '\n\r')` followed by `echo "next=${safe_next}" >> "$GITHUB_OUTPUT"`. This ensures newlines and carriage returns are stripped from the untrusted `inputs.version`-derived value before it is written to the special environment file.

