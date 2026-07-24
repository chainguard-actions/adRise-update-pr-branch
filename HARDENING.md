<!-- markdownlint-disable -->

# Hardening Report: adRise--update-pr-branch/v0.11.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **adRise--update-pr-branch/v0.11.1** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable tags rather than full 40-character commit SHAs, making the workflow vulnerable to supply-chain attacks if the tag is moved. Failing references:
- `actions/checkout@v2` (unit_test.yml)
- `coverallsapp/github-action@v1.1.2` (unit_test.yml)
- `actions/checkout@v4` (update-rolling-tags.yml)

Locations:

- `.github/workflows/unit_test.yml:13`
- `.github/workflows/unit_test.yml:19`
- `.github/workflows/update-rolling-tags.yml:9`

### script-injection (severity: high)

Sub-rule (a): `${{ github.event.release.tag_name }}` is interpolated directly inside `run:` shell command strings. Although the `release` event is typically triggered by maintainers, the tag name is still attacker-influenced data that flows through YAML template substitution before the shell sees it, enabling shell metacharacter injection. Offending lines:
- `TAG_NAME="${{ github.event.release.tag_name }}"` in the 'Update Rolling Tags' step
- `TAG_NAME="${{ github.event.release.tag_name }}"` and `${{ github.event.release.tag_name }}` in the 'Summary' step
Fix: assign the value via an `env:` block and reference it as a quoted shell variable (e.g., `"$TAG_NAME"`).

Locations:

- `.github/workflows/update-rolling-tags.yml:22`
- `.github/workflows/update-rolling-tags.yml:55`
- `.github/workflows/update-rolling-tags.yml:58`

### missing-permissions (severity: medium)

Neither workflow file declares a top-level `permissions:` block, and no job within them declares job-level permissions. Without explicit permissions, workflows run with the repository's default token permissions, which may be overly broad (e.g., write access to contents, pull-requests, etc.).

Locations:

- `.github/workflows/unit_test.yml:1`
- `.github/workflows/update-rolling-tags.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings across both workflow files:

1. **unpinned-uses**: Pinned all three action references to full commit SHAs:
   - `actions/checkout@v2` → `@0717577d45739eb3c851188b29f50ed6c0b2194e # v2` (unit_test.yml)
   - `coverallsapp/github-action@v1.1.2` → `@8cbef1dea373ebce56de0a14c68d6267baa10b44 # v1.1.2` (unit_test.yml)
   - `actions/checkout@v4` → `@11d5960a326750d5838078e36cf38b85af677262 # v4` (update-rolling-tags.yml)

2. **script-injection**: Moved `${{ github.event.release.tag_name }}` out of both `run:` blocks in update-rolling-tags.yml into `env:` blocks as `TAG_NAME`, then referenced as `$TAG_NAME` in the shell scripts.

3. **missing-permissions**: Added top-level `permissions:` blocks to both files:
   - `unit_test.yml`: `contents: read` (minimum for checkout)
   - `update-rolling-tags.yml`: `contents: write` (required to push tags)

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted `$TAG_NAME` in the `[[ ]]` conditional in `.github/workflows/update-rolling-tags.yml` line 32. Changed `if [[ ! $TAG_NAME =~ ... ]]` to `if [[ ! "$TAG_NAME" =~ ... ]]` to ensure the workflow-controllable env var is always double-quoted, preventing shell metacharacters in a crafted tag name from causing unexpected behavior.

