<!-- markdownlint-disable -->

# Hardening Report: yykamei--block-merge-based-on-time/v5.0.33

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **yykamei--block-merge-based-on-time/v5.0.33** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A ${{ }} expression is directly interpolated inside a run: shell command. The step `run: echo pr-blocked=${{ steps.block.outputs.pr-blocked }}` injects the step output value directly into the shell command string before the shell ever sees it, allowing an attacker who can influence that output to inject arbitrary shell commands.

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:21`

### unpinned-uses (severity: high)

Two workflow files reference actions using mutable branch refs (@main) instead of full 40-character SHA commit digests, making them vulnerable to supply-chain attacks if the referenced repository is compromised:
- `yykamei/block-merge-based-on-time@main` in block-merge-based-on-time.yaml (line 16)
- `yykamei/github-workflows-metrics@main` in metrics.yml (line 12)

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:16`
- `.github/workflows/metrics.yml:12`

### missing-permissions (severity: medium)

Two workflow files have no top-level `permissions:` key and no job-level `permissions:` blocks on any of their jobs, meaning they run with the default (overly broad) GITHUB_TOKEN permissions:
- ci.yml: no top-level permissions and none of the four jobs (ci, dependency-review, auto-build-trusted, auto-build-untrusted) define permissions.
- fetch-holidays.yml: no top-level permissions and the fetch-holidays job has no permissions block.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/fetch-holidays.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

1. script-injection (block-merge-based-on-time.yaml line 21): Moved `${{ steps.block.outputs.pr-blocked }}` out of the run: shell string into an env: block as PR_BLOCKED, then referenced it as $PR_BLOCKED in the shell command.
2. unpinned-uses: Pinned yykamei/block-merge-based-on-time@main → @323a1a93fa63cb187a0e87ec7ce178ce372091f7 # main in block-merge-based-on-time.yaml; pinned yykamei/github-workflows-metrics@main → @44ee33cf23b5ff6e90dcf3f886376b402dee220c # main in metrics.yml.
3. missing-permissions: Added top-level `permissions: {}` and job-level `permissions: contents: read` to ci.yml (all 4 jobs) and fetch-holidays.yml. The dependency-review job also gets `pull-requests: write` since the action posts review comments.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

In .github/workflows/release.yml, the 'Decide the next version' step now sanitizes the `next` variable before writing to $GITHUB_OUTPUT. Added `safe_next=$(printf '%s' "$next" | tr -d '\n\r')` immediately before the `echo "next=${safe_next}" >> "$GITHUB_OUTPUT"` line. This strips any embedded newlines or carriage returns from the value (which is derived from the untrusted `${{ inputs.version }}` workflow_dispatch input), preventing injection of additional key=value pairs into GITHUB_OUTPUT.

