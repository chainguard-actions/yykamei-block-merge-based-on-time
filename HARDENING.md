<!-- markdownlint-disable -->

# Hardening Report: yykamei--block-merge-based-on-time--/v5.0.23

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **yykamei--block-merge-based-on-time--/v5.0.23** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two workflow files reference actions by mutable branch names instead of full 40-character commit SHAs:
- `.github/workflows/block-merge-based-on-time.yaml`: `uses: yykamei/block-merge-based-on-time@main` (line 16)
- `.github/workflows/metrics.yml`: `uses: yykamei/github-workflows-metrics@main` (line 12)
These can be silently updated by the action owner, enabling supply-chain attacks.

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:16`
- `.github/workflows/metrics.yml:12`

### missing-permissions (severity: medium)

Two workflow files have no top-level `permissions:` key and no job-level `permissions:` on any of their jobs, meaning they run with the default (potentially broad) token permissions:
- `.github/workflows/ci.yml`: four jobs (ci, dependency-review, auto-build-trusted, auto-build-untrusted) all lack permissions.
- `.github/workflows/fetch-holidays.yml`: the fetch-holidays job lacks permissions.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/fetch-holidays.yml:1`

### script-injection (severity: high)

Rule (a): A `${{ ... }}` expression is interpolated directly inside a `run:` shell command string. In `.github/workflows/block-merge-based-on-time.yaml` line 22: `run: echo pr-blocked=${{ steps.block.outputs.pr-blocked }}`. The value of `steps.block.outputs.pr-blocked` flows through YAML template substitution before the shell sees it, allowing injection of shell metacharacters.

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:22`

### github-env-injection (severity: high)

In `.github/workflows/release.yml`, the 'Decide the next version' step writes a value derived from `${{ inputs.version }}` (mapped to env var `INPUT_VERSION`) to `$GITHUB_OUTPUT` without sanitization. The script sets `next="$INPUT_VERSION"` and then executes `echo "next=${next}" >> "$GITHUB_OUTPUT"` (line 44). An attacker-controlled `inputs.version` value containing newlines could inject arbitrary key-value pairs into GITHUB_OUTPUT. The required sanitization (`printf '%s' ... | tr -d '\n\r'`) is absent.

Locations:

- `.github/workflows/release.yml:44`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, github-env-injection

**Notes:**

Fixed all four findings: (1) Pinned yykamei/block-merge-based-on-time@main to full SHA 7bc5ebaff2a92272109ab654a0425d0a1b7e6b54 in block-merge-based-on-time.yaml, and yykamei/github-workflows-metrics@main to SHA 418918de095b08285b8897b789ea286ac1ff1199 in metrics.yml. (2) Added top-level `permissions: {}` and job-level permissions blocks to ci.yml (all four jobs get `contents: read`; dependency-review also gets `pull-requests: read`) and fetch-holidays.yml (fetch-holidays job gets `contents: read`). (3) Moved `${{ steps.block.outputs.pr-blocked }}` from the run: shell string into an env: block as PR_BLOCKED to prevent script injection. (4) Added `safe_next=$(printf '%s' "$next" | tr -d '\n\r')` before writing to GITHUB_OUTPUT in release.yml to prevent newline injection from the user-controlled inputs.version value.

