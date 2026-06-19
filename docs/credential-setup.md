# Credential and Integration Setup

To ensure security, this repository contains **no API keys, access tokens, client secrets, or user credentials**. All parameters are parameterized using n8n's credential manager. 

This guide details how to safely connect external services and prevent credentials from leaking when exporting or saving workflows.

---

## 1. Safety Guidelines (Keep Secrets Out of Files)

> [!CAUTION]
> Never hardcode API keys, passwords, or client secrets in n8n expression fields or code blocks. Always use n8n's native Credential system.

When exporting your workflow to share:
1.  Use the standard **Export Workflow** option. n8n automatically strips credential hashes and only references credential names (e.g., `googleCalendarOAuth2Api`).
2.  If you use **Environment Variables** in nodes (e.g., `process.env.TELEGRAM_TOKEN`), ensure these variables are declared in your n8n host environment configuration (Docker Compose file or `.env` file) and not in the JSON file.

---

## 2. Setting Up LLM API Credentials

The AI reasoning nodes require credentials for Google Gemini, OpenAI, or Anthropic.

### Google Gemini API
1.  Go to [Google AI Studio](https://aistudio.google.com/) and generate an API key.
2.  In n8n, click on **Credentials** on the left menu, select **New**, search for **Google Gemini API**, and paste your API key.
3.  Link this credential inside the `Google Gemini Chat Model` node in the workflow.

### OpenAI API
1.  Go to the [OpenAI Developer Portal](https://platform.openai.com/) and create a secret key.
2.  In n8n, create a **New Credential**, select **OpenAI API**, paste your secret key, and link it inside the `OpenAI Chat Model` node.

---

## 3. Connecting Workspace Integrations

If you choose to link external services for automated ingestion, configure them as follows:

### A. Google Sheets & Google Calendar (OAuth2 Setup)
To ingest tasks directly from spreadsheets or block out calendar events:
1.  Navigate to the [Google Cloud Console](https://console.cloud.google.com/).
2.  Create a new project and enable the **Google Calendar API** and **Google Sheets API**.
3.  Configure your **OAuth Consent Screen** (set publishing status to "Testing" and add your own email as a test user).
4.  Navigate to **Credentials** -> **Create Credentials** -> **OAuth Client ID**. Select *Web Application*.
5.  Add n8n's redirect URL to the **Authorized redirect URIs** list:
    *   *Cloud:* `https://<YOUR_SUBDOMAIN>.app.n8n.cloud/rest/oauth2-credential/callback`
    *   *Self-hosted:* `https://localhost:5678/rest/oauth2-credential/callback` (or your public domain).
6.  Copy the **Client ID** and **Client Secret** into the n8n Google Calendar/Sheets Credential block and click **Sign in with Google** to authorize.

### B. Notion Database Integration
To retrieve tasks from Notion databases:
1.  Go to [Notion Integrations](https://www.notion.so/my-integrations) and click **Create new integration**.
2.  Give it a name and select the workspace. Ensure "Read content" and "Update content" capabilities are active.
3.  Copy the **Internal Integration Token**.
4.  In Notion, open the specific database page you want to sync, click the three dots (`...`) -> **Connections**, search for your integration name, and click **Connect**.
5.  In n8n, create a **Notion API** credential and enter the token.

### C. Telegram Bot Setup (For Pushes & Alerts)
To deliver digests and alerts to your mobile device:
1.  Open Telegram, search for `@BotFather`, and send `/newbot`.
2.  Follow the prompts to name your bot and copy the generated **HTTP API Token**.
3.  In n8n, create a **Telegram API** credential and insert the token.
4.  Get your personal Chat ID using the `@userinfobot` on Telegram. In the workflow's Telegram node, input this ID as the target.
