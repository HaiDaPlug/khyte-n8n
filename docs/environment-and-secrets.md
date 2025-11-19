# Environment & Secrets

This document explains how to handle credentials, API keys, and configuration values in `khyte-n8n` workflows.

## Core Principle: Never Hardcode

**Never** put secrets directly in workflows:

- ❌ API keys in HTTP Request nodes
- ❌ Database passwords in SQL nodes
- ❌ Webhook URLs with tokens in them
- ❌ Email addresses (if sensitive)

**Always** use n8n's credential system or environment variables.

## Two Types of Configuration

### 1. Secrets (sensitive)

Things that must be protected:

- API keys and tokens
- Database passwords
- OAuth credentials
- Webhook URLs (if they contain secret tokens)
- SMTP passwords

**How to handle:** Use n8n's **Credentials** system.

### 2. Configuration (non-sensitive)

Values that change per deployment but aren't secret:

- Sheet IDs
- CRM instance URLs
- Email addresses (for notifications)
- Slack channel names
- Workflow-specific settings (thresholds, schedules)

**How to handle:** Use **Environment Variables** or **workflow settings**.

## Using n8n Credentials

### Built-in credential types

n8n has pre-built credential types for common services:

- Google Sheets API
- OpenAI API
- Slack API
- Gmail/SMTP
- PostgreSQL / MySQL
- HTTP Basic Auth / API Key Auth
- OAuth2

**How to use:**

1. In n8n UI, go to **Credentials** → **Add Credential**
2. Choose the credential type (e.g. "OpenAI API")
3. Fill in the required fields (API key, etc.)
4. Give it a name (e.g. "OpenAI - Production" or "OpenAI - Client ABC")
5. Save

Then in your workflow nodes, select this credential from the dropdown.

### Custom API credentials

For APIs without built-in credential types, use:

- **HTTP Request node** → **Generic Credential Type** → **API Key Auth** or **Header Auth**

Example (API key in header):

```
Credential Type: Header Auth
Name: X-API-Key
Value: your-api-key-here
```

### Per-client credentials

For consulting work, create separate credentials per client:

- `OpenAI - Khyte Internal`
- `OpenAI - Client ABC`
- `Slack - Khyte`
- `Slack - Client ABC`

This makes it easy to:

- Switch between client environments
- Hand off workflows to clients (they create their own credentials)
- Avoid accidental cross-client data leaks

## Using Environment Variables

Environment variables are for **non-secret configuration** that changes between environments.

### Setting environment variables

**Self-hosted n8n:**

Add to your `.env` file or `docker-compose.yml`:

```bash
# .env
SHEET_ID=1a2b3c4d5e6f7g8h9i0j
CRM_BASE_URL=https://api.pipedrive.com/v1
NOTIFICATION_EMAIL=hai@khyte.se
AI_MODEL=claude-3-5-sonnet-20241022
```

**n8n Cloud:**

Set in your instance settings:

1. Go to **Settings** → **Environments**
2. Add variables as `KEY=VALUE`

### Accessing in workflows

Use the syntax: `{{ $env.VARIABLE_NAME }}`

**Example (Set node):**

```json
{
  "sheetId": "{{ $env.SHEET_ID }}",
  "model": "{{ $env.AI_MODEL }}",
  "notificationEmail": "{{ $env.NOTIFICATION_EMAIL }}"
}
```

**Example (HTTP Request URL):**

```
{{ $env.CRM_BASE_URL }}/deals
```

### Common environment variables

Here are typical env vars for `khyte-n8n` workflows:

```bash
# AI
AI_MODEL=claude-3-5-sonnet-20241022
AI_TEMPERATURE=0.3
AI_MAX_TOKENS=2000

# Data sources
LEADS_SHEET_ID=abc123
CRM_BASE_URL=https://api.your-crm.com
DATABASE_HOST=db.example.com

# Notifications
NOTIFICATION_EMAIL=alerts@khyte.se
SLACK_CHANNEL=#automation-alerts
ERROR_NOTIFICATION_LEVEL=error  # or "warning" or "info"

# Workflow settings
LEAD_SCORE_THRESHOLD=7
FOLLOWUP_DELAY_DAYS=5
BATCH_SIZE=50
```

## API Keys and Tokens

### Storing API keys

**For common services (OpenAI, Anthropic, etc.):**

Use n8n credentials with the service-specific credential type.

**For generic APIs:**

Option 1: Use HTTP Request node's **Header Auth** credential type

```
Header Name: Authorization
Value: Bearer your-api-key
```

Option 2: Use environment variable + expression

```
Headers:
{
  "Authorization": "Bearer {{ $env.MY_API_KEY }}"
}
```

**Recommendation:** Use credentials for secrets, env vars for configuration.

### Rotating keys

When you need to rotate an API key:

1. Generate new key from the service
2. Update n8n credential (don't create a new one, just edit the existing)
3. Test workflows
4. Deactivate old key in the service

Using credential names (not hardcoded keys) makes rotation easy — update once, all workflows get the new key.

## Handling Multiple Environments

Many consultants work with:

- **Dev/test** environment (your own n8n instance)
- **Production** environment (client's n8n instance or your production instance)

### Strategy 1: Separate credentials per environment

Create credentials with environment prefixes:

- `OpenAI - Dev`
- `OpenAI - Prod`
- `Slack - Dev`
- `Slack - Prod`

When deploying to production, update workflows to use "Prod" credentials.

### Strategy 2: Same credential names, different values

Keep credential **names** the same across environments:

- `OpenAI API` (but different API key in dev vs prod)
- `Slack Webhook` (but different webhook URL in dev vs prod)

Export workflow from dev, import to prod — credentials auto-map if names match.

**Recommendation:** Strategy 2 is cleaner for workflows you deploy to multiple clients.

## Handing Off Workflows to Clients

When giving a workflow to a client:

### Step 1: Document required credentials

Create a checklist like:

```
Required Credentials:
- [ ] OpenAI API (credential type: OpenAI API)
  - API Key from platform.openai.com
- [ ] Google Sheets (credential type: Google Service Account)
  - Service account JSON from Google Cloud Console
- [ ] Slack Webhook (credential type: Webhook)
  - Webhook URL from Slack app settings
```

### Step 2: Use generic credential names

In your workflow, use names like:

- `OpenAI API` (not "Hai's OpenAI Key")
- `Google Sheets Access` (not "Khyte Sheets Credential")

This makes it clear what the client needs to create.

### Step 3: Export workflow

Export from n8n. The exported JSON will have credential **references** but not the actual secrets.

### Step 4: Client imports and maps credentials

Client:

1. Imports your workflow JSON
2. Creates their own credentials in their n8n instance
3. Maps workflow's credential references to their credentials

### Step 5: Set environment variables

Provide a list of required env vars:

```bash
# Add these to your n8n environment:
SHEET_ID=your-google-sheet-id
NOTIFICATION_EMAIL=your-email@example.com
LEAD_SCORE_THRESHOLD=7
```

Client sets these in their n8n environment settings.

## Common Services & Credentials

### OpenAI / Anthropic (Claude)

**Credential type:** API Key Auth (Header)

```
Header Name: Authorization
Value: Bearer sk-...
```

Or use built-in "OpenAI Account" credential type if available.

**Environment variables:**

```bash
AI_MODEL=gpt-4-turbo  # or claude-3-5-sonnet-20241022
AI_MAX_TOKENS=2000
AI_TEMPERATURE=0.3
```

### Google Sheets

**Credential type:** Google Sheets OAuth2 or Service Account

**For OAuth2:**
- Easier for manual use
- Requires user to authenticate

**For Service Account:**
- Better for automation
- Requires JSON key file from Google Cloud Console
- Sheet must be shared with service account email

**Environment variables:**

```bash
SHEET_ID=1a2b3c4d5e6f
SHEET_RANGE=Sheet1!A:Z
```

### Slack

**For incoming webhooks:**

**Credential type:** Webhook (or HTTP Request with URL as credential)

```
Webhook URL: https://hooks.slack.com/services/T00000000/B00000000/XXXXXXXXXXXXXXXXXXXX
```

**For Slack API:**

**Credential type:** Slack OAuth2

Get from: Slack app settings

**Environment variables:**

```bash
SLACK_CHANNEL=#automation-alerts
SLACK_USERNAME=n8n Bot
```

### Email (SMTP)

**Credential type:** SMTP

Required fields:

- Host (e.g. `smtp.gmail.com`)
- Port (e.g. `587`)
- User (email address)
- Password (or app-specific password for Gmail)

**Environment variables:**

```bash
EMAIL_FROM=automation@khyte.se
EMAIL_REPLY_TO=hai@khyte.se
```

### CRM (HubSpot, Pipedrive, etc.)

**Credential type:** Varies by CRM (often OAuth2 or API Key)

**Environment variables:**

```bash
CRM_BASE_URL=https://api.pipedrive.com/v1
CRM_PIPELINE_ID=1
```

### Databases (PostgreSQL, MySQL, Supabase)

**Credential type:** PostgreSQL / MySQL / Generic SQL

Required fields:

- Host
- Database name
- User
- Password
- Port

**For Supabase:**

```
Host: db.your-project.supabase.co
Port: 5432
Database: postgres
User: postgres
Password: your-password
SSL: enabled
```

**Environment variables:**

```bash
DB_TABLE_NAME=leads
DB_SCHEMA=public
```

## Security Best Practices

### 1. Principle of least privilege

Give credentials only the permissions they need:

- **Read-only** credentials for workflows that only read data
- **Scoped tokens** (e.g. only access specific sheets, not all of Google Drive)
- **Separate credentials per workflow** if they need different access levels

### 2. Audit credential usage

Periodically review:

- Which workflows use which credentials
- Whether old credentials can be deactivated
- Whether any credentials are shared across clients (shouldn't be)

### 3. Don't commit credentials to git

If you export workflows for version control:

- Never commit `.env` files with real values
- Use `.env.example` with placeholder values
- Add `.env` to `.gitignore`

### 4. Rotate regularly

For sensitive systems:

- Rotate API keys every 90 days (or per your security policy)
- Use different keys for dev vs prod
- Immediately rotate if a credential is exposed

### 5. Monitor for leaks

Check that credentials aren't being:

- Logged in workflow execution history (n8n doesn't log credentials, but custom logs might)
- Sent in notifications (don't include API keys in Slack messages)
- Written to sheets or databases (don't store API keys as data)

## Example: Full Setup for a Workflow

Let's say you're deploying the **Lead Enrichment Pipeline** workflow.

### Required credentials

Create these in n8n:

1. **Google Sheets OAuth2**
   - Name: `Google Sheets Access`
   - Authenticate with your Google account
   - Share the leads sheet with this account

2. **OpenAI API**
   - Name: `OpenAI API`
   - API Key: `sk-...` (from platform.openai.com)

3. **Slack Webhook**
   - Name: `Slack Notifications`
   - URL: `https://hooks.slack.com/services/...` (from Slack app)

### Required environment variables

Add to `.env` (self-hosted) or n8n environment settings (cloud):

```bash
# Data source
LEADS_SHEET_ID=1a2b3c4d5e6f7g8h9i0j
LEADS_SHEET_RANGE=Sheet1!A:Z

# AI settings
AI_MODEL=gpt-3.5-turbo
AI_TEMPERATURE=0.2
AI_MAX_TOKENS=1000

# Notifications
SLACK_CHANNEL=#lead-alerts
NOTIFICATION_EMAIL=hai@khyte.se

# Workflow settings
BATCH_SIZE=50
ENRICHMENT_REQUIRED_FIELDS=name,email,company
```

### In the workflow

**Google Sheets node:**
- Credential: Select "Google Sheets Access"
- Sheet ID: `{{ $env.LEADS_SHEET_ID }}`
- Range: `{{ $env.LEADS_SHEET_RANGE }}`

**HTTP Request node (to OpenAI):**
- Credential: Select "OpenAI API"
- Body: `{ "model": "{{ $env.AI_MODEL }}", "temperature": {{ $env.AI_TEMPERATURE }}, ... }`

**Slack node:**
- Credential: Select "Slack Notifications"
- Channel: `{{ $env.SLACK_CHANNEL }}`

Now you can change `AI_MODEL` or `SLACK_CHANNEL` without editing the workflow — just update the environment variable.

## Troubleshooting

### "Missing credential" error

**Cause:** Workflow references a credential that doesn't exist in this n8n instance.

**Solution:**
1. Check credential name in the node
2. Create a credential with that exact name
3. Or edit the node to select an existing credential

### "Invalid API key" error

**Cause:** Credential contains wrong or expired API key.

**Solution:**
1. Go to Credentials in n8n
2. Edit the credential
3. Update with correct API key
4. Test the workflow

### Environment variable not found

**Cause:** `{{ $env.VARIABLE_NAME }}` returns empty or undefined.

**Solution:**
1. Check spelling (exact match, case-sensitive)
2. Verify variable is set in n8n environment (self-hosted: `.env`, cloud: settings)
3. Restart n8n if you just added the variable (self-hosted)

### Credentials work in dev but not in prod

**Cause:** Different credential values in prod, or credential not created yet.

**Solution:**
1. Verify credential exists in prod n8n instance
2. Check that credential has correct values (API keys, URLs, etc.)
3. Ensure environment variables are set in prod environment

## Summary

**Secrets (API keys, passwords):**
- Use n8n **Credentials**
- Create per-client credentials for consulting work
- Never hardcode in workflows

**Configuration (sheet IDs, URLs, settings):**
- Use **Environment Variables**
- Document required variables
- Use `{{ $env.VAR_NAME }}` syntax

**When handing off to clients:**
- Export workflows (credentials stay behind)
- Provide credential + env var checklists
- Client creates their own credentials in their n8n instance

**Security:**
- Least privilege
- Rotate regularly
- Never commit to git
- Monitor for leaks

Follow these practices and you'll have workflows that are secure, portable, and easy to deploy across environments and clients.
