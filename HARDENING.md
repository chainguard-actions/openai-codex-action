<!-- markdownlint-disable -->

# Hardening Report: openai--codex-action/v1.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **openai--codex-action/v1.6** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Determine server info path' step constructs a path from $CODEX_HOME (sourced from steps.resolve_home.outputs.codex-home, a steps.*.outputs.* value) and $CODEX_RUN_ID (sourced from github.run_id), then writes it directly to $GITHUB_OUTPUT without sanitization. Neither value is passed through `printf '%s' ... | tr -d '\n\r'` before the write. A malicious value containing newlines in either variable could inject arbitrary key=value pairs into the GitHub output context.

Offending line:
  echo "server_info_file=$server_info_file" >> "$GITHUB_OUTPUT"

Fix: sanitize both components before use, e.g.:
  safe_home=$(printf '%s' "$CODEX_HOME" | tr -d '\n\r')
  safe_run_id=$(printf '%s' "$CODEX_RUN_ID" | tr -d '\n\r')
  echo "server_info_file=$safe_home/$safe_run_id.json" >> "$GITHUB_OUTPUT"

Locations:

- `action.yml:163`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Determine server info path' step in action.yml (around line 163). Both CODEX_HOME (sourced from steps.resolve_home.outputs.codex-home) and CODEX_RUN_ID (sourced from github.run_id) are now sanitized using `printf '%s' "$VAR" | tr -d '\n\r'` before being used to construct the server_info_file path that is written to $GITHUB_OUTPUT. This prevents newline injection attacks that could inject arbitrary key=value pairs into the GitHub output context.

