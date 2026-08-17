<!-- markdownlint-disable -->

# Hardening Report: devops-infra--action-commit-push/v1.3.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **devops-infra--action-commit-push/v1.3.4** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In entrypoint.sh, the variable BRANCH (derived from INPUT_TARGET_BRANCH, a user-controlled action input) and FILES_CHANGED (from git diff output, which can contain attacker-controlled filenames with embedded newlines) are written to $GITHUB_OUTPUT without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). Specifically, `echo "branch_name=${BRANCH}" >> "${GITHUB_OUTPUT}"` writes an unsanitized user-controlled value. An attacker can inject newlines into the branch name to poison GITHUB_OUTPUT with arbitrary key=value pairs.

Locations:

- `entrypoint.sh:282`

### unpinned-uses (severity: high)

Multiple unpinned references found:

1. action.yml uses `image: docker://devopsinfra/action-commit-push:v1.3.4` — a mutable Docker tag instead of a SHA digest (e.g. `@sha256:<64-hex-char-digest>`). This is vulnerable to supply-chain attacks if the tag is overwritten.

2. All 6 workflow files reference reusable workflows using mutable tags (`@v1` or `@master`) instead of pinned 40-character commit SHAs:
   - auto-pull-request-create.yml: `devops-infra/.github/.github/workflows/reusable-auto-pull-request-create.yml@v1`
   - auto-release-create.yml: `devops-infra/.github/.github/workflows/reusable-auto-release-create.yml@v1`
   - cron-dependency-update.yml: `devops-infra/.github/.github/workflows/reusable-cron-dependency-update.yml@v1`
   - manual-e2e-validate.yml: `devops-infra/triglav/.github/workflows/e2e-action-commit-push.yml@master`
   - manual-release-branch-prepare.yml: `devops-infra/.github/.github/workflows/reusable-manual-release-branch-prepare.yml@v1`
   - manual-release-create.yml: `devops-infra/.github/.github/workflows/reusable-manual-release-create.yml@v1`

Locations:

- `action.yml:71`
- `.github/workflows/auto-pull-request-create.yml:15`
- `.github/workflows/auto-release-create.yml:25`
- `.github/workflows/cron-dependency-update.yml:13`
- `.github/workflows/manual-e2e-validate.yml:22`
- `.github/workflows/manual-release-branch-prepare.yml:27`
- `.github/workflows/manual-release-create.yml:26`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, unpinned-uses

**Notes:**

Fixed github-env-injection in entrypoint.sh by sanitizing the BRANCH variable with `printf '%s' "${BRANCH}" | tr -d '\n\r'` before writing to GITHUB_OUTPUT. Fixed unpinned-uses by: (1) pinning the Docker image in action.yml to sha256:42d08d0fe03b03d17aaeee4e6b147c34d5c0cd461422c33e7f00117d338e398f, (2) pinning all 5 devops-infra/.github reusable workflow references from @v1 to SHA 88b425df3a0f0415f12a0678b84d065562363bdc, and (3) pinning the devops-infra/triglav workflow reference from @master to SHA 226f4dbb31815fc2d1892e7600b87972f051ec75. All original tags preserved as inline comments.

