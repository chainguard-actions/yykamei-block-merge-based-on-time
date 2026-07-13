<!-- markdownlint-disable -->

# Hardening Report: yykamei--block-merge-based-on-time/v5.0.25

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **yykamei--block-merge-based-on-time/v5.0.25** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two workflow files reference actions using mutable branch refs (@main) instead of pinned 40-character commit SHAs. This exposes the workflow to supply-chain attacks if the referenced branch is compromised or force-pushed. Affected references: `yykamei/block-merge-based-on-time@main` and `yykamei/github-workflows-metrics@main`.

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:15`
- `.github/workflows/metrics.yml:13`

### script-injection (severity: high)

Sub-rule (a): A `run:` block directly interpolates a GitHub Actions expression into the shell command string. The step `run: echo pr-blocked=${{ steps.block.outputs.pr-blocked }}` embeds the expression directly in the shell command before the shell ever sees it, allowing an attacker who can influence `steps.block.outputs.pr-blocked` to inject arbitrary shell commands.

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:21`

### missing-permissions (severity: medium)

Two workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without explicit permissions, workflows inherit the default repository token permissions (which may be broad). `ci.yml` and `fetch-holidays.yml` both lack any permissions declaration.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/fetch-holidays.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings: (1) Pinned yykamei/block-merge-based-on-time@main to commit SHA 3253a9329289117ba11c13e5cdbfee98778740ab and yykamei/github-workflows-metrics@main to commit SHA 3bda9ee1ba6731558f19bd95d362848a104efbc5 in their respective workflow files. (2) Fixed script injection in block-merge-based-on-time.yaml by moving ${{ steps.block.outputs.pr-blocked }} into an env: block as PR_BLOCKED and referencing it as $PR_BLOCKED in the shell command. (3) Added top-level permissions: {} to ci.yml and fetch-holidays.yml, with job-level permissions scoped to minimum needed: ci job gets contents: read, dependency-review gets contents: read + pull-requests: read, auto-build-trusted gets contents: write, auto-build-untrusted gets contents: read, and fetch-holidays gets contents: write + pull-requests: write.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in .github/workflows/release.yml. In the 'Decide the next version' step, added sanitization of the `next` variable before writing it to $GITHUB_OUTPUT. Changed `echo "next=${next}" >> "$GITHUB_OUTPUT"` to first compute `safe=$(printf '%s' "$next" | tr -d '\n\r')` and then use `echo "next=${safe}" >> "$GITHUB_OUTPUT"`. This strips any newline characters from the user-controlled `inputs.version` value that could otherwise inject additional key=value pairs into GITHUB_OUTPUT.

