<!-- markdownlint-disable -->

# Hardening Report: yykamei--block-merge-based-on-time/v5.0.44

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **yykamei--block-merge-based-on-time/v5.0.44** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two workflow files reference actions using mutable branch refs (`@main`) instead of full 40-character commit SHA digests, making them vulnerable to supply-chain attacks if the referenced branch is compromised or force-pushed:
- `yykamei/block-merge-based-on-time@main` in block-merge-based-on-time.yaml
- `yykamei/github-workflows-metrics@main` in metrics.yml
These should be pinned to a full SHA, e.g. `yykamei/block-merge-based-on-time@<40-char-sha> # main`.

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:14`
- `.github/workflows/metrics.yml:11`

### missing-permissions (severity: medium)

Two workflow files have no top-level `permissions:` key and no job-level `permissions:` block on any of their jobs. Without explicit permissions, GitHub Actions defaults to the repository's default token permissions (often `write-all` for older repos), granting excessive access. Minimal required permissions should be declared explicitly.
- `ci.yml`: 4 jobs (ci, dependency-review, auto-build-trusted, auto-build-untrusted) — none have permissions.
- `fetch-holidays.yml`: 1 job (fetch-holidays) — no permissions declared.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/fetch-holidays.yml:1`

### script-injection (severity: high)

Rule (a) violation: A `run:` block directly interpolates a `${{ }}` GitHub Actions expression into a shell command string. The expression `${{ steps.block.outputs.pr-blocked }}` is substituted into the shell command before the shell ever parses it, allowing a malicious value in that output to inject arbitrary shell commands.

Offending line:
```
run: echo pr-blocked=${{ steps.block.outputs.pr-blocked }}
```

Fix: Move the value into an `env:` variable and reference it as a quoted shell variable:
```yaml
env:
  PR_BLOCKED: ${{ steps.block.outputs.pr-blocked }}
run: echo "pr-blocked=$PR_BLOCKED"
```

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:20`

### github-env-injection (severity: high)

The 'Decide the next version' step in release.yml writes a value derived from `inputs.version` (a user-controlled `workflow_dispatch` input) to `$GITHUB_OUTPUT` without the required newline-stripping sanitization (`printf '%s' ... | tr -d '\n\r'`). The input is routed through the `INPUT_VERSION` env var and validated by a regex, but the check requires the specific sanitization pipeline before every write to special environment files when the source is untrusted input. A crafted multi-line value that bypasses the regex could inject additional key=value pairs into `$GITHUB_OUTPUT`.

Offending pattern:
```bash
next="$INPUT_VERSION"   # INPUT_VERSION = ${{ inputs.version }}
echo "next=${next}" >> "$GITHUB_OUTPUT"   # FAIL: no tr -d newlines
```

Fix:
```bash
safe=$(printf '%s' "$next" | tr -d '\n\r')
echo "next=${safe}" >> "$GITHUB_OUTPUT"
```

Locations:

- `.github/workflows/release.yml:36`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, github-env-injection

**Notes:**

Fixed all four findings:
1. unpinned-uses: Pinned yykamei/block-merge-based-on-time@main to SHA ce0d55d18b07dd035a62d9dd4d680743acc2ffc0 in block-merge-based-on-time.yaml, and yykamei/github-workflows-metrics@main to SHA ee4cc54ed556a638ebd9151d077d502c94980c0e in metrics.yml.
2. missing-permissions: Added top-level `permissions: {}` and job-level permissions blocks to ci.yml (4 jobs: ci, dependency-review, auto-build-trusted, auto-build-untrusted) and fetch-holidays.yml (1 job: fetch-holidays). Minimum permissions granted: contents: read for all jobs, plus pull-requests: write for dependency-review.
3. script-injection: Moved `${{ steps.block.outputs.pr-blocked }}` out of the run: shell string in block-merge-based-on-time.yaml into an env: block as PR_BLOCKED, referenced as "$PR_BLOCKED" in the shell command.
4. github-env-injection: Added `safe=$(printf '%s' "$next" | tr -d '\n\r')` before the GITHUB_OUTPUT write in release.yml's 'Decide the next version' step to strip newlines from the user-controlled input.version value.

