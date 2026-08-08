<!-- markdownlint-disable -->

# Hardening Report: yykamei--block-merge-based-on-time/v5.0.29

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **yykamei--block-merge-based-on-time/v5.0.29** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A ${{ }} expression is interpolated directly inside a run: shell command. The step `run: echo pr-blocked=${{ steps.block.outputs.pr-blocked }}` injects the `steps.block.outputs.pr-blocked` context value directly into the shell command string. An attacker who can influence the action's output could inject shell metacharacters. The value should be passed via an env: variable and double-quoted in the shell.

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:22`

### unpinned-uses (severity: high)

One or more `uses:` references are pinned to a mutable branch name rather than a full 40-character commit SHA, making the workflow vulnerable to supply-chain attacks if the referenced branch is compromised. Failing references:
- `uses: yykamei/block-merge-based-on-time@main` (line 16)
- `uses: yykamei/github-workflows-metrics@main` (line 13)

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:16`
- `.github/workflows/metrics.yml:13`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and no job-level `permissions:` key on any of its jobs. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad. A top-level `permissions: {}` or specific per-job permissions blocks should be added.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/fetch-holidays.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

1. block-merge-based-on-time.yaml: Fixed script injection by moving `${{ steps.block.outputs.pr-blocked }}` into an env: variable (PR_BLOCKED) and referencing it as "$PR_BLOCKED" in the shell. Also pinned `yykamei/block-merge-based-on-time@main` to SHA `1f346b61a046e3e12703028fabb869166eafbeb3 # main`. 2. metrics.yml: Pinned `yykamei/github-workflows-metrics@main` to SHA `6d666ef8bb2dc18c5efce53f387faee1bfc81a47 # main`. 3. ci.yml: Added `permissions: {}` at the top level and specific per-job permissions (ci: contents:read; dependency-review: contents:read + pull-requests:write; auto-build-trusted: contents:write; auto-build-untrusted: contents:read). 4. fetch-holidays.yml: Added `permissions: {}` at the top level and job-level `contents: write` + `pull-requests: write` for the fetch-holidays job.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in .github/workflows/release.yml at line 44. Added sanitization of the `next` variable before writing to $GITHUB_OUTPUT: `safe_next=$(printf '%s' "$next" | tr -d '\n\r')` and then used `safe_next` in the echo statement. This prevents potential newline injection via the `inputs.version` workflow_dispatch input, even though a regex validation is also present.

