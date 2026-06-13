<!-- markdownlint-disable -->

# Hardening Report: devops-infra--action-commit-push/v1.3.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **devops-infra--action-commit-push/v1.3.2** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yml uses a Docker image reference with a mutable version tag instead of an immutable SHA256 digest. The reference `docker://devopsinfra/action-commit-push:v1.3.2` can be silently replaced by a different image if the tag is moved, enabling a supply-chain attack. It should be pinned to a full SHA256 digest, e.g. `docker://devopsinfra/action-commit-push@sha256:<64-hex-char-digest> # v1.3.2`.

Locations:

- `action.yml:79`

### github-env-injection (severity: high)

entrypoint.sh writes user-controlled and externally-derived values to $GITHUB_OUTPUT without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). Specifically:

1. `branch_name=${BRANCH}` — BRANCH is derived from `INPUT_TARGET_BRANCH` (the `target_branch` action input, which is caller-controlled). An attacker can inject newlines into this value to poison subsequent output variable reads.

2. `echo -e "${FILES_CHANGED}"` — FILES_CHANGED comes from `git diff --staged --name-status`, which can contain attacker-crafted filenames with embedded newlines, allowing injection into the `files_changed` output via the heredoc block.

Neither write is preceded by the required sanitization pipeline. The fix is to sanitize each value before writing, e.g.:
```bash
safe_branch=$(printf '%s' "${BRANCH}" | tr -d '\n\r')
echo "branch_name=${safe_branch}" >> "${GITHUB_OUTPUT}"
```

Locations:

- `entrypoint.sh:246`
- `entrypoint.sh:247`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection

**Notes:**

1. action.yml line 79: Pinned Docker image from mutable tag `docker://devopsinfra/action-commit-push:v1.3.2` to immutable digest `docker://devopsinfra/action-commit-push@sha256:9fb0965e2323a8813a6266bcf22f6a4c0088e84246a6babe5b4925162bdf9b09 # v1.3.2`. 2. entrypoint.sh Finish block: Added sanitization of BRANCH and FILES_CHANGED values using `printf '%s' ... | tr -d '\n\r'` before writing to $GITHUB_OUTPUT, preventing newline injection attacks from caller-controlled inputs or attacker-crafted git filenames.

