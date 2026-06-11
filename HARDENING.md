<!-- markdownlint-disable -->

# Hardening Report: peter-evans--enable-pull-request-automerge/v3.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **peter-evans--enable-pull-request-automerge/v3.0.0** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Lowercase' step directly interpolates `${{ inputs.merge-method }}` inside a `run:` shell command string. Before the shell ever sees the command, GitHub Actions substitutes the expression value verbatim into the script text, allowing an attacker-controlled value to inject arbitrary shell commands. Offending line: `echo merge-method=$(echo "${{ inputs.merge-method }}" | tr '[:upper:]' '[:lower:]') >> $GITHUB_OUTPUT`

Locations:

- `action.yml:23`

### script-injection (severity: high)

Sub-rule (a): The 'Enable automerge' step directly interpolates three expressions into a `run:` shell command: `${{ inputs.repository }}` (double-quoted but still template-substituted before shell execution), `${{ steps.lowercase.outputs.merge-method }}` (completely unquoted — allows shell metacharacter injection), and `${{ inputs.pull-request-number }}` (double-quoted but still unsafe). Offending line: `run: gh pr merge -R "${{ inputs.repository }}" --${{ steps.lowercase.outputs.merge-method }} --auto "${{ inputs.pull-request-number }}"`

Locations:

- `action.yml:27`

### github-env-injection (severity: high)

The 'Lowercase' step writes `inputs.merge-method` — an untrusted caller-controlled input — to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`). A newline character embedded in the input value can inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially poisoning subsequent steps. Offending line: `echo merge-method=$(echo "${{ inputs.merge-method }}" | tr '[:upper:]' '[:lower:]') >> $GITHUB_OUTPUT`

Locations:

- `action.yml:23`

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

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

Fixed all 6 findings in action.yml:

1. Lowercase step: Moved `${{ inputs.merge-method }}` to env block as MERGE_METHOD. Added sanitization with `printf '%s' "$MERGE_METHOD" | tr -d '\n\r'` before writing to $GITHUB_OUTPUT to prevent newline injection.

2. Enable automerge step: Moved all three expressions to env block — `${{ inputs.repository }}` as REPOSITORY, `${{ steps.lowercase.outputs.merge-method }}` as MERGE_METHOD, and `${{ inputs.pull-request-number }}` as PR_NUMBER. The run: command now uses only plain shell variable references ($REPOSITORY, $MERGE_METHOD, $PR_NUMBER).

No ${{ }} expressions remain in any run: block.

