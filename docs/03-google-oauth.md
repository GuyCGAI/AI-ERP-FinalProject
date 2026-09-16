# Google OAuth (Gmail + Drive)

Two Google credentials are needed:

- **Gmail (OAuth2)** — WF3's notification path, WF4a (send cold emails), WF4b (read replies).
- **Google Drive (OAuth2)** — WF8 (upload + share the generated invoice HTML).

## Setup

1. [Google Cloud Console](https://console.cloud.google.com) → create a project (or reuse one) → enable the **Gmail API** and **Google Drive API**.
2. OAuth consent screen → External → add your own Google account as a test user.
3. Credentials → Create OAuth client ID → Web application → add `https://<your-n8n-domain>/rest/oauth2-credential/callback` as an authorized redirect URI (n8n shows you the exact URL when you create the credential).
4. In n8n: Credentials → Add credential → Gmail OAuth2 API, paste the Client ID/Secret, click "Connect" and authorize. Repeat for Google Drive OAuth2 API.

## Known limitation

The OAuth consent screen stays in **Testing** mode unless you go through Google's verification process, which is out of scope for a course project. In Testing mode, refresh tokens expire after **7 days** — you'll need to reconnect both credentials weekly.
