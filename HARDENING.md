<!-- markdownlint-disable -->

# Hardening Report: yykamei--block-merge-based-on-time/v5.0.40

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **yykamei--block-merge-based-on-time/v5.0.40** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A ${{ }} expression is directly interpolated inside a run: shell command. The step `run: echo pr-blocked=${{ steps.block.outputs.pr-blocked }}` embeds the expression directly in the shell string. If the output value contains shell metacharacters, it can lead to command injection.

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:20`

### github-env-injection (severity: high)

The 'Decide the next version' step in release.yml writes `echo "next=${next}" >> "$GITHUB_OUTPUT"` where `next` is derived from `$INPUT_VERSION`, which is set from `${{ inputs.version }}` (a workflow_dispatch user-controlled input). Although a regex format check is applied, the value is not sanitized with `printf '%s' ... | tr -d '\n\r'` before the write to $GITHUB_OUTPUT, leaving it vulnerable to newline injection that could poison subsequent steps reading the output.

Locations:

- `.github/workflows/release.yml:41`

### unpinned-uses (severity: high)

One or more `uses:` references are pinned to a mutable branch name rather than a full 40-character commit SHA, making the workflow vulnerable to supply-chain attacks if the referenced branch is compromised. Failing references:
- `yykamei/block-merge-based-on-time@main` (block-merge-based-on-time.yaml, line 14)
- `yykamei/github-workflows-metrics@main` (metrics.yml, line 12)

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:14`
- `.github/workflows/metrics.yml:12`

### missing-permissions (severity: medium)

ci.yml has no top-level `permissions:` key and none of its four jobs (ci, dependency-review, auto-build-trusted, auto-build-untrusted) define job-level `permissions:` blocks. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad.

Locations:

- `.github/workflows/ci.yml:1`

### missing-permissions (severity: medium)

fetch-holidays.yml has no top-level `permissions:` key and its single job (fetch-holidays) has no job-level `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad.

Locations:

- `.github/workflows/fetch-holidays.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all 5 findings: (1) script-injection in block-merge-based-on-time.yaml: moved ${{ steps.block.outputs.pr-blocked }} to env block as PR_BLOCKED; (2) github-env-injection in release.yml: sanitized 'next' value with printf+tr before writing to GITHUB_OUTPUT; (3) unpinned-uses: pinned yykamei/block-merge-based-on-time@main to SHA 2e0678236eabb8d89f7a2e3049a66191f61efb07 and yykamei/github-workflows-metrics@main to SHA 3e4b2ad0dca9abb9631bd8a9d7829f3c7990e830; (4) missing-permissions in ci.yml: added top-level permissions:{} and job-level permissions for all 4 jobs; (5) missing-permissions in fetch-holidays.yml: added top-level permissions:{} and job-level contents:read.

