<!-- markdownlint-disable -->

# Hardening Report: openai--codex-action/v1.9

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **openai--codex-action/v1.9** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Determine server info path' step, the variable `server_info_file` is constructed from `$CODEX_HOME` (sourced from `steps.resolve_home.outputs.codex-home`, a `steps.*.outputs.*` value) and `$CODEX_RUN_ID` (sourced from `github.run_id`, a `github.*` value). Both are untrusted per the check rules. The composed value is then written directly to `$GITHUB_OUTPUT` via `echo "server_info_file=$server_info_file" >> "$GITHUB_OUTPUT"` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A caller-controlled Codex home path or a crafted run ID containing newlines could inject arbitrary key=value pairs into the GitHub output environment.

Locations:

- `action.yml:175`

### missing-permissions (severity: medium)

The workflow file `.github/workflows/ci.yml` has no top-level `permissions:` key and its only job (`verify`) also has no job-level `permissions:` key. Without explicit permissions, the job inherits the repository's default token permissions, which may be overly broad (e.g., write access to contents). A minimal permissions block such as `permissions: contents: read` should be added.

Locations:

- `.github/workflows/ci.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, missing-permissions

**Notes:**

1. Fixed github-env-injection in action.yml line 175: Added sanitization of the `server_info_file` value using `printf '%s' "$server_info_file" | tr -d '\n\r'` before writing to $GITHUB_OUTPUT. The sanitized value is stored in `safe_server_info_file` which is then written to the output instead of the raw composed path. 2. Fixed missing-permissions in .github/workflows/ci.yml: Added a top-level `permissions: contents: read` block, which is the minimal permission required for the workflow (checkout only needs read access to repository contents).

