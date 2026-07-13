<!-- markdownlint-disable -->

# Hardening Report: yykamei--block-merge-based-on-time/v5.0.24

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **yykamei--block-merge-based-on-time/v5.0.24** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A GitHub Actions expression `${{ steps.block.outputs.pr-blocked }}` is directly interpolated inside a `run:` shell command string. `steps.*.outputs.*` is a workflow-controllable context that flows through YAML template substitution before the shell sees it, enabling script injection. Offending line: `run: echo pr-blocked=${{ steps.block.outputs.pr-blocked }}`

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:20`

### github-env-injection (severity: high)

The 'Decide the next version' step writes a value derived from `inputs.version` (via env var `INPUT_VERSION`) to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). Although a regex format check is applied, newline characters are not stripped before the write: `echo "next=${next}" >> "$GITHUB_OUTPUT"`. An attacker-supplied version string containing newlines could inject additional key=value pairs into GITHUB_OUTPUT.

Locations:

- `.github/workflows/release.yml:42`

### unpinned-uses (severity: high)

The following `uses:` references are pinned to mutable branch refs rather than full 40-character commit SHAs, making them vulnerable to supply-chain attacks: `yykamei/block-merge-based-on-time@main` (block-merge-based-on-time.yaml line 14), `yykamei/github-workflows-metrics@main` (metrics.yml line 10).

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:14`
- `.github/workflows/metrics.yml:10`

### missing-permissions (severity: medium)

ci.yml has no top-level `permissions:` key and none of its jobs (ci, dependency-review, auto-build-trusted, auto-build-untrusted) define job-level permissions. This means the workflow runs with the default (potentially broad) token permissions.

Locations:

- `.github/workflows/ci.yml:1`

### missing-permissions (severity: medium)

fetch-holidays.yml has no top-level `permissions:` key and the `fetch-holidays` job has no job-level permissions block. This means the workflow runs with the default (potentially broad) token permissions.

Locations:

- `.github/workflows/fetch-holidays.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all 5 findings: (1) script-injection in block-merge-based-on-time.yaml: moved `${{ steps.block.outputs.pr-blocked }}` to env block as PR_BLOCKED; (2) github-env-injection in release.yml: added `safe_next=$(printf '%s' "$next" | tr -d '\n\r')` before writing to GITHUB_OUTPUT; (3) unpinned-uses: pinned yykamei/block-merge-based-on-time@main to SHA f752081105af15ff3d2602a9585cd1e2a4b67d28 and yykamei/github-workflows-metrics@main to SHA 3bda9ee1ba6731558f19bd95d362848a104efbc5; (4) missing-permissions in ci.yml: added top-level `permissions: contents: read` and job-level `contents: read, pull-requests: write` for dependency-review; (5) missing-permissions in fetch-holidays.yml: added top-level `permissions: contents: read`.

