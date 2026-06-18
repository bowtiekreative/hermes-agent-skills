---
name: proton-pass-cli
description: "Retrieve secrets, credentials, and tokens from Proton Pass vaults using the pass-cli tool — login with PAT, session management, vault discovery, item access, and auto-recovery from expired sessions."
version: 1.2.0
tags:
  - proton-pass
  - password-manager
  - credentials
  - secrets
  - vaults
  - pat
  - keyring
platforms: [linux, macos]
metadata:
  hermes:
    triggers:
      - "pass-cli"
      - "proton pass"
      - "vault"
      - "retrieve credential"
      - "fetch token"
      - "password manager"
      - "login to .*"
      - "get api key"
    related_skills: [user-space-install, github-auth]
---

# Proton Pass CLI

Retrieve and manage secrets from Proton Pass vaults using `pass-cli`.

## Session Architecture

Proton Pass CLI uses two critical environment variables:

| Variable | Purpose | Typical value |
|---|---|---|
| `PROTON_PASS_KEY_PROVIDER` | Key storage backend. **Must be `fs`** in containers/servers without a system keyring (D-Bus/libsecret). | `fs` |
| `PROTON_PASS_SESSION_DIR` | Isolated session directory. **Must be stable** — use a fixed name per session so login state persists across commands. | `/tmp/pass-agent-hermes` |

**Always set both** before any `pass-cli` command. Without `PROTON_PASS_KEY_PROVIDER=fs`, the CLI will crash on headless environments with `Error accessing credential` from the system keyring.

### ⚠️ Critical: Session Dir Stability

The `PROTON_PASS_SESSION_DIR` **must be identical** for login AND all subsequent commands. Using a dynamic value like `$(date +%s)` creates a new directory on every shell invocation, causing "no session" errors on every command after login. Use a **fixed name** per session (e.g. `/tmp/pass-agent-hermes`).

## Prerequisites

- `pass-cli` installed (see [Installation](#installation))
- Proton Pass personal access token (PAT) stored in memory for re-auth
- `curl` and `jq` available (installer dependencies)

## Installation

```bash
# Install to ~/.local/bin (no sudo needed) — default install script
export PROTON_PASS_CLI_INSTALL_DIR="$HOME/.local/bin"
curl -fsSL https://proton.me/download/pass-cli/install.sh | bash
```

**Fallback (if install script times out):** download the binary directly. The script outputs the download URL in its `[INFO]` lines. Use the version from the manifest URL:

```bash
# Determine latest version, then download directly
LATEST=$(curl -fsSL https://proton.me/download/pass-cli/versions.json | grep -o '"latest":"[^"]*"' | cut -d'"' -f4)
# Or use a known-good version (e.g. 2.1.4)
curl -fsSL -o "$HOME/.local/bin/pass-cli" "https://proton.me/download/pass-cli/${LATEST:-2.1.4}/pass-cli-linux-x86_64"
chmod +x "$HOME/.local/bin/pass-cli"
```

## Login (PAT-based)

Never use interactive password login in an agent context — always use a Personal Access Token.

```bash
# 1. Set up isolated session with STABLE directory name
export PROTON_PASS_KEY_PROVIDER=fs
export PROTON_PASS_SESSION_DIR="/tmp/pass-agent-hermes"
mkdir -p "$PROTON_PASS_SESSION_DIR"

# 2. Login with PAT from env var
PROTON_PASS_PERSONAL_ACCESS_TOKEN="pst_<token>::<key>" pass-cli login

# 3. Verify
pass-cli info
# Expected: exit code 0, shows "Personal Access Token: [Agent] <name>"
```

**IMPORTANT:** The `PROTON_PASS_SESSION_DIR` you set before login is where the session cookie is stored. Every subsequent `pass-cli` command in the same task must use the **same** directory — set the env var consistently.

## Session Monitoring (Mandatory)

**Before every single `pass-cli` command**, verify the session is alive:

```bash
pass-cli info > /dev/null 2>&1 && echo "SESSION_OK" || echo "SESSION_EXPIRED"
```

If `SESSION_EXPIRED`:

1. `pass-cli logout --force` — clear stale session
2. Re-run login with the PAT
3. `pass-cli info` — verify
4. Retry the original command

**Check periodically during long-running tasks** — sessions may expire mid-task.

### Error Diagnosis

If **any** `pass-cli` command fails, carefully read the **entire output** to diagnose the issue before retrying. Look for:

- **Authentication errors** → session expired, re-authenticate
- **"No session"** → `PROTON_PASS_SESSION_DIR` mismatch between login and current command
- **"Failed to parse token"** → the PAT token string may have been truncated or redacted by tooling; store it in a shell variable first: `PASS_PAT='<full token>'` then use `PROTON_PASS_PERSONAL_ACCESS_TOKEN="$PASS_PAT" pass-cli login`
- **"Unexpected argument"** → wrong flag syntax for the subcommand (see pitfalls)
- **Permission denied** → PAT viewer-only limitation

## Session and Connection Health

- `pass-cli info` — Show current account type and session details
- `pass-cli test` — Verify the connection to the Proton Pass API

## Vault Discovery

```bash
# List all vaults
pass-cli vault list --output json

# List shares (vaults + direct-item grants)
pass-cli share list --output json
```

## Item Discovery

```bash
# List items in a specific vault (both forms work in v2.1.4+)
pass-cli item list "Vault Name" --output json
pass-cli item list --vault-name "Vault Name" --output json

# List all accessible items
pass-cli item list --output json
```

> ⚠️ **Syntax asymmetry note:** In v2.1.3 and earlier, `item list` required the vault name as a **positional** argument (`pass-cli item list "Business"`) and `--vault-name` caused an `unexpected argument` error. In v2.1.4+, both positional and `--vault-name` flag forms work. By contrast, `item view` consistently uses `--vault-name "X" --item-title "Y" [--field "F"]` across all versions — never positional.

## Retrieving Secrets

**IMPORTANT: The `PROTON_PASS_AGENT_REASON` env var is REQUIRED** for these commands. The reason must describe why the credential is needed.

```bash
# View full item as JSON
PROTON_PASS_AGENT_REASON="Need X to do Y" pass-cli item view \
  --vault-name "Vault Name" \
  --item-title "Item Title" \
  --output json

# Get a single field
PROTON_PASS_AGENT_REASON="..." pass-cli item view \
  --vault-name "Vault" \
  --item-title "Item" \
  --field "Field Name"

# Using pass:// URI format
PROTON_PASS_AGENT_REASON="..." pass-cli item view "pass://SHARE_ID/ITEM_ID/FIELD"
```

### Commands that require PROTON_PASS_AGENT_REASON:
- `item view` — reading any item or field
- `item create` (and all sub-commands)
- `item update`
- `item trash` / `item untrash`
- `vault update`

## Credential Field Types

| Field type | JSON key | Notes |
|---|---|---|
| Login/password | `content.content.Login` sections | Standard login items |
| Hidden/API key | `extra_fields[].content.Hidden` | Custom items store API keys here |
| Text | `extra_fields[].content.Text` | Used for labels/descriptions |
| Timestamp | `extra_fields[].content.Timestamp` | Expiry dates |

## Auto-Recovery Sequence

If any `pass-cli` command fails with an authentication error:

1. `pass-cli logout --force` — clear stale session
2. Re-run login with the PAT (stored in memory)
3. `pass-cli info` — verify
4. Retry the original command

If the regular logout errors (e.g. "no session"), `pass-cli logout --force` still works.

## CRITICAL: Never Display Secret Values

Secret values (passwords, API keys, tokens) must never appear in agent responses. Fetch them server-side and pipe them directly into the consuming command. If the user asks about a credential's value, say "authentication succeeded" — never echo or repeat the value back.

**✅ Do this — pipe directly into the target command:**
```bash
PROTON_PASS_AGENT_REASON="reason" pass-cli item view --field "API Key" ... |
  gh auth login --with-token
```

**❌ Never do this — fetching then displaying:**
```bash
TOKEN=$(pass-cli item view ...)
echo "The token is: $TOKEN"    # WRONG
```

## Agent PAT Limitations

Agent PATs have **viewer-only** access by default, regardless of share role. This means:

- ✅ Can read items (`item view`, `item list`, `vault list`)
- ❌ Cannot create, update, or delete items (`item create`, `item update`, `item trash`)
- ❌ Cannot write to vaults even with "Owner" in share list

If the agent needs write access, the account owner must grant `editor` role:
```bash
pass-cli agent access grant <NAME> --vault-name <VAULT> --role editor
```

## Agent Audit Logging

Agents record every audited action with the reason encrypted in an audit log.
Inspect the log as the account owner:

```bash
pass-cli agent monitor <AGENT_NAME> [--limit N] [--output json]
```

Each entry shows which item was accessed, from which vault, the action, and the reason provided.

## Using the Agent-Prefixed Form

The agent-prefixed form is also valid and clearer when running as an agent:
```bash
PROTON_PASS_AGENT_REASON="..." pass-cli agent item view \
  --vault-name "Business" --item-title "GitHub" --field "API Key"
```

## Pitfalls

- **`item list` vault-name syntax varies by version**: v2.1.3 and earlier required **positional** (`pass-cli item list "Business"`); `--vault-name` caused an error. v2.1.4+ accepts both `--vault-name "Business"` and positional syntax. Check version with `pass-cli --version`.
- **Stale timestamped session dirs hijack auth**: If a prior run logged in with `PROTON_PASS_SESSION_DIR=/tmp/pass-agent-$(date +%s)`, you'll see leftover dirs like `/tmp/pass-agent-hermes-1781320484` that each hold a dead/invalidated session, while the stable `/tmp/pass-agent-hermes` has none. Symptom: user insists "my Proton hasn't expired" but `pass-cli info` says "no session". Recovery: ignore the timestamped dirs (they're dead), `pass-cli logout --force`, then re-login into the STABLE dir. The user's Proton *app* session is unrelated to the CLI *agent* session — they must be told these are separate.
- **Session dir must be stable**: Using `$(date +%s)` or any dynamic value in `PROTON_PASS_SESSION_DIR` causes "no session" errors — the login stores its cookie in a dir that subsequent commands don't find. Use a fixed name per task (e.g. `/tmp/pass-agent-hermes`).
- **PAT token redacted by security scanners**: Shell history/tooling may redact the token in the command itself. Workaround: store it in a shell variable first (`PASS_PAT='<full token>'`) then use `PROTON_PASS_PERSONAL_ACCESS_TOKEN=***` for the actual call.
- **Hidden field values may appear truncated in JSON output**: The `Hidden` field content may show abbreviated values. Try retrieving with `--field "Field Name"` or pass:// URI for the full value.
- **PAT token is the single source of truth**: Store it in memory, never in a file. If logging in with PAT fails with "Already authenticated", the session is still good — just verify with `pass-cli info`.
- **Already authenticated error**: If login says "Already authenticated", logout first: `pass-cli logout --force`.
- **Agent PATs are viewer-only by default**: `share_role: "Owner"` does NOT grant write access on agent PATs.

## Known Vaults (Ryan's Account)

As discovered via `pass-cli vault list --output json` on v2.1.4 (session Jun 17, 2026):

| Vault | Share Role | Notes |
|-------|-----------|-------|
| **Personal** | Owner | Web logins (github.com, accounts.google.com) |
| **Business** | Owner | API tokens (GMAIL, EMAILIT_API, MARKUPGO, ROBOMOTION_API_TOKEN, PUBLR_API) |
| **GitHub** | Owner | GitHub-specific vault (API token, credentials) |
| **Wave Accounting** | Owner | Wave accounting credentials |
| **Wordpress Plugin Tools** | Owner | WordPress-related items |
| **Hosting and Domain** | Owner | Hostinger VPS and domain management |
| **Memories** | Owner | |
| **Skills** | Owner | |
| **Messaging** | Owner | Messaging-related items |

> Items like GitHub API token and Wave Accounting that were previously listed under **Business** vault now reside in their own dedicated vaults. Check each vault with `pass-cli item list "Vault Name" --output json` to see current contents.

## Reference Files

- `references/vault-structure.md` — Vault layout and known items for this user's Proton Pass account
- `references/hostinger-vps-api.md` — Hostinger VPS API access (item in Business vault, token retrieval, endpoint patterns, known VPS list)

## Quick Reference

```bash
pass-cli login                                         # Authenticate with PAT from env
pass-cli logout                                        # End the session
pass-cli logout --force                                # Forcefully end session (use if regular logout errors)
pass-cli info                                          # Session details
pass-cli test                                          # API connection health
pass-cli vault list --output json                      # List vaults
pass-cli share list --output json                      # List vaults + direct-item grants
pass-cli item list <NAME> --output json                # List items in a vault (positional)
pass-cli item list --vault-name <NAME> --output json   # List items in a vault (flag, v2.1.4+)
pass-cli item list --output json                       # List all accessible items
pass-cli item view --vault-name <VAULT> \
  --item-title <TITLE> [--field <FIELD>]                # Read an item (or field only)
pass-cli agent instructions                             # Full CLI usage instructions
```
