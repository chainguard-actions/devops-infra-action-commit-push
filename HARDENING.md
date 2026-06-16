<!-- markdownlint-disable -->

# Hardening Report: devops-infra--action-commit-push/v1.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **devops-infra--action-commit-push/v1.4.0** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yml uses a Docker image reference with a mutable version tag (`docker://devopsinfra/action-commit-push:v1.4.0`) instead of a SHA digest. This means the image could be replaced with a different (potentially malicious) version without changing the action reference. It should be pinned to a specific SHA digest, e.g. `docker://devopsinfra/action-commit-push@sha256:<64-hex-char-digest> # v1.4.0`.

Locations:

- `action.yml:63`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Replaced the mutable Docker image tag `docker://devopsinfra/action-commit-push:v1.4.0` with the immutable digest `docker://devopsinfra/action-commit-push@sha256:9ef8bd7dbf6a80ebf398ec95a400cdd2e5ea57d619340eb39afe58e10aa4b508 # v1.4.0` in action.yml line 63. The original tag is preserved as a comment for readability.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed github-env-injection in entrypoint.sh: Added sanitization of the BRANCH variable before writing it to $GITHUB_OUTPUT. Introduced SAFE_BRANCH variable computed via `printf '%s' "${BRANCH}" | tr -d '\n\r'` to strip all newline and carriage return characters. The heredoc block now writes `branch_name=${SAFE_BRANCH}` instead of `branch_name=${BRANCH}`, preventing an attacker from injecting additional key=value pairs into the GitHub output context via a crafted branch name containing embedded newlines.

