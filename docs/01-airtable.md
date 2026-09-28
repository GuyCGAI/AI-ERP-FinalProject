# Airtable Setup

Base name: **AI-ERP** — 9 tables: Customers, Leads, Products, Orders, Invoices, TaxInvoices, Receipts, Tasks, Files.

## 1. Create the base

Create the base and its tables with the exact field names and types from [`schema/schema.json`](../schema/schema.json). The base starts empty — no demo data.

## 2. Add the `Created` field

Every table needs a field named `Created` of type **Created time** (the Orders table already has one). If your tool can't create "Created time" fields through the API, add it by hand to **each table**:

- Field name: **`Created`** (exactly)
- Field type: **Created time**

The Airtable Triggers in WF1 (Invoices, TaxInvoices, Receipts) and WF3 (Leads) poll on this field — without it they never fire.

## 3. Conventions

- Relationships are plain text foreign keys (`CustomerId = CUST-0001`), not Airtable link fields — simpler, and easier to show in the app.
- `Status` fields are single selects; the workflows write with `typecast` on, so the values in `schema.json` must exist as choices.
- Invoices, TaxInvoices and Receipts have a `PdfUrl` field: WF8 writes the Google Drive link of the generated document there, and the app shows it as the "מסמך" link.
- Records created from the app get running IDs from WF13 (`CUST-0001`, `LEAD-0006`, `ORD-0001`, `TASK-0001`, invoices `INV-1001` with `DocNumber` 1001, 1002, …).
- `VATRate` is a percent field stored as a fraction: `0.18` = 18%. From 01/01/2025 the rate is 18% (17% before) — WF1 rejects a document whose VAT doesn't match its `IssueDate`.

## 4. Connect it to n8n

n8n → Credentials → Airtable OAuth2 → connect, and grant access to the **AI-ERP** base (Airtable OAuth grants are per base — a base created later isn't included automatically).
