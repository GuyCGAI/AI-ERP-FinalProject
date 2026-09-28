# Working in this repo

This repo documents an n8n + Airtable project. There is no application code to build or test here — `schema/`, `templates/`, `mock/`, and `docs/` describe a system that actually runs on n8n Cloud, Airtable and Lovable, not in this repository.

## Scope

**n8n Cloud + Airtable + a Lovable admin app.** No Docker, no backend, nothing installed or run locally. If a task can be solved in a workflow node, it does not get code — and that includes n8n's own Code node: the workflows use Edit Fields, IF / Switch, Aggregate, the HTML node and Airtable formulas only.

The **live n8n instance is the source of truth** for workflows. The repo keeps no exports (they carry the owner's Telegram chat id and base ids) — `docs/screenshots/` holds run screenshots for reference.

## Keep in sync

- **schema/schema.json** describes the 9 live Airtable tables in the `AI-ERP` base. If you change a workflow's field mapping, update this file.
- **templates/*.html** mirror WF8's `Build HTML` node (an HTML node template). If you edit one, update the other.
- **mock/policies/*.md** is a readable copy of the policy text. The live text sits in WF6's `עריכת שדות` node (one field per topic) — edit it there and re-run WF6.
- **Products** live in Airtable; WF7 embeds them (key `products`). After changing products, re-run WF7.
- **docs/04-workflows.md** is the map of every workflow — node names, tool names and credential names must match n8n.
- **docs/05-app.md** is the WF13 API contract the Lovable app depends on (row fields, payload fields, Hebrew labels). Change WF13 and the doc together.

## Gotchas

- The vector store is n8n's **Simple Vector Store** (in memory). It's wiped whenever n8n restarts — re-run WF6 and WF7.
- Airtable search nodes v2.2 return `{ id, createdTime, fields }`; v2.1 (WF4a) returns the fields flat. Write expressions for the version in front of you.
- An AI Agent tool's name comes from its node name with non-Latin letters stripped — name tools in English.
- A Telegram trigger and its reply nodes must use the same bot credential.

## Conventions

- All user-facing text and document content is **Hebrew (RTL)**.
- Secrets live in n8n credentials and the Lovable secret only — never in source, never in git.
- Relationships are text foreign keys (`CUST-0001`), not Airtable links.

There's no CI, no package manager, and nothing to `npm install`. Treat this as living documentation for a cloud system, not a codebase.
