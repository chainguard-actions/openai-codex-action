<!-- markdownlint-disable -->

# Hardening Report: openai--codex-action/v1.12

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **openai--codex-action/v1.12** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Determine server info path' step writes `echo "server_info_file=$server_info_file" >> "$GITHUB_OUTPUT"`. The value is built from CODEX_HOME, which is set from `steps.resolve_home.outputs.codex-home`. That output comes from the caller-supplied `inputs['codex-home']` override. The value is written without first running `printf '%s' ... | tr -d '\n\r'`, so a newline in it could add extra step outputs.

Locations:

- `action.yml:199`

### script-injection (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. Rule (b) violation: The 'Install Codex CLI' and 'Install Codex Responses API proxy' steps expand `${CODEX_VERSION}` (sourced from `inputs['codex-version']`, an attacker-controlled input) inside a double-quoted string passed to bash: `npm install -g "@openai/codex@${CODEX_VERSION}"`. Bash still interprets command substitutions (`$(...)` and backticks) inside double-quoted strings, so a malicious value such as `$(evil_command)` in the `codex-version` input would be executed by the shell. The variable must be validated or the npm install invocation must be restructured to avoid inline expansion of untrusted input.

Locations:

- `action.yml:155`
- `action.yml:161`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index ff7088b..4619f52 100644
--- a/action.yml
+++ b/action.yml
@@ -127,6 +127,23 @@ outputs:
 runs:
   using: "composite"
   steps:
     - name: Validate Windows safety strategy
       if: ${{ runner.os == 'Windows' }}
       shell: bash
@@ -138,14 +155,12 @@ runs:
           echo "because no viable sandboxing options are available at this time." >&2
           exit 1
         fi
-
     - name: Ensure Node.js available
       # Pin to a commit hash because some repositories require it:
       # https://github.com/openai/codex-action/issues/43
       uses: actions/setup-node@53b83947a5a98c8d113130e565377fae1a50d02f # v6.3.0
       with:
         node-version: "24"
-
     - name: Check repository write access
       env:
         ACTION_PATH: ${{ github.action_path }}
@@ -159,19 +174,26 @@ runs:
           --allow-bots "$ALLOW_BOTS" \
           --allow-bot-users "$ALLOW_BOT_USERS" \
           --allow-users "$ALLOW_USERS"
-
     - name: Install Codex CLI
       shell: bash
       env:
         CODEX_VERSION: ${{ inputs['codex-version'] }}
-      run: npm install -g "@openai/codex@${CODEX_VERSION}"
-
+      run: |
+        if [ -n "$CODEX_VERSION" ] && ! printf '%s' "$CODEX_VERSION" | grep -qE '^[A-Za-z0-9_.~^<>=*|-]+$'; then
+          echo "Invalid codex-version value" >&2
+          exit 1
+        fi
+        npm install -g "@openai/codex@${CODEX_VERSION}"
     - name: Install Codex Responses API proxy
       shell: bash
       env:
         CODEX_VERSION: ${{ inputs['codex-version'] }}
-      run: npm install -g "@openai/codex-responses-api-proxy@${CODEX_VERSION}"
-
+      run: |
+        if [ -n "$CODEX_VERSION" ] && ! printf '%s' "$CODEX_VERSION" | grep -qE '^[A-Za-z0-9_.~^<>=*|-]+$'; then
+          echo "Invalid codex-version value" >&2
+          exit 1
+        fi
+        npm install -g "@openai/codex-responses-api-proxy@${CODEX_VERSION}"
     - name: Resolve Codex home
       id: resolve_home
       shell: bash
@@ -187,7 +209,6 @@ runs:
           --safety-strategy "$SAFETY_STRATEGY" \
           --codex-user "$CODEX_USER" \
           --github-run-id "$CODEX_RUN_ID"
-
     - name: Determine server info path
       id: derive_server_info
       shell: bash
@@ -195,9 +216,11 @@ runs:
         CODEX_HOME: ${{ steps.resolve_home.outputs.codex-home }}
         CODEX_RUN_ID: ${{ github.run_id }}
       run: |
-        server_info_file="$CODEX_HOME/$CODEX_RUN_ID.json"
-        echo "server_info_file=$server_info_file" >> "$GITHUB_OUTPUT"
-
+        safe_codex_home="$(printf '%s' "$CODEX_HOME" | tr -d '\n\r')"
+        safe_codex_run_id="$(printf '%s' "$CODEX_RUN_ID" | tr -d '\n\r')"
+        server_info_file="${safe_codex_home}/${safe_codex_run_id}.json"
+        safe_server_info_file="$(printf '%s' "$server_info_file" | tr -d '\n\r')"
+        echo "server_info_file=${safe_server_info_file}" >> "$GITHUB_OUTPUT"
     - name: Check Responses API proxy status
       id: start_proxy
       if: ${{ inputs['openai-api-key'] != '' }}
@@ -211,7 +234,6 @@ runs:
         else
           echo "server_info_file_exists=false" >> "$GITHUB_OUTPUT"
         fi
-
     # This is its own step to minimize the runtime logic that has access to the
     # API key. Note we use `env -u PROXY_API_KEY` to ensure extra copies of the
     # key do not end up in the memory of the `codex-responses-api-proxy`
@@ -240,7 +262,6 @@ runs:
         (
           exec env -u PROXY_API_KEY -u NODE_OPTIONS NODE_OPTIONS=--disable-sigusr1 "${args[@]}" <<< "$PROXY_API_KEY"
         ) &
-
     - name: Wait for Responses API proxy
       if: ${{ inputs['openai-api-key'] != '' && steps.start_proxy.outputs.server_info_file_exists == 'false' }}
       shell: bash
@@ -262,7 +283,6 @@ runs:
           sudo chmod 444 "$SERVER_INFO_FILE"
           sudo chown root "$SERVER_INFO_FILE"
         fi
-
     # This step has an output named `port`.
     - name: Read server info
       id: read_server_info
@@ -272,7 +292,6 @@ runs:
         ACTION_PATH: ${{ github.action_path }}
         SERVER_INFO_FILE: ${{ steps.derive_server_info.out
... (truncated)
```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, script-injection, prior-fix

**Notes:**

Changes in action.yml:
- github-env-injection: the 'Determine server info path' step now strips newlines and carriage returns from server_info_file (printf '%s' | tr -d '\n\r') before writing it to $GITHUB_OUTPUT.
- script-injection: both npm install steps now check codex-version against an allowlist of semver/dist-tag characters ^[A-Za-z0-9_.~^<>=*|-]+$ and stop with an error if it doesn't match. An empty value still installs the way upstream did.
- prior-fix: this puts back the security changes from the previous harden. The blank-line removals from that diff were left out because they change nothing. The rest of the diff was cut off, and the steps after it already use env vars and quoting.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

In action.yml, the 'Determine server info path' step now strips newline and carriage-return characters (printf '%s' ... | tr -d '\n\r') from server_info_file before writing it to $GITHUB_OUTPUT.

