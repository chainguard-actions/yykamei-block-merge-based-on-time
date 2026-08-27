<!-- markdownlint-disable -->

# Hardening Report: yykamei--block-merge-based-on-time/v5.0.39

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **yykamei--block-merge-based-on-time/v5.0.39** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two workflow files reference actions using mutable branch refs (`@main`) instead of pinned full-length SHA digests, making them vulnerable to supply-chain attacks if the referenced repository is compromised or the branch is force-pushed.

- `.github/workflows/block-merge-based-on-time.yaml` line 15: `uses: yykamei/block-merge-based-on-time@main`
- `.github/workflows/metrics.yml` line 13: `uses: yykamei/github-workflows-metrics@main`

These should be pinned to a full 40-character commit SHA (e.g. `uses: yykamei/block-merge-based-on-time@<sha> # main`).

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:15`
- `.github/workflows/metrics.yml:13`

### script-injection (severity: high)

Sub-rule (a): A `${{ }}` expression is interpolated directly inside a `run:` shell command string. In `.github/workflows/block-merge-based-on-time.yaml` line 21, the step `run: echo pr-blocked=${{ steps.block.outputs.pr-blocked }}` embeds the action output directly into the shell command via YAML template substitution before the shell ever sees it. If the output value contains shell metacharacters (`;`, `|`, `&`, `$(...)`, etc.), they will be interpreted by the shell, enabling command injection. The value should be passed via an `env:` variable and the env var referenced with double-quotes: `env: { PR_BLOCKED: "${{ steps.block.outputs.pr-blocked }}" }` and `run: echo "pr-blocked=$PR_BLOCKED"`.

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:21`

### missing-permissions (severity: medium)

Two workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without explicit permissions, GitHub Actions defaults to the repository's default token permissions (which may be `read-all` or `write-all` depending on org/repo settings), granting the `GITHUB_TOKEN` broader access than necessary.

- `.github/workflows/ci.yml`: Contains four jobs (`ci`, `dependency-review`, `auto-build-trusted`, `auto-build-untrusted`) with no permissions declared at any level.
- `.github/workflows/fetch-holidays.yml`: Contains one job (`fetch-holidays`) with no permissions declared at any level.

Add a top-level `permissions: {}` block (or minimal specific scopes) to each file.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/fetch-holidays.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings: (1) Pinned yykamei/block-merge-based-on-time@main to SHA 37e89da846e881a30a0790691ca2967d3c8592b3 and yykamei/github-workflows-metrics@main to SHA 4ff4a6bba410696d7d450c7b7e4f5eea367a0514 in their respective workflow files. (2) Fixed script injection in block-merge-based-on-time.yaml by moving ${{ steps.block.outputs.pr-blocked }} into an env: block as PR_BLOCKED and referencing it safely as "$PR_BLOCKED" in the run: command. (3) Added top-level `permissions: {}` to ci.yml and fetch-holidays.yml to enforce least-privilege for the GITHUB_TOKEN.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in .github/workflows/release.yml at the 'Decide the next version' step. The `next` variable (derived from user-controlled `$INPUT_VERSION`) is now sanitized with `safe=$(printf '%s' "$next" | tr -d '\n\r')` before being written to $GITHUB_OUTPUT as `echo "next=${safe}" >> "$GITHUB_OUTPUT"`. This prevents newline injection attacks where a malicious version input could inject arbitrary key-value pairs into the GitHub output context.

