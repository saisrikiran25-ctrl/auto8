# Setup Guide

This guide walks you through importing the **Student Deadline Command Center Automation** workflow into n8n, configuring connection endpoints, setting up your preferred AI model provider, and running validation tests.

---

## 1. Prerequisites

Before starting, ensure you have:
*   An active **n8n instance** (Self-hosted, Desktop app, or n8n Cloud).
*   Access to an LLM provider API key (e.g., **Google Gemini**, **OpenAI**, or **Anthropic**).
*   Input systems set up (such as a Google Form, Notion database, or email forwarding address).

---

## 2. Step 1: Import the n8n Workflow

1.  Navigate to the `/workflow` directory of this project and open [student-deadline-command-center.json](../workflow/student-deadline-command-center.json).
2.  Select all (`Ctrl+A` or `Cmd+A`) and copy the raw JSON contents to your clipboard.
3.  Open your n8n workspace, create a **New Workflow**, and click anywhere on the canvas.
4.  Paste (`Ctrl+V` or `Cmd+V`) the JSON. n8n will automatically render all triggers, code blocks, conditional switches, and AI nodes.

---

## 3. Step 2: Configure Provider & Model Settings

The workflow uses a **Provider Config Node** to manage global variables. This ensures you do not have to update settings in individual LLM nodes.

1.  Locate the node labeled `Set Provider Config` (near the top-left of the canvas).
2.  Double-click the node to open its parameters.
3.  Set the values for your active provider:
    *   `model_provider`: Choose from `gemini`, `openai`, or `anthropic`.
    *   `model_name`: Set the specific model (e.g., `gemini-1.5-flash`, `gpt-4o-mini`, or `claude-3-5-sonnet`).
    *   `temperature`: Default is `0.1` (recommended to keep low for predictable, structured output).
4.  Open the corresponding **LLM Model Connection Node** linked to the AI chain (e.g., `Google Gemini Chat Model` or `OpenAI Chat Model`).
5.  Select or create your API credential in the node's credential field.

---

## 4. Step 3: Connect Input Channels

The intake node begins with a **Webhook Trigger**. You can feed data into this trigger from various sources:

### Option A: Google Forms (Recommended for Manual Ingest)
1.  Create a Google Form with fields matching our input schema: *Student Name, Task Title, Course Name, Task Type, Due Date, Estimated Hours, Grade Weight (%), and Notes.*
2.  Use a Google Sheets script or an n8n Google Sheets trigger to capture form entries and POST them to your n8n Webhook URL.

### Option B: Notion Database integration
1.  Connect n8n to your Notion workspace using an integration token.
2.  Replace the Webhook Trigger node with a Notion `On Database Page Added` trigger.
3.  Map the Notion properties into the normalization script node (`Normalize Student Input`).

### Option C: Google Calendar / ICS Ingestion
1.  Use the n8n Google Calendar node to retrieve events.
2.  Filter events that match keywords (e.g., "Exam", "Due", "HW") and map the event times to the `due_date` field.

---

## 5. Step 4: Running Test Payloads

To verify the installation:
1.  Click **Listen for Test Event** on the n8n Webhook node.
2.  Using a utility like `cURL`, Postman, or a Python script, send a POST request with the sample payload from [student-deadline-input-payload-example.json](../schemas/student-deadline-input-payload-example.json) to your test webhook URL.

3.  Verify the path executions:
    *   If `analysis_mode` is `prioritize`, the flow should execute the prioritization branch and return Mode A results.
    *   If `analysis_mode` is `weekly_plan`, it should run through both AI engines and output Mode B results.

---

## 6. Interpreting Outputs

The workflow produces a unified, structured JSON structure:
*   `status`: Indicates `success` or `fallback`.
*   `mode`: The active operating mode (`prioritize` or `weekly_plan`).
*   `payload`: Contains the prioritized deadline list or the scheduled weekly calendar blocks.
*   `warnings`: List of overload risks, calendar clashes, or deterministic error flags.
*   `needs_human_review`: A boolean flag. If `true`, the data should be manually verified because of invalid/missing dates or massive overload alerts.

---

## 7. Common First-Run Checks & Troubleshooting

*   **Error: `Invalid Input Schema`**
    *   *Cause:* The payload missing `student_id` or the `deadline_items` array is formatted incorrectly.
    *   *Solution:* Ensure the payload exactly matches the fields in `schemas/student-deadline-input-payload-example.json`.
*   **Error: `LLM Output Parse Failed`**
    *   *Cause:* The LLM returned markdown blocks (like ` ```json `) instead of raw text, causing JSON parse errors in subsequent nodes.
    *   *Solution:* Ensure the LLM nodes are configured with `Response Format: JSON Object` (or ensure you are using a model that natively supports JSON outputs).
*   **Error: `Timezone Mismatch`**
    *   *Cause:* Dates parsed in 12-hour format or lacking timezone offset headers.
    *   *Solution:* Normalization node expects full ISO-8601 format (e.g., `YYYY-MM-DDTHH:mm:ssZ`). Ensure input dates are standardized before sending.
