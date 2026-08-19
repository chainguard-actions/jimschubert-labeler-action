<!-- markdownlint-disable -->

# Hardening Report: jimschubert--labeler-action/v2.1.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **jimschubert--labeler-action/v2.1.4** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files and action.yml reference actions/images by mutable tag or branch instead of a pinned 40-character commit SHA, making them vulnerable to supply-chain attacks.

action.yml: `image: docker://jimschubert/labeler-action:v2` (tag, not SHA digest)
.github/workflows/build.yml: `jimschubert/.workflows/...@main` (branch ref)
.github/workflows/community.yml: `jimschubert/labeler-action@v2`
.github/workflows/dependabot-auto-merge.yml: `dependabot/fetch-metadata@v2`
.github/workflows/docker-rebuild.yml: `actions/checkout@v6`, `actions/setup-go@v6`, `docker/setup-qemu-action@v3`, `docker/setup-buildx-action@v3`, `docker/login-action@v3.7.0`, `goreleaser/goreleaser-action@v6`, `actions/attest-build-provenance@v3`
.github/workflows/pr.yml: `jimschubert/.workflows/...@main` (branch ref)
.github/workflows/publish-action.yml: `actions/checkout@v6`
.github/workflows/update-major-tag.yml: `actions/checkout@v6`

Locations:

- `action.yml:31`
- `.github/workflows/build.yml:43`
- `.github/workflows/community.yml:26`
- `.github/workflows/dependabot-auto-merge.yml:32`
- `.github/workflows/docker-rebuild.yml:79`
- `.github/workflows/docker-rebuild.yml:84`
- `.github/workflows/docker-rebuild.yml:88`
- `.github/workflows/docker-rebuild.yml:92`
- `.github/workflows/docker-rebuild.yml:96`
- `.github/workflows/docker-rebuild.yml:130`
- `.github/workflows/docker-rebuild.yml:178`
- `.github/workflows/pr.yml:26`
- `.github/workflows/publish-action.yml:26`
- `.github/workflows/update-major-tag.yml:37`

### script-injection (severity: high)

Multiple workflow `run:` blocks directly interpolate `${{ ... }}` expressions into shell commands, allowing script injection. Attacker-controlled values flow directly into the shell before quoting can occur.

(a) publish-action.yml — `${{ github.event.inputs.major_version }}` interpolated directly in a run block: `MAJOR_VERSION="${{ github.event.inputs.major_version }}"`; also `run: git tag -f ${{ github.event.inputs.major_version }} ${{ github.event.inputs.target }}` and `run: git push origin ${{ github.event.inputs.major_version }} --force`.

(a) update-major-tag.yml — `${{ inputs.tag }}` interpolated directly in a run block: `if [ -n "${{ inputs.tag }}" ]` / `TAG="${{ inputs.tag }}"`; also `git tag -fa ${{ steps.version.outputs.major }} ${{ steps.version.outputs.tag }}` and `git push origin ${{ steps.version.outputs.major }} --force`.

(a) docker-rebuild.yml — `${{ inputs.tag }}` and `${{ github.repository }}` interpolated directly in a run block: `TAG="${{ inputs.tag }}"` and `gh api repos/${{ github.repository }}/tags`.

(a) dependabot-auto-merge.yml — `${{ github.event.pull_request.number }}` and `${{ github.repository }}` interpolated directly in run blocks: `gh pr review ${{ github.event.pull_request.number }} --approve --repo ${{ github.repository }}` and `gh pr merge ${{ github.event.pull_request.number }} --auto --squash --repo ${{ github.repository }}`.

Locations:

- `.github/workflows/publish-action.yml:32`
- `.github/workflows/publish-action.yml:40`
- `.github/workflows/publish-action.yml:42`
- `.github/workflows/update-major-tag.yml:46`
- `.github/workflows/update-major-tag.yml:47`
- `.github/workflows/update-major-tag.yml:74`
- `.github/workflows/update-major-tag.yml:75`
- `.github/workflows/docker-rebuild.yml:57`
- `.github/workflows/docker-rebuild.yml:58`
- `.github/workflows/docker-rebuild.yml:65`
- `.github/workflows/dependabot-auto-merge.yml:38`
- `.github/workflows/dependabot-auto-merge.yml:43`

### github-env-injection (severity: high)

Run blocks write values derived from user-controlled inputs to `$GITHUB_OUTPUT` without sanitization (no `printf '%s' ... | tr -d '\n\r'` step).

docker-rebuild.yml: The variable `TAG` is set from `${{ inputs.tag }}` (a workflow_dispatch input) and then written unsanitized: `echo "tag=$TAG" >> $GITHUB_OUTPUT`.

update-major-tag.yml: The variable `TAG` is set from `${{ inputs.tag }}` and `MAJOR` is derived from it; both are written unsanitized: `echo "tag=$TAG" >> $GITHUB_OUTPUT` and `echo "major=$MAJOR" >> $GITHUB_OUTPUT`.

Locations:

- `.github/workflows/docker-rebuild.yml:75`
- `.github/workflows/update-major-tag.yml:71`
- `.github/workflows/update-major-tag.yml:72`

### missing-permissions (severity: medium)

The following workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any job. Without explicit permissions, workflows run with the repository's default token permissions (which may be overly broad, e.g. `write` on all scopes for some repository configurations).

- community.yml: no permissions declared
- pr.yml: no permissions declared
- publish-action.yml: no permissions declared (pushes tags, needs at minimum `contents: write`)
- update-major-tag.yml: no permissions declared (pushes tags, needs at minimum `contents: write`)

Locations:

- `.github/workflows/community.yml:1`
- `.github/workflows/pr.yml:1`
- `.github/workflows/publish-action.yml:1`
- `.github/workflows/update-major-tag.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, missing-permissions

**Notes:**

Fixed all 4 findings across 8 files:

1. unpinned-uses: Pinned all action references to full 40-char commit SHAs and the Docker image in action.yml to its sha256 digest. Affected: action.yml, build.yml, community.yml, dependabot-auto-merge.yml, docker-rebuild.yml (7 actions), pr.yml, publish-action.yml, update-major-tag.yml.

2. script-injection: Moved all ${{ }} expressions from run: blocks into env: blocks and referenced them as plain shell variables. Affected: publish-action.yml (major_version, target inputs), update-major-tag.yml (inputs.tag), docker-rebuild.yml (inputs.tag, github.repository), dependabot-auto-merge.yml (pull_request.number, github.repository).

3. github-env-injection: Added printf '%s' ... | tr -d '\n\r' sanitization before writing to $GITHUB_OUTPUT. Affected: docker-rebuild.yml (find-tag step), update-major-tag.yml (version step, both tag and major outputs).

4. missing-permissions: Added top-level permissions blocks to community.yml (contents:read, issues:write, pull-requests:write), pr.yml (contents:read), publish-action.yml (contents:write), and update-major-tag.yml (contents:write).

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all three findings in .github/workflows/docker-rebuild.yml:
1. (script-injection, lines 55/59) Moved secrets.DOCKER_USERNAME and secrets.DOCKER_PASSWORD from inline ${{ }} interpolation in run: to an env: block; shell script now references $DOCKER_USERNAME and $DOCKER_PASSWORD.
2. (script-injection, line 133) Moved steps.goreleaser.outputs.artifacts from inline ${{ }} interpolation in run: to an env: block as GORELEASER_ARTIFACTS; shell script now uses "$GORELEASER_ARTIFACTS".
3. (github-env-injection, lines 155/156) Added printf '%s' ... | tr -d '\n\r' sanitization for IMAGE_NAME and DIGEST before writing to $GITHUB_OUTPUT, preventing newline injection attacks.

