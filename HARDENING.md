<!-- markdownlint-disable -->

# Hardening Report: yykamei--block-merge-based-on-time/v5.0.32

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **yykamei--block-merge-based-on-time/v5.0.32** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A ${{ }} expression is interpolated directly inside a run: shell command. The step `run: echo pr-blocked=${{ steps.block.outputs.pr-blocked }}` injects the value of `steps.block.outputs.pr-blocked` directly into the shell command string before the shell ever sees it. Although `steps.*.outputs.*` is not directly attacker-controlled here, any expression inside ${{ }} in a run: block is a script-injection risk as it bypasses shell quoting. The value should be passed via an env: variable and referenced as `$ENV_VAR` instead.

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:20`

### unpinned-uses (severity: high)

Two workflow files reference actions by mutable branch names instead of full 40-character commit SHAs, making them vulnerable to supply-chain attacks if the referenced branch is compromised or force-pushed:
- `uses: yykamei/block-merge-based-on-time@main` (branch ref)
- `uses: yykamei/github-workflows-metrics@main` (branch ref)
These should be pinned to a full SHA digest, e.g. `uses: yykamei/block-merge-based-on-time@<40-char-sha> # main`.

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:15`
- `.github/workflows/metrics.yml:10`

### missing-permissions (severity: medium)

Two workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without explicit permissions, GitHub Actions defaults to the repository's default token permissions (which may be broad). Each workflow should declare minimal required permissions:
- `ci.yml`: no top-level permissions; jobs `ci`, `dependency-review`, `auto-build-trusted`, and `auto-build-untrusted` all lack job-level permissions.
- `fetch-holidays.yml`: no top-level permissions; the `fetch-holidays` job lacks job-level permissions.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/fetch-holidays.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

1. block-merge-based-on-time.yaml: Fixed script-injection by moving `${{ steps.block.outputs.pr-blocked }}` into an env: variable (PR_BLOCKED) and referencing it as `$PR_BLOCKED` in the shell. Also pinned `yykamei/block-merge-based-on-time@main` to full SHA `e48448b3a1ec260265f4f38d240e7ecec2357c06 # main`. 2. metrics.yml: Pinned `yykamei/github-workflows-metrics@main` to full SHA `44ee33cf23b5ff6e90dcf3f886376b402dee220c # main`. 3. ci.yml: Added top-level `permissions: {}` and job-level permissions for each job: `ci` (contents: read), `dependency-review` (contents: read, pull-requests: write), `auto-build-trusted` (contents: read), `auto-build-untrusted` (contents: read). 4. fetch-holidays.yml: Added top-level `permissions: {}` and job-level `permissions: contents: read` for the fetch-holidays job (git-push and pr-create use the app token, not GITHUB_TOKEN).

