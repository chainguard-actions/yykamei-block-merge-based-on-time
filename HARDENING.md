<!-- markdownlint-disable -->

# Hardening Report: yykamei--block-merge-based-on-time/v5.0.42

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **yykamei--block-merge-based-on-time/v5.0.42** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two workflow files reference actions using mutable branch names instead of pinned full-length SHA digests, making them vulnerable to supply-chain attacks if the referenced branch is compromised:
- `.github/workflows/block-merge-based-on-time.yaml` line 16: `uses: yykamei/block-merge-based-on-time@main`
- `.github/workflows/metrics.yml` line 13: `uses: yykamei/github-workflows-metrics@main`
These should be pinned to a full 40-character commit SHA (e.g. `uses: yykamei/block-merge-based-on-time@<sha> # main`).

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:16`
- `.github/workflows/metrics.yml:13`

### script-injection (severity: high)

Sub-rule (a): A `run:` block directly interpolates a GitHub Actions expression `${{ steps.block.outputs.pr-blocked }}` inside a shell command string. The `steps.*.outputs.*` context is workflow-controllable and flows through YAML template substitution before the shell sees it, allowing an attacker to inject shell metacharacters. The offending line is: `run: echo pr-blocked=${{ steps.block.outputs.pr-blocked }}`. This should be moved to an `env:` variable and the env var should be double-quoted in the shell command.

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:22`

### missing-permissions (severity: medium)

Two workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs, meaning they run with the default (potentially broad) token permissions:
- `ci.yml`: No top-level permissions and none of its four jobs (ci, dependency-review, auto-build-trusted, auto-build-untrusted) declare job-level permissions.
- `fetch-holidays.yml`: No top-level permissions and its single job (fetch-holidays) has no job-level permissions.
Each file should declare a top-level `permissions:` block with the minimal required scopes, or each job should declare its own `permissions:` block.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/fetch-holidays.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings:
1. unpinned-uses: Pinned yykamei/block-merge-based-on-time@main to SHA e3dead35a6e0a34de89c4a7db88eea941346b7a3 in block-merge-based-on-time.yaml, and yykamei/github-workflows-metrics@main to SHA ee4cc54ed556a638ebd9151d077d502c94980c0e in metrics.yml.
2. script-injection: Moved ${{ steps.block.outputs.pr-blocked }} out of the run: shell string into an env: block as PR_BLOCKED, and referenced it as "$PR_BLOCKED" in the shell command.
3. missing-permissions: Added permissions: {} at the top level of ci.yml and fetch-holidays.yml, plus job-level permissions blocks with minimal required scopes for each job (contents: read for read-only jobs, contents: write for jobs that push commits, pull-requests: read/write where needed).

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed two security findings: (1) In block-merge-based-on-time.yaml, replaced `echo "pr-blocked=${PR_BLOCKED}"` with `printf 'pr-blocked=%s\n' "${PR_BLOCKED}"` to prevent shell metacharacters in the PR_BLOCKED value from being interpreted. (2) In release.yml, added `safe_next=$(printf '%s' "${next}" | tr -d '\n\r')` before writing to GITHUB_OUTPUT to strip embedded newlines/carriage returns that could enable header injection attacks.

