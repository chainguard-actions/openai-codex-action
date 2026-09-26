<!-- markdownlint-disable -->

# Hardening Report: openai--codex-action/v1.12

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **openai--codex-action/v1.12** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Determine server info path' step writes an unsanitized value to $GITHUB_OUTPUT. The variable `server_info_file` is constructed from `$CODEX_HOME` (sourced from `steps.resolve_home.outputs.codex-home`, a step output — untrusted) and `$CODEX_RUN_ID` (sourced from `github.run_id` — untrusted). The line `echo "server_info_file=$server_info_file" >> "$GITHUB_OUTPUT"` writes this composed value without first applying the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A newline injected into either source value could allow an attacker to inject arbitrary key=value pairs into $GITHUB_OUTPUT, potentially overwriting subsequent step outputs.

Locations:

- `action.yml:183`

### script-injection (severity: high)

Rule (b) violation: The 'Install Codex CLI' and 'Install Codex Responses API proxy' steps expand `${CODEX_VERSION}` (sourced from `inputs['codex-version']`, an attacker-controlled input) inside a double-quoted string passed to bash: `npm install -g "@openai/codex@${CODEX_VERSION}"`. Bash still interprets command substitutions (`$(...)` and backticks) inside double-quoted strings, so a malicious value such as `$(evil_command)` in the `codex-version` input would be executed by the shell. The variable must be validated or the npm install invocation must be restructured to avoid inline expansion of untrusted input.

Locations:

- `action.yml:155`
- `action.yml:161`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Fixed two security findings in hardened/action/action.yml:

1. github-env-injection: In the 'Determine server info path' step, added sanitization of CODEX_HOME and CODEX_RUN_ID using `printf '%s' "$VAR" | tr -d '\n\r'` before composing the server_info_file path and writing it to $GITHUB_OUTPUT. The final composed value is also sanitized.

2. script-injection: In both 'Install Codex CLI' and 'Install Codex Responses API proxy' steps, added input validation that checks CODEX_VERSION against a strict allowlist regex (^[A-Za-z0-9_.~^<>=*|-]+$) before using it in the npm install command. This blocks any value containing command substitution characters like $(...) or backticks while allowing all valid npm version specifiers (semver ranges, tags, etc.).

