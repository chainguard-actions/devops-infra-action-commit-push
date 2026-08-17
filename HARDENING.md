<!-- markdownlint-disable -->

# Hardening Report: devops-infra--action-commit-push/v1.3.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **devops-infra--action-commit-push/v1.3.3** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yml uses a Docker image reference with a mutable version tag instead of a SHA digest. `image: docker://devopsinfra/action-commit-push:v1.3.3` should be pinned to a specific SHA256 digest (e.g., `image: docker://devopsinfra/action-commit-push@sha256:<64-hex-char-digest>`) to prevent supply-chain attacks via tag mutation.

Locations:

- `action.yml:68`

### github-env-injection (severity: high)

entrypoint.sh writes unsanitized, potentially attacker-controlled values to $GITHUB_OUTPUT without applying the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). Specifically: (1) `branch_name=${BRANCH}` — where BRANCH is derived from the action input `INPUT_TARGET_BRANCH` (an untrusted caller-controlled value); (2) `${FILES_CHANGED}` written via heredoc — where FILES_CHANGED comes from `git diff --staged --name-status`, which can contain attacker-controlled filenames with embedded newlines. Newlines in these values can be used to inject additional key=value pairs into GITHUB_OUTPUT, potentially overwriting other outputs or causing unexpected behavior in downstream steps.

Locations:

- `entrypoint.sh:243`
- `entrypoint.sh:244`
- `entrypoint.sh:246`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection

**Notes:**

1. action.yml line 68: Pinned Docker image from `docker://devopsinfra/action-commit-push:v1.3.3` to `docker://devopsinfra/action-commit-push:v1.3.3@sha256:0b8941187b4e9e7eeb6ec835fba4ae7511ee57d7af2af295f886c33907cb61bb` to prevent supply-chain attacks via tag mutation. The tag is preserved inline for readability. 2. entrypoint.sh lines 243-246: Added sanitization of BRANCH and FILES_CHANGED values before writing to $GITHUB_OUTPUT. Both values are now passed through `printf '%s' ... | tr -d '\n\r'` to strip embedded newlines/carriage-returns that could be used to inject additional key=value pairs into GITHUB_OUTPUT.

### Iteration 2

**Fixes applied:** unpinned-uses

**Notes:**

Pinned all six unpinned `uses:` references to full 40-character SHA commit hashes:
- auto-pull-request-create.yml: devops-infra/.github@v1 → @88b425df3a0f0415f12a0678b84d065562363bdc # v1
- auto-release-create.yml: devops-infra/.github@v1 → @88b425df3a0f0415f12a0678b84d065562363bdc # v1
- cron-dependency-update.yml: devops-infra/.github@v1 → @88b425df3a0f0415f12a0678b84d065562363bdc # v1
- manual-e2e-validate.yml: devops-infra/triglav@master → @6b7cf10d8042334e484bdc14ea1c9b4b335cea3e # master
- manual-release-branch-prepare.yml: devops-infra/.github@v1 → @88b425df3a0f0415f12a0678b84d065562363bdc # v1
- manual-release-create.yml: devops-infra/.github@v1 → @88b425df3a0f0415f12a0678b84d065562363bdc # v1
Original tags/branches preserved as inline comments for readability.

