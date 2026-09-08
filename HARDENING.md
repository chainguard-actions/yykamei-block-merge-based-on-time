<!-- markdownlint-disable -->

# Hardening Report: yykamei--block-merge-based-on-time/v5.0.43

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **yykamei--block-merge-based-on-time/v5.0.43** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A ${{ }} expression is directly interpolated inside a run: shell command. The step `run: echo pr-blocked=${{ steps.block.outputs.pr-blocked }}` embeds the step output directly into the shell command string before the shell ever sees it. If the output contains shell metacharacters, they will be interpreted by the shell. The value should be passed via an env: variable and then referenced as a quoted shell variable instead.

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:21`

### unpinned-uses (severity: high)

Two workflow steps use mutable branch refs instead of pinned full-length SHA commit hashes, making them vulnerable to supply-chain attacks if the referenced branch is compromised: (1) `uses: yykamei/block-merge-based-on-time@main` in block-merge-based-on-time.yaml; (2) `uses: yykamei/github-workflows-metrics@main` in metrics.yml. These should be pinned to a specific 40-character commit SHA.

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:15`
- `.github/workflows/metrics.yml:12`

### missing-permissions (severity: medium)

Two workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without explicit permissions, the GITHUB_TOKEN is granted its default (potentially broad) permissions. (1) ci.yml — four jobs (ci, dependency-review, auto-build-trusted, auto-build-untrusted) with no permissions declared. (2) fetch-holidays.yml — one job (fetch-holidays) with no permissions declared. Both files should declare minimal required permissions.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/fetch-holidays.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

1. script-injection (block-merge-based-on-time.yaml line 21): Moved `${{ steps.block.outputs.pr-blocked }}` out of the run: shell string into an env: block as PR_BLOCKED, then referenced it as `$PR_BLOCKED` in the shell command. 2. unpinned-uses: Pinned `yykamei/block-merge-based-on-time@main` → SHA c59434d125febefb7842980ec9dbb547264320f0 in block-merge-based-on-time.yaml; pinned `yykamei/github-workflows-metrics@main` → SHA ee4cc54ed556a638ebd9151d077d502c94980c0e in metrics.yml. 3. missing-permissions: Added top-level `permissions: {}` to ci.yml and fetch-holidays.yml, plus job-level minimal permissions: ci job gets `contents: read`; dependency-review gets `contents: read, pull-requests: read`; auto-build-trusted gets `contents: write, pull-requests: read`; auto-build-untrusted gets `contents: read, pull-requests: read`; fetch-holidays gets `contents: write, pull-requests: write`.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

In .github/workflows/release.yml, added sanitization before writing the `next` version variable to $GITHUB_OUTPUT. The value is now passed through `printf '%s' "$next" | tr -d '\n\r'` to strip newline/carriage-return characters before the echo write, satisfying the required sanitization pipeline.

