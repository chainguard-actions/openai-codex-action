<!-- markdownlint-disable -->

# Hardening Report: openai--codex-action--/v1.11

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **openai--codex-action--/v1.11** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### missing-permissions (severity: medium)

The workflow file .github/workflows/ci.yml has no top-level `permissions:` key and no job-level `permissions:` key on any of its jobs. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad.

Locations:

- `.github/workflows/ci.yml:1`

### github-env-injection (severity: high)

In action.yml, the 'Determine server info path' step writes a value derived from untrusted inputs to $GITHUB_OUTPUT without sanitization. Specifically, `CODEX_HOME` (sourced from `steps.resolve_home.outputs.codex-home`, a step output) and `CODEX_RUN_ID` (sourced from `github.run_id`, a github context value) are concatenated into `server_info_file` and written with `echo "server_info_file=$server_info_file" >> "$GITHUB_OUTPUT"`. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before the write. A newline embedded in either value could inject additional key=value pairs into GITHUB_OUTPUT.

Locations:

- `action.yml:196`

## Iteration Notes

### Iteration 1

**Fixes applied:** missing-permissions, github-env-injection

**Notes:**

1. Added `permissions: contents: read` top-level block to .github/workflows/ci.yml — the workflow only needs read access to repository contents for checkout, build, and git status checks. 2. Fixed github-env-injection in action.yml's 'Determine server info path' step: both CODEX_HOME (from steps.resolve_home.outputs.codex-home) and CODEX_RUN_ID (from github.run_id) are now sanitized using `printf '%s' "$VAR" | tr -d '\n\r'` before being concatenated into server_info_file and written to $GITHUB_OUTPUT.

