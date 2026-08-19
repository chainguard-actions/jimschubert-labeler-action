<!-- markdownlint-disable -->

# Hardening Report: jimschubert--labeler-action/v1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **jimschubert--labeler-action/v1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml uses a Docker image with a mutable tag instead of a SHA digest. The image reference `docker://jimschubert/labeler-action:latest` uses the `:latest` tag, which can change at any time and is vulnerable to supply-chain attacks. It should be pinned to a specific SHA digest, e.g. `docker://jimschubert/labeler-action@sha256:<64-hex-char-digest>`.

Locations:

- `action.yml:26`

### missing-permissions (severity: medium)

The workflow file .github/workflows/community.yml has no top-level `permissions:` key and the single job `labeler` also has no job-level `permissions:` key. Without explicit permissions, the workflow inherits the default repository permissions (which may be write-all), violating the principle of least privilege. A minimal permissions block such as `permissions: contents: read` should be added.

Locations:

- `.github/workflows/community.yml:15`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions

**Notes:**

1. Pinned the Docker image in action.yml from `docker://jimschubert/labeler-action:latest` to `docker://jimschubert/labeler-action:latest@sha256:137acbf8c270199f4a390d5336a557979b5f52341aa91dcf9303b9584a3836be`, preserving the `docker://` scheme and `:latest` tag inline. 2. Added a top-level `permissions:` block to `.github/workflows/community.yml` with `issues: write` and `pull-requests: write` — the minimum permissions needed for the labeler action to apply labels to issues and pull requests.

