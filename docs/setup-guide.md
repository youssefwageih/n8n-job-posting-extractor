# Setup and Configuration Guide

This guide walks you through setting up the 3 external connections needed for the **Job Posting Extractor & CV Matcher** n8n workflow.

---

## 1. Telegram Bot Setup

1. Open Telegram and search for `@BotFather`.
2. Send `/newbot` and follow the prompts to choose a bot name and username.
3. Save the **HTTP API Bot Token** provided by BotFather (e.g., `123456789:ABCdef...`).
4. In your n8n workspace:
   - Navigate to **Credentials** > **New Credential** > **Telegram API**.
   - Paste your Access Token and save the credential.
5. In the workflow, assign this credential to:
   - `Telegram Trigger`
   - `Get CV File`
   - `Confirm CV Saved`
   - `CV Read Failed`
   - `Send Summary`
   - `Reply Not a Job Posting`
   - `Report Fetch Failure`

> **Note**: If you run n8n on n8n Cloud, Telegram webhooks work automatically. If self-hosting locally, ensure your n8n instance is reachable via a public HTTPS URL (e.g. using Cloudflare Tunnel, ngrok, or reverse proxy).

---

## 2. n8n Data Table Setup (`CV Store`)

The workflow uses an n8n Data Table to store each candidate's resume text, keyed by their Telegram chat ID.

1. In n8n, navigate to **Data Tables** (in the left navigation panel).
2. Click **Create Table** and name it: `CV Store`.
3. Add these 4 columns:

| Column Name   | Type   | Purpose |
| ------------- | ------ | ------- |
| `chat_id`     | String | Unique Telegram chat ID used to match the user |
| `file_name`   | String | Name of the uploaded resume PDF file |
| `cv_text`     | String | Extracted text content of the candidate's CV |
| `uploaded_at` | String | Upload timestamp formatted as `yyyy-MM-dd HH:mm` |

4. When you import the workflow, verify that both the **Save CV** and **Get Saved CV** nodes point to this `CV Store` table.

---

## 3. OpenAI API Credential

1. Sign up or log into [platform.openai.com](https://platform.openai.com/).
2. Create an API Key in the **API Keys** section.
3. In n8n:
   - Go to **Credentials** > **New Credential** > **OpenAI API**.
   - Paste your API key and save.
4. Open the workflow and verify the **OpenAI Chat Model** node is linked to your credential (model defaults to `gpt-4o-mini`).

---

## 4. Google Sheets Setup

The automation automatically records every processed job lead into a Google Spreadsheet.

### Step 4.1: Create the Spreadsheet
1. Create a new Google Spreadsheet (e.g. named `Job Analyzer`).
2. Rename the active sheet tab to `Sheet1` (or your preferred tab name).
3. Set the first row with these exact headers (or import `templates/google_sheets_template.csv`):
   - `Date`
   - `Job Title`
   - `Pay`
   - `Tools & Skills`
   - `Education`
   - `Experience`
   - `Other`
   - `Key Tasks`
   - `Notes`
   - `Link`
   - `CV Fit`

### Step 4.2: Connect Google Sheets in n8n
1. Create a **Google Sheets OAuth2** credential in n8n (or configure a Service Account).
2. Open the **Save to Google Sheet** node in the workflow.
3. Select your credential.
4. Under **Document**, select **By ID** and paste your spreadsheet ID (found in the URL: `https://docs.google.com/spreadsheets/d/<SPREADSHEET_ID>/edit`).

---

## 5. Activating and Verifying the Workflow

1. Toggle the workflow switch in n8n to **Active**.
2. Open your Telegram bot in the Telegram app and send `/start`.
3. **Upload Resume**: Send a text-based PDF resume. The bot will confirm:
   > *"Got your CV. Every job posting you send from now on will include a Fit / Partial fit / Miss check against it..."*
4. **Send Job Link or Description**:
   - Send a URL to a job opening (e.g., from a company careers page).
   - Or paste a job description text directly.
5. The bot will reply with a structured breakdown and append a new row into your Google Sheet.
