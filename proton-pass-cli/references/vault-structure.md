# Vault Structure — Ryan Perez

Discovered via `pass-cli vault list --output json` on v2.1.4 (session Jun 17, 2026).
PAT agent name: "TheBrain".

## Vaults

| # | Vault | Share Role | Purpose / Contents |
|---|-------|-----------|-------------------|
| 1 | **Personal** | Owner | Web logins (github.com, accounts.google.com) |
| 2 | **Business** | Owner | API tokens (GMAIL, EMAILIT_API, MARKUPGO, ROBOMOTION_API_TOKEN, PUBLR_API) |
| 3 | **GitHub** | Owner | GitHub-specific items (API token, credentials) |
| 4 | **Wave Accounting** | Owner | Wave accounting credentials |
| 5 | **Wordpress Plugin Tools** | Owner | WordPress-related items |
| 6 | **Hosting and Domain** | Owner | Hostinger VPS + domain management |
| 7 | **Memories** | Owner | |
| 8 | **Skills** | Owner | |
| 9 | **Messaging** | Owner | |

## Known Items by Vault

### Personal
| Item | Type | Notes |
|------|------|-------|
| github.com | login | GitHub web login credentials |
| accounts.google.com | login | Google accounts |

### Business
| Item | Type | Field | Purpose |
|------|------|-------|---------|
| GMAIL | custom | (Hidden) | Gmail API credentials |
| EMAILIT_API | custom | (Hidden) | Emailit API key |
| MARKUPGO | custom | (Hidden) | MarkupGo API key |
| ROBOMOTION_API_TOKEN | custom | (Hidden) | Robomotion API token |
| PUBLR_API | custom | (Hidden) | Publr API key |

### GitHub
| Item | Type | Field | Purpose |
|------|------|-------|---------|
| GitHub | custom | API Key (Hidden) | GitHub API token — used for `gh auth login --with-token` |

### Wave Accounting
| Item | Type | Notes |
|------|------|-------|
| WAVE accounting | custom | Wave accounting credentials |

### Hosting and Domain
| Item | Type | Notes |
|------|------|-------|
| Hostinger | custom | Hostinger VPS API key (48-char Bearer token) |

> **Note:** Items previously listed under **Business** vault (GitHub API token, Wave Accounting) now reside in their own dedicated vaults. Run `pass-cli item list "Vault Name" --output json` to discover items in any vault.
