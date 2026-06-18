# Proton Pass CLI — Agent Instructions Reference

Source: `pass-cli agent instructions` (v2.1.3)

## Agent System

Agents are PATs with audit logging. Each audited action requires `PROTON_PASS_AGENT_REASON`,
which is encrypted and stored in the audit log for review via `pass-cli agent monitor <NAME>`.

## Audited Commands (require PROTON_PASS_AGENT_REASON)
- item view
- item create (all variants)
- item update
- item trash
- item untrash
- item move
- vault update

## Agent Management (by account owner only)

```bash
# Create agent with vault access
pass-cli agent create <NAME> --expiration <1d|1w|1m|3m|6m|1y> [--vault <NAME>]...
# Output JSON: {"token": "PROTON_PASS_PERSONAL_ACCESS_TOKEN=pst_xxx::KEY", ...}

# List agents
pass-cli agent list

# Grant access
pass-cli agent access grant <NAME> --vault-name <VAULT> [--role viewer|editor|manager]

# Revoke access
pass-cli agent access revoke <NAME> --share-id <SHARE_ID>

# Renew token
pass-cli agent renew <NAME> --expiration <EXPIRATION>

# Delete agent
pass-cli agent delete <NAME>

# Monitor audit log
pass-cli agent monitor [<NAME>] [--limit <N>]
```

## Agent Login Flow

```bash
export PROTON_PASS_SESSION_DIR="/tmp/pass-agent-<name>"
pass-cli info 2>/dev/null ||
  PROTON_PASS_PERSONAL_ACCESS_TOKEN=pst_xxx::KEY pass-cli login
```

## Agent Item Access

```bash
PROTON_PASS_AGENT_REASON="reason" pass-cli item view \
  --vault-name "Vault" --item-title "Item" --field password

# Or with agent prefix:
PROTON_PASS_AGENT_REASON="reason" pass-cli agent item view \
  --vault-name "Vault" --item-title "Item" --field password
```

## Important Notes
- Agent sessions cannot manage/act on PATs while logged in with a PAT
- `PROTON_PASS_AGENT_REASON` max 300 chars, non-empty
- Default role is viewer (read-only). Write operations require editor+.
