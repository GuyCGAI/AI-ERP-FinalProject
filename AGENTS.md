# Working in this repo

This repo documents an n8n + Airtable + Qdrant project. There is no application code to build or test here — `schema/`, `templates/`, `mock/policies/`, and `docs/` are the source of truth for a system that actually runs on n8n Cloud and Airtable, not in this repository.

- **schema/schema.json** describes the 4 live Airtable tables. If you change a workflow's field mapping, update this file to match.
- **templates/invoice.html** is copied verbatim from WF8's "בניית HTML" node. If you edit the template in n8n, paste the change back here too (and vice versa).
- **mock/policies/*.md** is the exact corpus WF6 embeds into Qdrant. Editing a file here does nothing to the live vector store — you have to re-run WF6's form upload for the change to take effect.
- **docs/04-workflows.md** is the map of what each workflow ID does. Keep the workflow IDs and credential names there in sync with n8n if you rename or rebuild anything.

There's no CI, no package manager, and nothing to `npm install`. Treat this as living documentation for a cloud system, not a codebase.
