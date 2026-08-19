<!-- markdownlint-disable -->

# Hardening Report: openai--codex-action/v1.8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **openai--codex-action/v1.8** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Determine server info path' step of action.yml, the value `server_info_file=$server_info_file` is written to $GITHUB_OUTPUT without sanitization. The variable `server_info_file` is composed from `$CODEX_HOME` (sourced from `steps.resolve_home.outputs.codex-home`, a step output) and `$CODEX_RUN_ID` (sourced from `github.run_id`, a GitHub context value). Both are untrusted inputs per the check rules. The required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`) is not applied before the write, allowing a newline injection attack that could poison subsequent steps reading from GITHUB_OUTPUT. The offending line is: `echo "server_info_file=$server_info_file" >> "$GITHUB_OUTPUT"`

Locations:

- `action.yml:158`

### missing-permissions (severity: medium)

The workflow file ci.yml has no top-level `permissions:` key, and its only job (`verify`) also has no job-level `permissions:` key. Without explicit permissions, the job inherits the default repository token permissions, which may be overly broad (e.g., write access to contents). A minimal permissions block such as `permissions: contents: read` should be added.

Locations:

- `.github/workflows/ci.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, missing-permissions

**Notes:**

1. Fixed github-env-injection in action.yml 'Determine server info path' step: added sanitization of the server_info_file value using `printf '%s' "$server_info_file" | tr -d '\n\r'` before writing to $GITHUB_OUTPUT, preventing newline injection via CODEX_HOME (step output) or CODEX_RUN_ID (github.run_id). 2. Fixed missing-permissions in .github/workflows/ci.yml: added top-level `permissions: contents: read` block, which is the minimum permission needed for the workflow (repository checkout).

