<!-- markdownlint-disable -->

# Hardening Report: openai--codex-action/v1.10

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **openai--codex-action/v1.10** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Determine server info path' step, the variable `server_info_file` is constructed from `$CODEX_HOME` (sourced from `steps.resolve_home.outputs.codex-home`, a workflow-controllable step output) and `$CODEX_RUN_ID` (from `github.run_id`). This composed value is written directly to `$GITHUB_OUTPUT` via `echo "server_info_file=$server_info_file" >> "$GITHUB_OUTPUT"` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker who can influence the step output (e.g. via a malicious codex-home input) could inject newlines to poison subsequent GITHUB_OUTPUT entries.

Locations:

- `action.yml:172`

### missing-permissions (severity: medium)

The workflow file ci.yml has no top-level `permissions:` key and the single job `verify` also has no job-level `permissions:` key. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (write access to contents, packages, etc.). A minimal permissions block (e.g. `permissions: contents: read`) should be added.

Locations:

- `.github/workflows/ci.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, missing-permissions

**Notes:**

1. Fixed github-env-injection in action.yml (line 172): Added sanitization of the server_info_file value using `safe_server_info_file="$(printf '%s' "$server_info_file" | tr -d '\n\r')"` before writing to $GITHUB_OUTPUT, preventing newline injection via a malicious codex-home input. 2. Fixed missing-permissions in .github/workflows/ci.yml: Added a top-level `permissions: contents: read` block to restrict the workflow token to the minimum required permissions (read-only repository access for checkout, build, and test steps).

