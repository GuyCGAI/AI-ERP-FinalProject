# AI-ERP with n8n

A small, learning-oriented ERP system for a fictional Israeli electronics business — built entirely on **n8n Cloud**, **Airtable**, and **Qdrant**, with three AI agents and a RAG-grounded customer service bot. Final project for the AI & Automation course.

Everything runs in the cloud. Nothing to install, nothing to run locally, no server to deploy.

## What's in it

- **3 AI agents** — a manager (analytics, owner-only), a customer service agent (RAG over policies + products), and a sales agent (cold outreach + reply tracking).
- **10 n8n workflows** (+ 1 webhook gateway for an admin app) covering invoicing, lead intake, sales, RAG ingestion, and document generation.
- **RAG over Qdrant** — two collections (`policies`, `products`), embedded with OpenAI `text-embedding-3-small`, queried through the AI Agent's native vector-store tool.
- **Israeli tax rules built in** — 18% VAT from 01/01/2025 (17% before), running invoice numbers, HTML invoice generation.
- **Airtable as the single database** — 4 tables (of a 14-table full model) are enough to run everything.

## Architecture

```
                    ┌─────────────────────────────┐
 Telegram (owner)   │                              │
 Telegram (customer)│                              ├──► Airtable  (data)
 Gmail               ──►        n8n Cloud           │
 Schedule triggers   │   (workflows + AI agents)    ├──► Gmail     (sales emails)
 Admin app webhook   │                              │
                    │                              ├──► Google Drive (invoice docs)
                    └─────────────────────────────┘
                                   │
                                   ▼
                             Qdrant Cloud
                          (policies / products)
```

n8n sits in the middle. Triggers come in from two Telegram bots, Gmail, scheduled timers, and a single webhook the admin app calls. Everything that needs to persist goes to Airtable; documents go to Google Drive; embeddings go to Qdrant.

## Quick start

1. **Accounts**: n8n Cloud, Airtable, OpenAI, Qdrant Cloud (free tier), two Telegram bots via @BotFather, a Google account for Gmail + Drive.
2. **Airtable**: build the base and 4 tables — see [`docs/01-airtable.md`](docs/01-airtable.md) and [`schema/schema.json`](schema/schema.json).
3. **Telegram**: create both bots and note their tokens — see [`docs/02-telegram-bots.md`](docs/02-telegram-bots.md).
4. **Google**: connect Gmail + Drive OAuth2 credentials — see [`docs/03-google-oauth.md`](docs/03-google-oauth.md).
5. **n8n**: import/build the 10 workflows, re-select the Airtable base/table and credentials in each node — see [`docs/04-workflows.md`](docs/04-workflows.md) for the full list and what each one does.
6. **Qdrant**: get a cluster URL + API key, add it as an n8n credential, then run WF6 and WF7 once each (manual form upload) to populate the `policies` and `products` collections.
7. Activate WF1, WF3, WF4a, WF4b, WF5, WF8, WF9, WF13. Try the customer bot ("מה מדיניות ההחזרות?") and the owner bot ("מה ההכנסות?").

## Repo layout

```
├── docs/            setup guides for Airtable, Telegram, Google OAuth, and the workflow map
├── mock/policies/   the Hebrew policy corpus embedded into Qdrant by WF6
├── schema/          schema.json - the single source of truth for the 4 Airtable tables
├── templates/       the HTML invoice template used by WF8
├── AGENTS.md         notes for anyone (human or AI) editing this repo
└── README.md         this file
```

## The 10 workflows (summary)

See [`docs/04-workflows.md`](docs/04-workflows.md) for the full table with triggers and credentials.

| # | Name | Trigger |
|---|------|---------|
| WF1 | אימות חשבוניות ותור הפקה | New Invoice in Airtable |
| WF3 | קליטת אנשי קשר וסינון כפילויות | Webhook |
| WF4a | סוכן מכירות — מיילים קרים | Every 3h |
| WF4b | סוכן מכירות — בדיקת תשובות | Every 30 min |
| WF5 | סוכן שירות לקוחות | Telegram (customer bot) |
| WF6 | מדיניות ← מאגר וקטורי | Manual |
| WF7 | מוצרים ← מאגר וקטורי | Manual |
| WF8 | הפקת מסמך חשבונית + דרייב | Every 1 min |
| WF9 | סוכן המנהל | Telegram (owner bot) |
| WF13 | Webhook לאפליקציה | Webhook |

## Known limitations (intentional)

- No error handling or retries — a failed execution shows up red in n8n's **Executions** tab, on purpose, so failures are visible instead of silently retried.
- No PDF conversion — invoices are HTML on Google Drive; converting to PDF is a manual right-click away.
- Google OAuth stays in Testing mode — refresh tokens expire every 7 days.
- Running invoice numbers can collide if two invoices are created in the same polling window.
- The manager agent (WF9) gets no tools and sees at most 100 invoices — by design, to keep it simple and predictable.
- Vector search runs through n8n's native `Qdrant Vector Store` node (`retrieve-as-tool` mode), but that same node's `insert` mode currently fails against recent Qdrant server versions (upstream n8n bug). WF6/WF7 work around it with a manual OpenAI Embeddings + Qdrant REST API pipeline — see [`docs/04-workflows.md`](docs/04-workflows.md) for details and the tracking issues.

## Credits

Project structure inspired by [tomerfooks/jb-erp-ai](https://github.com/tomerfooks/jb-erp-ai), built independently for the same course assignment.
