<!-- markdownlint-disable -->

# Hardening Report: yykamei--block-merge-based-on-time/v5.0.36

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **yykamei--block-merge-based-on-time/v5.0.36** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two workflow files reference actions using mutable branch refs (@main) instead of pinned 40-character commit SHAs, making them vulnerable to supply-chain attacks if the referenced repository is compromised.

- `.github/workflows/block-merge-based-on-time.yaml`: `uses: yykamei/block-merge-based-on-time@main`
- `.github/workflows/metrics.yml`: `uses: yykamei/github-workflows-metrics@main`

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:14`
- `.github/workflows/metrics.yml:13`

### script-injection (severity: high)

Rule (a) violation: A `${{ }}` expression is interpolated directly inside a `run:` shell command string. The step output `steps.block.outputs.pr-blocked` is injected into the shell command before the shell ever sees it, allowing an attacker who can influence that output to inject arbitrary shell commands.

Offending line: `run: echo pr-blocked=${{ steps.block.outputs.pr-blocked }}`

Fix: move the value into an `env:` variable and reference it as a quoted shell variable: `run: echo "pr-blocked=$PR_BLOCKED"` with `env: PR_BLOCKED: ${{ steps.block.outputs.pr-blocked }}`.

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:19`

### missing-permissions (severity: medium)

Two workflow files have no top-level `permissions:` key and contain jobs that also lack job-level `permissions:` blocks, meaning those jobs run with the default (potentially broad) token permissions.

- `.github/workflows/ci.yml`: No top-level permissions. Jobs `ci`, `auto-build-trusted`, and `auto-build-untrusted` all lack job-level permissions blocks. Only `dependency-review` uses `actions/dependency-review-action` which has its own permissions, but the job itself has none declared.
- `.github/workflows/fetch-holidays.yml`: No top-level permissions and the `fetch-holidays` job has no job-level permissions block.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/fetch-holidays.yml:1`

### github-env-injection (severity: high)

In the `release.yml` workflow, the `Decide the next version` step writes the value of `$INPUT_VERSION` (sourced from `${{ inputs.version }}`, a user-controlled `workflow_dispatch` input) to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). Although a regex check (`grep -qE '^[0-9]+\.[0-9]+\.[0-9]+$'`) is applied, the check rule requires explicit newline-stripping sanitization before every write to a special environment file when the source is user-controlled.

Offending code:
```
next="$INPUT_VERSION"
...
echo "next=${next}" >> "$GITHUB_OUTPUT"
```

Fix:
```
safe=$(printf '%s' "$INPUT_VERSION" | tr -d '\n\r')
next="$safe"
...
echo "next=${next}" >> "$GITHUB_OUTPUT"
```

Locations:

- `.github/workflows/release.yml:40`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions, github-env-injection

**Notes:**

Fixed all four findings:
1. unpinned-uses: Pinned yykamei/block-merge-based-on-time@main to SHA 4563474068e843c13cc0510a82228171128d21ba and yykamei/github-workflows-metrics@main to SHA 4ff4a6bba410696d7d450c7b7e4f5eea367a0514.
2. script-injection: Moved ${{ steps.block.outputs.pr-blocked }} out of the run: shell string into an env: block (PR_BLOCKED), referencing it as "$PR_BLOCKED" in the shell command.
3. missing-permissions: Added top-level permissions: {} to ci.yml and fetch-holidays.yml, plus job-level permissions: contents: read to all jobs lacking them (ci, dependency-review, auto-build-trusted, auto-build-untrusted, fetch-holidays). dependency-review also gets pull-requests: write for PR comment posting.
4. github-env-injection: Added safe=$(printf '%s' "$INPUT_VERSION" | tr -d '\n\r') sanitization in release.yml before using the user-controlled version input, applying it to both the regex validation and the next variable assignment.

