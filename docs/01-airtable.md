# Airtable Setup

Base name: **AI-ERP** — 8 tables: Customers, Leads, Products, Invoices, TaxInvoices, Receipts, Tasks, Files.

## 1. Create the base

Create the base and its tables with the exact field names and types from [`schema/schema.json`](../schema/schema.json). The base starts empty — no demo data.

## 2. Add the `Created` field by hand

Airtable's API can't create "Created time" fields, so after the tables exist, add one manually to **each of the 8 tables**:

- Field name: **`Created`** (exactly)
- Field type: **Created time**

The Airtable Triggers in WF1 (Invoices, TaxInvoices, Receipts) and WF3 (Leads) poll on this field — without it they never fire.

## 3. Conventions

- Relationships are plain text foreign keys (`CustomerId = CUST-0001`), not Airtable link fields — simpler, and easier to show in the app.
- `Status` fields are single selects; the workflows write with `typecast` on, so the values in `schema.json` must exist as choices.
- `VATRate` is a percent field stored as a fraction: `0.18` = 18%. From 01/01/2025 the rate is 18% (17% before) — WF1 rejects a document whose VAT doesn't match its `IssueDate`.

## 4. Connect it to n8n

n8n → Credentials → Airtable OAuth2 → connect, and grant access to the **AI-ERP** base (Airtable OAuth grants are per base — a base created later isn't included automatically).
