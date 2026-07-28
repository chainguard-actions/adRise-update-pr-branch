<!-- markdownlint-disable -->

# Hardening Report: adRise--update-pr-branch/v0.11.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **adRise--update-pr-branch/v0.11.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Update Rolling Tags' workflow directly interpolates `${{ github.event.release.tag_name }}` inside `run:` shell blocks. This is a script-injection vulnerability: the expression is substituted into the shell script before the shell parses it, allowing an attacker who can create a release with a crafted tag name to inject arbitrary shell commands. Offending lines:
- Line 24: `TAG_NAME="${{ github.event.release.tag_name }}"`
- Line 57: `echo "The following rolling tags have been updated for release \`${{ github.event.release.tag_name }}\`:"`
- Line 59: `TAG_NAME="${{ github.event.release.tag_name }}"`
Fix: use an `env:` block to pass the value as an environment variable and reference it as `"$TAG_NAME"` in the shell script.

Locations:

- `.github/workflows/update-rolling-tags.yml:24`
- `.github/workflows/update-rolling-tags.yml:57`
- `.github/workflows/update-rolling-tags.yml:59`

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable tags or version strings rather than immutable 40-character commit SHAs. A tag can be moved to point to a different (potentially malicious) commit at any time, making this a supply-chain risk.
- `actions/checkout@v2` (unit_test.yml, line 14)
- `coverallsapp/github-action@v1.1.2` (unit_test.yml, line 19)
- `actions/checkout@v4` (update-rolling-tags.yml, line 12)
Fix: pin each reference to a full SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `.github/workflows/unit_test.yml:14`
- `.github/workflows/unit_test.yml:19`
- `.github/workflows/update-rolling-tags.yml:12`

### missing-permissions (severity: medium)

Neither workflow file defines a `permissions:` block at the top level or at the job level. Without explicit permissions, workflows run with the default token permissions (which may be `write-all` depending on repository settings), granting broader access than necessary. Both `unit_test.yml` and `update-rolling-tags.yml` are affected.
Fix: add a top-level `permissions:` block with the minimum required scopes (e.g. `contents: read` for unit tests, `contents: write` for the tag-pushing workflow).

Locations:

- `.github/workflows/unit_test.yml:1`
- `.github/workflows/update-rolling-tags.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across both workflow files:

1. script-injection (update-rolling-tags.yml): Moved all three `${{ github.event.release.tag_name }}` interpolations out of `run:` shell blocks into `env:` blocks (one per step). The shell scripts now reference `$TAG_NAME` as a plain environment variable.

2. unpinned-uses: Pinned all three action references to full 40-char commit SHAs: actions/checkout@v2 → 0717577d45739eb3c851188b29f50ed6c0b2194e, coverallsapp/github-action@v1.1.2 → 8cbef1dea373ebce56de0a14c68d6267baa10b44, actions/checkout@v4 → 11d5960a326750d5838078e36cf38b85af677262.

3. missing-permissions: Added top-level `permissions: contents: read` to unit_test.yml and `permissions: contents: write` to update-rolling-tags.yml (write is required to push tags).

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed the unquoted variable expansion in update-rolling-tags.yml line 32. Changed `if [[ ! $TAG_NAME =~ ^v[0-9]+\.[0-9]+\.[0-9]+$ ]]` to `if [[ ! "$TAG_NAME" =~ ^v[0-9]+\.[0-9]+\.[0-9]+$ ]]`. The TAG_NAME variable (sourced from github.event.release.tag_name via the env block) was expanded unquoted, which could allow shell metacharacters in the value to be interpreted before the regex match. Adding double quotes ensures the value is treated as a single string.

