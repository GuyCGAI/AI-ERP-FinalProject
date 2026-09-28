# Working in this repo

This repo documents an n8n + Airtable project. There is no application code to build or test here — `schema/`, `templates/`, `mock/`, and `docs/` are the source of truth for a system that actually runs on n8n Cloud and Airtable, not in this repository.

- **schema/schema.json** describes the 9 live Airtable tables in the `AI-ERP` base. If you change a workflow's field mapping, update this file to match.
- **templates/invoice.html** mirrors WF8's `Build HTML` Code node. If you edit one, update the other.
- **mock/policies/*.md** is a readable copy of the policy text. The live text sits in WF6's `עריכת שדות` node (one field per topic) — edit it there and re-run WF6; keep this copy in sync.
- **Products** live in Airtable; WF7 embeds them (key `products`). After changing products, re-run WF7.
- The vector store is n8n's **Simple Vector Store** (in memory). It's wiped whenever n8n restarts — re-run WF6 and WF7.
- **docs/04-workflows.md** is the map of what each workflow does. Keep workflow names, tool names and credential names there in sync with n8n.

There's no CI, no package manager, and nothing to `npm install`. Treat this as living documentation for a cloud system, not a codebase.
