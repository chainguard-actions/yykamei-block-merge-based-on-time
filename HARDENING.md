<!-- markdownlint-disable -->

# Hardening Report: yykamei--block-merge-based-on-time/v5.0.20

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **yykamei--block-merge-based-on-time/v5.0.20** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a) violation: A ${{ }} expression is directly interpolated inside a run: shell command string. The step `run: echo pr-blocked=${{ steps.block.outputs.pr-blocked }}` injects the `steps.block.outputs.pr-blocked` context value directly into the shell command before the shell ever sees it. An attacker who can influence the action's output could inject shell metacharacters. The value should be passed via an env: variable and then double-quoted in the shell.

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:20`

### unpinned-uses (severity: high)

One or more `uses:` references are pinned to a mutable branch name (`@main`) instead of a full 40-character commit SHA. This means the action code can change at any time without notice, enabling supply-chain attacks. Failing references:
- `uses: yykamei/block-merge-based-on-time@main` (block-merge-based-on-time.yaml, line 14)
- `uses: yykamei/github-workflows-metrics@main` (metrics.yml, line 13)

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:14`
- `.github/workflows/metrics.yml:13`

### missing-permissions (severity: medium)

These workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any job. Without explicit permissions, the workflow inherits the repository's default token permissions (which may be broad). Each workflow should declare minimal required permissions at the top level or per job.
- ci.yml: 4 jobs (ci, dependency-review, auto-build-trusted, auto-build-untrusted) — none have permissions defined.
- fetch-holidays.yml: 1 job (fetch-holidays) — no permissions defined.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/fetch-holidays.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

1. script-injection (block-merge-based-on-time.yaml line 20): Moved `${{ steps.block.outputs.pr-blocked }}` into an env: variable PR_BLOCKED and referenced it as "$PR_BLOCKED" in the shell run command. 2. unpinned-uses: Pinned yykamei/block-merge-based-on-time@main → SHA 6f9af9cb2267234702a351ac04edb35abc4fb9c8 (block-merge-based-on-time.yaml line 14) and yykamei/github-workflows-metrics@main → SHA 778d9db33ef12a32c03416e9ccd2e3f1d9af6b22 (metrics.yml line 13), both with # main comments. 3. missing-permissions: Added job-level permissions to all 4 jobs in ci.yml (ci: contents:read; dependency-review: contents:read + pull-requests:write; auto-build-trusted: contents:write; auto-build-untrusted: contents:read) and to the fetch-holidays job in fetch-holidays.yml (contents:write + pull-requests:write).

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in .github/workflows/release.yml at the 'Decide the next version' step. Added sanitization using `safe_next=$(printf '%s' "$next" | tr -d '\n\r')` before writing to $GITHUB_OUTPUT, replacing the direct `echo "next=${next}" >> "$GITHUB_OUTPUT"` with `echo "next=${safe_next}" >> "$GITHUB_OUTPUT"`. This ensures that even if a crafted `inputs.version` value somehow bypassed the regex check, newline characters cannot be used to inject additional key=value pairs into GITHUB_OUTPUT.

