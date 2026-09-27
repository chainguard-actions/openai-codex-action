<!-- markdownlint-disable -->

# Hardening Report: openai--codex-action/v1.10

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **openai--codex-action/v1.10** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Determine server info path' step writes an unsanitized value derived from untrusted inputs to $GITHUB_OUTPUT. Specifically, it constructs `server_info_file="$CODEX_HOME/$CODEX_RUN_ID.json"` where CODEX_HOME comes from `steps.resolve_home.outputs.codex-home` (a steps.*.outputs.* value) and CODEX_RUN_ID comes from `github.run_id` (a github.* value). Both are classified as untrusted per the check's scope. The composed value is then written directly to $GITHUB_OUTPUT via `echo "server_info_file=$server_info_file" >> "$GITHUB_OUTPUT"` without the required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`). A malicious actor who can influence the step output (e.g. via a crafted codex-home input or run_id) could inject newlines to poison subsequent GITHUB_OUTPUT entries.

Locations:

- `action.yml:162`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Determine server info path' step in hardened/action/action.yml (around line 162). The composed path value `$CODEX_HOME/$CODEX_RUN_ID.json` is now sanitized with `printf '%s' "$server_info_file" | tr -d '\n\r'` before being written to $GITHUB_OUTPUT, preventing newline injection attacks that could poison subsequent GITHUB_OUTPUT entries.

