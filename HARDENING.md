<!-- markdownlint-disable -->

# Hardening Report: jimschubert--labeler-action/v2.1.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **jimschubert--labeler-action/v2.1.3** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks directly interpolate ${{ }} expressions into shell commands (rule a), allowing script injection. 

- dependabot-auto-merge.yml line 38: `run: gh pr review ${{ github.event.pull_request.number }} --approve --repo ${{ github.repository }}` — github context values interpolated directly into shell.
- dependabot-auto-merge.yml line 43: `run: gh pr merge ${{ github.event.pull_request.number }} --auto --squash --repo ${{ github.repository }}` — same issue.
- docker-rebuild.yml line 51: `if [ -z "${{ secrets.DOCKER_USERNAME }}" ]` — expression in run: block.
- docker-rebuild.yml line 63: `if [ -n "${{ inputs.tag }}" ]` / `TAG="${{ inputs.tag }}"` — attacker-controlled workflow_dispatch input interpolated directly.
- docker-rebuild.yml line 68: `TAG=$(gh api repos/${{ github.repository }}/tags ...)` — github context in shell.
- docker-rebuild.yml line 96: `ARTIFACTS='${{ steps.goreleaser.outputs.artifacts }}'` — step output interpolated directly into shell variable assignment.
- publish-action.yml line 30: `MAJOR_VERSION="${{ github.event.inputs.major_version }}"` — workflow_dispatch input in run: block.
- publish-action.yml line 38: `run: git tag -f ${{ github.event.inputs.major_version }} ${{ github.event.inputs.target }}` — attacker-controlled inputs used directly as git tag arguments.
- publish-action.yml line 39: `run: git push origin ${{ github.event.inputs.major_version }} --force` — same.
- update-major-tag.yml line 44: `if [ -n "${{ inputs.tag }}" ]` / `TAG="${{ inputs.tag }}"` — workflow_dispatch input interpolated directly.
- update-major-tag.yml line 72: `git tag -fa ${{ steps.version.outputs.major }} ${{ steps.version.outputs.tag }} -m "Update ${{ steps.version.outputs.major }} to ${{ steps.version.outputs.tag }}"` — step outputs interpolated directly into git command.

Locations:

- `.github/workflows/dependabot-auto-merge.yml:38`
- `.github/workflows/dependabot-auto-merge.yml:43`
- `.github/workflows/docker-rebuild.yml:51`
- `.github/workflows/docker-rebuild.yml:63`
- `.github/workflows/docker-rebuild.yml:68`
- `.github/workflows/docker-rebuild.yml:96`
- `.github/workflows/publish-action.yml:30`
- `.github/workflows/publish-action.yml:38`
- `.github/workflows/publish-action.yml:39`
- `.github/workflows/update-major-tag.yml:44`
- `.github/workflows/update-major-tag.yml:72`

### github-env-injection (severity: high)

Untrusted input values are written to $GITHUB_OUTPUT without the required sanitization step (printf '%s' ... | tr -d '\n\r'). In docker-rebuild.yml, ${{ inputs.tag }} is interpolated into the shell and assigned to TAG, which is then written to $GITHUB_OUTPUT via `echo "tag=$TAG" >> $GITHUB_OUTPUT` with no newline stripping. In update-major-tag.yml, ${{ inputs.tag }} similarly flows into TAG and is written to $GITHUB_OUTPUT unsanitized, allowing newline injection to poison subsequent steps' environment.

Locations:

- `.github/workflows/docker-rebuild.yml:80`
- `.github/workflows/update-major-tag.yml:67`

### unpinned-uses (severity: high)

All uses: references across workflow files use mutable tags or branch refs instead of pinned 40-character SHA commit hashes, making the action vulnerable to supply-chain attacks if any referenced action is compromised or its tag is moved.

Failing references:
- community.yml: `jimschubert/labeler-action@v2`
- dependabot-auto-merge.yml: `dependabot/fetch-metadata@v2`
- docker-rebuild.yml: `actions/checkout@v6`, `actions/setup-go@v5`, `docker/setup-qemu-action@v3`, `docker/setup-buildx-action@v3`, `docker/login-action@v3.7.0`, `goreleaser/goreleaser-action@v6`, `actions/attest-build-provenance@v2`
- lint.yml: `actions/checkout@v5`, `actions/cache@v4`, `actions/setup-go@v6`, `golangci/golangci-lint-action@v8`
- publish-action.yml: `actions/checkout@v5`
- update-major-tag.yml: `actions/checkout@v6`
- build.yml: `jimschubert/.workflows/.github/workflows/go-build-goreleaser.yml@main` (mutable branch)
- pr.yml: `jimschubert/.workflows/.github/workflows/go-pr.yml@main` (mutable branch)

Additionally, action.yml uses a Docker image with a mutable tag instead of a SHA digest: `image: docker://jimschubert/labeler-action:v2`

Locations:

- `.github/workflows/community.yml:27`
- `.github/workflows/dependabot-auto-merge.yml:33`
- `.github/workflows/docker-rebuild.yml:76`
- `.github/workflows/docker-rebuild.yml:82`
- `.github/workflows/docker-rebuild.yml:86`
- `.github/workflows/docker-rebuild.yml:90`
- `.github/workflows/docker-rebuild.yml:94`
- `.github/workflows/docker-rebuild.yml:103`
- `.github/workflows/docker-rebuild.yml:130`
- `.github/workflows/lint.yml:14`
- `.github/workflows/lint.yml:19`
- `.github/workflows/lint.yml:25`
- `.github/workflows/lint.yml:29`
- `.github/workflows/publish-action.yml:24`
- `.github/workflows/update-major-tag.yml:36`
- `.github/workflows/build.yml:43`
- `.github/workflows/pr.yml:22`
- `action.yml:31`

### missing-permissions (severity: medium)

The following workflow files have no top-level permissions: block and no job-level permissions: block on any of their jobs. Without explicit permissions, workflows inherit the default repository permissions (which may be read/write for all scopes), violating the principle of least privilege.

- community.yml: no permissions block at all
- lint.yml: no permissions block at all
- pr.yml: no permissions block at all
- publish-action.yml: no permissions block at all (this workflow pushes tags via workflow_dispatch, making this especially risky)
- update-major-tag.yml: no permissions block at all (this workflow pushes tags and uses a PAT)

Locations:

- `.github/workflows/community.yml:1`
- `.github/workflows/lint.yml:1`
- `.github/workflows/pr.yml:1`
- `.github/workflows/publish-action.yml:1`
- `.github/workflows/update-major-tag.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all 4 findings across 9 files:

1. script-injection: Moved all ${{ }} expressions from run: blocks into env: sections in dependabot-auto-merge.yml (PR number and repo), docker-rebuild.yml (DOCKER_USERNAME/PASSWORD secrets, inputs.tag, github.repository, goreleaser artifacts), publish-action.yml (major_version and target inputs), and update-major-tag.yml (inputs.tag, step outputs for major/tag).

2. github-env-injection: In docker-rebuild.yml and update-major-tag.yml, values written to $GITHUB_OUTPUT are now sanitized with `printf '%s' "$VAR" | tr -d '\n\r'` before writing.

3. unpinned-uses: Pinned all 15 action references to full 40-char SHA hashes (jimschubert/labeler-action, dependabot/fetch-metadata, actions/checkout v5 and v6, actions/setup-go v5 and v6, actions/cache, docker/setup-qemu-action, docker/setup-buildx-action, docker/login-action, goreleaser/goreleaser-action, actions/attest-build-provenance, golangci/golangci-lint-action, jimschubert/.workflows reusable workflow). Also pinned the Docker image in action.yml to its sha256 digest.

4. missing-permissions: Added permissions blocks to community.yml, lint.yml, pr.yml, publish-action.yml, and update-major-tag.yml with minimal required permissions.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in .github/workflows/docker-rebuild.yml (lines 178-179). In the 'Extract Docker image info for attestation' step, IMAGE_NAME and DIGEST (derived from the untrusted goreleaser artifacts step output) were written directly to $GITHUB_OUTPUT without sanitization. Added sanitization using `printf '%s' "$VAR" | tr -d '\n\r'` for both values before writing to $GITHUB_OUTPUT, storing results in safe_image_name and safe_digest variables respectively.

