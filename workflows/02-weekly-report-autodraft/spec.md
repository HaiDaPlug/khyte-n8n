# Workflow: Weekly Report Autodraft

## Summary

Automatically compiles a weekly business report by aggregating data from multiple sources: CRM activity, project management tools, financial metrics, and team updates. Generates a draft report with AI-powered summaries and insights, ready for quick review and sending.

**Time saved:** 90-120 minutes weekly → 10 minutes review/edit
**Best for:** Consultants, agency owners, department heads sending regular updates

## Use Cases

1. **Client update reports**
   - Pull project milestones from Asana/Trello
   - Summarize deliverables and progress
   - Include next week's priorities

2. **Internal team digest**
   - CRM wins and pipeline changes
   - Support ticket trends
   - Team capacity and utilization

3. **Investor/stakeholder updates**
   - Revenue metrics from Stripe/QuickBooks
   - Growth KPIs and trends
   - Key wins and challenges

## Inputs & Dependencies

**Required:**
- **Data sources:** At least 2 of:
  - CRM (HubSpot, Pipedrive, etc.)
  - Project management (Asana, ClickUp, Trello)
  - Financial (Stripe, QuickBooks)
  - Google Sheets (manual tracking)
- **AI API access:** OpenAI or Anthropic (for summary generation)
- **Output destination:** Email, Google Docs, or Slack
- **n8n credentials:**
  - OAuth2 for each data source
  - OpenAI/Anthropic API key
  - Email/Slack credentials

**Data requirements:**
- Access to last 7 days of activity
- Configured date range for "this week"
- Template for report structure

## High-Level Flow

```
1. Trigger (scheduled weekly: Friday 4pm or Monday 9am)
   ↓
2. Calculate date range (this week)
   ↓
3. Parallel data collection:
   3a. Fetch CRM activity (deals, contacts, pipeline changes)
   3b. Fetch project milestones (completed tasks, deadlines)
   3c. Fetch financial metrics (revenue, expenses, invoices)
   3d. Fetch team updates (tickets, messages, availability)
   ↓
4. Aggregate all data
   ↓
5. For each section:
   5a. Format raw data
   5b. Call AI to summarize with insights
   5c. Structure into report section
   ↓
6. Combine sections into full report
   ↓
7. Write to Google Doc or format as email
   ↓
8. Send notification with link/preview
   ↓
9. Log completion
```

## Node-by-Node Implementation

### 1. Trigger: Schedule Weekly (Schedule Trigger node)

```
Trigger: Cron
Schedule: Every Friday at 4:00 PM
Expression: 0 16 * * 5

Alternative: Every Monday at 9:00 AM
Expression: 0 9 * * 1
```

### 2. Calculate Date Range (Function node)

```javascript
const now = new Date();
const dayOfWeek = now.getDay();

// Calculate Monday of this week
const monday = new Date(now);
monday.setDate(now.getDate() - (dayOfWeek === 0 ? 6 : dayOfWeek - 1));
monday.setHours(0, 0, 0, 0);

// Calculate Sunday (end of week)
const sunday = new Date(monday);
sunday.setDate(monday.getDate() + 6);
sunday.setHours(23, 59, 59, 999);

return {
  weekStart: monday.toISOString(),
  weekEnd: sunday.toISOString(),
  weekLabel: `Week of ${monday.toLocaleDateString('en-US', { month: 'short', day: 'numeric' })}`,
  year: monday.getFullYear(),
  weekNumber: getWeekNumber(monday)
};

function getWeekNumber(date) {
  const d = new Date(Date.UTC(date.getFullYear(), date.getMonth(), date.getDate()));
  const dayNum = d.getUTCDay() || 7;
  d.setUTCDate(d.getUTCDate() + 4 - dayNum);
  const yearStart = new Date(Date.UTC(d.getUTCFullYear(), 0, 1));
  return Math.ceil((((d - yearStart) / 86400000) + 1) / 7);
}
```

### 3. Log Workflow Start (Audit Log subflow)

```json
{
  "workflowName": "weekly-report-autodraft",
  "eventType": "start",
  "payloadSummary": "Generating report for {{ $json.weekLabel }}"
}
```

### 4a. Fetch CRM Activity (CRM node - HubSpot/Pipedrive)

**Example for HubSpot:**

```
Operation: Get All
Resource: Deals
Filters:
  - closedate >= {{ $('Calculate Date Range').item.json.weekStart }}
  - closedate <= {{ $('Calculate Date Range').item.json.weekEnd }}
Additional Fields: dealname, amount, dealstage, closedate
```

**Also get pipeline changes:**

```
Operation: Get All
Resource: Deals
Filters:
  - hs_lastmodifieddate >= {{ $('Calculate Date Range').item.json.weekStart }}
Return Fields: dealname, dealstage, hs_lastmodifieddate
```

### 4b. Fetch Project Milestones (Asana/ClickUp/Trello node)

**Example for ClickUp:**

```
Operation: Get All
Resource: Tasks
List ID: {{ $env.CLICKUP_LIST_ID }}
Filters:
  - status: completed
  - date_done >= {{ $('Calculate Date Range').item.json.weekStart }}
  - date_done <= {{ $('Calculate Date Range').item.json.weekEnd }}
```

### 4c. Fetch Financial Metrics (Stripe/QuickBooks node)

**Example for Stripe:**

```
Operation: Get All
Resource: Invoices
Filters:
  - created >= {{ Math.floor(new Date($('Calculate Date Range').item.json.weekStart).getTime() / 1000) }}
  - created <= {{ Math.floor(new Date($('Calculate Date Range').item.json.weekEnd).getTime() / 1000) }}
  - status: paid
```

**Also get charges/payments:**

```
Operation: Get All
Resource: Charges
Filters: (same date range)
Return: amount, currency, status, created
```

### 4d. Fetch Team Updates (Google Sheets or other source)

```
Operation: Get All
Sheet ID: {{ $env.TEAM_UPDATES_SHEET_ID }}
Range: Updates!A:E
Filter: Where "Week" column matches {{ $('Calculate Date Range').item.json.weekNumber }}
```

### 5. Process CRM Data (Function node)

```javascript
const deals = $input.all();
const weekStart = $('Calculate Date Range').item.json.weekStart;

const closedDeals = deals.filter(d => d.json.closedate && d.json.dealstage === 'closedwon');
const lostDeals = deals.filter(d => d.json.closedate && d.json.dealstage === 'closedlost');
const pipelineChanges = deals.filter(d => d.json.hs_lastmodifieddate >= weekStart);

const totalRevenue = closedDeals.reduce((sum, d) => sum + (d.json.amount || 0), 0);

return {
  section: 'sales',
  closedDeals: closedDeals.length,
  closedDealsList: closedDeals.map(d => ({
    name: d.json.dealname,
    amount: d.json.amount,
    closeDate: d.json.closedate
  })),
  lostDeals: lostDeals.length,
  totalRevenue,
  pipelineActivity: pipelineChanges.length,
  rawData: deals
};
```

### 6. Process Project Data (Function node)

```javascript
const tasks = $input.all();

const completedTasks = tasks.filter(t => t.json.status === 'completed');
const byProject = {};

completedTasks.forEach(task => {
  const project = task.json.list?.name || 'Other';
  if (!byProject[project]) {
    byProject[project] = [];
  }
  byProject[project].push({
    name: task.json.name,
    completedDate: task.json.date_done
  });
});

return {
  section: 'projects',
  totalCompleted: completedTasks.length,
  byProject,
  rawData: tasks
};
```

### 7. Process Financial Data (Function node)

```javascript
const invoices = $input.all();

const paidInvoices = invoices.filter(i => i.json.status === 'paid');
const totalRevenue = paidInvoices.reduce((sum, i) => sum + (i.json.amount_paid || 0), 0);

// Convert from cents to dollars
const revenueDollars = totalRevenue / 100;

return {
  section: 'financial',
  paidInvoices: paidInvoices.length,
  totalRevenue: revenueDollars,
  currency: paidInvoices[0]?.json.currency || 'usd',
  invoiceList: paidInvoices.map(i => ({
    number: i.json.number,
    amount: i.json.amount_paid / 100,
    customer: i.json.customer_name
  })),
  rawData: invoices
};
```

### 8. Aggregate All Sections (Merge node)

```
Mode: Merge By Position
Input 1: CRM data
Input 2: Project data
Input 3: Financial data
Input 4: Team updates
```

### 9. Build Report Prompt for AI (Function node)

```javascript
const sections = $input.all();
const dateRange = $('Calculate Date Range').item.json;

const salesData = sections.find(s => s.json.section === 'sales');
const projectData = sections.find(s => s.json.section === 'projects');
const financialData = sections.find(s => s.json.section === 'financial');

const prompt = `Generate a professional weekly business report for ${dateRange.weekLabel}.

**Sales & CRM Activity:**
- Closed deals: ${salesData.json.closedDeals}
- Total revenue: $${salesData.json.totalRevenue}
- Lost deals: ${salesData.json.lostDeals}
- Pipeline activity: ${salesData.json.pipelineActivity} deals updated
- Deal details: ${JSON.stringify(salesData.json.closedDealsList, null, 2)}

**Project Delivery:**
- Completed tasks: ${projectData.json.totalCompleted}
- By project: ${JSON.stringify(projectData.json.byProject, null, 2)}

**Financial Summary:**
- Invoices paid: ${financialData.json.paidInvoices}
- Revenue: $${financialData.json.totalRevenue}

Please write:
1. Executive summary (2-3 sentences)
2. Key highlights (3-5 bullet points)
3. Sales section (paragraph with insights)
4. Project section (paragraph with insights)
5. Financial section (paragraph)
6. Next week priorities (3 items - infer from data)

Tone: Professional but conversational. Focus on progress and momentum.`;

return {
  prompt,
  dateRange,
  sections
};
```

### 10. Call AI for Report Generation (AI Call Wrapper subflow)

```json
{
  "prompt": "={{ $json.prompt }}",
  "mode": "text",
  "systemInstructions": "You are a business analyst creating weekly reports for a small business owner. Be concise, insightful, and focus on what matters. Highlight wins and flag concerns.",
  "temperature": 0.4,
  "model": "gpt-4-turbo"
}
```

### 11. Check AI Success (IF node)

```
Condition: {{ $json.success === true }}
True: Format report
False: Handle error (send raw data instead)
```

### 12. Format Report Document (Function node)

```javascript
const aiOutput = $input.item.json.outputText;
const dateRange = $('Build Report Prompt for AI').item.json.dateRange;
const sections = $('Build Report Prompt for AI').item.json.sections;

const salesData = sections.find(s => s.section === 'sales');
const projectData = sections.find(s => s.section === 'projects');
const financialData = sections.find(s => s.section === 'financial');

// Build formatted report
const report = `# Weekly Business Report
${dateRange.weekLabel}

---

${aiOutput}

---

## Detailed Metrics

**Sales:**
- Deals closed: ${salesData.closedDeals}
- Revenue: $${salesData.totalRevenue.toLocaleString()}
- Pipeline moves: ${salesData.pipelineActivity}

**Projects:**
- Tasks completed: ${projectData.totalCompleted}
- Active projects: ${Object.keys(projectData.byProject).length}

**Financial:**
- Invoices paid: ${financialData.paidInvoices}
- Total collected: $${financialData.totalRevenue.toLocaleString()}

---

*Generated automatically on ${new Date().toLocaleString()}*
`;

return {
  report,
  reportTitle: `Weekly Report - ${dateRange.weekLabel}`,
  dateRange
};
```

### 13. Write to Google Doc (Google Docs node)

```
Operation: Create Document
Document Name: {{ $json.reportTitle }}
Folder ID: {{ $env.REPORTS_FOLDER_ID }}
Content: {{ $json.report }}
```

**Alternative: Format as Email HTML:**

```javascript
const report = $input.item.json.report;
const htmlReport = report
  .replace(/^# (.+)$/gm, '<h1>$1</h1>')
  .replace(/^## (.+)$/gm, '<h2>$1</h2>')
  .replace(/^\*\*(.+)\*\*$/gm, '<strong>$1</strong>')
  .replace(/^- (.+)$/gm, '<li>$1</li>')
  .replace(/\n\n/g, '<p></p>');

return {
  htmlReport,
  subject: $input.item.json.reportTitle
};
```

### 14. Send Notification with Link (Notification Dispatch subflow)

```json
{
  "severity": "info",
  "message": "Weekly report ready for review",
  "context": {
    "Week": "={{ $('Calculate Date Range').item.json.weekLabel }}",
    "Document": "={{ $json.documentUrl }}",
    "Preview": "Report generated successfully"
  },
  "workflowName": "weekly-report-autodraft",
  "channels": ["email", "slack"]
}
```

**For email with full report:**

```
To: {{ $env.REPORT_EMAIL_TO }}
Subject: {{ $json.subject }}
Email Type: HTML
Body: {{ $json.htmlReport }}
```

### 15. Handle AI Error - Send Raw Data (Function node)

```javascript
const sections = $('Build Report Prompt for AI').item.json.sections;
const dateRange = $('Build Report Prompt for AI').item.json.dateRange;

const fallbackReport = `Weekly Report - ${dateRange.weekLabel}

AI summary generation failed. Here's the raw data:

${JSON.stringify(sections, null, 2)}

Please compile manually.`;

return {
  report: fallbackReport,
  reportTitle: `[MANUAL] Weekly Report - ${dateRange.weekLabel}`,
  isFallback: true
};
```

### 16. Log Completion (Audit Log subflow)

```json
{
  "workflowName": "weekly-report-autodraft",
  "eventType": "success",
  "payloadSummary": "Report generated for {{ $('Calculate Date Range').item.json.weekLabel }}",
  "details": {
    "documentUrl": "={{ $('Write to Google Doc').item.json.documentUrl }}",
    "sections": "={{ $('Build Report Prompt for AI').item.json.sections.length }}"
  }
}
```

## Subflows Used

- **ai-call-wrapper:** For report summary generation (step 10)
- **notification-dispatch:** For completion notification (step 14)
- **audit-log:** For start/completion logging (steps 3 & 16)

Optional:
- **safe-http-call:** If fetching from custom APIs

## Customization Tips

### Add more data sources

**Email metrics (Gmail/Outlook):**
- Count emails sent/received this week
- Track response times
- Identify top contacts

**Analytics (Google Analytics, Plausible):**
- Website traffic trends
- Top pages and conversion rates
- User behavior insights

**Social media (Buffer, Hootsuite):**
- Posts published
- Engagement metrics
- Follower growth

### Customize report structure

Create a template document and populate it instead of generating from scratch:

```javascript
// Load template
const template = `
# Weekly Report - {{WEEK}}

## Executive Summary
{{EXECUTIVE_SUMMARY}}

## Sales Performance
{{SALES_SECTION}}
...
`;

// Replace placeholders
const finalReport = template
  .replace('{{WEEK}}', dateRange.weekLabel)
  .replace('{{EXECUTIVE_SUMMARY}}', aiOutput.summary)
  .replace('{{SALES_SECTION}}', aiOutput.sales);
```

### Add comparison to previous week

Before the AI call, fetch last week's data and include in prompt:

```javascript
const prompt = `...

**Last Week Comparison:**
- Revenue: $${lastWeekRevenue} → $${thisWeekRevenue} (${percentChange}%)
- Deals: ${lastWeekDeals} → ${thisWeekDeals}

Highlight trends and changes in your summary.`;
```

### Send to different recipients

Add logic to customize report based on audience:

```javascript
// For clients: Hide internal metrics
if (recipientType === 'client') {
  delete financialData.rawData;
  delete salesData.pipelineActivity;
}

// For team: Add granular details
if (recipientType === 'team') {
  includeIndividualPerformance = true;
}
```

### Generate charts/visualizations

Use QuickChart or similar to create charts from data:

```javascript
const chartUrl = `https://quickchart.io/chart?c={
  type: 'bar',
  data: {
    labels: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri'],
    datasets: [{
      label: 'Revenue by Day',
      data: ${JSON.stringify(dailyRevenue)}
    }]
  }
}`;

// Include in report
report += `\n![Revenue Chart](${chartUrl})\n`;
```

### Schedule multiple reports

Create variants for different stakeholders:

- **Monday AM:** Internal team digest (detailed)
- **Friday PM:** Client update (high-level)
- **Month-end:** Extended report with monthly rollup

Duplicate workflow and adjust:
- Data sources
- AI prompt tone
- Recipients
- Schedule

## Environment Variables Needed

```bash
# Data sources
HUBSPOT_API_KEY=your-hubspot-key
CLICKUP_API_KEY=your-clickup-key
STRIPE_API_KEY=your-stripe-key
TEAM_UPDATES_SHEET_ID=your-sheet-id

# AI
AI_MODEL=gpt-4-turbo
AI_TEMPERATURE=0.4

# Output
REPORTS_FOLDER_ID=google-drive-folder-id
REPORT_EMAIL_TO=recipient@company.com

# Notifications
SLACK_WEBHOOK_URL=your-slack-webhook
NOTIFICATION_EMAIL=your@email.com

# Audit
AUDIT_LOG_SHEET_ID=your-audit-sheet-id
```

## Performance Notes

- **Processing time:** 2-5 minutes (depends on data volume)
- **AI cost:** ~$0.10-0.30 per report (GPT-4)
- **API calls:** 3-6 external APIs per run
- **Recommended:** Run during off-hours to avoid rate limits

## Common Issues & Solutions

**Issue: Missing data from one source**
- Solution: Add fallback values and note in report: "CRM data unavailable, showing cached values"

**Issue: Report too generic/boring**
- Solution: Add more context to AI prompt (your business goals, what you care about)

**Issue: Date range confusion (week boundaries)**
- Solution: Make week start day configurable in environment variables

**Issue: Too much data, report is overwhelming**
- Solution: Add threshold filters (only show deals > $1000, tasks from priority projects)

**Issue: AI summary misses important details**
- Solution: Use structured prompting with required sections, or add post-processing to inject key metrics

This workflow saves hours every week while ensuring stakeholders stay informed with minimal manual effort.
