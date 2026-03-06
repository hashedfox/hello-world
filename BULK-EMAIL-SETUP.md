# n8n Bulk Custom Email Workflow

Automatically sends personalized, individually unique emails at scale from a Google Sheets spreadsheet via your email account.

## Workflow Architecture

```
Manual Trigger
  → Read Spreadsheet (Google Sheets)
    → Filter Valid Emails (regex + empty check)
      → Rate Limiter (1 email per batch)
        → Compose Personalized Email (Code node - template engine)
          → Send Email (Gmail OAuth2 or SMTP)
            → Delay Between Emails (3s default)
              → Log Sent Email
                → Write Log to Sheet
                  → Loop Back → Rate Limiter (next email)
```

## Setup Instructions

### 1. Import the Workflow

1. Open your n8n instance
2. Go to **Workflows → Import from File**
3. Select `n8n-bulk-email-workflow.json`

### 2. Configure Credentials

#### Gmail (default)
1. In n8n, go to **Settings → Credentials → Add Credential → Gmail OAuth2**
2. Follow the OAuth2 flow to connect your Google account
3. Open the **Send Email (Gmail)** node and select your credential

#### SMTP (alternative - Outlook, custom domain, etc.)
1. Disable the **Send Email (Gmail)** node
2. Enable the **Send Email (SMTP)** node
3. Create an SMTP credential with your provider's settings:
   - **Host**: e.g., `smtp.office365.com`, `smtp.gmail.com`
   - **Port**: 587 (TLS) or 465 (SSL)
   - **User**: your email address
   - **Password**: your app password

### 3. Configure Google Sheets

1. Add a **Google Sheets OAuth2** credential in n8n
2. Open the **Read Spreadsheet** node and set your Sheet's document ID
3. Update the sheet name if it's not "Sheet1"

### 4. Prepare Your Spreadsheet

Create a Google Sheet with these columns (header row required):

| email | first_name | last_name | company | subject | body_template | custom_field_1 | custom_field_2 |
|-------|-----------|-----------|---------|---------|---------------|----------------|----------------|
| jane@example.com | Jane | Doe | Acme Corp | Hi {{first_name}} - Quick question about {{company}} | {{greeting}}\n\nI saw {{company}} is doing great work... | Custom intro text | Custom CTA text |

**Required columns:**
- `email` - recipient address

**Optional columns (used for personalization):**
- `first_name`, `last_name`, `company`
- `subject` - supports `{{placeholder}}` syntax
- `body_template` - full HTML/text body with placeholders
- `custom_field_1`, `custom_field_2` - any extra personalization data

### 5. Template Placeholders

Use these in both `subject` and `body_template` columns:

| Placeholder | Replaced With |
|-------------|---------------|
| `{{greeting}}` | "Hi FirstName," or "Hello," |
| `{{first_name}}` | first_name column |
| `{{last_name}}` | last_name column |
| `{{company}}` | company column |
| `{{custom_field_1}}` | custom_field_1 column |
| `{{custom_field_2}}` | custom_field_2 column |

If no `body_template` is provided, a default professional template is used automatically.

## Features

- **Unique emails**: Each email is individually composed from per-row spreadsheet data
- **Email validation**: Filters out empty or malformed email addresses before sending
- **Rate limiting**: 3-second delay between sends (adjustable in the Delay node)
- **Batch processing**: Processes one email at a time to avoid spam triggers
- **Send logging**: Writes status, recipient, subject, and timestamp to a "Send Log" sheet tab
- **Error handling**: Failed sends are captured and logged; the workflow continues with remaining emails
- **Dual provider support**: Gmail OAuth2 (enabled) and SMTP (disabled, ready to swap)

## Adjusting Send Rate

Edit the **Delay Between Emails** node to change the pause between sends:
- **Conservative**: 5-10 seconds (recommended for large batches 500+)
- **Default**: 3 seconds (good for batches under 500)
- **Aggressive**: 1 second (only if your provider allows it)

## Gmail Daily Limits

- **Free Gmail**: ~500 emails/day
- **Google Workspace**: ~2,000 emails/day

Plan your batch sizes accordingly.

## Adding a "Send Log" Tab

Create a second tab in your spreadsheet named **Send Log** with columns:
- `status`
- `recipient`
- `subject`
- `timestamp`

The workflow will automatically append results here after each send.
