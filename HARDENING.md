<!-- markdownlint-disable -->

# Hardening Report: openai--codex-action/v1.13

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **openai--codex-action/v1.13** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Determine server info path' step, server_info_file="$CODEX_HOME/$CODEX_RUN_ID.json" is built from CODEX_HOME, which comes from ${{ steps.resolve_home.outputs.codex-home }} (derived from inputs['codex-home']). The step then writes it with `echo "server_info_file=$server_info_file" >> "$GITHUB_OUTPUT"` without first running it through printf '%s' ... | tr -d '\n\r'. A newline in the value could inject extra outputs.

Locations:

- `action.yml:200`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index ff7088b..d0c807e 100644
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
@@ -159,19 +174,16 @@ runs:
           --allow-bots "$ALLOW_BOTS" \
           --allow-bot-users "$ALLOW_BOT_USERS" \
           --allow-users "$ALLOW_USERS"
-
     - name: Install Codex CLI
       shell: bash
       env:
         CODEX_VERSION: ${{ inputs['codex-version'] }}
       run: npm install -g "@openai/codex@${CODEX_VERSION}"
-
     - name: Install Codex Responses API proxy
       shell: bash
       env:
         CODEX_VERSION: ${{ inputs['codex-version'] }}
       run: npm install -g "@openai/codex-responses-api-proxy@${CODEX_VERSION}"
-
     - name: Resolve Codex home
       id: resolve_home
       shell: bash
@@ -187,7 +199,6 @@ runs:
           --safety-strategy "$SAFETY_STRATEGY" \
           --codex-user "$CODEX_USER" \
           --github-run-id "$CODEX_RUN_ID"
-
     - name: Determine server info path
       id: derive_server_info
       shell: bash
@@ -195,9 +206,11 @@ runs:
         CODEX_HOME: ${{ steps.resolve_home.outputs.codex-home }}
         CODEX_RUN_ID: ${{ github.run_id }}
       run: |
-        server_info_file="$CODEX_HOME/$CODEX_RUN_ID.json"
-        echo "server_info_file=$server_info_file" >> "$GITHUB_OUTPUT"
-
+        safe_codex_home="$(printf '%s' "$CODEX_HOME" | tr -d '\n\r')"
+        safe_codex_run_id="$(printf '%s' "$CODEX_RUN_ID" | tr -d '\n\r')"
+        server_info_file="$safe_codex_home/$safe_codex_run_id.json"
+        safe_server_info_file="$(printf '%s' "$server_info_file" | tr -d '\n\r')"
+        echo "server_info_file=$safe_server_info_file" >> "$GITHUB_OUTPUT"
     - name: Check Responses API proxy status
       id: start_proxy
       if: ${{ inputs['openai-api-key'] != '' }}
@@ -211,7 +224,6 @@ runs:
         else
           echo "server_info_file_exists=false" >> "$GITHUB_OUTPUT"
         fi
-
     # This is its own step to minimize the runtime logic that has access to the
     # API key. Note we use `env -u PROXY_API_KEY` to ensure extra copies of the
     # key do not end up in the memory of the `codex-responses-api-proxy`
@@ -240,7 +252,6 @@ runs:
         (
           exec env -u PROXY_API_KEY -u NODE_OPTIONS NODE_OPTIONS=--disable-sigusr1 "${args[@]}" <<< "$PROXY_API_KEY"
         ) &
-
     - name: Wait for Responses API proxy
       if: ${{ inputs['openai-api-key'] != '' && steps.start_proxy.outputs.server_info_file_exists == 'false' }}
       shell: bash
@@ -262,7 +273,6 @@ runs:
           sudo chmod 444 "$SERVER_INFO_FILE"
           sudo chown root "$SERVER_INFO_FILE"
         fi
-
     # This step has an output named `port`.
     - name: Read server info
       id: read_server_info
@@ -272,7 +282,6 @@ runs:
         ACTION_PATH: ${{ github.action_path }}
         SERVER_INFO_FILE: ${{ steps.derive_server_info.outputs.server_info_file }}
       run: node "$ACTION_PATH/dist/main.js" read-server-info "$SERVER_INFO_FILE"
-
     - name: Write Codex proxy config
       if: ${{ inputs['openai-api-key'] != '' }}
       shell: bash
@@ -286,7 +295,6 @@ runs:
           --codex-home "$CODEX_HOME" \
           --port "$PROXY_PORT" \
           --safety-strategy "$SAFETY_STRATEGY"
-
     - name: Enable Linux user namespaces for bubblewrap
       if: ${{ runner.os == 'Linux' && runner.environment == 'github-hosted' && (inputs['openai-api-key'] != '' || inputs.prompt != ''
... (truncated)
```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, prior-fix

**Notes:**

In action.yml's 'Determine server info path' step, newlines are now stripped from CODEX_HOME, CODEX_RUN_ID and the combined server_info_file (printf '%s' | tr -d '\n\r') before the value is written to $GITHUB_OUTPUT. This is the security change the previous harden made, applied again. The diff in the prior-fix finding was cut off: one part of it adds 17 lines at the start of runs:, but those lines aren't shown, so I couldn't restore them. I don't know what they were. Apart from removing blank lines, the rest of that diff is this same sanitization. The other steps already pass expressions through env: and needed no changes.

