<!-- markdownlint-disable -->

# Hardening Report: adRise--update-pr-branch/v0.10.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **adRise--update-pr-branch/v0.10.2** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: `${{ github.event.release.tag_name }}` is interpolated directly inside `run:` shell command strings in the 'Update Rolling Tags' and 'Summary' steps. GitHub Actions substitutes the expression into the shell script before the shell parses it, so a release tag name containing shell metacharacters (e.g. `; malicious_command #`) would be executed as arbitrary shell code. Offending lines:
- Line 24: `TAG_NAME="${{ github.event.release.tag_name }}"`
- Line 59: `echo "...release \`${{ github.event.release.tag_name }}\`..."`
- Line 61: `TAG_NAME="${{ github.event.release.tag_name }}"`
Fix: set the value in an `env:` block and reference it as `"$TAG_NAME"` in the shell script.

Locations:

- `.github/workflows/update-rolling-tags.yml:24`
- `.github/workflows/update-rolling-tags.yml:59`
- `.github/workflows/update-rolling-tags.yml:61`

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable tags or version strings instead of immutable 40-character commit SHAs, making the workflow vulnerable to supply-chain attacks if the referenced tag is moved or overwritten.
- `actions/checkout@v2` (unit_test.yml, line 13)
- `coverallsapp/github-action@v1.1.2` (unit_test.yml, line 18)
- `actions/checkout@v4` (update-rolling-tags.yml, line 12)
Fix: pin each reference to a full SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `.github/workflows/unit_test.yml:13`
- `.github/workflows/unit_test.yml:18`
- `.github/workflows/update-rolling-tags.yml:12`

### missing-permissions (severity: medium)

Neither workflow file defines a top-level `permissions:` block, and no job within either file defines its own `permissions:` block. Without explicit permissions, workflows inherit the repository's default token permissions, which may be broader than necessary (e.g. write access to contents, pull-requests, etc.). Fix: add a minimal `permissions:` block at the top level of each workflow, granting only the scopes actually required (e.g. `contents: write` for the tag-pushing workflow, `checks: read` for the unit-test workflow).

Locations:

- `.github/workflows/unit_test.yml:1`
- `.github/workflows/update-rolling-tags.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across two workflow files:

1. **script-injection** (update-rolling-tags.yml): Moved `${{ github.event.release.tag_name }}` from inline `run:` shell strings into `env:` blocks for both the 'Update Rolling Tags' step and the 'Summary' step. Shell scripts now reference the safe `$TAG_NAME` environment variable.

2. **unpinned-uses**: Pinned all three action references to full commit SHAs:
   - `actions/checkout@v2` → `@ee0669bd1cc54295c223e0bb666b733df41de1c5 # v2`
   - `coverallsapp/github-action@v1.1.2` → `@8cbef1dea373ebce56de0a14c68d6267baa10b44 # v1.1.2`
   - `actions/checkout@v4` → `@34e114876b0b11c390a56381ad16ebd13914f8d5 # v4`

3. **missing-permissions**: Added top-level `permissions:` blocks:
   - `unit_test.yml`: `contents: read` (minimal for checkout)
   - `update-rolling-tags.yml`: `contents: write` (required for force-pushing tags)

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed the unquoted `$TAG_NAME` variable in the `[[ ]]` conditional in the 'Update Rolling Tags' step of `.github/workflows/update-rolling-tags.yml`. Changed `if [[ ! $TAG_NAME =~ ^v[0-9]+\.[0-9]+\.[0-9]+$ ]]; then` to `if [[ ! "$TAG_NAME" =~ ^v[0-9]+\.[0-9]+\.[0-9]+$ ]]; then`. The variable is already sourced from the `env:` block (not directly from a `${{ }}` expression in the run script), so this single quoting fix is sufficient to address the script injection risk.

