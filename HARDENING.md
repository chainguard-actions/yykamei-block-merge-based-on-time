<!-- markdownlint-disable -->

# Hardening Report: yykamei--block-merge-based-on-time/v5.0.30

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **yykamei--block-merge-based-on-time/v5.0.30** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two workflow files reference actions by mutable branch names instead of pinned 40-character commit SHAs, making them vulnerable to supply-chain attacks:
- `.github/workflows/block-merge-based-on-time.yaml`: `uses: yykamei/block-merge-based-on-time@main`
- `.github/workflows/metrics.yml`: `uses: yykamei/github-workflows-metrics@main`

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:14`
- `.github/workflows/metrics.yml:13`

### missing-permissions (severity: medium)

Two workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs, meaning they run with the default (potentially broad) repository permissions:
- `ci.yml`: 4 jobs (ci, dependency-review, auto-build-trusted, auto-build-untrusted) — none have permissions defined.
- `fetch-holidays.yml`: 1 job (fetch-holidays) — no permissions defined.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/fetch-holidays.yml:1`

### script-injection (severity: high)

Rule (a) violation: A `run:` block directly interpolates a `${{ ... }}` expression inside a shell command string. In `.github/workflows/block-merge-based-on-time.yaml`, the step `run: echo pr-blocked=${{ steps.block.outputs.pr-blocked }}` embeds `steps.block.outputs.pr-blocked` directly into the shell command before the shell ever sees it, allowing an attacker who can influence that output value to inject arbitrary shell commands.

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:20`

### github-env-injection (severity: high)

In `.github/workflows/release.yml`, the 'Decide the next version' step sets `INPUT_VERSION: ${{ inputs.version }}` in its `env:` block (an untrusted workflow_dispatch input), then assigns `next="$INPUT_VERSION"` and writes `echo "next=${next}" >> "$GITHUB_OUTPUT"` without first sanitizing the value with `printf '%s' "$next" | tr -d '\n\r'`. A newline character in `inputs.version` could inject additional key=value pairs into GITHUB_OUTPUT.

Locations:

- `.github/workflows/release.yml:43`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, github-env-injection

**Notes:**

Fixed all 4 findings:
1. unpinned-uses: Pinned yykamei/block-merge-based-on-time@main → SHA 2453849e8fc55d0cbef5a31081caf0d2c3fa37f2 and yykamei/github-workflows-metrics@main → SHA 44ee33cf23b5ff6e90dcf3f886376b402dee220c.
2. missing-permissions: Added top-level `permissions: {}` and job-level permissions to ci.yml (ci/dependency-review: contents:read; auto-build-trusted: contents:write; auto-build-untrusted: contents:read) and fetch-holidays.yml (contents:write, pull-requests:write).
3. script-injection: Moved `${{ steps.block.outputs.pr-blocked }}` into an env: block as PR_BLOCKED and referenced it as $PR_BLOCKED in the shell command.
4. github-env-injection: Added `safe_next=$(printf '%s' "$next" | tr -d '\n\r')` before writing to GITHUB_OUTPUT in release.yml to sanitize the untrusted inputs.version value.

