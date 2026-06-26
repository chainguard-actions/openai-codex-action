<!-- markdownlint-disable -->

# Hardening Report: openai--codex-action/v1.9

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **openai--codex-action/v1.9** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Determine server info path' step writes a value derived from `steps.resolve_home.outputs.codex-home` (an untrusted step output per the check spec) to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). The variable `CODEX_HOME` is set from `${{ steps.resolve_home.outputs.codex-home }}` in the `env:` block, then used unsanitized in: `echo "server_info_file=$server_info_file" >> "$GITHUB_OUTPUT"`. A malicious value containing newlines could inject additional key=value pairs into the GitHub output context.

Locations:

- `action.yml:166`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Determine server info path' step in action.yml. The step now sanitizes the `server_info_file` value before writing it to $GITHUB_OUTPUT by using `safe_server_info_file=$(printf '%s' "$server_info_file" | tr -d '\n\r')` and then echoing the sanitized value. This prevents a malicious `CODEX_HOME` value containing newlines from injecting additional key=value pairs into the GitHub output context.

