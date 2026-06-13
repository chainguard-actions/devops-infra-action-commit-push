<!-- markdownlint-disable -->

# Hardening Report: devops-infra--action-commit-push/v1.3.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **devops-infra--action-commit-push/v1.3.3** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yml uses a Docker image reference with a mutable version tag (`docker://devopsinfra/action-commit-push:v1.3.3`) instead of an immutable SHA digest. This means the image content could change without notice, enabling a supply-chain attack. It should be pinned to a specific SHA digest, e.g. `docker://devopsinfra/action-commit-push@sha256:<64-hex-char-digest>`.

Locations:

- `action.yml:70`

### github-env-injection (severity: high)

In entrypoint.sh, the `# Finish` block writes `branch_name=${BRANCH}` directly to `$GITHUB_OUTPUT` without first sanitizing the value with `printf '%s' "$BRANCH" | tr -d '\n\r'`. The variable `BRANCH` is derived from `INPUT_TARGET_BRANCH`, which is an action input (untrusted). A caller supplying a `target_branch` value containing embedded newlines could inject additional key=value pairs into GITHUB_OUTPUT, potentially overwriting other outputs or poisoning downstream steps. The fix is to sanitize before writing: `safe_branch=$(printf '%s' "${BRANCH}" | tr -d '\n\r')` and then `echo "branch_name=${safe_branch}" >> "${GITHUB_OUTPUT}"`.

Locations:

- `entrypoint.sh:215`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection

**Notes:**

1. Pinned Docker image in action.yml from mutable tag `devopsinfra/action-commit-push:v1.3.3` to immutable digest `devopsinfra/action-commit-push@sha256:0b8941187b4e9e7eeb6ec835fba4ae7511ee57d7af2af295f886c33907cb61bb # v1.3.3`. 2. Fixed github-env-injection in entrypoint.sh by sanitizing the BRANCH variable (derived from untrusted INPUT_TARGET_BRANCH) with `safe_branch=$(printf '%s' "${BRANCH}" | tr -d '\n\r')` before writing `branch_name=${safe_branch}` to $GITHUB_OUTPUT.

