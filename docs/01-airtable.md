# Airtable Setup

Base name: **AI-ERP Claude**.

## 1. Create the base and 4 tables

Create the base, then add these 4 tables with the **exact** field names and types from [`schema/schema.json`](../schema/schema.json):

- **Invoices** — InvoiceNumber, CustomerId, Amount, VatAmount, Total, Status, PdfUrl, Created
- **Leads** — Name, Email, Company, Status, Created
- **Products** — Name, Category, Price, Description, InStock
- **Tasks** — Title, Status

Two things that cause an hour of debugging if you skip them:

1. `Created` must be type **Created time** in Invoices and Leads — WF1 and WF3's Airtable Triggers poll on it.
2. `Status` must be plain **text**, not Single select — the workflows write values (`מוכן להפקה`, `הופק`, `Contacted`, `Replied`...) that wouldn't exist as pre-defined choices.

The base starts empty — no demo data. Foreign keys between tables (e.g. `CustomerId`) are plain text, not Airtable link fields.

## 2. Get a Personal Access Token

Airtable → account icon → Developer Hub → Personal access tokens → Create token. Scopes needed: `data.records:read`, `data.records:write`, `schema.bases:read`. Attach it to this base only.

## 3. Wire it into n8n

Credentials → Add credential → Airtable (OAuth2 or Personal Access Token, either works). In every Airtable node in every workflow: re-select the base (it will show as a placeholder `appXXXXXXXXXXXXXX` until you pick it from the list) and re-select the table — the workflows are otherwise pre-wired.
