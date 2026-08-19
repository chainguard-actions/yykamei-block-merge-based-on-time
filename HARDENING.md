<!-- markdownlint-disable -->

# Hardening Report: yykamei--block-merge-based-on-time/v5.0.15

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **yykamei--block-merge-based-on-time/v5.0.15** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): A ${{ }} expression is directly interpolated inside a run: shell command. The step `run: echo pr-blocked=${{ steps.block.outputs.pr-blocked }}` embeds the step output directly into the shell string before the shell ever sees it. An attacker who can influence the action's output could inject shell metacharacters. The value should be passed via an env: variable and then referenced as a quoted shell variable (e.g., `echo "pr-blocked=$PR_BLOCKED"`).

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:20`

### github-env-injection (severity: high)

The 'Decide the next version' step in release.yml writes a value derived from the untrusted input `inputs.version` (via the env var `$INPUT_VERSION`) to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). The variable `next` is set from `$INPUT_VERSION` and then written with `echo "next=${next}" >> "$GITHUB_OUTPUT"`. A newline character in the input could inject additional key=value pairs into GITHUB_OUTPUT.

Locations:

- `.github/workflows/release.yml:36`

### unpinned-uses (severity: high)

Two workflow files reference actions using mutable branch refs instead of full 40-character commit SHAs, making them vulnerable to supply-chain attacks if the referenced branch is compromised: (1) `yykamei/block-merge-based-on-time@main` in block-merge-based-on-time.yaml; (2) `yykamei/github-workflows-metrics@main` in metrics.yml.

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:14`
- `.github/workflows/metrics.yml:12`

### missing-permissions (severity: medium)

Two workflow files have no top-level `permissions:` key and no job-level `permissions:` blocks on any of their jobs, meaning they run with the default (potentially broad) token permissions: (1) ci.yml — contains jobs `ci`, `dependency-review`, `auto-build-trusted`, and `auto-build-untrusted`, none of which declare permissions; (2) fetch-holidays.yml — the `fetch-holidays` job has no permissions block. Each file should declare a minimal top-level `permissions:` block (e.g., `permissions: {}`) and grant only the specific scopes required.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/fetch-holidays.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all four findings: (1) script-injection in block-merge-based-on-time.yaml: moved `${{ steps.block.outputs.pr-blocked }}` to an env var `PR_BLOCKED` and referenced it as `"$PR_BLOCKED"` in the shell; (2) github-env-injection in release.yml: added `safe_next=$(printf '%s' "${next}" | tr -d '\n\r')` before writing to GITHUB_OUTPUT; (3) unpinned-uses: pinned `yykamei/block-merge-based-on-time@main` to SHA 9457d7741e55ec4c3ad76085251c8868eea29b23 and `yykamei/github-workflows-metrics@main` to SHA 778d9db33ef12a32c03416e9ccd2e3f1d9af6b22; (4) missing-permissions: added `permissions: {}` top-level and per-job `permissions: contents: read` (plus `pull-requests: read` for dependency-review) to ci.yml and fetch-holidays.yml.

