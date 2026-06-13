<!-- markdownlint-disable -->

# Hardening Report: devops-infra--action-commit-push/v1.3.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **devops-infra--action-commit-push/v1.3.4** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action uses a Docker image referenced by a mutable version tag rather than an immutable SHA256 digest. `image: docker://devopsinfra/action-commit-push:v1.3.4` can be silently replaced with a different (potentially malicious) image at any time. It should be pinned to a specific SHA256 digest, e.g. `image: docker://devopsinfra/action-commit-push@sha256:<64-hex-char-digest>`

Locations:

- `action.yml:57`

### github-env-injection (severity: high)

In entrypoint.sh, the final output block writes `echo "branch_name=${BRANCH}"` directly to `$GITHUB_OUTPUT` without sanitization. `BRANCH` is set from the user-controlled input `INPUT_TARGET_BRANCH` (via `BRANCH="${INPUT_TARGET_BRANCH:-$(get_current_branch)}"`). An attacker supplying a `target_branch` value containing newline characters could inject arbitrary key=value pairs into `$GITHUB_OUTPUT`, potentially overwriting other outputs or poisoning downstream steps. The required sanitization (`safe=$(printf '%s' "$BRANCH" | tr -d '\n\r')`) is absent before the write.

Locations:

- `entrypoint.sh:243`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection

**Notes:**

1. Fixed unpinned-uses in action.yml line 57: replaced `docker://devopsinfra/action-commit-push:v1.3.4` with `docker://devopsinfra/action-commit-push@sha256:42d08d0fe03b03d17aaeee4e6b147c34d5c0cd461422c33e7f00117d338e398f # v1.3.4` using the real digest from the Docker registry. 2. Fixed github-env-injection in entrypoint.sh near line 243: introduced `SAFE_BRANCH="$(printf '%s' "${BRANCH}" | tr -d '\n\r')"` before the GITHUB_OUTPUT block, and replaced `echo "branch_name=${BRANCH}"` with `echo "branch_name=${SAFE_BRANCH}"` to strip any attacker-injected newline characters from the user-controlled INPUT_TARGET_BRANCH value.

