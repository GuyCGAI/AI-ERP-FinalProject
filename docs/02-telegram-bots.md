# Telegram Bots

The project uses **two separate bots** — mixing them up means customers and the business owner end up talking to the same chat.

| Bot | Used by | Purpose |
|-----|---------|---------|
| Bot #1 (owner) | WF9 | Manager analytics — revenue, invoice count, unpaid total. Restricted to the owner's Telegram chat ID. |
| Bot #2 (customer) | WF5 | Customer service — RAG over policies and products. |

## Create both bots

1. Open a chat with **@BotFather** on Telegram.
2. `/newbot` → give it a name and a unique username → save the API token it returns.
3. Repeat for the second bot.
4. In n8n: Credentials → Add credential → Telegram API, once per bot, with each token. Name them clearly (e.g. `Telegram - owner`, `Telegram - customers`) so they don't get confused later.
5. Attach the correct credential to WF9's Telegram Trigger + reply node (owner bot), and to WF5's Telegram Trigger + reply node (customer bot).

## Restricting WF9 to the owner

Message either bot once, then check n8n's **Executions** tab for that workflow — the trigger payload includes `message.chat.id`. Paste that numeric ID into the **"האם זה הבעלים?"** IF node in WF9 (it ships with a placeholder there). Anyone else gets a polite refusal instead of the analytics.
