---
name: accountant-agent
description: "Financial modeling, budget classification, Supabase database storage, and investor-ready reporting for Antoinette & Friends web series. Includes full classification taxonomy, Supabase schema, industry benchmarks, and modeling methodology."
version: 1.1.0
tags:
  - finance
  - accounting
  - supabase
  - financial-modeling
  - investor
  - budget
platforms: [linux]
metadata:
  hermes:
    triggers:
      - "financial model"
      - "budget"
      - "accountant"
      - "investor"
      - "revenue projection"
      - "supabase"
---

# Accountant Agent — Antoinette & Friends

This skill provides the system prompt ("soul") for an accountant AI agent that classifies financial data, stores it in Supabase, and builds investor-ready financial models for the Antoinette & Friends web series.

## System Prompt ("The Soul")

The full accountant agent system prompt is in `references/agent-soul.md`. Load it when you need to:

- Classify budget line items using the standard taxonomy
- Store financial data in Supabase
- Build 3-year revenue/cost projections
- Create investor-ready reports

## Key Financial Data (Antoinette & Friends)

### Per-Episode Budget: $19,050

| Phase | Amount |
|-------|--------|
| Pre-Production | $2,950 |
| Production Day | $11,150 |
| Post-Production | $4,950 |
| **Total Per Episode** | **$19,050** |

### Crew Rates (Production Day)
| Role | Day Rate |
|------|----------|
| Norman Seeff (Creative Collaborator) | $2,500 |
| Director of Photography | $1,200 |
| A Camera Operator | $900 |
| Antoinette (Talent/Host) | $1,200 |
| B Camera Operator | $700 |
| Audio Engineer | $600 |
| Hair & Makeup | $650 |
| C Camera Operator | $550 |
| Producer (Ryan Perez) | $500 |
| D Camera/BTS | $400 |
| Production Assistant | $300 |
| Craft Services | $750 |
| Equipment Rental | $800 |

### Post-Production Rates
| Role | Rate |
|------|------|
| Lead Editor (3 days) | $400/day |
| Color Correction (1 day) | $500/day |
| Audio Post (1 day) | $350/day |
| Motion Graphics | $400 flat |
| Social Media (Jen Nida) | $600 flat |
| Business Mgmt (Helen Cohen) | $1,000 flat |

## Revenue Streams (Projected — Base Case)

| Stream | Year 1 | Year 2 | Year 3 | Assumption Basis |
|--------|--------|--------|--------|-----------------|
| YouTube AdSense | $3,000 | $6,000 | $18,000 | RPM $5/1K, 600K→3.6M views |
| YouTube Memberships | $2,400 | $6,000 | $18,000 | 50% of ad rev from members/super chat |
| Sponsorships | $7,500 | $30,000 | $90,000 | $20-50 CPM, 3→12 deals/yr |
| Odeum Subscriptions | $25,164 | $83,880 | $251,640 | $6.99/mo, 300→3K subs, 10% platform fee |
| Production Services | $20,000 | $48,000 | $90,000 | 4→12 projects @ $5K-7.5K each |
| Gallery / Merch | $5,000 | $15,000 | $30,000 | Norman Seeff prints + branded merch |
| Affiliate & Other | $2,000 | $5,000 | $10,000 | Links, partnerships |
| **TOTAL REVENUE** | **$65,064** | **$193,880** | **$507,640** | |

See `references/financial-model.md` for full P&L, cost breakdown, sensitivity analysis, and KPIs.

## Funding Ask

**$350,000** — covers 12-episode season ($228.6K), capital equipment ($57K), marketing ($36K), Odeum build ($5K), and 6-month operating reserve ($23.4K). Path to profitability in Year 4.

## Industry Benchmarks

- **Creator Economy Market**: $252B (2025), growing 23.3% CAGR → $1.3T by 2033
- **YouTube Interview/Talk Show RPM**: $4-8 per 1,000 views
- **Sponsorship Rates**: $20-50 CPM for integrated mentions
- **OTT Subscription**: $4.99-9.99/month, 1-5% conversion, 5-10% churn
- **YouTube Typical**: Channels with 10-50K subs earn $100-1,000/month AdSense
- **YouTube Typical**: Channels with 50-100K subs earn $1,000-10,000/month total

## Show Facts for Context

- Host: Antoinette Peragine (former actress/comedian, pivoted at pandemic)
- Format: Cinematic interview series — 1-2 episodes/month
- Notable guests: Darryl Anka (131K views), Moon Unit Zappa (50K views), Sharon Gless
- Creative Director: Norman Seeff (legendary rock photographer)
- Community: "Angel Community" — engaged supportive sisterhood
- Distribution: YouTube, Odeum (planned), Apple TV/Roku/Fire TV, Instagram, TikTok, Facebook
- Production Services: Also offers legacy interviews, brand storytelling, custom content
- Gallery: Antoinette Peragine Gallery × Norman Seeff prints
- Social: 15.8K Facebook followers, active Instagram/TikTok

## How to Build Financial Model in Sturppy

1. Navigate to **app.sturppy.com** and log in with saved credentials
2. Click **"New Project"** (top-right corner)
3. Template selection — two paths:
   - **Using a template**: Click "Entertainer or artist" → check subcategories ("Film/videography" + "Youtuber"). If a "Use this template" button appears, click it. If none appears, use the blank path instead.
   - **Blank template** (fallback): Click **"None of these templates work for me"** → then click **"Use this template"** on the blank/custom template. This creates an empty project you populate from scratch.
4. Fill in each section via the sidebar:
   - **Acquisition** — Marketing spend, subscriber growth targets, CAC assumptions
   - **Revenues** — All 7 revenue streams with 3-year projections (see references/financial-model.md)
   - **Expenses** — Per-episode budget ($19,050) scaled to production schedule (12→24 eps/year)
   - **Assumptions** — Industry benchmarks (RPM rates, conversion rates, churn, inflation)
   - **Statements** — P&L, Cash Flow, Balance Sheet auto-generated from above inputs
   - **KPIs** — Key metrics dashboard (subscribers, views, burn rate, runway, revenue vs expenses)
   - **Valuation** — Investor-ready valuation section
5. Export / Share with investors

### Pitfalls
- **Browser tool may be unstable** with Sturppy (502 errors common). If the browser crashes mid-session, retry. The login state persists for a while.
- **Template buttons may not appear clearly** after selecting subcategories — the "None of these templates work for me" → blank template path is more reliable.
- **Sturppy auto-generates statements** from your revenue/expense/acquisition inputs. Don't manually enter in Statements tab — fill in the upstream tabs instead.
- **Name your project** something meaningful (e.g. "Antoinette & Friends") so the breadcrumb is identifiable.
- **Verify before reporting**: after creating a project, navigate back to the project list or check the breadcrumb to confirm it actually saved. The browser may show a "success" click but crash before the server processes it — the project doesn't exist until the breadcrumb or dashboard confirms it.
- **Don't auto-retrieve credentials** — do NOT use pass-cli to log into services without explicit user permission. Wait for the user to say "use these credentials" or "log into this" before pulling any tokens or passwords.
- **Sturppy revenue onboarding** uses a wizard flow: after naming a revenue stream and selecting its type (e.g. "Ad Revenue"), it asks Product Type and Pricing Type before creating the line item. Complete the wizard before moving to the next stream.

## Supabase Database

The schema is defined in the accountant soul prompt. Use these tables:
- `financial_models` — Model metadata
- `budget_line_items` — All cost/revenue line items
- `revenue_projections` — 3-year revenue by stream
- `kpi_targets` — KPI benchmarks and targets