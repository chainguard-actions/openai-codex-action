<!-- markdownlint-disable -->

# Hardening Report: openai--codex-action/v1.9

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **openai--codex-action/v1.9** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Determine server info path' step, the value written to $GITHUB_OUTPUT is composed from $CODEX_HOME (sourced from steps.resolve_home.outputs.codex-home — a step output, treated as untrusted) and $CODEX_RUN_ID (sourced from github.run_id — a github.* context, treated as untrusted). The concatenated path is written directly to $GITHUB_OUTPUT without the required sanitization step (printf '%s' "$VAR" | tr -d '\n\r'). An attacker who can influence the step output or run_id could inject newlines to poison subsequent GITHUB_OUTPUT entries.

Offending line:
  echo "server_info_file=$server_info_file" >> "$GITHUB_OUTPUT"

Fix: sanitize each component before use, e.g.:
  safe_home=$(printf '%s' "$CODEX_HOME" | tr -d '\n\r')
  safe_run_id=$(printf '%s' "$CODEX_RUN_ID" | tr -d '\n\r')
  echo "server_info_file=$safe_home/$safe_run_id.json" >> "$GITHUB_OUTPUT"

Locations:

- `action.yml:175`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Determine server info path' step in action.yml (around line 175). Replaced the direct construction and writing of server_info_file to $GITHUB_OUTPUT with a sanitized version: both CODEX_HOME (from step output) and CODEX_RUN_ID (from github.run_id) are now passed through `printf '%s' "$VAR" | tr -d '\n\r'` to strip any embedded newlines before the path is written to $GITHUB_OUTPUT.

