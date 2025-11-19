# Workflow: Lead Enrichment Pipeline

## Summary

Automatically enrich raw lead data with AI-powered analysis and external API lookups. Takes basic lead information (name, company, website) and adds structured intelligence: industry classification, company size estimation, seniority detection, and relevant tags.

**Time saved:** 5-10 minutes per lead → 30 seconds automated
**Best for:** Agencies, consultants, sales teams processing inbound leads

## Use Cases

1. **Agency lead qualification**
   - New leads from contact forms or LinkedIn exports
   - Enrich with industry, size, decision-maker status
   - Prioritize by fit score

2. **Sales pipeline preparation**
   - Import raw leads from events, webinars, downloads
   - Add context before sales team touches them
   - Flag high-value prospects

3. **CRM data cleanup**
   - Enrich existing CRM records with missing fields
   - Standardize industry/company data
   - Add AI-generated outreach angles

## Inputs & Dependencies

**Required:**
- **Lead data source:** Google Sheets, CSV upload, or CRM
- **AI API access:** OpenAI or Anthropic (for enrichment)
- **n8n credentials:**
  - Google Sheets OAuth2 (if using Sheets)
  - OpenAI/Anthropic API key

**Lead data format (minimum fields):**
- `name` — Contact name
- `email` — Email address
- `company` — Company name

**Optional but helpful:**
- `website` — Company website
- `title` — Job title
- `phone` — Phone number
- `notes` — Any additional context

## High-Level Flow

```
1. Trigger (new row in sheet / scheduled batch)
   ↓
2. Fetch unprocessed leads
   ↓
3. For each lead:
   3a. Normalize data (trim, lowercase email, etc.)
   3b. Fetch company info (optional: Clearbit/similar API)
   3c. Call AI to enrich:
       - Classify industry
       - Estimate company size
       - Detect seniority level
       - Generate tags
       - Suggest outreach angle
   3d. Write enriched data back to sheet/CRM
   ↓
4. Send summary notification
   ↓
5. Log completion to audit log
```

## Node-by-Node Implementation

### 1. Trigger: New Row or Schedule (Trigger node)

**Option A: Google Sheets Trigger (real-time)**
```
Trigger: On row added
Sheet ID: {{ $env.LEADS_SHEET_ID }}
Trigger column: Status (triggers when status is empty or "new")
```

**Option B: Schedule Trigger (batch processing)**
```
Trigger: Cron - every 30 minutes
Expression: */30 * * * *
```

### 2. Fetch Unprocessed Leads (Google Sheets node)

```
Operation: Get All
Sheet ID: {{ $env.LEADS_SHEET_ID }}
Range: Sheet1!A:Z

Filter: Where "Status" column is empty or "new"
```

### 3. Check if Leads Exist (IF node)

```
Condition: {{ $json.length > 0 }}
True: Process leads
False: End (no new leads)
```

### 4. Log Workflow Start (Audit Log subflow)

```json
{
  "workflowName": "lead-enrichment-pipeline",
  "eventType": "start",
  "payloadSummary": "Processing {{ $json.length }} leads"
}
```

### 5. Loop Through Leads (Split In Batches node)

```
Batch Size: 10
Options: Keep batches for downstream
```

### 6. Normalize Lead Data (Function node)

```javascript
const lead = $input.item.json;

return {
  id: lead.id || lead.rowNumber,
  name: (lead.name || '').trim(),
  email: (lead.email || '').toLowerCase().trim(),
  company: (lead.company || '').trim(),
  website: (lead.website || '').trim().replace(/\/$/, ''),
  title: (lead.title || '').trim(),
  rawData: lead
};
```

### 7. Validate Required Fields (IF node)

```
Condition: {{ $json.email !== '' && $json.company !== '' }}
True: Proceed to enrichment
False: Skip (mark as invalid)
```

### 8. Optional: Fetch Company Data (HTTP Request node)

If using an external API (Clearbit, Hunter.io, etc.):

```
Method: GET
URL: https://company.clearbit.com/v2/companies/find?domain={{ $json.website }}
Headers:
  Authorization: Bearer {{ $env.CLEARBIT_API_KEY }}
Continue On Fail: true
```

Skip this step if you don't have external APIs.

### 9. Build AI Enrichment Prompt (Function node)

```javascript
const lead = $input.item.json;
const companyData = $('Fetch Company Data').item?.json || {};

const prompt = `Extract and enrich information for this lead:

Name: ${lead.name}
Email: ${lead.email}
Company: ${lead.company}
Website: ${lead.website}
Title: ${lead.title}

${companyData.description ? `Company description: ${companyData.description}` : ''}

Please analyze and return the following:
1. industry: Best-fit industry category
2. companySize: Estimate (small/medium/large)
3. seniority: Job seniority (owner/executive/manager/specialist/other)
4. tags: Array of relevant tags (e.g. ["agency", "local", "b2b"])
5. fitScore: Relevance for automation consulting (1-10)
6. outreachAngle: One-sentence personalized angle for outreach`;

return {
  prompt,
  lead,
  companyData
};
```

### 10. Call AI for Enrichment (AI Call Wrapper subflow)

```json
{
  "prompt": "={{ $json.prompt }}",
  "mode": "json",
  "systemInstructions": "You are a B2B lead analyst. Analyze leads for automation and AI consulting relevance. Return valid JSON matching the requested schema.",
  "temperature": 0.2,
  "model": "gpt-4-turbo"
}
```

### 11. Check AI Success (IF node)

```
Condition: {{ $json.success === true }}
True: Extract enrichment data
False: Handle error
```

### 12. Extract Enrichment Data (Function node)

```javascript
const enrichment = $input.item.json.outputJson;
const lead = $('Build AI Enrichment Prompt').item.json.lead;

return {
  ...lead,
  industry: enrichment.industry,
  companySize: enrichment.companySize,
  seniority: enrichment.seniority,
  tags: enrichment.tags.join(', '),
  fitScore: enrichment.fitScore,
  outreachAngle: enrichment.outreachAngle,
  enrichedAt: new Date().toISOString(),
  status: 'enriched'
};
```

### 13. Write Enriched Data Back (Google Sheets node)

```
Operation: Update
Sheet ID: {{ $env.LEADS_SHEET_ID }}
Range: Find row by ID/email

Update fields:
- industry: {{ $json.industry }}
- companySize: {{ $json.companySize }}
- seniority: {{ $json.seniority }}
- tags: {{ $json.tags }}
- fitScore: {{ $json.fitScore }}
- outreachAngle: {{ $json.outreachAngle }}
- enrichedAt: {{ $json.enrichedAt }}
- status: enriched
```

### 14. Handle Enrichment Errors (Function node)

For leads that failed enrichment:

```javascript
const lead = $('Build AI Enrichment Prompt').item.json.lead;
const error = $input.item.json.error;

return {
  ...lead,
  status: 'enrichment_failed',
  errorMessage: error.message,
  enrichedAt: new Date().toISOString()
};
```

### 15. Aggregate Results (Aggregate node)

```
Aggregate All Items
```

### 16. Build Summary (Function node)

```javascript
const items = $input.all();

const total = items.length;
const succeeded = items.filter(i => i.json.status === 'enriched').length;
const failed = items.filter(i => i.json.status === 'enrichment_failed').length;

return {
  total,
  succeeded,
  failed,
  successRate: ((succeeded / total) * 100).toFixed(1) + '%'
};
```

### 17. Send Notification (Notification Dispatch subflow)

```json
{
  "severity": "info",
  "message": "Lead enrichment completed",
  "context": {
    "Total leads": "={{ $json.total }}",
    "Enriched": "={{ $json.succeeded }}",
    "Failed": "={{ $json.failed }}",
    "Success rate": "={{ $json.successRate }}"
  },
  "workflowName": "lead-enrichment-pipeline",
  "channels": ["slack"]
}
```

### 18. Log Completion (Audit Log subflow)

```json
{
  "workflowName": "lead-enrichment-pipeline",
  "eventType": "success",
  "payloadSummary": "Enriched {{ $('Build Summary').item.json.succeeded }} of {{ $('Build Summary').item.json.total }} leads",
  "details": "={{ $('Build Summary').item.json }}"
}
```

## Subflows Used

- **ai-call-wrapper:** For AI-powered enrichment (step 10)
- **notification-dispatch:** For completion notification (step 17)
- **audit-log:** For start/completion logging (steps 4 & 18)

Optional:
- **safe-http-call:** If using external company data APIs (step 8)

## Customization Tips

### Adjust AI prompt for your ideal customer profile

Modify the enrichment prompt to focus on your specific criteria:

```javascript
const prompt = `...

Focus on these criteria for fit score:
- Local businesses in Borås/Sweden area: +3 points
- Agency or professional services: +2 points
- 5-50 employees: +2 points
- Decision maker (owner/director): +2 points

Outreach angle should mention automation and time savings.`;
```

### Add more data sources

**Company website scraping (careful with ToS):**
- Use a scraping API to fetch company website content
- Pass to AI for deeper analysis

**LinkedIn data (from exports only):**
- If lead has LinkedIn URL (manually added or from export)
- Don't scrape, but use URL for context

**CRM historical data:**
- Check if company already in CRM
- Include past interaction notes in AI prompt

### Different enrichment tiers

**Basic tier (free/cheap):**
- Use GPT-3.5-turbo
- No external APIs
- Basic classification only

**Premium tier:**
- Use GPT-4-turbo
- Add Clearbit/Hunter.io
- Deeper analysis with competitive intel

**Customize in AI Call node:**
```json
{
  "model": "={{ $env.ENRICHMENT_TIER === 'premium' ? 'gpt-4-turbo' : 'gpt-3.5-turbo' }}"
}
```

### Trigger on specific lead sources

Add a filter after trigger:

```javascript
// Only process leads from LinkedIn
if ($json.source !== 'LinkedIn') {
  return null;  // Skip
}
return $json;
```

### Write to CRM instead of Sheets

Replace Google Sheets update node with CRM node (HubSpot, Pipedrive, etc.):

```
Operation: Update Contact
CRM Contact ID: {{ $json.crmId }}
Fields:
  industry: {{ $json.industry }}
  company_size: {{ $json.companySize }}
  ...
```

### Add human review for high-fit leads

After enrichment, check fit score:

```javascript
if ($json.fitScore >= 8) {
  // Send special notification
  // Or create task for manual review
}
```

## Environment Variables Needed

```bash
# Data source
LEADS_SHEET_ID=your-google-sheet-id

# AI
AI_MODEL=gpt-4-turbo
AI_TEMPERATURE=0.2

# Optional: External APIs
CLEARBIT_API_KEY=your-clearbit-key

# Notifications
SLACK_WEBHOOK_URL=your-slack-webhook
NOTIFICATION_EMAIL=your@email.com

# Audit
AUDIT_LOG_SHEET_ID=your-audit-sheet-id
```

## Performance Notes

- **Processing time:** ~30-60 seconds per lead (mostly AI calls)
- **Cost:** ~$0.01-0.05 per lead (depending on model)
- **Recommended batch size:** 10-50 leads at a time
- **Add delays:** Consider 500ms between API calls to avoid rate limits

## Common Issues & Solutions

**Issue: AI returns invalid JSON**
- Solution: The `ai-call-wrapper` subflow handles this with automatic retry

**Issue: Rate limits on AI API**
- Solution: Add a `Wait` node (1-2 seconds) between leads, or reduce batch size

**Issue: Missing company website**
- Solution: Try to infer from email domain, or mark as "manual review needed"

**Issue: Enrichment is too generic**
- Solution: Add more context to the AI prompt (your ICP, example good/bad leads)

This workflow is the foundation of intelligent lead management for consultants and agencies.
