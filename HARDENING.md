<!-- markdownlint-disable -->

# Hardening Report: jimschubert--labeler-action/v2.1.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **jimschubert--labeler-action/v2.1.4** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yml uses a Docker image with a mutable tag (`docker://jimschubert/labeler-action:v2`) instead of a SHA digest. This means the image can be silently replaced with a different version, enabling supply-chain attacks. The image reference should use a SHA digest, e.g. `docker://jimschubert/labeler-action@sha256:<64-hex-char-digest>`.

Locations:

- `action.yml:33`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Replaced the mutable Docker image tag `docker://jimschubert/labeler-action:v2` with the immutable SHA digest `docker://jimschubert/labeler-action@sha256:b303f564fd7df178f6aea821080c9fc9b2cf9453e69c99d0baf32f48f2fa3a96 # v2` in action.yml line 33. The original tag is preserved as a comment for readability.

