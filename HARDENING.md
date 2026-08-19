<!-- markdownlint-disable -->

# Hardening Report: yykamei--block-merge-based-on-time/v5.0.21

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **yykamei--block-merge-based-on-time/v5.0.21** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two workflow files reference actions using mutable branch refs (@main) instead of pinned 40-character commit SHAs. This exposes the workflow to supply-chain attacks if the referenced repository is compromised or the branch is force-pushed.

- `.github/workflows/block-merge-based-on-time.yaml`: `uses: yykamei/block-merge-based-on-time@main`
- `.github/workflows/metrics.yml`: `uses: yykamei/github-workflows-metrics@main`

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:14`
- `.github/workflows/metrics.yml:12`

### script-injection (severity: high)

Rule (a) violation: A `run:` block directly interpolates a `${{ ... }}` expression into a shell command string. The expression `${{ steps.block.outputs.pr-blocked }}` is substituted into the shell command before the shell parses it, allowing any special characters in the value to be interpreted by the shell.

Offending line: `run: echo pr-blocked=${{ steps.block.outputs.pr-blocked }}`

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:20`

### missing-permissions (severity: medium)

Two workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without explicit permissions, workflows inherit the default repository token permissions (which may be `write-all` depending on repository settings), granting unnecessarily broad access.

- `.github/workflows/ci.yml`: Four jobs (`ci`, `dependency-review`, `auto-build-trusted`, `auto-build-untrusted`) — none have permissions defined.
- `.github/workflows/fetch-holidays.yml`: One job (`fetch-holidays`) — no permissions defined.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/fetch-holidays.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings:
1. unpinned-uses: Pinned yykamei/block-merge-based-on-time@main → @3253a9329289117ba11c13e5cdbfee98778740ab # main in block-merge-based-on-time.yaml; pinned yykamei/github-workflows-metrics@main → @fc84095f180148ab40840107fa1cb0809dcbdfd1 # main in metrics.yml.
2. script-injection: Moved ${{ steps.block.outputs.pr-blocked }} out of the run: shell string into an env: block (PR_BLOCKED), referenced as $PR_BLOCKED in the shell command.
3. missing-permissions: Added permissions: {} at top level and minimal job-level permissions (contents: read for most jobs; contents: read + pull-requests: write for dependency-review) to ci.yml and fetch-holidays.yml.

