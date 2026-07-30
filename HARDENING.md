<!-- markdownlint-disable -->

# Hardening Report: yykamei--block-merge-based-on-time/v5.0.28

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **yykamei--block-merge-based-on-time/v5.0.28** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two workflow files reference actions by mutable branch names instead of pinned full-length commit SHAs, making them vulnerable to supply-chain attacks if the referenced branch is compromised.
- `.github/workflows/block-merge-based-on-time.yaml`: `uses: yykamei/block-merge-based-on-time@main`
- `.github/workflows/metrics.yml`: `uses: yykamei/github-workflows-metrics@main`

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:14`
- `.github/workflows/metrics.yml:12`

### missing-permissions (severity: medium)

Two workflow files have no top-level `permissions:` key and no job-level `permissions:` keys on any of their jobs, meaning they run with the default (potentially broad) GITHUB_TOKEN permissions.
- `ci.yml`: 4 jobs (ci, dependency-review, auto-build-trusted, auto-build-untrusted) all lack permissions.
- `fetch-holidays.yml`: the single `fetch-holidays` job lacks permissions.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/fetch-holidays.yml:1`

### script-injection (severity: high)

Sub-rule (a): A `${{ }}` expression is interpolated directly inside a `run:` shell command string. The value `steps.block.outputs.pr-blocked` is substituted into the shell command before the shell parses it, allowing injection of shell metacharacters if the output contains them.

Offending line: `run: echo pr-blocked=${{ steps.block.outputs.pr-blocked }}`

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:21`

### github-env-injection (severity: high)

In the 'Decide the next version' step of release.yml, the workflow_dispatch input `inputs.version` is mapped to the env var `INPUT_VERSION` (`${{ inputs.version }}`), then assigned to the shell variable `next`, and finally written to `$GITHUB_OUTPUT` via `echo "next=${next}" >> "$GITHUB_OUTPUT"` without the required sanitization step (`printf '%s' "$next" | tr -d '\n\r'`). A malicious value containing newlines could inject arbitrary key=value pairs into the GitHub output context.

Locations:

- `.github/workflows/release.yml:34`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, github-env-injection

**Notes:**

Fixed all four findings:
1. unpinned-uses: Pinned yykamei/block-merge-based-on-time@main → @3b76640d9f6d2496871efd9f702374ad29850be5 # main in block-merge-based-on-time.yaml, and yykamei/github-workflows-metrics@main → @7215e6518f63e75ec0e40608edb9cf51ec18d557 # main in metrics.yml.
2. missing-permissions: Added top-level `permissions: {}` and job-level permissions blocks to ci.yml (4 jobs) and fetch-holidays.yml (1 job).
3. script-injection: Moved ${{ steps.block.outputs.pr-blocked }} into an env: block (PR_BLOCKED) and referenced it as $PR_BLOCKED in the run: shell command in block-merge-based-on-time.yaml.
4. github-env-injection: Added `safe_next=$(printf '%s' "$next" | tr -d '\n\r')` before writing to $GITHUB_OUTPUT in release.yml's 'Decide the next version' step.

