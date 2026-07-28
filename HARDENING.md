<!-- markdownlint-disable -->

# Hardening Report: yykamei--block-merge-based-on-time/v5.0.27

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **yykamei--block-merge-based-on-time/v5.0.27** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two workflow files reference actions by mutable branch name (@main) instead of a full 40-character commit SHA. This exposes the workflow to supply-chain attacks if the referenced repository is compromised or the branch is force-pushed.

- `.github/workflows/block-merge-based-on-time.yaml`: `uses: yykamei/block-merge-based-on-time@main`
- `.github/workflows/metrics.yml`: `uses: yykamei/github-workflows-metrics@main`

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:15`
- `.github/workflows/metrics.yml:12`

### missing-permissions (severity: medium)

Two workflow files have no top-level `permissions:` key and no job-level `permissions:` on any of their jobs. Without explicit permissions, the GITHUB_TOKEN is granted its default (potentially broad) permissions.

- `ci.yml`: No top-level permissions; jobs `ci`, `dependency-review`, `auto-build-trusted`, and `auto-build-untrusted` all lack job-level permissions.
- `fetch-holidays.yml`: No top-level permissions; job `fetch-holidays` lacks job-level permissions.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/fetch-holidays.yml:1`

### script-injection (severity: high)

Sub-rule (a): A `run:` block directly interpolates a `${{ }}` expression inside the shell command string. The value `${{ steps.block.outputs.pr-blocked }}` is substituted by the Actions template engine before the shell ever sees it, allowing a malicious step output to inject arbitrary shell commands.

Offending line:
```
- run: echo pr-blocked=${{ steps.block.outputs.pr-blocked }}
```

Fix: move the value into an `env:` variable and reference it as a quoted shell variable:
```yaml
env:
  PR_BLOCKED: ${{ steps.block.outputs.pr-blocked }}
run: echo "pr-blocked=$PR_BLOCKED"
```

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:20`

### github-env-injection (severity: high)

In the 'Decide the next version' step of release.yml, the user-controlled input `inputs.version` is mapped to the env var `INPUT_VERSION` and then written to `$GITHUB_OUTPUT` without the required sanitization (`printf '%s' ... | tr -d '\n\r'`). Although a regex check validates the format, the value is assigned to `next` and written directly:

```bash
next="$INPUT_VERSION"
...
echo "next=${next}" >> "$GITHUB_OUTPUT"
```

A value containing embedded newlines (e.g. injected via a crafted `workflow_dispatch` input) could inject additional key=value pairs into `$GITHUB_OUTPUT`, potentially overwriting subsequent step outputs. The sanitization step must be applied immediately before the write.

Locations:

- `.github/workflows/release.yml:38`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, github-env-injection

**Notes:**

Fixed all four findings:
1. unpinned-uses: Pinned yykamei/block-merge-based-on-time@main to SHA 955a8ea0a9354593cff886662233c20e20489006 in block-merge-based-on-time.yaml, and yykamei/github-workflows-metrics@main to SHA 4cee3bbef92657b0c1247260419a0341da68ba11 in metrics.yml.
2. missing-permissions: Added permissions: {} at top level of ci.yml and fetch-holidays.yml, plus minimal job-level permissions for all four jobs in ci.yml (contents: read for ci and auto-build-untrusted; contents: read + pull-requests: write for dependency-review; contents: write + pull-requests: read for auto-build-trusted) and fetch-holidays job (contents: write + pull-requests: write).
3. script-injection: Moved ${{ steps.block.outputs.pr-blocked }} into an env: block as PR_BLOCKED and referenced it as $PR_BLOCKED in the shell command in block-merge-based-on-time.yaml.
4. github-env-injection: Added sanitization in release.yml using printf '%s' "$next" | tr -d '\n\r' before writing to $GITHUB_OUTPUT to prevent newline injection attacks.

