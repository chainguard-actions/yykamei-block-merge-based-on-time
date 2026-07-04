<!-- markdownlint-disable -->

# Hardening Report: yykamei--block-merge-based-on-time--/v5.0.22

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **yykamei--block-merge-based-on-time--/v5.0.22** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A ${{ }} expression is interpolated directly inside a run: shell command. The step `run: echo pr-blocked=${{ steps.block.outputs.pr-blocked }}` embeds the steps context value directly into the shell command string before the shell ever sees it, enabling script injection if the value contains shell metacharacters.

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:21`

### github-env-injection (severity: high)

The 'Decide the next version' step in release.yml maps the workflow_dispatch input `inputs.version` to the env var INPUT_VERSION, then writes `echo "next=${next}" >> "$GITHUB_OUTPUT"` where `next` is derived from `$INPUT_VERSION` without the required sanitization (`printf '%s' ... | tr -d '\n\r'`). An attacker-controlled version input containing newlines could inject arbitrary entries into GITHUB_OUTPUT.

Locations:

- `.github/workflows/release.yml:43`

### unpinned-uses (severity: high)

The following uses: references are pinned to mutable branch names rather than full 40-character commit SHAs, making them vulnerable to supply-chain attacks: (1) `yykamei/block-merge-based-on-time@main` in block-merge-based-on-time.yaml; (2) `yykamei/github-workflows-metrics@main` in metrics.yml.

Locations:

- `.github/workflows/block-merge-based-on-time.yaml:15`
- `.github/workflows/metrics.yml:13`

### missing-permissions (severity: medium)

ci.yml has no top-level `permissions:` key and none of its jobs (ci, dependency-review, auto-build-trusted, auto-build-untrusted) define job-level `permissions:` blocks, so the workflow runs with the default (broad) token permissions. Similarly, fetch-holidays.yml has no top-level `permissions:` key and its only job (fetch-holidays) has no job-level `permissions:` block.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/fetch-holidays.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all four findings: (1) script-injection in block-merge-based-on-time.yaml line 21 - moved ${{ steps.block.outputs.pr-blocked }} to env block as PR_BLOCKED; (2) github-env-injection in release.yml line 43 - added safe_next=$(printf '%s' "$next" | tr -d '\n\r') before writing to GITHUB_OUTPUT; (3) unpinned-uses - pinned yykamei/block-merge-based-on-time@main to SHA ac8c38e9fafa44ff9fba910bb3189242d77e00bf and yykamei/github-workflows-metrics@main to SHA 418918de095b08285b8897b789ea286ac1ff1199; (4) missing-permissions - added top-level 'permissions: contents: read' to ci.yml and fetch-holidays.yml.

