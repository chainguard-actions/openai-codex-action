<!-- markdownlint-disable -->

# Hardening Report: openai--codex-action/v1.12

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **openai--codex-action/v1.12** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b) violation: In the 'Install Codex CLI' and 'Install Codex Responses API proxy' steps, the env var CODEX_VERSION (sourced from inputs['codex-version'], an untrusted caller-controlled input) is expanded as ${CODEX_VERSION} inside a double-quoted string in a run: block: `npm install -g "@openai/codex@${CODEX_VERSION}"`. Although outer double-quotes prevent word splitting, bash still evaluates command substitutions ($(...) or backticks) embedded in the value, allowing an attacker-supplied version string to execute arbitrary shell commands.

Locations:

- `action.yml:163`
- `action.yml:170`

### github-env-injection (severity: high)

In the 'Determine server info path' step, the variable server_info_file is constructed from $CODEX_HOME (set from steps.resolve_home.outputs.codex-home, a step output treated as untrusted) and written directly to $GITHUB_OUTPUT without sanitization: `server_info_file="$CODEX_HOME/$CODEX_RUN_ID.json"` followed by `echo "server_info_file=$server_info_file" >> "$GITHUB_OUTPUT"`. A newline character embedded in the step output value could inject additional key=value pairs into GITHUB_OUTPUT. The required sanitization step (`printf '%s' ... | tr -d '\n\r'`) is absent.

Locations:

- `action.yml:178`

### missing-permissions (severity: medium)

The workflow file ci.yml has no top-level `permissions:` key and the single job 'verify' also has no job-level `permissions:` key. This means the workflow runs with GitHub's default token permissions (which include write access to contents and other scopes), violating the principle of least privilege.

Locations:

- `.github/workflows/ci.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, missing-permissions

**Notes:**

Fixed three security findings: (1) script-injection in 'Install Codex CLI' and 'Install Codex Responses API proxy' steps by sanitizing CODEX_VERSION through `printf '%s' "$CODEX_VERSION" | tr -d '\n\r'` into a safe_version variable before embedding in npm install command; (2) github-env-injection in 'Determine server info path' step by sanitizing server_info_file with `printf '%s' ... | tr -d '\n\r'` before writing to $GITHUB_OUTPUT; (3) missing-permissions in ci.yml by adding top-level `permissions: contents: read` block.

