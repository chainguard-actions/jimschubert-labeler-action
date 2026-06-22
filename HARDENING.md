<!-- markdownlint-disable -->

# Hardening Report: jimschubert--labeler-action/v1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **jimschubert--labeler-action/v1** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yml `runs.image:` field references a Docker image using a mutable `:latest` tag (`docker://jimschubert/labeler-action:latest`) instead of an immutable SHA digest (e.g., `docker://jimschubert/labeler-action@sha256:<64-hex-char-digest>`). This means the action can silently pull a different (potentially malicious) image on every run, creating a supply-chain attack vector.

Locations:

- `action.yml:26`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Replaced the mutable `docker://jimschubert/labeler-action:latest` reference in action.yml (line 26) with the immutable digest `docker://jimschubert/labeler-action@sha256:b303f564fd7df178f6aea821080c9fc9b2cf9453e69c99d0baf32f48f2fa3a96 # latest`. The digest was resolved via the Docker Registry HTTP API v2.

