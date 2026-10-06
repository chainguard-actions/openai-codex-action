<!-- markdownlint-disable -->

# Hardening Report: openai--codex-action/v1.13

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **openai--codex-action/v1.13** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Determine server info path' step writes an unsanitized value to $GITHUB_OUTPUT. The variable `server_info_file` is constructed from `$CODEX_HOME` (sourced from `steps.resolve_home.outputs.codex-home`, a step output treated as untrusted) and `$CODEX_RUN_ID` (sourced from `github.run_id`, a github context value). The write `echo "server_info_file=$server_info_file" >> "$GITHUB_OUTPUT"` is performed without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A newline embedded in either source value could inject additional key=value pairs into GITHUB_OUTPUT, potentially overwriting subsequent step outputs.

Locations:

- `action.yml:183`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Determine server info path' step in action.yml (around line 183) to sanitize values before writing to $GITHUB_OUTPUT. Both CODEX_HOME (from steps.resolve_home.outputs.codex-home, treated as untrusted) and CODEX_RUN_ID (from github.run_id) are now sanitized with `printf '%s' "$VAR" | tr -d '\n\r'` before being used to construct the server_info_file path. A final sanitization pass is also applied to the full constructed path before writing to GITHUB_OUTPUT, preventing newline injection that could overwrite subsequent step outputs.

