<!-- markdownlint-disable -->

# Hardening Report: yykamei--block-merge-based-on-time/v5.0.34

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **yykamei--block-merge-based-on-time/v5.0.34** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: A `run:` block directly interpolates a `${{ ... }}` expression inside the shell command string. The step `run: echo pr-blocked=${{ steps.block.outputs.pr-blocked }}` embeds `steps.block.outputs.pr-blocked` directly into the shell command without going through an env var. Any `${{ ... }}` expression inside a `run:` block is a script-injection risk because the value is substituted into the shell command before the shell parses it, allowing metacharacter injection.

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:21`

### github-env-injection (severity: high)

The `run:` block in the 'Decide the next version' step writes the variable `next` (derived from `$INPUT_VERSION`, which is set from `inputs.version` — a workflow_dispatch user-controlled input) to `$GITHUB_OUTPUT` via `echo "next=${next}" >> "$GITHUB_OUTPUT"` without the required sanitization step (`printf '%s' "$next" | tr -d '\n\r'`). An attacker-supplied version string containing newlines could inject arbitrary key=value pairs into GITHUB_OUTPUT.

Locations:

- `.github/workflows/release.yml:45`

### unpinned-uses (severity: high)

One or more `uses:` references are pinned to a mutable branch ref (`@main`) rather than an immutable 40-character commit SHA. Mutable refs can be silently updated to point to malicious code. Failing references:
- `uses: yykamei/block-merge-based-on-time@main` (line 15)
- `uses: yykamei/github-workflows-metrics@main` (line 12 in metrics.yml)

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:15`
- `.github/workflows/metrics.yml:12`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and none of its jobs define job-level `permissions:`. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (e.g., write access to contents and pull requests). All 4 jobs (ci, dependency-review, auto-build-trusted, auto-build-untrusted) lack permissions blocks.

Locations:

- `.github/workflows/ci.yml:1`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and the single job `fetch-holidays` has no job-level `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad.

Locations:

- `.github/workflows/fetch-holidays.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all 5 findings across 5 workflow files:
1. block-merge-based-on-time.yaml: Moved ${{ steps.block.outputs.pr-blocked }} to env var PR_BLOCKED (script-injection fix) and pinned yykamei/block-merge-based-on-time@main to SHA 05653dc8500f28d8079d59d7c7c41b868e37a8dc (unpinned-uses fix).
2. release.yml: Added printf/tr sanitization before writing 'next' to GITHUB_OUTPUT to prevent newline injection from user-controlled inputs.version (github-env-injection fix).
3. metrics.yml: Pinned yykamei/github-workflows-metrics@main to SHA 44ee33cf23b5ff6e90dcf3f886376b402dee220c (unpinned-uses fix).
4. ci.yml: Added top-level permissions: {} and job-level permissions blocks for all 4 jobs with minimal required permissions (missing-permissions fix).
5. fetch-holidays.yml: Added top-level permissions: {} and job-level permissions: contents: read for the fetch-holidays job (missing-permissions fix).

