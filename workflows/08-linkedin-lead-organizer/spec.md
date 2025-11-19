# Workflow: LinkedIn Lead Organizer

## Summary

Processes LinkedIn connection data (from CSV exports or manually curated lists) to organize, enrich, and prioritize leads. Uses AI to analyze profiles, categorize connections, and suggest personalized outreach strategies. **ToS-compliant: uses only official LinkedIn CSV exports and manual data entry - no scraping or automation of LinkedIn.com.**

**Time saved:** 45-60 minutes weekly → 5 minutes reviewing organized leads
**Best for:** Consultants, sales professionals, business development, recruiters

## Use Cases

1. **Sales pipeline from networking**
   - Export LinkedIn connections CSV
   - Enrich with company data and AI analysis
   - Prioritize by fit score for outreach

2. **Event follow-up**
   - Manually create list of people met at conference
   - LinkedIn profile URLs (manually collected)
   - Generate personalized follow-up angles

3. **Recruiter candidate tracking**
   - Export saved candidates from LinkedIn
   - Analyze skills and experience
   - Match to open positions

## IMPORTANT: LinkedIn ToS Compliance

**✅ ALLOWED:**
- Exporting your connections via LinkedIn's official CSV export
- Manually entering LinkedIn profile URLs
- Using publicly available company websites
- Enriching data with third-party APIs (Clearbit, Hunter.io)

**❌ NOT ALLOWED (and not included in this workflow):**
- Scraping LinkedIn.com pages
- Automated browsing of LinkedIn
- Using LinkedIn's API without authorization
- Extracting data from LinkedIn without user action

**This workflow complies with LinkedIn's Terms of Service by:**
- Only processing data YOU manually export or enter
- Not accessing LinkedIn.com programmatically
- Using external enrichment APIs (not LinkedIn data)

## Inputs & Dependencies

**Required:**
- **LinkedIn data source:** One of:
  - CSV export from LinkedIn Connections (Settings → Data Privacy → Download Data)
  - Manually created CSV with: Name, LinkedIn URL, Company, Title
  - Google Sheet with manually entered lead information
- **AI API access:** OpenAI or Anthropic (for analysis)
- **Enrichment APIs (optional):**
  - Clearbit (company data)
  - Hunter.io (email finding)
- **n8n credentials:**
  - Google Sheets OAuth2 (if using Sheets)
  - OpenAI/Anthropic API key
  - Optional: Clearbit, Hunter.io API keys

**Data format (CSV or Sheet columns):**
- `First Name` (required)
- `Last Name` (required)
- `Company` (required)
- `Position` / `Title` (required)
- `LinkedIn URL` (optional but helpful)
- `Email Address` (if available)
- `Connected On` (date - optional)
- `Notes` (optional)

## High-Level Flow

```
1. Trigger (manual upload of CSV / new rows in Sheet)
   ↓
2. Parse and validate lead data
   ↓
3. For each lead:
   3a. Check if already processed (deduplicate)
   3b. Normalize data (name, company, title)
   3c. Optional: Enrich with company data (Clearbit)
   3d. Optional: Find email (Hunter.io)
   3e. Call AI to analyze:
       - Industry and company category
       - Seniority level
       - Relevance/fit score for your business
       - Personalized outreach angle
       - Suggested next steps
   ↓
4. Categorize leads:
   4a. High priority (immediate outreach)
   4b. Medium priority (nurture)
   4c. Low priority (long-term network)
   ↓
5. Write enriched data to organized Sheet:
   5a. Main leads database
   5b. Separate tabs by category/priority
   ↓
6. Generate outreach suggestions document
   ↓
7. Send summary notification
   ↓
8. Log completion
```

## Node-by-Node Implementation

### 1. Trigger: Manual File Upload or Sheet Update

**Option A: Manual CSV Upload (Webhook)**

```
HTTP Method: POST
Path: /webhook/linkedin-leads-upload
Authentication: Header Auth
Content-Type: multipart/form-data

Expected: CSV file upload
```

**Option B: Google Sheets Trigger**

```
Trigger: On row added
Sheet ID: {{ $env.LINKEDIN_LEADS_SHEET_ID }}
Sheet: Raw Leads
Trigger column: Status (empty = new)
```

**Option C: Manual Trigger with CSV from Google Drive**

```
Trigger: Manual
→ Fetch file from Google Drive folder
→ Parse CSV
```

### 2. Parse CSV Data (CSV Parse node or Google Sheets node)

**If CSV upload:**

```
Operation: Parse CSV
Input: {{ $json.csvData }}
Options:
  - Delimiter: ,
  - Include header: Yes
  - Columns to include: All
```

**If Google Sheets:**

```
Operation: Get All
Sheet ID: {{ $env.LINKEDIN_LEADS_SHEET_ID }}
Range: Raw Leads!A:J
Filter: Where Status is empty or "new"
```

### 3. Validate and Normalize Lead Data (Function node)

```javascript
const leads = $input.all();

const validatedLeads = leads.map(item => {
  const lead = item.json;

  // Normalize names
  const firstName = (lead['First Name'] || lead.firstName || '').trim();
  const lastName = (lead['Last Name'] || lead.lastName || '').trim();
  const fullName = `${firstName} ${lastName}`.trim();

  // Normalize company and title
  const company = (lead.Company || lead.company || '').trim();
  const title = (lead.Position || lead.Title || lead.position || '').trim();

  // LinkedIn URL
  let linkedinUrl = (lead['URL'] || lead['LinkedIn URL'] || lead.linkedinUrl || '').trim();

  // Validate required fields
  const isValid = firstName && lastName && company && title;

  if (!isValid) {
    return {
      ...lead,
      validationError: 'Missing required fields (name, company, or title)',
      status: 'invalid'
    };
  }

  // Generate unique ID
  const uniqueId = `${firstName.toLowerCase()}-${lastName.toLowerCase()}-${company.toLowerCase().replace(/\s+/g, '-')}`;

  return {
    uniqueId,
    firstName,
    lastName,
    fullName,
    company,
    title,
    linkedinUrl,
    email: (lead['Email Address'] || lead.email || '').trim(),
    connectedOn: lead['Connected On'] || lead.connectedOn || null,
    notes: lead.Notes || lead.notes || '',
    source: 'linkedin-export',
    status: 'new',
    rawData: lead
  };
});

return validatedLeads.filter(lead => lead.status !== 'invalid');
```

### 4. Check for Duplicates (Function node + Database lookup)

Query existing leads database:

```
Operation: Get All
Sheet ID: {{ $env.LEADS_DATABASE_SHEET_ID }}
Range: Leads Database!A:Z
```

Then check for duplicates:

```javascript
const newLeads = $('Validate and Normalize Lead Data').all();
const existingLeads = $input.all();

// Build lookup of existing leads
const existingLookup = new Set(
  existingLeads.map(lead => lead.json.uniqueId)
);

// Filter out duplicates
const deduplicatedLeads = newLeads.filter(lead =>
  !existingLookup.has(lead.json.uniqueId)
);

return {
  newLeads: deduplicatedLeads,
  duplicatesSkipped: newLeads.length - deduplicatedLeads.length,
  totalProcessed: newLeads.length
};
```

### 5. Split Leads for Processing (Split In Batches node)

```
Batch Size: 10
Options: Keep input data
Input: {{ $json.newLeads }}
```

### 6. Optional: Enrich with Company Data (HTTP Request - Clearbit)

```
Method: GET
URL: https://company.clearbit.com/v2/companies/find
Parameters:
  domain: {{ $json.company.toLowerCase().replace(/\s+/g, '') }}.com
Headers:
  Authorization: Bearer {{ $env.CLEARBIT_API_KEY }}
Continue On Fail: Yes (company might not be found)
```

Parse Clearbit response:

```javascript
const lead = $('Validate and Normalize Lead Data').item.json;
const clearbitData = $input.item.json;

let enrichedData = { ...lead };

if (clearbitData.name) {
  enrichedData.companyFullName = clearbitData.name;
  enrichedData.companyDomain = clearbitData.domain;
  enrichedData.companyIndustry = clearbitData.category?.industry;
  enrichedData.companySize = clearbitData.metrics?.employees;
  enrichedData.companyLocation = clearbitData.geo?.city + ', ' + clearbitData.geo?.country;
  enrichedData.companyDescription = clearbitData.description;
}

return enrichedData;
```

### 7. Optional: Find Email Address (HTTP Request - Hunter.io)

Only if email not already present:

```javascript
if ($json.email && $json.email !== '') {
  return null;  // Skip, email already known
}
return $json;
```

```
Method: GET
URL: https://api.hunter.io/v2/email-finder
Parameters:
  domain: {{ $json.companyDomain }}
  first_name: {{ $json.firstName }}
  last_name: {{ $json.lastName }}
  api_key: {{ $env.HUNTER_API_KEY }}
Continue On Fail: Yes
```

Parse Hunter response:

```javascript
const lead = $input.item.json;
const hunterData = $('HTTP Request - Hunter.io').item.json;

if (hunterData.data?.email && hunterData.data?.score > 50) {
  lead.email = hunterData.data.email;
  lead.emailConfidence = hunterData.data.score;
  lead.emailSource = 'hunter.io';
}

return lead;
```

### 8. Build AI Analysis Prompt (Function node)

```javascript
const lead = $input.item.json;

const prompt = `Analyze this LinkedIn lead for business development prioritization.

**Lead Information:**
- Name: ${lead.fullName}
- Title: ${lead.title}
- Company: ${lead.company}
${lead.companySize ? `- Company size: ${lead.companySize} employees` : ''}
${lead.companyIndustry ? `- Industry: ${lead.companyIndustry}` : ''}
${lead.companyDescription ? `- Company description: ${lead.companyDescription}` : ''}
${lead.notes ? `- Notes: ${lead.notes}` : ''}

**Your Business Context:**
- You provide automation consulting and AI implementation services
- Ideal clients: SMBs (10-200 employees), agencies, professional services
- Focus: Time-saving automation, process optimization, AI integration

**Analysis needed:**

Return JSON:
{
  "industry": "Specific industry category",
  "seniority": "owner|c-level|director|manager|specialist|other",
  "decisionMaker": true/false (can they buy services?),
  "companyFit": "excellent|good|moderate|poor",
  "fitReason": "Why this fit score",
  "fitScore": 1-10 (relevance for automation consulting),
  "painPoints": ["likely pain point 1", "likely pain point 2"],
  "outreachAngle": "Personalized one-sentence approach (mention their role/industry)",
  "suggestedTopic": "Specific automation or AI topic to discuss",
  "priority": "high|medium|low",
  "nextSteps": "Specific suggested action (e.g., 'Send LinkedIn message about X', 'Email case study', 'Invite to webinar')"
}

Be specific and actionable. Base fit score on: company size, decision-making power, industry fit, and likely need for automation.`;

return {
  prompt,
  lead
};
```

### 9. Call AI for Lead Analysis (AI Call Wrapper subflow)

```json
{
  "prompt": "={{ $json.prompt }}",
  "mode": "json",
  "systemInstructions": "You are a B2B sales analyst specializing in automation and AI consulting. Analyze leads for relevance and provide specific, actionable outreach strategies. Be realistic about fit scores.",
  "temperature": 0.3,
  "model": "gpt-4-turbo"
}
```

### 10. Check AI Success (IF node)

```
Condition: {{ $json.success === true }}
True: Extract analysis
False: Use fallback (manual review)
```

### 11. Extract and Merge Analysis (Function node)

```javascript
const analysis = $input.item.json.outputJson;
const lead = $('Build AI Analysis Prompt').item.json.lead;

return {
  ...lead,
  // AI analysis fields
  industry: analysis.industry,
  seniority: analysis.seniority,
  decisionMaker: analysis.decisionMaker,
  companyFit: analysis.companyFit,
  fitReason: analysis.fitReason,
  fitScore: analysis.fitScore,
  painPoints: analysis.painPoints.join('; '),
  outreachAngle: analysis.outreachAngle,
  suggestedTopic: analysis.suggestedTopic,
  priority: analysis.priority,
  nextSteps: analysis.nextSteps,

  // Metadata
  processedAt: new Date().toISOString(),
  status: 'enriched'
};
```

### 12. Categorize by Priority (Function node)

```javascript
const leads = $input.all();

const highPriority = leads.filter(l => l.json.priority === 'high' && l.json.fitScore >= 7);
const mediumPriority = leads.filter(l => l.json.priority === 'medium' || (l.json.fitScore >= 4 && l.json.fitScore < 7));
const lowPriority = leads.filter(l => l.json.priority === 'low' || l.json.fitScore < 4);

return {
  highPriority,
  mediumPriority,
  lowPriority,
  summary: {
    total: leads.length,
    high: highPriority.length,
    medium: mediumPriority.length,
    low: lowPriority.length
  }
};
```

### 13. Write to Leads Database - Main Tab (Google Sheets node)

```
Operation: Append Rows
Sheet ID: {{ $env.LEADS_DATABASE_SHEET_ID }}
Sheet: All Leads

Columns:
  Unique ID: {{ $json.uniqueId }}
  Full Name: {{ $json.fullName }}
  Company: {{ $json.company }}
  Title: {{ $json.title }}
  Email: {{ $json.email }}
  LinkedIn URL: {{ $json.linkedinUrl }}
  Industry: {{ $json.industry }}
  Seniority: {{ $json.seniority }}
  Decision Maker: {{ $json.decisionMaker }}
  Fit Score: {{ $json.fitScore }}
  Priority: {{ $json.priority }}
  Outreach Angle: {{ $json.outreachAngle }}
  Suggested Topic: {{ $json.suggestedTopic }}
  Next Steps: {{ $json.nextSteps }}
  Company Fit: {{ $json.companyFit }}
  Pain Points: {{ $json.painPoints }}
  Processed At: {{ $json.processedAt }}
  Status: {{ $json.status }}
```

### 14. Write High Priority Leads to Separate Tab (Google Sheets node)

Loop through high priority leads:

```
Operation: Append Row
Sheet ID: {{ $env.LEADS_DATABASE_SHEET_ID }}
Sheet: High Priority - Action Needed

Columns: (same as above, plus)
  Follow-up Due: {{ new Date(Date.now() + 86400000 * 2).toISOString().split('T')[0] }}  // 2 days from now
```

### 15. Generate Outreach Suggestions Document (Function node)

```javascript
const { highPriority, mediumPriority, summary } = $input.item.json;

const outreachDoc = `# LinkedIn Lead Outreach Plan
Generated: ${new Date().toLocaleDateString()}

## Summary
- **Total leads processed:** ${summary.total}
- **High priority:** ${summary.high}
- **Medium priority:** ${summary.medium}
- **Low priority:** ${summary.low}

---

## High Priority Leads (Action This Week)

${highPriority.map((lead, i) => `
### ${i + 1}. ${lead.json.fullName} - ${lead.json.company}

**Role:** ${lead.json.title} (${lead.json.seniority})
**Fit Score:** ${lead.json.fitScore}/10 - ${lead.json.companyFit}
**Why relevant:** ${lead.json.fitReason}

**Outreach Strategy:**
- **Opening line:** ${lead.json.outreachAngle}
- **Topic to discuss:** ${lead.json.suggestedTopic}
- **Next step:** ${lead.json.nextSteps}

${lead.json.email ? `**Email:** ${lead.json.email}` : '**Email:** Not found - reach via LinkedIn'}
${lead.json.linkedinUrl ? `**LinkedIn:** ${lead.json.linkedinUrl}` : ''}

**Pain points to address:**
${lead.json.painPoints.split('; ').map(p => `- ${p}`).join('\n')}

---
`).join('\n')}

## Medium Priority Leads (Nurture)

${mediumPriority.slice(0, 10).map((lead, i) => `
${i + 1}. **${lead.json.fullName}** (${lead.json.company}) - ${lead.json.title}
   - Fit: ${lead.json.fitScore}/10
   - Angle: ${lead.json.outreachAngle}
   - Next: ${lead.json.nextSteps}
`).join('\n')}

${mediumPriority.length > 10 ? `\n*(+ ${mediumPriority.length - 10} more medium priority leads in database)*` : ''}

---

## Tips for LinkedIn Outreach

1. **Personalize each message** - Reference their role, company, or recent activity
2. **Lead with value** - How can you help them specifically?
3. **Be specific** - Mention a concrete automation or AI use case for their industry
4. **Low pressure** - Suggest a short call or resource, not a sales pitch
5. **Follow up** - If no response in 5-7 days, send a gentle follow-up

---

*This document generated automatically from LinkedIn lead analysis*
`;

return {
  outreachDoc,
  documentTitle: `LinkedIn Outreach Plan - ${new Date().toLocaleDateString()}`
};
```

### 16. Save Outreach Doc to Google Docs (Google Docs node)

```
Operation: Create Document
Document Name: {{ $json.documentTitle }}
Folder ID: {{ $env.OUTREACH_DOCS_FOLDER_ID }}
Content: {{ $json.outreachDoc }}
```

### 17. Send Summary Notification (Notification Dispatch subflow)

```json
{
  "severity": "info",
  "message": "LinkedIn leads processed and organized",
  "context": {
    "Total leads": "={{ $('Categorize by Priority').item.json.summary.total }}",
    "High priority": "={{ $('Categorize by Priority').item.json.summary.high }}",
    "Medium priority": "={{ $('Categorize by Priority').item.json.summary.medium }}",
    "Outreach doc": "={{ $('Save Outreach Doc to Google Docs').item.json.documentUrl }}",
    "Database": "https://docs.google.com/spreadsheets/d/{{ $env.LEADS_DATABASE_SHEET_ID }}"
  },
  "workflowName": "linkedin-lead-organizer",
  "channels": ["email", "slack"]
}
```

### 18. Log Completion (Audit Log subflow)

```json
{
  "workflowName": "linkedin-lead-organizer",
  "eventType": "success",
  "payloadSummary": "Processed {{ $('Categorize by Priority').item.json.summary.total }} LinkedIn leads",
  "details": {
    "highPriority": "={{ $('Categorize by Priority').item.json.summary.high }}",
    "duplicatesSkipped": "={{ $('Check for Duplicates').item.json.duplicatesSkipped }}"
  }
}
```

## Subflows Used

- **ai-call-wrapper:** For lead analysis (step 9)
- **notification-dispatch:** For completion notification (step 17)
- **audit-log:** For workflow logging (step 18)

Optional:
- **safe-http-call:** For enrichment APIs (steps 6 & 7)

## Customization Tips

### Adjust fit scoring criteria

Customize for your ideal customer profile:

```javascript
// In AI prompt
const icp = `
**Your Ideal Customer Profile:**
- Company size: 20-100 employees
- Industries: Marketing agencies, SaaS companies, professional services
- Location: Nordics preferred
- Role: Decision makers (founder, COO, VP Operations)
- Budget indicators: Recent funding, growth signals
`;

prompt += `\n\n${icp}\n\nScore fit based on alignment with this ICP.`;
```

### Add event/conference tracking

If leads are from specific events:

```javascript
// Add to lead data
lead.source = 'Conference: TechCrunch Disrupt 2024';
lead.connectionContext = 'Met at booth, discussed AI automation';

// Mention in outreach angle prompt
prompt += `\nConnection context: ${lead.connectionContext}`;
```

### Integration with CRM

Auto-create CRM contacts for high-priority leads:

```javascript
if (lead.priority === 'high' && lead.email) {
  // Create HubSpot contact
  await createContact({
    email: lead.email,
    firstname: lead.firstName,
    lastname: lead.lastName,
    company: lead.company,
    jobtitle: lead.title,
    lead_source: 'LinkedIn',
    fit_score: lead.fitScore
  });
}
```

### Schedule follow-up reminders

Create tasks or calendar events:

```javascript
// For high priority leads, create follow-up task
const followUpDate = new Date();
followUpDate.setDate(followUpDate.getDate() + 2);  // 2 days from now

await createTask({
  title: `Reach out to ${lead.fullName}`,
  description: `Outreach angle: ${lead.outreachAngle}`,
  dueDate: followUpDate
});
```

### Track outreach results

Add columns to track:
- Outreach sent (date)
- Response received (yes/no)
- Meeting scheduled (yes/no)
- Deal created (yes/no)

### Batch outreach drafts

Generate ready-to-send messages:

```javascript
const messageTemplate = `Hi ${lead.firstName},

${lead.outreachAngle}

${lead.suggestedTopic}

Would you be open to a brief 15-minute call next week to explore this?

Best,
[Your Name]`;

// Save to "Drafts" sheet for easy copy-paste
```

## Environment Variables Needed

```bash
# LinkedIn data source
LINKEDIN_LEADS_SHEET_ID=your-sheet-id-for-raw-imports

# Leads database
LEADS_DATABASE_SHEET_ID=your-main-database-sheet-id

# Enrichment APIs (optional)
CLEARBIT_API_KEY=your-clearbit-key
HUNTER_API_KEY=your-hunter-key

# AI
AI_MODEL=gpt-4-turbo
AI_TEMPERATURE=0.3

# Output
OUTREACH_DOCS_FOLDER_ID=google-drive-folder-id

# Notifications
SLACK_WEBHOOK_URL=your-slack-webhook
NOTIFICATION_EMAIL=your@email.com

# CRM (optional)
HUBSPOT_API_KEY=your-hubspot-key

# Audit
AUDIT_LOG_SHEET_ID=your-audit-sheet-id
```

## Performance Notes

- **Processing time:** 10-20 seconds per lead (with AI analysis)
- **Costs:**
  - AI analysis: ~$0.02-0.05 per lead (GPT-4)
  - Clearbit: ~$0.05 per lookup
  - Hunter.io: ~$0.01 per email found
- **Batch size:** Process 50-100 leads per run
- **Recommended:** Run on-demand when you have new connections to process

**Optimization:**
- Skip enrichment for low-fit leads
- Use GPT-3.5-turbo for initial categorization, GPT-4 for high-priority only
- Cache company data to avoid duplicate enrichment lookups

## Common Issues & Solutions

**Issue: Many leads have no email address**
- Solution: Use LinkedIn messaging instead, or manually request email during connection

**Issue: Clearbit/Hunter.io miss small companies**
- Solution: Expected for SMBs, rely more on AI analysis of title/company name

**Issue: Fit scores too high/low across the board**
- Solution: Adjust ICP description in AI prompt, provide examples of good/bad fits

**Issue: Outreach angles too generic**
- Solution: Add more context to prompt (your specific services, recent case studies, your value prop)

**Issue: Duplicate leads with slight name variations**
- Solution: Improve uniqueId generation, use fuzzy matching

**Issue: LinkedIn CSV export format changes**
- Solution: Check LinkedIn's current export format, update column name mappings

**ToS Compliance Reminder:**
- This workflow never accesses LinkedIn.com directly
- All data comes from manual exports or manual entry
- Respects LinkedIn's prohibition on automated data collection
- Uses third-party enrichment APIs for additional data (not LinkedIn's data)

This workflow helps you turn your LinkedIn network into organized, actionable business opportunities while staying fully compliant with platform policies.
