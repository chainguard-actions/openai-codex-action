<!-- markdownlint-disable -->

# Hardening Report: openai--codex-action/v1.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **openai--codex-action/v1.6** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Determine server info path' step, the env vars CODEX_HOME (sourced from `steps.resolve_home.outputs.codex-home`, a step output) and CODEX_RUN_ID (sourced from `github.run_id`, a github.* context) are concatenated and written to $GITHUB_OUTPUT without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker who can influence the codex-home step output or the run_id could inject newlines to poison subsequent GITHUB_OUTPUT entries. The offending line is: `echo "server_info_file=$server_info_file" >> "$GITHUB_OUTPUT"`

Locations:

- `action.yml:155`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Determine server info path' step in action.yml (around line 155). Both CODEX_HOME (from step output `steps.resolve_home.outputs.codex-home`) and CODEX_RUN_ID (from `github.run_id`) are now sanitized with `printf '%s' "$VAR" | tr -d '\n\r'` before being concatenated into `server_info_file`. The final concatenated value is also sanitized before being written to $GITHUB_OUTPUT, preventing newline injection attacks.

