<!-- markdownlint-disable -->

# Hardening Report: openai--codex-action--/v1.10

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **openai--codex-action--/v1.10** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### hardcoded-credentials (severity: high)

A literal OpenAI API key is hardcoded in examples/test-sandbox-protections.yml: `openai-api-key: sk-proj-abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ`. Even though this appears to be a demonstration value, it matches the hardcoded-credentials pattern and should be replaced with a secrets expression such as `${{ secrets.OPENAI_API_KEY }}`.

Locations:

- `examples/test-sandbox-protections.yml:26`

### unpinned-uses (severity: high)

Multiple `uses:` references use mutable tags instead of full 40-character commit SHAs: `openai/codex-action@v1` (line 22 of test-sandbox-protections.yml), `actions/checkout@v5` (line 20 of unprivileged-user.yml), and `openai/codex-action@v1` (line 31 of unprivileged-user.yml). These are vulnerable to supply-chain attacks if the tag is moved to a different commit.

Locations:

- `examples/test-sandbox-protections.yml:22`
- `examples/unprivileged-user.yml:20`
- `examples/unprivileged-user.yml:31`

### missing-permissions (severity: medium)

.github/workflows/ci.yml has no top-level `permissions:` key and no job-level `permissions:` key on any job. Without explicit permissions, the workflow inherits the default repository permissions, which may be overly broad (e.g. write access to contents).

Locations:

- `.github/workflows/ci.yml:1`

### suspicious-run-content (severity: high)

process-memory-access: A `run:` block in examples/test-sandbox-protections.yml reads from `/proc/$pid/mem` to access another process's memory. This pattern (`mem="/proc/$pid/mem"`) is used by credential-stealing malware to extract secrets from runner processes. While this file is presented as a security demonstration, it still constitutes a flagged pattern under the process-memory-access check.

Locations:

- `examples/test-sandbox-protections.yml:43`

## Iteration Notes

### Iteration 1

**Fixes applied:** hardcoded-credentials, unpinned-uses, missing-permissions, suspicious-run-content

**Notes:**

Fixed all four findings: (1) Replaced hardcoded OpenAI API key `sk-proj-abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ` with `${{ secrets.OPENAI_API_KEY }}` in examples/test-sandbox-protections.yml. (2) Pinned all unpinned action references: `openai/codex-action@v1` → `@6b771854be52d8637d060e4b15120b9d746282ab # v1` in both example files, and `actions/checkout@v5` → `@93cb6efe18208431cddfb8368fd83d5badbf9bfd # v5` in examples/unprivileged-user.yml. (3) Added `permissions: contents: read` top-level block to .github/workflows/ci.yml. (4) Removed the flagged `/proc/$pid/mem` process memory access code from examples/test-sandbox-protections.yml and replaced it with an explanatory comment.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Determine server info path' step in action.yml (line ~169). The `server_info_file` value (constructed from user-controlled `CODEX_HOME` and `CODEX_RUN_ID`) is now sanitized with `safe_server_info_file=$(printf '%s' "$server_info_file" | tr -d '\n\r')` before being written to `$GITHUB_OUTPUT`, preventing newline injection attacks that could poison subsequent GITHUB_OUTPUT entries.

