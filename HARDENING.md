<!-- markdownlint-disable -->

# Hardening Report: adRise--update-pr-branch/v0.9.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **adRise--update-pr-branch/v0.9.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The workflow file uses action references pinned to mutable tags instead of immutable 40-character commit SHAs. This exposes the workflow to supply-chain attacks if the tag is moved or the upstream repository is compromised. Failing references: `actions/checkout@v2` (line 14) and `coverallsapp/github-action@v1.1.2` (line 20). Each should be pinned to a full SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v2`.

Locations:

- `.github/workflows/unit_test.yml:14`
- `.github/workflows/unit_test.yml:20`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and the only job (`unit_test`) also has no job-level `permissions:` key. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (write access to contents, pull-requests, etc.). A minimal permissions block such as `permissions: read-all` or specific scopes should be added at the top level or on the job.

Locations:

- `.github/workflows/unit_test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions

**Notes:**

Fixed .github/workflows/unit_test.yml: (1) Pinned actions/checkout@v2 to full SHA ee0669bd1cc54295c223e0bb666b733df41de1c5 and coverallsapp/github-action@v1.1.2 to full SHA 8cbef1dea373ebce56de0a14c68d6267baa10b44, preserving original tags as inline comments. (2) Added top-level `permissions: contents: read` block to restrict the workflow token to the minimum required permissions.

