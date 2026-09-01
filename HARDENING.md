<!-- markdownlint-disable -->

# Hardening Report: yykamei--block-merge-based-on-time/v5.0.41

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **yykamei--block-merge-based-on-time/v5.0.41** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: A ${{ ... }} expression is directly interpolated inside a run: shell command. The step `run: echo pr-blocked=${{ steps.block.outputs.pr-blocked }}` embeds `steps.block.outputs.pr-blocked` (a steps.*.outputs.* context value) directly into the shell command string before the shell ever sees it. An attacker who can influence the action output could inject shell metacharacters. The value should be passed via an env: variable and the expansion double-quoted instead.

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:22`

### github-env-injection (severity: high)

The 'Decide the next version' step writes a value derived from the user-controlled `inputs.version` input to $GITHUB_OUTPUT without sanitization. The flow is: `inputs.version` → env var `INPUT_VERSION` → shell variable `next` → `echo "next=${next}" >> "$GITHUB_OUTPUT"`. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before the write, allowing a newline-injection attack that could poison subsequent steps' environment via GITHUB_OUTPUT.

Locations:

- `.github/workflows/release.yml:43`

### unpinned-uses (severity: high)

One or more `uses:` references are pinned to a mutable branch name rather than a full 40-character commit SHA, making the workflow vulnerable to supply-chain attacks if the referenced branch is compromised. Failing references: `yykamei/block-merge-based-on-time@main` (line 16) and `yykamei/github-workflows-metrics@main` (line 12 in metrics.yml).

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:16`
- `.github/workflows/metrics.yml:12`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and none of its jobs (ci, dependency-review, auto-build-trusted, auto-build-untrusted) define a job-level `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (write-all by default for many repositories). Each job should declare the minimal permissions it requires.

Locations:

- `.github/workflows/ci.yml:1`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and the single job `fetch-holidays` has no job-level `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad. The job should declare the minimal permissions it requires (e.g. `contents: write` for pushing commits).

Locations:

- `.github/workflows/fetch-holidays.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all 5 findings across 5 workflow files:
1. block-merge-based-on-time.yaml: Pinned yykamei/block-merge-based-on-time@main to SHA 57a7613c9af8c4c5cd292d8835355a83d09619c7; moved steps.block.outputs.pr-blocked into env var PR_BLOCKED to fix script-injection.
2. release.yml: Added printf/tr sanitization before writing 'next' to GITHUB_OUTPUT to prevent newline injection.
3. metrics.yml: Pinned yykamei/github-workflows-metrics@main to SHA 3e4b2ad0dca9abb9631bd8a9d7829f3c7990e830.
4. ci.yml: Added minimal permissions blocks to all four jobs (ci: contents:read; dependency-review: contents:read + pull-requests:write; auto-build-trusted: contents:write; auto-build-untrusted: contents:read).
5. fetch-holidays.yml: Added permissions block (contents:write, pull-requests:write) to the fetch-holidays job.

