# n8n Workflows

All 10 workflows (plus the app gateway, WF13) live in the **My project** team project on n8n Cloud. Numbering follows the original course brief and is intentionally non-sequential — WF2, WF10, WF11 and WF12 (reports, reminders, schema validation, product sync) were dropped to keep the build lean.

| # | Name | Trigger | What it does |
|---|------|---------|---------------|
| WF1 | אימות חשבוניות ותור הפקה | Airtable Trigger — new record in `Invoices` | Computes a running invoice number, VAT (18% from 01/01/2025, 17% before), and `Total`; sets `Status = מוכן להפקה`. |
| WF3 | קליטת אנשי קשר וסינון כפילויות | Webhook | Dedupes an incoming lead by email against `Leads`, creates the record if new. |
| WF4a | סוכן מכירות: מיילים קרים | Schedule — every 3h | Picks one `New` lead, drafts a short Hebrew cold email with an LLM chain, sends it via Gmail, marks the lead `Contacted`. Capped at 1 lead/run on purpose. |
| WF4b | סוכן מכירות: בדיקת תשובות | Gmail Trigger — every 30 min | Matches unread replies to a `Contacted` lead by sender email, flips it to `Replied`. |
| WF5 | סוכן שירות לקוחות | Telegram Trigger (customer-facing bot) | RAG agent with two Qdrant retriever tools (`policies`, `products`) and per-chat memory. |
| WF6 | מדיניות למאגר וקטורי | Manual (Form) | Embeds `mock/policies/*.md` into the Qdrant `policies` collection. Auto-creates the collection on first run. |
| WF7 | מוצרים למאגר וקטורי | Manual (Form) | Embeds the product catalog into the Qdrant `products` collection. |
| WF8 | הפקת מסמך חשבונית והעלאה לדרייב | Schedule — every 1 min | Renders `templates/invoice.html` for invoices with `Status = מוכן להפקה`, uploads to Google Drive, shares it, writes `PdfUrl` + `Status = הופק` back to Airtable. |
| WF9 | סוכן המנהל | Telegram Trigger (owner-only bot) | Precomputes revenue / invoice count / unpaid total via Airtable + Summarize, and only asks the LLM to phrase the Hebrew answer — no tools, no free-form data access. |
| WF13 | Webhook לאפליקציה | Webhook — `POST /app-gateway` | Single entry point for the admin app: `{action:"chat", message}` routes to a RAG agent, `{action:"create", table, payload}` writes a record to any of the 4 tables. |

## Known deviations from a "textbook" build

- **WF6/WF7 use a file-upload form**, not the brief's "hardcoded policy text in an Edit Fields node." Kept because it's more realistic for iterating on real documents.
- **Qdrant instead of the in-memory vector store.** The built-in n8n "Simple Vector Store" node works fine for retrieval but its `insert` mode currently throws `fetch failed` against recent Qdrant server versions — a known n8n bug (upstream `@qdrant/js-client-rest` / undici dispatcher mismatch, see n8n-io/n8n#37690, #37706, #37907). WF6/WF7 embed and upsert manually via `HTTP Request` (OpenAI Embeddings API + Qdrant REST API) to work around it. WF5/WF13 use the native `Qdrant Vector Store` node in `retrieve-as-tool` mode, which is unaffected (only `insert` is broken).
- **Sales cold-email batch size is capped at 1** per run, per the brief's explicit safety note ("so a mistake doesn't send dozens of emails").

## Credentials used

| Type | Name | Used by |
|------|------|---------|
| Airtable (OAuth2) | Airtable account 3 / 4 | All Airtable read/write nodes |
| OpenAI | OpenAI account 6 | Chat models, embeddings |
| Qdrant API | Qdrant account 6 | WF5, WF6, WF7, WF13 |
| Telegram | Telegram account 4 (owner bot), Telegram account 5 (customer bot) | WF9, WF5 |
| Gmail (OAuth2) | Gmail account 2 | WF3, WF4a, WF4b |
| Google Drive (OAuth2) | Google Drive account | WF8 |
