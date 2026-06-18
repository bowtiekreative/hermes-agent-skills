---
name: proton-pass-credential-retrieval
description: "Retrieve credentials from Proton Pass CLI for authentication to services, with secure handling rules."
version: 1.3.0
author: Hermes Agent
license: MIT
platforms: [linux]
metadata:
  hermes:
    tags: [Proton Pass, Credentials, Secrets, Authentication, pass-cli, Agents]
---

# Proton Pass Credential Retrieval

This skill covers how to retrieve credentials from Proton Pass (pass-cli) for authenticating to services like GitHub, APIs, and other tools.

## Prerequisites

- `pass-cli` installed at `~/.local/bin/pass-cli` (user-space install, no sudo)
- PAT stored in memory for re-authentication
- Environment: container with no system keyring
- We are an **Agent PAT** — audited commands require `PROTON_PASS_AGENT_REASON`

## CRITICAL: Never Display Secret Values

Secret values (passwords, API keys, tokens) must never appear in agent responses. Fetch them server-side and pipe them directly into the consuming command. If the user asks about a credential's value, say "authentication succeeded" or "the token is being used" — never echo or repeat the value back.

✅ **Do this — pipe directly into the target command:**
```bash
PROTON_PASS_AGENT_REASON="reason" pass-cli item view --field "API Key" ... |
  gh auth login --with-token
```

❌ **Never do this — fetching then displaying:**
```bash
TOKEN=*** item view ...)
echo "The token is: $TOKEN"    # WRONG — do not display
```

## Agent PAT Limitations

Agent PATs have **viewer-only** access by default. This means:
- ✅ Can read items (`item view`, `item list`, `vault list`)
- ❌ Cannot create, update, or delete items (`item create`, `item update`, `item trash`)
- ❌ Cannot write to vaults even with "Owner" in share list

If the agent needs write access, the account owner must create the item directly or grant `editor` role via:
```bash
pass-cli agent access grant <NAME> --vault-name <VAULT> --role editor
```

## Environment Setup

Always use these env vars when working with pass-cli in this container:

```bash
export PROTON_PASS_KEY_PROVIDER=fs          # File-based key store (no system keyring)
export PROTON_PASS_SESSION_DIR="/tmp/pass-agent-hermes"  # Isolated session dir
```

## Agent System Overview

Proton Pass CLI supports **Agents** — PATs with audit logging. When an agent performs operations like `item view`, it must supply a reason via `PROTON_PASS_AGENT_REASON`. The reason is encrypted and stored in an audit log inspectable via `pass-cli agent monitor <NAME>`.

**Audited commands** (require `PROTON_PASS_AGENT_REASON`):
- `item view`
- `item create` (all variants)
- `item update`
- `item trash`
- `item untrash`
- `item move`
- `vault update`

You can use either form:
```bash
# Standard form (works with agent PATs)
PROTON_PASS_AGENT_REASON="reason" pass-cli item view ...

# Agent-prefixed form
PROTON_PASS_AGENT_REASON="reason" pass-cli agent item view ...
```

## Session Health Check

Before ANY command, verify the session is alive:

```bash
pass-cli info
# Exit code 0 = authenticated
# Non-zero = need to re-authenticate
```

## Authentication (if session expired)

```bash
# 1. Force logout stale session
pass-cli logout --force

# 2. Set up isolated session dir
export PROTON_PASS_SESSION_DIR="/tmp/pass-agent-<unique-name>"
mkdir -p "$PROTON_PASS_SESSION_DIR"

# 3. Login with PAT
PROTON_PASS_PERSONAL_ACCESS_TOKEN="<token>" pass-cli login

# 4. Verify
pass-cli info
```

## Listing Vaults & Items

```bash
# List vaults
pass-cli vault list --output json

# List items in a vault
pass-cli item list "Vault Name" --output json

# List all accessible items
PROTON_PASS_AGENT_REASON="reason" pass-cli item list --output json
```

## Retrieving Credentials

**IMPORTANT:** Every `item view` command requires `PROTON_PASS_AGENT_REASON` env var (max 300 chars).

```bash
# Full item details (JSON)
PROTON_PASS_AGENT_REASON="Why you need this" pass-cli item view \
  --vault-name "Vault Name" \
  --item-title "Item Title" \
  --output json

# Single field — pipe directly to consuming command, never display
PROTON_PASS_AGENT_REASON="Why you need this" pass-cli item view \
  --vault-name "Vault Name" \
  --item-title "Item Title" \
  --field "Field Name"

# Via pass:// URI format
PROTON_PASS_AGENT_REASON="..." pass-cli item view \
  "pass://SHARE_ID/ITEM_ID"

# Via pass:// URI with specific field
PROTON_PASS_AGENT_REASON="..." pass-cli item view \
  "pass://SHARE_ID/ITEM_ID/FIELD_NAME"

# Agent-prefixed form (also valid)
PROTON_PASS_AGENT_REASON="..." pass-cli agent item view \
  --vault-name "Vault Name" --item-title "Title" --field "Field"
```

## Vault Structure

Discovered via `pass-cli vault list --output json` on v2.1.4 (Jun 17, 2026):

- **Personal** — web logins (github.com, accounts.google.com)
- **Business** — API tokens (GMAIL, EMAILIT_API, MARKUPGO, ROBOMOTION_API_TOKEN, PUBLR_API)
- **GitHub** — GitHub-specific API token and credentials
- **Wave Accounting** — Wave accounting credentials
- **Wordpress Plugin Tools** — WordPress-related items
- **Hosting and Domain** — Hostinger VPS + domain management
- **Memories** — stored memories
- **Skills** — stored skills
- **Messaging** — messaging-related items

## Re-authentication Flow

If any command fails with auth error:
1. `pass-cli logout --force`
2. Re-login with PAT
3. `pass-cli info` to verify
4. Retry original command

## ⚠️ CRITICAL: User-Provided Credentials Take Priority

If the user hands you a username/password or token directly for a tool or website, **use it immediately** — do NOT go looking for it in pass-cli or any other vault first. The user is the authority on what credentials to use. Pulling credentials from Proton Pass without the user's explicit permission is a violation of trust.

**❌ Wrong:** User says "login to Sturppy with ryan@bowtiekreative.com and password X" — then I open pass-cli to "check" if the password is in the vault.

**✅ Right:** Use the exact credentials the user provided, piping them into the target command.

This rule overrides any "always check pass-cli first" instinct. The user will explicitly tell you when to use vault-stored credentials.

## Known Vault Items

| Vault | Item | Type | Description |
|-------|------|------|-------------|
| Business | GMAIL | custom | Gmail API credentials |
| Business | EMAILIT_API | custom | Emailit API key |
| Business | MARKUPGO | custom | MarkupGo API key |
| Business | ROBOMOTION_API_TOKEN | custom | Robomotion API token |
| Business | PUBLR_API | custom | Publr API key |
| GitHub | GitHub | custom | GitHub API token |
| Wave Accounting | WAVE accounting | custom | Wave accounting credentials |
| Personal | github.com | login | GitHub web login credentials |
| Personal | accounts.google.com | login | Google accounts |
