<!-- markdownlint-disable -->

# Hardening Report: yykamei--block-merge-based-on-time/v5.0.38

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **yykamei--block-merge-based-on-time/v5.0.38** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: A ${{ }} expression is directly interpolated inside a run: shell command string. The line `run: echo pr-blocked=${{ steps.block.outputs.pr-blocked }}` embeds the step output directly into the shell command before the shell ever sees it, allowing an attacker who can influence that output to inject arbitrary shell commands.

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:21`

### github-env-injection (severity: high)

The env var INPUT_VERSION is sourced from ${{ inputs.version }} (a workflow_dispatch user-controlled input) and then written to $GITHUB_OUTPUT via `echo "next=${next}" >> "$GITHUB_OUTPUT"` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker-supplied newline in the version input could inject additional key=value pairs into GITHUB_OUTPUT.

Locations:

- `.github/workflows/release.yml:37`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and none of its jobs (ci, dependency-review, auto-build-trusted, auto-build-untrusted) define job-level permissions. Without explicit permissions, the workflow inherits the repository default (typically write-all for private repos), granting excessive access.

Locations:

- `.github/workflows/ci.yml:1`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and the fetch-holidays job has no job-level permissions block. Without explicit permissions, the workflow inherits the repository default, granting excessive access.

Locations:

- `.github/workflows/fetch-holidays.yml:1`

### unpinned-uses (severity: high)

The following `uses:` reference is pinned to a mutable branch ref (`@main`) rather than a full 40-character commit SHA, making it vulnerable to supply-chain attacks if the referenced repository is compromised: `uses: yykamei/block-merge-based-on-time@main` (line 15).

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:15`

### unpinned-uses (severity: high)

The following `uses:` reference is pinned to a mutable branch ref (`@main`) rather than a full 40-character commit SHA, making it vulnerable to supply-chain attacks if the referenced repository is compromised: `uses: yykamei/github-workflows-metrics@main` (line 13).

Locations:

- `.github/workflows/metrics.yml:13`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, missing-permissions, unpinned-uses

**Notes:**

Fixed all 6 findings across 5 workflow files:
1. block-merge-based-on-time.yaml: (a) Pinned yykamei/block-merge-based-on-time@main to SHA 195ac424703c965fc550a343dfdf90aab879c017; (b) Fixed script-injection by moving ${{ steps.block.outputs.pr-blocked }} into an env block as PR_BLOCKED.
2. release.yml: Fixed github-env-injection by sanitizing the 'next' variable with `printf '%s' "${next}" | tr -d '\n\r'` before writing to GITHUB_OUTPUT.
3. ci.yml: Added top-level `permissions: {}` and job-level permissions (contents:read for ci/auto-build-trusted/auto-build-untrusted; contents:read + pull-requests:read for dependency-review).
4. fetch-holidays.yml: Added top-level `permissions: {}` and job-level `permissions: contents: read`.
5. metrics.yml: Pinned yykamei/github-workflows-metrics@main to SHA 4ff4a6bba410696d7d450c7b7e4f5eea367a0514.

