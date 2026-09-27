# n8n Workflows

All workflows live in the **My project** team project on n8n Cloud and use the Airtable base **AI-ERP** (8 tables, see [`schema/schema.json`](../schema/schema.json)). Numbering follows the course brief; WF2, WF10, WF11 and WF12 (reports, reminders, schema validation, product sync) were dropped to keep the build lean.

| # | Name | Trigger | Flow |
|---|------|---------|------|
| WF1 | Tax Doc Validation | 3 Airtable Triggers — new Invoice / TaxInvoice / Receipt | `Validate` (Code) → `Is Valid?` → **true:** `Add To File Queue` (Files, Status=Pending) · **false:** `Mark Invalid` (Status=Invalid + ValidationError) |
| WF3 | Contact Intake | Airtable Trigger — new Lead | `Normalize` → `Find Same Email` → `Count Matches` → `Duplicate?` → **true:** `Mark Dead` · **false:** `Keep As New` |
| WF4a | Sales Cold Emails | Every 3 hours | `Search New Leads` (Status=New, 1 per run) → `Write Cold Email` (LLM chain) → `Send Email` (Gmail) → `Mark Contacted` (+ GmailThreadId) |
| WF4b | Sales Reply Check | Gmail Trigger — subject "AI Electronics", every minute | `Extract Fields` → `Find Lead By Thread` (GmailThreadId) → `Is A Lead?` (a lead's thread, and received after our last email to them) → `Draft Reply` (AI Agent) → `Gmail Reply` → `Mark Replied` |
| WF5 | Customer Service | Telegram (customer bot) | `סוכן שירות` (AI Agent + Simple Memory) with two *Answer questions with a vector store* tools: `search_policies` (key `policies`) and `search_products` (key `products`) → reply in Telegram |
| WF6 | Policies Embedding | Manual (form upload) | `On form submission` → `Simple Vector Store` (insert, key `policies`) with `Embeddings` + `Load Documents` |
| WF7 | Products Embedding | Manual | `All Products` (Airtable) → `Build Text` → `Simple Vector Store` (insert, key `products`) with `Embeddings` + `Default Data Loader` + `Text Splitter` (800/100) |
| WF8 | File Pipeline | Every minute | `Pending Files` → `Get Source Record` → `Build HTML` (Hebrew, RTL) → `Convert to File` → `Google Drive Upload` → `Mark Done` (+ DriveLink) |
| WF9 | Manager Agent | Telegram (owner bot) | `Is Owner?` → `Manager Agent` (Chat Memory; tools: policies vector store, `search_tasks`, `create_task`, `search_invoices_tax_receipt`, `Create_Invoice`, `Create_Tax_Invoice`, `Create_Receipt`) → `Send Answer` · not owner → `Deny` |
| WF13 | App Gateway | Webhook `POST /app-gateway` | For the admin app: `{action:"chat", message}` → RAG agent; `{action:"create", table, payload}` → writes a record to any table |

## Screenshots

### WF1 — Tax Doc Validation
Three Airtable triggers (one per document table) feed a single `Validate` Code node: required fields, VAT rate by issue date, VAT amount and total arithmetic, and a customer business number on tax invoices. Valid documents are queued for WF8; invalid ones are marked with the reason.

![WF1](screenshots/wf1-tax-doc-validation.png)

### WF3 — Contact Intake
A new lead's email is normalized (trimmed, lower-case), checked against existing leads with the same email, and marked `Dead` if another lead already has it.

![WF3](screenshots/wf3-contact-intake.png)

### WF4a — Sales Cold Emails
Every 3 hours: take one `New` lead, have the LLM write a personal Hebrew email, send it from Gmail, and store the Gmail thread ID on the lead.

![WF4a](screenshots/wf4a-sales-cold-emails.png)

### WF4b — Sales Reply Check
New Gmail messages are matched to a lead by thread ID; if it's a lead's reply, the agent drafts an answer, replies in the same thread, and marks the lead `Replied`.

![WF4b](screenshots/wf4b-sales-reply-check.png)

### WF5 — Customer Service
Telegram customer bot → service agent with memory and two RAG tools: `search_policies` and `search_products`.

![WF5](screenshots/wf5-customer-service.png)

### WF6 — Policies Embedding
Upload the policy files through the form → embedded into the in-memory store under the key `policies`.

![WF6](screenshots/wf6-policies-embedding.png)

### WF7 — Products Embedding
Read every product from Airtable → build one Hebrew text per product → embed under the key `products`.

![WF7](screenshots/wf7-products-embedding.png)

### WF8 — File Pipeline
Every minute: pending rows in `Files` → fetch the source document → Hebrew RTL HTML → Google Drive → mark `Done` with the link.

![WF8](screenshots/wf8-file-pipeline.png)

### WF9 — Manager Agent
Owner-only Telegram bot. The agent has memory, the policies knowledge base, and Airtable tools to search/create tasks, search documents, and create invoices, tax invoices and receipts (after the owner confirms).

![WF9](screenshots/wf9-manager-agent.png)

### WF13 — App Gateway
One webhook for the admin app: `chat` goes to a RAG agent with the same `search_policies` / `search_products` tools as WF5, `create` writes a record to the requested table, anything else returns an error.

![WF13](screenshots/wf13-app-gateway.png)

## How the pieces connect

- A document created anywhere (Airtable UI, the app via WF13, or the manager bot via WF9) is picked up by **WF1**, validated, and queued in **Files** → **WF8** renders it to Google Drive and writes the link back.
- A lead created anywhere is de-duplicated by **WF3** → emailed by **WF4a** (which stores the Gmail thread) → replies are answered by **WF4b**.
- **WF6/WF7** fill the in-memory vector store that **WF5**, **WF9** and **WF13** read from.

## Differences from the reference structure

- **Models:** OpenAI (`gpt-5-mini`, `text-embedding-3-small`) instead of GLM / bge-m3.
- **WF8** has no HTML→PDF step. The course brief rules out an external conversion service, so the invoice stays HTML on Drive (manual PDF: open in Google Docs → Download → PDF).
- **WF8** has an extra `Get Source Record` node — the Files queue row only stores a pointer, so the HTML step needs the actual document data.
- **WF9**'s document search tool is named `search_invoices_tax_receipt` (no slashes, so it's a valid tool name).
- **WF5**'s tools are named in English (`search_policies`, `search_products`): n8n builds the tool name from the node name and strips non-Latin letters, so two Hebrew-named tools both became `_` and collided.

## Credentials

| Type | Name | Used by |
|------|------|---------|
| Airtable (OAuth2) | Airtable account 3 | Every Airtable node and tool |
| OpenAI | OpenAI account 6 | All chat models and embeddings |
| Telegram | Telegram account 6 (owner bot) | WF9 — trigger *and* both send nodes must use the same bot credential |
| Telegram | Telegram account 4 (customer bot) | WF5 |
| Gmail (OAuth2) | Gmail account 2 | WF4a, WF4b — must be the same mailbox |
| Google Drive (OAuth2) | Google Drive account | WF8 |
