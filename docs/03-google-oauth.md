# 3. Google OAuth (Gmail + Drive)

Needed for the **sales agent** (sends and reads email) and the **file pipeline** (uploads documents to Drive). One Google Cloud project covers both.

## 1. Create a project and enable the APIs

1. [Google Cloud Console](https://console.cloud.google.com/) → **Create project** → name it `ai-erp`.
2. **APIs & Services → Library** → enable both:
   - **Gmail API**
   - **Google Drive API**

## 2. Configure the consent screen

**APIs & Services → OAuth consent screen**

- User type: **External**
- App name: `AI-ERP`, and your email where required.
- **Test users** → add your own Google account. While the app is in *Testing*, only listed test users can authorize it.
- Save. You don't need to publish or get verified for a course project.

## 3. Create the OAuth client

**APIs & Services → Credentials → Create credentials → OAuth client ID**

- Application type: **Web application**
- Name: `n8n`
- **Authorized redirect URIs** → add exactly:
  ```
  https://<your-instance>.app.n8n.cloud/rest/oauth2-credential/callback
  ```
  Don't guess it: open the Gmail or Drive credential in n8n and copy the **OAuth Redirect URL** it shows — it must match character for character.

Copy the **Client ID** and **Client secret**.

## 4. Add the credentials in n8n

Create **two** credentials — both use the same client id and secret:

| Type | Name in this project |
|---|---|
| Gmail OAuth2 API | Gmail account 2 |
| Google Drive OAuth2 API | Google Drive account |

For each: paste the client id + secret → **Sign in with Google** → pick your test-user account → approve. Google warns that the app is unverified; choose **Advanced → Go to AI-ERP (unsafe)** — it's your own app.

## Which workflows use what

| Credential | Workflows |
|---|---|
| Gmail | **WF4a** sends the cold email and stores its Gmail thread id on the lead · **WF4b** reads replies and answers in the same thread. Both must use the **same mailbox**, otherwise WF4b never sees the threads WF4a started |
| Google Drive | **WF8** uploads the generated invoice / tax invoice / receipt (HTML, `text/html`) to My Drive |

## Gotchas

- **Cold emails go to real inboxes.** WF4a sends one email per run to a lead with `Status = New`. In this project every demo lead has a `+alias` of the project's own mailbox (`guycgai+lead1@gmail.com` … `+lead6`), and WF13 gives every lead created from the app such an alias — so no stranger ever gets an email, and you can answer the email yourself to test WF4b.
- **WF4b only answers replies.** It skips messages that arrived before our last email to the lead (`LastContactedAt`), so the copy of our own cold email in the same mailbox is never answered.
- **Refresh tokens expire after 7 days** while the consent screen is in *Testing* mode. When emails stop going out or Drive uploads fail, reconnect both credentials (or publish the app).
- Gmail sending limits (~500/day on a personal account) don't matter at this scale.
