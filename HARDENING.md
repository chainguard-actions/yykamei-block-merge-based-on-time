<!-- markdownlint-disable -->

# Hardening Report: yykamei--block-merge-based-on-time/v5.0.24

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **yykamei--block-merge-based-on-time/v5.0.24** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two workflow files reference actions by mutable branch name (@main) rather than a pinned 40-character commit SHA, making them vulnerable to supply-chain attacks if the referenced branch is compromised.

- `.github/workflows/block-merge-based-on-time.yaml` line 16: `uses: yykamei/block-merge-based-on-time@main`
- `.github/workflows/metrics.yml` line 13: `uses: yykamei/github-workflows-metrics@main`

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:16`
- `.github/workflows/metrics.yml:13`

### script-injection (severity: high)

Sub-rule (a): A `${{ ... }}` expression is interpolated directly inside a `run:` shell command string. In `.github/workflows/block-merge-based-on-time.yaml` line 22, the step output `steps.block.outputs.pr-blocked` is embedded directly in the shell command: `run: echo pr-blocked=${{ steps.block.outputs.pr-blocked }}`. Although `steps.*.outputs.*` is not directly attacker-controlled here, any expression inside a `run:` block is subject to YAML template substitution before the shell ever sees it, making this a script-injection risk. The value should be passed via an `env:` variable and referenced as `$ENV_VAR` instead.

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:22`

### github-env-injection (severity: high)

In `.github/workflows/release.yml`, the workflow input `inputs.version` is mapped to the env var `INPUT_VERSION` and then assigned to the shell variable `next` (after a regex validation check). The value is then written directly to `$GITHUB_OUTPUT` via `echo "next=${next}" >> "$GITHUB_OUTPUT"` without the required sanitization step (`printf '%s' "$next" | tr -d '\n\r'`). A crafted version string containing newlines could inject arbitrary key-value pairs into `$GITHUB_OUTPUT`, potentially overwriting subsequent step outputs.

Locations:

- `.github/workflows/release.yml:35`

### permissions (severity: medium)

Two workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs, meaning they run with the default (potentially broad) repository permissions.

- `.github/workflows/ci.yml`: No top-level permissions and none of the four jobs (`ci`, `dependency-review`, `auto-build-trusted`, `auto-build-untrusted`) declare job-level permissions.
- `.github/workflows/fetch-holidays.yml`: No top-level permissions and the `fetch-holidays` job has no job-level permissions.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/fetch-holidays.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, permissions

**Notes:**

Fixed all four findings:
1. unpinned-uses: Pinned yykamei/block-merge-based-on-time@main → @6f9af9cb2267234702a351ac04edb35abc4fb9c8 # main in block-merge-based-on-time.yaml; pinned yykamei/github-workflows-metrics@main → @778d9db33ef12a32c03416e9ccd2e3f1d9af6b22 # main in metrics.yml.
2. script-injection: In block-merge-based-on-time.yaml line 22, moved ${{ steps.block.outputs.pr-blocked }} into an env: block as PR_BLOCKED and referenced it as $PR_BLOCKED in the shell command.
3. github-env-injection: In release.yml, added safe_next=$(printf '%s' "$next" | tr -d '\n\r') before writing to $GITHUB_OUTPUT to strip any embedded newlines.
4. permissions: Added top-level permissions: {} and job-level permissions (contents: read for ci/auto-build-trusted/auto-build-untrusted, contents: read + pull-requests: write for dependency-review) to ci.yml; added top-level permissions: {} and job-level contents: read to fetch-holidays.yml.

