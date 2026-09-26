<!-- markdownlint-disable -->

# Hardening Report: openai--codex-action/v1.8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **openai--codex-action/v1.8** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Determine server info path' step writes an unsanitized value to $GITHUB_OUTPUT. The variable `server_info_file` is constructed from `$CODEX_HOME` (sourced from `steps.resolve_home.outputs.codex-home`, a prior step output) and `$CODEX_RUN_ID` (sourced from `github.run_id`). Both are untrusted inputs per the check rules. The write `echo "server_info_file=$server_info_file" >> "$GITHUB_OUTPUT"` is not preceded by the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A malicious caller could inject newlines into the step output to poison subsequent GITHUB_OUTPUT entries.

Locations:

- `action.yml:163`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Determine server info path' step in action.yml (around line 163). The step now sanitizes the `server_info_file` value before writing it to $GITHUB_OUTPUT. Added `safe_server_info_file="$(printf '%s' "$server_info_file" | tr -d '\n\r')"` and changed the echo to use `$safe_server_info_file` instead of `$server_info_file`. This prevents newline injection attacks where CODEX_HOME (from a prior step output) or CODEX_RUN_ID (from github.run_id) could contain newlines that would poison subsequent GITHUB_OUTPUT entries.

