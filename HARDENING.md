<!-- markdownlint-disable -->

# Hardening Report: openai--codex-action/v1.11

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **openai--codex-action/v1.11** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Determine server info path' step builds server_info_file="$CODEX_HOME/$CODEX_RUN_ID.json". CODEX_HOME comes from steps.resolve_home.outputs.codex-home, which is derived from inputs['codex-home']. The step then writes it to $GITHUB_OUTPUT with `echo "server_info_file=$server_info_file" >> "$GITHUB_OUTPUT"`. Nothing strips newlines first (no printf '%s' ... | tr -d '\n\r'), so a value with a newline in it could add extra output entries.

Locations:

- `action.yml:186`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 50e3079..71cf36b 100644
--- a/action.yml
+++ b/action.yml
@@ -117,6 +117,23 @@ outputs:
 runs:
   using: "composite"
   steps:
     - name: Validate Windows safety strategy
       if: ${{ runner.os == 'Windows' }}
       shell: bash
@@ -128,14 +145,12 @@ runs:
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
@@ -149,19 +164,16 @@ runs:
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
@@ -177,7 +189,6 @@ runs:
           --safety-strategy "$SAFETY_STRATEGY" \
           --codex-user "$CODEX_USER" \
           --github-run-id "$CODEX_RUN_ID"
-
     - name: Determine server info path
       id: derive_server_info
       shell: bash
@@ -187,7 +198,6 @@ runs:
       run: |
         server_info_file="$CODEX_HOME/$CODEX_RUN_ID.json"
         echo "server_info_file=$server_info_file" >> "$GITHUB_OUTPUT"
-
     - name: Check Responses API proxy status
       id: start_proxy
       if: ${{ inputs['openai-api-key'] != '' }}
@@ -201,7 +211,6 @@ runs:
         else
           echo "server_info_file_exists=false" >> "$GITHUB_OUTPUT"
         fi
-
     # This is its own step to minimize the runtime logic that has access to the
     # API key. Note we use `env -u PROXY_API_KEY` to ensure extra copies of the
     # key do not end up in the memory of the `codex-responses-api-proxy`
@@ -227,7 +236,6 @@ runs:
         (
           printenv PROXY_API_KEY | env -u PROXY_API_KEY "${args[@]}"
         ) &
-
     - name: Wait for Responses API proxy
       if: ${{ inputs['openai-api-key'] != '' && steps.start_proxy.outputs.server_info_file_exists == 'false' }}
       shell: bash
@@ -249,7 +257,6 @@ runs:
           sudo chmod 444 "$SERVER_INFO_FILE"
           sudo chown root "$SERVER_INFO_FILE"
         fi
-
     # This step has an output named `port`.
     - name: Read server info
       id: read_server_info
@@ -259,7 +266,6 @@ runs:
         ACTION_PATH: ${{ github.action_path }}
         SERVER_INFO_FILE: ${{ steps.derive_server_info.outputs.server_info_file }}
       run: node "$ACTION_PATH/dist/main.js" read-server-info "$SERVER_INFO_FILE"
-
     - name: Write Codex proxy config
       if: ${{ inputs['openai-api-key'] != '' }}
       shell: bash
@@ -273,7 +279,6 @@ runs:
           --codex-home "$CODEX_HOME" \
           --port "$PROXY_PORT" \
           --safety-strategy "$SAFETY_STRATEGY"
-
     - name: Enable Linux user namespaces for bubblewrap
       if: ${{ runner.os == 'Linux' && runner.environment == 'github-hosted' && (inputs['openai-api-key'] != '' || inputs.prompt != '' || inputs['prompt-file'] != '') }}
       shell: bash
@@ -299,7 +304,6 @@ runs:
           echo "Disabling kernel.apparmor_restrict_unprivileged_userns for bubblewrap."
           sudo sysctl -w kernel.apparmor_restrict_unprivileged_userns=0
         fi
-
     - name: Drop sudo privilege, if appropriate
       if: ${{ inputs['safety-strategy'] == 'drop-sudo' && inputs['openai-api-key'] != '' }}
       shell: bash
@@ -318,7 +322,6 @@ runs:
             exit 1
             ;;
         esac
-
     - name: Verify sudo privilege re
... (truncated)
```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, prior-fix

**Notes:**

github-env-injection: in the 'Determine server info path' step, newlines are now removed from server_info_file with printf '%s' | tr -d '\n\r' before it is written to $GITHUB_OUTPUT.

prior-fix: I checked the parts of the previous patch that were not cut off. In action.yml, all inputs and context values already go through env: blocks and are quoted in the shell, and actions/setup-node is pinned to a full commit SHA. The rest of the visible diff only deletes blank lines between steps, which has no security effect, so I did not repeat it.

The patch was cut off. It adds 17 lines at the top of the steps list that aren't visible, so I couldn't tell what they were and didn't add them. Whoever reviews this should check what those lines were.

