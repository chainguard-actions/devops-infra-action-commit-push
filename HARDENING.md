<!-- markdownlint-disable -->

# Hardening Report: devops-infra--action-commit-push/v1.5.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **devops-infra--action-commit-push/v1.5.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All six workflow files reference reusable workflows using mutable tag or branch refs instead of pinned 40-character SHA digests. This allows supply-chain attacks if the referenced tag or branch is moved or overwritten. Failing references:
- auto-pull-request-create.yml: devops-infra/.github/.github/workflows/reusable-auto-pull-request-create.yml@v1
- auto-release-create.yml: devops-infra/.github/.github/workflows/reusable-auto-release-create.yml@v1
- cron-dependency-update.yml: devops-infra/.github/.github/workflows/reusable-cron-dependency-update.yml@v1
- manual-e2e-validate.yml: devops-infra/triglav/.github/workflows/e2e-action-commit-push.yml@master
- manual-release-branch-prepare.yml: devops-infra/.github/.github/workflows/reusable-manual-release-branch-prepare.yml@v1
- manual-release-create.yml: devops-infra/.github/.github/workflows/reusable-manual-release-create.yml@v1

Additionally, action.yml references a Docker image with a mutable version tag: `image: docker://devopsinfra/action-commit-push:v1.5.0`. This should use a SHA digest (e.g. `docker://devopsinfra/action-commit-push@sha256:<64-hex-char-digest>`) to ensure immutability.

Locations:

- `.github/workflows/auto-pull-request-create.yml:14`
- `.github/workflows/auto-release-create.yml:24`
- `.github/workflows/cron-dependency-update.yml:14`
- `.github/workflows/manual-e2e-validate.yml:18`
- `.github/workflows/manual-release-branch-prepare.yml:22`
- `.github/workflows/manual-release-create.yml:22`
- `action.yml:72`

### github-env-injection (severity: high)

In entrypoint.sh, the final block writes `branch_name=${BRANCH}` directly to `$GITHUB_OUTPUT` without sanitization. `BRANCH` is derived from `INPUT_TARGET_BRANCH` (the `target_branch` action input), which is fully user-controlled. A caller can supply a value containing newline characters to inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially overwriting other outputs or poisoning downstream steps. The required sanitization step (`safe=$(printf '%s' "$BRANCH" | tr -d '\n\r')`) is absent before the write. The `FILES_CHANGED` value is also written via a heredoc (`files_changed<<EOF ... EOF`) which is safer for the content itself, but the `branch_name` line remains unsanitized.

Locations:

- `entrypoint.sh:349`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection

**Notes:**

Fixed all 7 unpinned-uses locations: pinned 5 devops-infra/.github reusable workflow references to SHA 88b425df3a0f0415f12a0678b84d065562363bdc (v1), pinned devops-infra/triglav reference to SHA 226f4dbb31815fc2d1892e7600b87972f051ec75 (master), and pinned the Docker image in action.yml to devopsinfra/action-commit-push:v1.5.0@sha256:4fbcf57ccf8ef7059c46026d3e0f65d16478c3eab47b535f4b7c99ca59c6ca94 (preserving docker:// scheme and tag). Fixed github-env-injection in entrypoint.sh by sanitizing BRANCH with printf + tr -d '\n\r' into SAFE_BRANCH before writing branch_name to GITHUB_OUTPUT.

