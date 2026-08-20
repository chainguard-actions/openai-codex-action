<!-- markdownlint-disable -->

# Hardening Report: openai--codex-action/v1.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **openai--codex-action/v1.6** was hardened automatically. 7 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in .github/workflows/ci.yml are pinned to mutable tags rather than full 40-character commit SHAs: `actions/checkout@v6` (line 14), `pnpm/action-setup@v4` (line 17), `actions/setup-node@v6` (line 21). These can be silently updated by the upstream maintainer, enabling supply-chain attacks.

Locations:

- `.github/workflows/ci.yml:14`
- `.github/workflows/ci.yml:17`
- `.github/workflows/ci.yml:21`

### unpinned-uses (severity: high)

`uses: contributor-assistant/github-action@v2.6.1` in cla.yml is pinned to a mutable version tag rather than a full commit SHA. This workflow also uses `pull_request_target`, making an unpinned action especially dangerous.

Locations:

- `.github/workflows/cla.yml:19`

### unpinned-uses (severity: high)

Example files reference actions with mutable tags: `openai/codex-action@v1` in test-sandbox-protections.yml (line 24), and `actions/checkout@v5` (line 18) and `openai/codex-action@v1` (line 33) in unprivileged-user.yml. These should be pinned to full commit SHAs.

Locations:

- `examples/test-sandbox-protections.yml:24`
- `examples/unprivileged-user.yml:18`
- `examples/unprivileged-user.yml:33`

### permissions (severity: medium)

missing-permissions: .github/workflows/ci.yml has no top-level `permissions:` key and no job-level `permissions:` key on any job. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad.

Locations:

- `.github/workflows/ci.yml:1`

### hardcoded-credentials (severity: high)

A literal OpenAI API key is hardcoded in examples/test-sandbox-protections.yml: `openai-api-key: sk-proj-abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ`. Even if intended as a placeholder/example, this matches the real key format and should be replaced with a `${{ secrets.* }}` reference.

Locations:

- `examples/test-sandbox-protections.yml:28`

### suspicious-run-content (severity: high)

process-memory-access: A `run:` block in examples/test-sandbox-protections.yml reads directly from `/proc/$pid/mem` to dump another process's memory. Matching pattern: `/proc/$pid/mem`. While this is presented as a demonstration of an attack, the code is live and executable in the workflow.

Locations:

- `examples/test-sandbox-protections.yml:40`

### github-env-injection (severity: high)

In the 'Determine server info path' step of action.yml, the value `server_info_file` — composed from `$CODEX_HOME` (sourced from `steps.resolve_home.outputs.codex-home`, a workflow-controllable step output) and `$CODEX_RUN_ID` (from `github.run_id`) — is written directly to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A newline embedded in the step output could inject additional key=value pairs into GITHUB_OUTPUT.

Locations:

- `action.yml:121`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, hardcoded-credentials, suspicious-run-content, github-env-injection

**Notes:**

Fixed all 7 findings across 5 files:

1. ci.yml: Pinned actions/checkout@v6→@d23441a48e516b6c34aea4fa41551a30e30af803, pnpm/action-setup@v4→@fc06bc1257f339d1d5d8b3a19a8cae5388b55320 (v4.4.0), actions/setup-node@v6→@249970729cb0ef3589644e2896645e5dc5ba9c38; added top-level `permissions: contents: read`.

2. cla.yml: Pinned contributor-assistant/github-action@v2.6.1→@ca4a40a7d1004f18d9960b404b97e5f30a505a08.

3. examples/test-sandbox-protections.yml: Pinned openai/codex-action@v1→@c25d10f3f498316d4b2496cc4c6dd58057a7b031 (v1.6); replaced hardcoded API key `sk-proj-abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ` with `${{ secrets.OPENAI_API_KEY }}`; removed live /proc/$pid/mem memory-scanning code and replaced with a safe explanatory comment; moved matrix.safety-strategy expression to env block.

4. examples/unprivileged-user.yml: Pinned actions/checkout@v5→@fbc6f3992d24b796d5a048ff273f7fcc4a7b6c09; pinned openai/codex-action@v1→@c25d10f3f498316d4b2496cc4c6dd58057a7b031 (v1.6).

5. action.yml: Added `printf '%s' "$server_info_file" | tr -d '\n\r'` sanitization before writing server_info_file to $GITHUB_OUTPUT to prevent newline injection.

