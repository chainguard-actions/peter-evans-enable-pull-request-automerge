<!-- markdownlint-disable -->

# Hardening Report: peter-evans--enable-pull-request-automerge/v3.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **peter-evans--enable-pull-request-automerge/v3.0.0** was hardened automatically. 7 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): the 'Lowercase' step puts an input expression straight into the run script: `echo merge-method=$(echo "${{ inputs.merge-method }}" | tr '[:upper:]' '[:lower:]') >> $GITHUB_OUTPUT`. Someone who controls the input can run their own shell commands. Fix: move it into env (e.g. MERGE_METHOD) and use "$MERGE_METHOD" with quotes.

Locations:

- `action.yml:23`

### script-injection (severity: high)

Rule (a): the 'Enable automerge' step puts expressions straight into the run script: `gh pr merge -R "${{ inputs.repository }}" --${{ steps.lowercase.outputs.merge-method }} --auto "${{ inputs.pull-request-number }}"`. Fix: move inputs.repository, steps.lowercase.outputs.merge-method and inputs.pull-request-number into env vars and expand them with quotes.

Locations:

- `action.yml:27`

### github-env-injection (severity: high)

The 'Lowercase' step writes a value taken from inputs.merge-method to $GITHUB_OUTPUT without the `printf '%s' ... | tr -d '\n\r'` step: `echo merge-method=$(echo "${{ inputs.merge-method }}" | tr ...) >> $GITHUB_OUTPUT`. A value with a newline in it can add extra output entries. The $GITHUB_OUTPUT path is also unquoted.

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

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 686e7c9..ffeabf5 100644
--- a/action.yml
+++ b/action.yml
@@ -16,17 +16,39 @@ inputs:
 runs:
   using: composite
   steps:
     - name: Lowercase
       id: lowercase
       shell: bash
+      env:
+        MERGE_METHOD: ${{ inputs.merge-method }}
       run: |
-        echo merge-method=$(echo "${{ inputs.merge-method }}" | tr '[:upper:]' '[:lower:]') >> $GITHUB_OUTPUT
-
+        safe=$(printf '%s' "$MERGE_METHOD" | tr '[:upper:]' '[:lower:]' | tr -d '\n\r')
+        echo "merge-method=$safe" >> "$GITHUB_OUTPUT"
     - name: Enable automerge
       shell: bash
-      run: gh pr merge -R "${{ inputs.repository }}" --${{ steps.lowercase.outputs.merge-method }} --auto "${{ inputs.pull-request-number }}"
       env:
         GH_TOKEN: ${{ inputs.token }}
+        INPUT_REPOSITORY: ${{ inputs.repository }}
+        INPUT_MERGE_METHOD: ${{ steps.lowercase.outputs.merge-method }}
+        INPUT_PR_NUMBER: ${{ inputs.pull-request-number }}
+      run: gh pr merge -R "$INPUT_REPOSITORY" --"$INPUT_MERGE_METHOD" --auto "$INPUT_PR_NUMBER"
 branding:
   icon: 'git-pull-request'
   color: 'gray-dark'

```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection, prior-fix

**Notes:**

Lowercase step: inputs.merge-method moved to env MERGE_METHOD, lowercased, newlines stripped, written to quoted "$GITHUB_OUTPUT". Enable automerge step: repository, merge-method output and PR number moved to env vars and expanded quoted in gh pr merge. Re-applies previous harden patch.

