<!-- markdownlint-disable -->

# Hardening Report: peter-evans--enable-pull-request-automerge/v3.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **peter-evans--enable-pull-request-automerge/v3.0.0** was hardened automatically. 11 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): Multiple ${{ }} expressions are interpolated directly inside run: shell commands in action.yml. Step 'Lowercase' interpolates ${{ inputs.merge-method }} directly in the shell command. Step 'Enable automerge' interpolates ${{ inputs.repository }}, ${{ steps.lowercase.outputs.merge-method }}, and ${{ inputs.pull-request-number }} directly in the shell command. These values flow through YAML template substitution before the shell sees them, enabling command injection by an attacker who controls the inputs.

Locations:

- `action.yml:22`
- `action.yml:27`

### github-env-injection (severity: high)

Step 'Lowercase' in action.yml writes the untrusted input value ${{ inputs.merge-method }} to $GITHUB_OUTPUT without the required sanitization step (printf '%s' ... | tr -d '\n\r'). The command `echo merge-method=$(echo "${{ inputs.merge-method }}" | tr '[:upper:]' '[:lower:]') >> $GITHUB_OUTPUT` does not strip newlines from the value before writing, allowing a newline-injection attack to poison subsequent GITHUB_OUTPUT entries.

Locations:

- `action.yml:22`

### script-injection (severity: high)

Rule (a): ${{ github.event.inputs.main_version }} and ${{ github.event.inputs.target }} are interpolated directly inside run: shell commands in update-major-version.yml. The 'Tag new target' step runs `git tag -f ${{ github.event.inputs.main_version }} ${{ github.event.inputs.target }}` and the 'Push new tag' step runs `git push origin ${{ github.event.inputs.main_version }} --force`. These workflow_dispatch inputs are attacker-controllable and flow through YAML template substitution before the shell parses them, enabling command injection.

Locations:

- `.github/workflows/update-major-version.yml:29`
- `.github/workflows/update-major-version.yml:31`

### script-injection (severity: high)

Rule (a): ${{ steps.cpr.outputs.pull-request-number }} is interpolated directly inside a run: shell command in ci.yml. The 'Auto approve' step runs `gh pr review --approve "${{ steps.cpr.outputs.pull-request-number }}"`. The steps.*.outputs.* context is workflow-controllable and flows through YAML template substitution before the shell parses it, enabling command injection.

Locations:

- `.github/workflows/ci.yml:88`

### unpinned-uses (severity: high)

Multiple uses: references across workflow files use mutable tag refs instead of full 40-character commit SHAs, making them vulnerable to supply-chain attacks if the referenced tag is moved. Failing references include: actions/checkout@v3, actions/upload-artifact@v3, actions/download-artifact@v3, peter-evans/create-pull-request@v4, peter-evans/enable-pull-request-automerge@v2 (in automerge-dependabot.yml), and actions/checkout@v3 (in update-major-version.yml).

Locations:

- `.github/workflows/ci.yml:19`
- `.github/workflows/ci.yml:20`
- `.github/workflows/ci.yml:31`
- `.github/workflows/ci.yml:38`
- `.github/workflows/ci.yml:47`
- `.github/workflows/automerge-dependabot.yml:9`
- `.github/workflows/update-major-version.yml:19`

### missing-permissions (severity: medium)

automerge-dependabot.yml has no top-level permissions: key and its only job ('automerge') also has no job-level permissions: key. This means the workflow runs with the default (potentially broad) token permissions.

Locations:

- `.github/workflows/automerge-dependabot.yml:1`

### missing-permissions (severity: medium)

test-required-status-check.yml has no top-level permissions: key and its only job ('check') also has no job-level permissions: key. This means the workflow runs with the default (potentially broad) token permissions.

Locations:

- `.github/workflows/test-required-status-check.yml:1`

### missing-permissions (severity: medium)

update-major-version.yml has no top-level permissions: key and its only job ('tag') also has no job-level permissions: key. This means the workflow runs with the default (potentially broad) token permissions.

Locations:

- `.github/workflows/update-major-version.yml:1`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.merge-method }}" appears directly in run: block of step "Lowercase"; move to env: map

Locations:

- `action.yml:23`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.repository }}" appears directly in run: block of step "Enable automerge"; move to env: map

Locations:

- `action.yml:28`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.pull-request-number }}" appears directly in run: block of step "Enable automerge"; move to env: map

Locations:

- `action.yml:28`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all findings across action.yml and workflow files:

1. action.yml: Moved all ${{ }} expressions out of run: blocks into env: maps. In Lowercase step, MERGE_METHOD env var is sanitized with tr -d '\n\r' before writing to $GITHUB_OUTPUT. In Enable automerge step, INPUT_REPOSITORY, INPUT_MERGE_METHOD, and INPUT_PR_NUMBER env vars replace inline expressions.

2. ci.yml: Fixed script-injection in Auto approve step by moving ${{ steps.cpr.outputs.pull-request-number }} to PR_NUMBER env var. Pinned all unpinned actions: actions/checkout@v3→SHA, actions/upload-artifact@v3→SHA, actions/download-artifact@v3→SHA (×2), peter-evans/create-pull-request@v4→SHA (×2).

3. update-major-version.yml: Fixed script-injection in Tag new target and Push new tag steps by moving ${{ github.event.inputs.main_version }} and ${{ github.event.inputs.target }} to env vars. Pinned actions/checkout@v3→SHA. Added permissions: contents: write.

4. automerge-dependabot.yml: Pinned peter-evans/enable-pull-request-automerge@v2→SHA. Added permissions: pull-requests: write, contents: write.

5. test-required-status-check.yml: Added permissions: {} (no permissions needed for this workflow).

