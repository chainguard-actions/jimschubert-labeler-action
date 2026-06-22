<!-- markdownlint-disable -->

# Hardening Report: jimschubert--labeler-action/v2.1.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **jimschubert--labeler-action/v2.1.3** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yml references a Docker image using a mutable tag (`docker://jimschubert/labeler-action:v2`) instead of an immutable SHA digest. This means the image could be silently replaced with a different (potentially malicious) version without any change to the action definition. The image reference should be pinned to a specific SHA digest, e.g. `docker://jimschubert/labeler-action@sha256:<64-hex-char-digest>`.

Locations:

- `action.yml:31`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned the Docker image reference in action.yml from the mutable tag `docker://jimschubert/labeler-action:v2` to the immutable digest `docker://jimschubert/labeler-action@sha256:b303f564fd7df178f6aea821080c9fc9b2cf9453e69c99d0baf32f48f2fa3a96 # v2`. The original tag is preserved as a comment for readability.

