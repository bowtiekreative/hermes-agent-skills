# Hostinger VPS API Access

Retrieved from Proton Pass **Hosting and Domain** vault, item "Hostinger", field "API Key".

## API Endpoint
`https://api.hostinger.com/api/vps/v1/` (v1)
`https://developers.hostinger.com/api/[product]/v1/` (product-specific)

## Swagger/OpenAPI docs
`https://developers.hostinger.com/` → select product (VPS, hPanel, DNS, etc.)

## Authentication
- Bearer token (48 chars), passed as `Authorization: Bearer <token>`
- Stored in Proton Pass **Hosting and Domain** vault, item "Hostinger", field "API Key"
- Secret field is currently empty; Expiry Date and Permissions are unset

## VPS Endpoints Used
- `GET /virtual-machines` — list all VPS instances
- Response includes: id, hostname, plan, state, cpus, memory, disk, bandwidth, ipv4, ipv6, template (OS), subscription_id

## Known VPS on this account
| ID | Hostname | Plan | State | CPU | RAM | Disk | IP |
|----|----------|------|-------|-----|-----|------|-----|
| 620544 | srv620544.hstgr.cloud | KVM 4 | running | 4 | 16 GB | 50 GB | 212.1.213.81 |
| 523949 | srv523949.hstgr.cloud | KVM 1 | destroyed | 1 | 4 GB | 50 GB | — |

## Shell Quick Reference
```bash
# Fetch token from Proton Pass (server-side only, never displayed)
TOKEN=$(PROTON_PASS_AGENT_REASON="accessing Hostinger VPS API" \
  pass-cli item view --vault-name "Hosting and Domain" --item-title "Hostinger" --field "API Key")

# List VMs
curl -s -H "Authorization: Bearer $TOKEN" -H "Accept: application/json" \
  "https://api.hostinger.com/api/vps/v1/virtual-machines"

# Get single VM
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://api.hostinger.com/api/vps/v1/virtual-machines/$VM_ID"

unset TOKEN
```
