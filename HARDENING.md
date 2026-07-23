<!-- markdownlint-disable -->

# Hardening Report: yykamei--block-merge-based-on-time/v5.0.26

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **yykamei--block-merge-based-on-time/v5.0.26** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two workflow files reference actions using mutable branch names (@main) instead of pinned 40-character commit SHAs. An attacker who compromises the referenced repository could push malicious code that runs in this workflow.

- `.github/workflows/block-merge-based-on-time.yaml`: `uses: yykamei/block-merge-based-on-time@main`
- `.github/workflows/metrics.yml`: `uses: yykamei/github-workflows-metrics@main`

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:15`
- `.github/workflows/metrics.yml:11`

### script-injection (severity: high)

Sub-rule (a): A GitHub Actions expression is interpolated directly inside a `run:` shell command string. The offending line is:

  `run: echo pr-blocked=${{ steps.block.outputs.pr-blocked }}`

The value of `steps.block.outputs.pr-blocked` is substituted into the shell command before the shell parses it, allowing an attacker who can influence that output to inject arbitrary shell commands (e.g. via a crafted PR). The fix is to pass the value through an `env:` variable and double-quote the expansion: `echo "pr-blocked=$PR_BLOCKED"`.

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:21`

### github-env-injection (severity: high)

The `Decide the next version` step in release.yml writes the variable `next` to `$GITHUB_OUTPUT` without the required sanitization (`printf '%s' ... | tr -d '\n\r'`). The `next` variable is derived from `$INPUT_VERSION`, which is set from `${{ inputs.version }}` — a workflow_dispatch user-controlled input. Although a regex check (`^[0-9]+\.[0-9]+\.[0-9]+$`) is applied, the check specification requires the explicit `tr -d '\n\r'` sanitization step before every write to a special environment file when the source is untrusted input. The offending line is:

  `echo "next=${next}" >> "$GITHUB_OUTPUT"`

Locations:

- `.github/workflows/release.yml:30`

### missing-permissions (severity: medium)

Two workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without explicit permissions, GitHub Actions defaults to the repository's default token permissions (often `write-all` for older repositories), granting more access than necessary.

- `ci.yml`: Four jobs (`ci`, `dependency-review`, `auto-build-trusted`, `auto-build-untrusted`) — none have permissions declared, and there is no top-level permissions block.
- `fetch-holidays.yml`: The `fetch-holidays` job has no permissions declared, and there is no top-level permissions block.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/fetch-holidays.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, missing-permissions

**Notes:**

Fixed all four findings:
1. unpinned-uses: Pinned yykamei/block-merge-based-on-time@main → @9457d7741e55ec4c3ad76085251c8868eea29b23 and yykamei/github-workflows-metrics@main → @778d9db33ef12a32c03416e9ccd2e3f1d9af6b22 with # main comments.
2. script-injection: Moved ${{ steps.block.outputs.pr-blocked }} into an env: block as PR_BLOCKED and used echo "pr-blocked=$PR_BLOCKED" in the run step.
3. github-env-injection: Added safe_next=$(printf '%s' "$next" | tr -d '\n\r') sanitization before writing to $GITHUB_OUTPUT in release.yml.
4. missing-permissions: Added top-level 'permissions: contents: read' to ci.yml and fetch-holidays.yml; added job-level 'permissions: contents: read, pull-requests: write' to the dependency-review job in ci.yml.

