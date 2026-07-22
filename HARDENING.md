<!-- markdownlint-disable -->

# Hardening Report: adRise--update-pr-branch/v0.9.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **adRise--update-pr-branch/v0.9.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The workflow file uses action references pinned to mutable tags instead of immutable full-length SHA digests. `actions/checkout@v2` and `coverallsapp/github-action@v1.1.2` can be silently updated by the upstream maintainer, enabling supply-chain attacks. Each should be pinned to a full 40-character commit SHA (e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`).

Locations:

- `.github/workflows/unit_test.yml:14`
- `.github/workflows/unit_test.yml:20`

### missing-permissions (severity: medium)

The workflow file `.github/workflows/unit_test.yml` has no top-level `permissions:` block and the only job (`unit_test`) also has no job-level `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (e.g. `write` access to contents). A minimal `permissions:` block should be added (e.g. `contents: read`).

Locations:

- `.github/workflows/unit_test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions

**Notes:**

Fixed .github/workflows/unit_test.yml: (1) Pinned actions/checkout@v2 to full SHA 0717577d45739eb3c851188b29f50ed6c0b2194e and coverallsapp/github-action@v1.1.2 to full SHA 8cbef1dea373ebce56de0a14c68d6267baa10b44, preserving original tags as comments. (2) Added top-level `permissions: contents: read` block to restrict the workflow token to the minimum required access.

