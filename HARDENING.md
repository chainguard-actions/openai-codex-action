<!-- markdownlint-disable -->

# Hardening Report: openai--codex-action/v1.11

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **openai--codex-action/v1.11** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Determine server info path' step constructs a file path from `$CODEX_HOME` (sourced from `steps.resolve_home.outputs.codex-home`, which is ultimately derived from the user-controlled input `inputs['codex-home']`) and writes it to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker could inject newlines into the `codex-home` input to poison subsequent step outputs. The offending line is: `echo "server_info_file=$server_info_file" >> "$GITHUB_OUTPUT"`

Locations:

- `action.yml:178`

### missing-permissions (severity: medium)

The workflow file ci.yml has no top-level `permissions:` key and no job-level `permissions:` key on any of its jobs. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (e.g., `write` access to contents). A minimal permissions block (e.g., `permissions: contents: read`) should be added.

Locations:

- `.github/workflows/ci.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, missing-permissions

**Notes:**

1. Fixed github-env-injection in action.yml 'Determine server info path' step: added sanitization of the server_info_file value using `printf '%s' "$server_info_file" | tr -d '\n\r'` before writing to $GITHUB_OUTPUT, preventing newline injection from the user-controlled codex-home input. 2. Fixed missing-permissions in .github/workflows/ci.yml: added top-level `permissions: contents: read` block, which is the minimal permission required for the workflow (repository checkout).

