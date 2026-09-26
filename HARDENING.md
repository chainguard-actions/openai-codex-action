<!-- markdownlint-disable -->

# Hardening Report: openai--codex-action/v1.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **openai--codex-action/v1.6** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Determine server info path' step of action.yml, the shell variable `server_info_file` is constructed from `$CODEX_HOME` (which is the output of the `resolve_home` step, itself derived from the user-controlled input `inputs['codex-home']`) and then written directly to `$GITHUB_OUTPUT` without sanitization:

```
server_info_file="$CODEX_HOME/$CODEX_RUN_ID.json"
echo "server_info_file=$server_info_file" >> "$GITHUB_OUTPUT"
```

An attacker who controls the `codex-home` input can inject newlines into `$CODEX_HOME`, which would allow them to inject arbitrary key=value pairs into `$GITHUB_OUTPUT`, potentially overwriting outputs consumed by later steps. The required sanitization step (`printf '%s' "$server_info_file" | tr -d '\n\r'`) is missing before the write.

Locations:

- `action.yml:163`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Determine server info path' step in action.yml (line 163). The `server_info_file` value, which is constructed from the user-controlled `$CODEX_HOME` input, is now sanitized with `printf '%s' "$server_info_file" | tr -d '\n\r'` before being written to `$GITHUB_OUTPUT`. This prevents an attacker from injecting newlines into the `codex-home` input to inject arbitrary key=value pairs into `$GITHUB_OUTPUT`.

