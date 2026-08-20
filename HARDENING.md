<!-- markdownlint-disable -->

# Hardening Report: yykamei--block-merge-based-on-time/v5.0.35

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **yykamei--block-merge-based-on-time/v5.0.35** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A ${{ }} expression is directly interpolated inside a run: shell command. The line `run: echo pr-blocked=${{ steps.block.outputs.pr-blocked }}` passes the step output value through YAML template substitution before the shell ever sees it, allowing an attacker who controls that output to inject shell metacharacters. Fix: move the value into an env: variable and reference it as a quoted shell variable, e.g. `echo "pr-blocked=$PR_BLOCKED"`.

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:21`

### github-env-injection (severity: high)

The 'Decide the next version' step in release.yml writes `next=${next}` to $GITHUB_OUTPUT where `next` is derived from `$INPUT_VERSION`, which is set from `inputs.version` (a workflow_dispatch user-controlled input). No sanitization (`printf '%s' ... | tr -d '\n\r'`) is applied before the write. A newline embedded in the version input could inject arbitrary key=value pairs into GITHUB_OUTPUT, poisoning subsequent steps.

Locations:

- `.github/workflows/release.yml:37`

### unpinned-uses (severity: high)

Two workflow files reference actions by mutable branch names instead of immutable 40-character commit SHAs, making them vulnerable to supply-chain attacks if the referenced branch is compromised or force-pushed:
- `yykamei/block-merge-based-on-time@main` in block-merge-based-on-time.yaml (line 15)
- `yykamei/github-workflows-metrics@main` in metrics.yml (line 12)
Fix: pin each reference to a full SHA, e.g. `uses: yykamei/block-merge-based-on-time@<40-char-sha> # main`.

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:15`
- `.github/workflows/metrics.yml:12`

### missing-permissions (severity: medium)

Two workflow files have no top-level `permissions:` block and no job-level `permissions:` block on any of their jobs. Without explicit permissions, the GITHUB_TOKEN is granted the repository's default permissions (often broad write access), violating the principle of least privilege.
- ci.yml: no permissions declared at top level or on any of its four jobs (ci, dependency-review, auto-build-trusted, auto-build-untrusted).
- fetch-holidays.yml: no permissions declared at top level or on the fetch-holidays job.
Fix: add a top-level `permissions: {}` (deny-all) and grant only the specific scopes each job requires.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/fetch-holidays.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all four findings: (1) script-injection in block-merge-based-on-time.yaml: moved ${{ steps.block.outputs.pr-blocked }} into env var PR_BLOCKED and referenced as "$PR_BLOCKED"; (2) github-env-injection in release.yml: sanitized 'next' variable with printf '%s' | tr -d '\n\r' before writing to GITHUB_OUTPUT; (3) unpinned-uses: pinned yykamei/block-merge-based-on-time@main to SHA ae7f50a8f8b0b2b88d2361794a696430d936a2db and yykamei/github-workflows-metrics@main to SHA 4ff4a6bba410696d7d450c7b7e4f5eea367a0514; (4) missing-permissions: added top-level permissions: {} and job-level permissions blocks (contents: read, plus pull-requests: read for dependency-review) to ci.yml and fetch-holidays.yml.

