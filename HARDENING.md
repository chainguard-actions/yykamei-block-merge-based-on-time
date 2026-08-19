<!-- markdownlint-disable -->

# Hardening Report: yykamei--block-merge-based-on-time/v5.0.19

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **yykamei--block-merge-based-on-time/v5.0.19** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A ${{ }} expression is directly interpolated inside a run: shell command. The step `run: echo pr-blocked=${{ steps.block.outputs.pr-blocked }}` interpolates `steps.block.outputs.pr-blocked` directly into the shell command string before the shell ever sees it, allowing an attacker who can influence that output value to inject arbitrary shell commands.

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:22`

### github-env-injection (severity: high)

The 'Decide the next version' step in release.yml writes `echo "next=${next}" >> "$GITHUB_OUTPUT"` where `next` is derived from the env var `INPUT_VERSION`, which is set to `${{ inputs.version }}` (a workflow_dispatch user-controlled input). The value is not sanitized with `printf '%s' ... | tr -d '\n\r'` before being written to $GITHUB_OUTPUT, allowing newline injection that could poison subsequent step outputs or environment variables.

Locations:

- `.github/workflows/release.yml:44`

### unpinned-uses (severity: high)

Two workflow files reference actions by mutable branch name (`@main`) rather than a pinned 40-character commit SHA. This means the action code can change at any time without notice, enabling supply-chain attacks. Failing references: `yykamei/block-merge-based-on-time@main` and `yykamei/github-workflows-metrics@main`.

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:16`
- `.github/workflows/metrics.yml:12`

### missing-permissions (severity: medium)

ci.yml has no top-level `permissions:` key and none of its four jobs (ci, dependency-review, auto-build-trusted, auto-build-untrusted) define job-level permissions. Without explicit permissions, the workflow inherits the repository default (typically `contents: write` for private repos or the organization default), granting broader access than necessary. This is especially risky given the workflow triggers on `pull_request_target`.

Locations:

- `.github/workflows/ci.yml:1`

### missing-permissions (severity: medium)

fetch-holidays.yml has no top-level `permissions:` key and its only job (`fetch-holidays`) has no job-level `permissions:` block. The workflow inherits the repository default permissions, which may be broader than required.

Locations:

- `.github/workflows/fetch-holidays.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all 5 findings across 5 workflow files:
1. block-merge-based-on-time.yaml: Moved steps.block.outputs.pr-blocked into env block (PR_BLOCKED) to fix script-injection; pinned yykamei/block-merge-based-on-time@main to SHA 3253a9329289117ba11c13e5cdbfee98778740ab.
2. metrics.yml: Pinned yykamei/github-workflows-metrics@main to SHA fc84095f180148ab40840107fa1cb0809dcbdfd1.
3. release.yml: Added printf/tr sanitization before writing 'next' to GITHUB_OUTPUT to prevent newline injection from user-controlled inputs.version.
4. ci.yml: Added top-level permissions: {} and per-job permissions (contents: read for ci/auto-build-trusted/auto-build-untrusted; contents: read + pull-requests: read for dependency-review).
5. fetch-holidays.yml: Added top-level permissions: {} and job-level permissions: contents: read.

