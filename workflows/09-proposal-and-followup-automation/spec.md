# Workflow: Proposal and Follow-up Automation

## Summary

Automates the proposal creation and follow-up process for client projects. Takes project requirements from a form or CRM, generates a customized proposal using AI and templates, sends it to the prospect, and manages automated follow-ups until response. Tracks proposal status and sends reminders.

**Time saved:** 2-3 hours per proposal → 15 minutes review/send
**Best for:** Consultants, agencies, freelancers, professional services

## Use Cases

1. **Consulting proposals**
   - Prospect fills out discovery form
   - Auto-generate scope, timeline, pricing
   - Send proposal PDF + follow-up sequence

2. **Agency project quotes**
   - Sales team enters project details in CRM
   - Create branded proposal document
   - Track opens and send reminders

3. **Freelance service quotes**
   - Client requests quote via website
   - Generate proposal from template
   - Auto-follow-up if no response in 3 days

## Inputs & Dependencies

**Required:**
- **Project data source:** One of:
  - Typeform/JotForm (project request form)
  - CRM deal/opportunity
  - Google Sheet (manual entry)
  - Webhook (from website)
- **Proposal template:** Google Docs or similar
- **AI API access:** OpenAI or Anthropic (for customization)
- **Email service:** Gmail, SendGrid, or similar
- **n8n credentials:**
  - Form/CRM API credentials
  - Google Docs OAuth2
  - OpenAI/Anthropic API key
  - Email credentials

**Project data requirements:**
- Client name and email
- Project description/requirements
- Desired timeline
- Budget range (optional)
- Any specific deliverables

## High-Level Flow

```
1. Trigger (form submission / CRM opportunity created / manual trigger)
   ↓
2. Extract project requirements
   ↓
3. Validate required information
   ↓
4. Call AI to analyze requirements:
   4a. Scope of work breakdown
   4b. Estimated timeline
   4c. Suggested pricing (based on your rates)
   4d. Potential challenges/risks
   4e. Value proposition points
   ↓
5. Generate proposal document:
   5a. Load template
   5b. Fill in client details
   5c. Insert AI-generated sections
   5d. Calculate pricing table
   5e. Add terms and next steps
   ↓
6. Convert to PDF (if needed)
   ↓
7. Send proposal email with PDF attached
   ↓
8. Schedule follow-up sequence:
   8a. Day 3: Gentle check-in if no response
   8b. Day 7: More direct follow-up
   8c. Day 14: Final follow-up or archive
   ↓
9. Track proposal status:
   9a. Opened (if tracking enabled)
   9b. Responded
   9c. Accepted/Rejected
   ↓
10. If accepted: Trigger onboarding workflow
    If rejected: Log and move to archive
    If no response: Continue follow-ups
   ↓
11. Log completion
```

## Node-by-Node Implementation

### 1. Trigger: Form Submission or CRM Update

**Option A: Typeform Trigger**

```
Event: New Response
Form ID: {{ $env.PROPOSAL_REQUEST_FORM_ID }}
```

**Option B: CRM Trigger (HubSpot)**

```
Event: Deal Created or Updated
Pipeline: Sales
Deal Stage: Proposal Requested
```

**Option C: Webhook (from website)**

```
HTTP Method: POST
Path: /webhook/proposal-request
Authentication: Header Auth
```

### 2. Extract and Normalize Project Data (Function node)

```javascript
const input = $input.item.json;

// Handle different source formats
let projectData = {};

if (input.form_response) {
  // Typeform format
  const answers = input.form_response.answers;
  projectData = {
    clientName: answers.find(a => a.field.ref === 'client_name')?.text,
    clientEmail: answers.find(a => a.field.ref === 'email')?.email,
    companyName: answers.find(a => a.field.ref === 'company')?.text,
    projectDescription: answers.find(a => a.field.ref === 'project_description')?.text,
    desiredTimeline: answers.find(a => a.field.ref === 'timeline')?.choice?.label,
    budgetRange: answers.find(a => a.field.ref === 'budget')?.choice?.label,
    specificRequirements: answers.find(a => a.field.ref === 'requirements')?.text,
    source: 'typeform'
  };
} else if (input.dealname) {
  // HubSpot deal format
  projectData = {
    clientName: input.contact_name || input.firstname + ' ' + input.lastname,
    clientEmail: input.contact_email || input.email,
    companyName: input.company || input.dealname,
    projectDescription: input.description || input.deal_description,
    desiredTimeline: input.timeline || 'Not specified',
    budgetRange: input.amount ? `$${input.amount}` : 'Not specified',
    specificRequirements: input.notes || '',
    dealId: input.dealId,
    source: 'hubspot'
  };
} else {
  // Generic webhook format
  projectData = {
    clientName: input.clientName,
    clientEmail: input.clientEmail,
    companyName: input.companyName,
    projectDescription: input.projectDescription,
    desiredTimeline: input.timeline,
    budgetRange: input.budget,
    specificRequirements: input.requirements || '',
    source: 'webhook'
  };
}

// Generate unique proposal ID
projectData.proposalId = `PROP-${Date.now()}`;
projectData.createdAt = new Date().toISOString();

return projectData;
```

### 3. Validate Required Fields (IF node)

```javascript
const data = $input.item.json;

const requiredFields = ['clientName', 'clientEmail', 'projectDescription'];
const missing = requiredFields.filter(field => !data[field] || data[field].trim() === '');

if (missing.length > 0) {
  return {
    valid: false,
    missingFields: missing
  };
}

return {
  valid: true,
  data
};
```

```
Condition: {{ $json.valid === true }}
True: Continue with proposal generation
False: Send notification about missing data
```

### 4. Build AI Analysis Prompt (Function node)

```javascript
const project = $input.item.json;

const yourRates = {
  hourlyRate: parseFloat(process.env.HOURLY_RATE) || 150,
  dayRate: parseFloat(process.env.DAY_RATE) || 1200,
  projectMinimum: parseFloat(process.env.PROJECT_MINIMUM) || 3000
};

const prompt = `Analyze this project request and create a detailed proposal breakdown.

**Client Information:**
- Name: ${project.clientName}
- Company: ${project.companyName}
- Email: ${project.clientEmail}

**Project Request:**
${project.projectDescription}

**Requirements:**
${project.specificRequirements || 'None specified'}

**Timeline:** ${project.desiredTimeline}
**Budget:** ${project.budgetRange}

**Your Service Context:**
- You provide automation consulting and AI implementation
- Hourly rate: $${yourRates.hourlyRate}
- Day rate: $${yourRates.dayRate}
- Project minimum: $${yourRates.projectMinimum}
- Typical deliverables: Workflow automation, AI integration, process optimization, training

**Generate comprehensive proposal details:**

Return JSON:
{
  "projectTitle": "Concise project name",
  "executiveSummary": "2-3 sentences explaining what you'll deliver and the value",
  "scopeOfWork": {
    "phases": [
      {
        "name": "Phase name",
        "description": "What happens in this phase",
        "deliverables": ["deliverable 1", "deliverable 2"],
        "estimatedHours": number,
        "duration": "X weeks"
      }
    ]
  },
  "timeline": {
    "totalDuration": "X weeks/months",
    "startDate": "Upon acceptance",
    "milestones": [
      {"phase": "phase name", "deadline": "Week X"}
    ]
  },
  "pricing": {
    "pricingModel": "fixed|hourly|monthly",
    "totalEstimate": number (in dollars),
    "breakdown": [
      {"item": "description", "hours": number, "rate": number, "total": number}
    ],
    "paymentTerms": "50% upfront, 50% on completion (or your standard terms)"
  },
  "assumptions": ["assumption 1", "assumption 2"],
  "outOfScope": ["what's not included 1", "what's not included 2"],
  "risks": ["potential challenge 1", "mitigation strategy"],
  "valueProposition": ["key benefit 1", "key benefit 2", "key benefit 3"],
  "nextSteps": ["step 1", "step 2", "step 3"]
}

Be specific and realistic. Pricing should account for complexity and your rates. Include buffer for unexpected challenges.`;

return {
  prompt,
  project,
  yourRates
};
```

### 5. Call AI for Proposal Generation (AI Call Wrapper subflow)

```json
{
  "prompt": "={{ $json.prompt }}",
  "mode": "json",
  "systemInstructions": "You are an expert consultant creating detailed, professional proposals. Be thorough but concise. Pricing should be realistic and account for complexity. Focus on value delivered, not just hours worked.",
  "temperature": 0.4,
  "model": "gpt-4-turbo"
}
```

### 6. Check AI Success (IF node)

```
Condition: {{ $json.success === true }}
True: Build proposal document
False: Use fallback template
```

### 7. Format Proposal Document (Function node)

```javascript
const proposal = $input.item.json.outputJson;
const project = $('Build AI Analysis Prompt').item.json.project;
const today = new Date().toLocaleDateString('en-US', { year: 'numeric', month: 'long', day: 'numeric' });

const proposalDoc = `# Project Proposal

**${proposal.projectTitle}**

**Prepared for:** ${project.clientName}, ${project.companyName}
**Prepared by:** ${process.env.YOUR_NAME || 'Your Company'}
**Date:** ${today}
**Proposal ID:** ${project.proposalId}
**Valid until:** ${new Date(Date.now() + 30*24*60*60*1000).toLocaleDateString()}

---

## Executive Summary

${proposal.executiveSummary}

---

## Scope of Work

${proposal.scopeOfWork.phases.map((phase, i) => `
### Phase ${i + 1}: ${phase.name}

${phase.description}

**Deliverables:**
${phase.deliverables.map(d => `- ${d}`).join('\n')}

**Estimated effort:** ${phase.estimatedHours} hours
**Duration:** ${phase.duration}
`).join('\n---\n')}

---

## Timeline

**Total project duration:** ${proposal.timeline.totalDuration}
**Anticipated start:** ${proposal.timeline.startDate}

**Milestones:**
${proposal.timeline.milestones.map((m, i) => `${i + 1}. ${m.phase} - ${m.deadline}`).join('\n')}

---

## Investment

**Pricing model:** ${proposal.pricing.pricingModel}
**Total estimate:** $${proposal.pricing.totalEstimate.toLocaleString()}

**Breakdown:**
${proposal.pricing.breakdown.map(item => `
- **${item.item}**
  - ${item.hours} hours × $${item.rate}/hour = $${item.total.toLocaleString()}
`).join('\n')}

**Payment terms:** ${proposal.pricing.paymentTerms}

---

## Assumptions

${proposal.assumptions.map((a, i) => `${i + 1}. ${a}`).join('\n')}

---

## Out of Scope

The following are not included in this proposal and would require separate discussion:

${proposal.outOfScope.map((item, i) => `${i + 1}. ${item}`).join('\n')}

---

## Potential Challenges & Mitigations

${proposal.risks.map((risk, i) => `${i + 1}. ${risk}`).join('\n')}

---

## Value & Benefits

${proposal.valueProposition.map((v, i) => `${i + 1}. ${v}`).join('\n')}

---

## Next Steps

${proposal.nextSteps.map((step, i) => `${i + 1}. ${step}`).join('\n')}

---

## Terms & Conditions

- This proposal is valid for 30 days from the date above
- Work will commence upon signed agreement and receipt of initial payment
- Changes to scope will be documented in writing and may affect timeline and pricing
- Standard terms and conditions apply (see attached)

---

## Let's Move Forward

I'm excited about the possibility of working together on this project. If you have any questions or would like to discuss any aspect of this proposal, please don't hesitate to reach out.

To accept this proposal, simply reply to this email with your confirmation, and I'll send over the formal agreement.

Best regards,

${process.env.YOUR_NAME}
${process.env.YOUR_EMAIL}
${process.env.YOUR_PHONE || ''}
`;

return {
  proposalDoc,
  proposalData: {
    ...project,
    ...proposal,
    totalPrice: proposal.pricing.totalEstimate,
    status: 'sent',
    sentAt: new Date().toISOString()
  }
};
```

### 8. Create Proposal Document (Google Docs node)

```
Operation: Create Document
Document Name: Proposal - {{ $json.proposalData.projectTitle }} - {{ $json.proposalData.companyName }}
Folder ID: {{ $env.PROPOSALS_FOLDER_ID }}
Content: {{ $json.proposalDoc }}
```

### 9. Convert to PDF (Google Drive node)

```
Operation: Export
File ID: {{ $json.documentId }}
Format: PDF
Download: Yes
```

### 10. Send Proposal Email (Email node)

```
To: {{ $('Extract and Normalize Project Data').item.json.clientEmail }}
Cc: {{ $env.YOUR_EMAIL }}
Subject: Proposal: {{ $('Format Proposal Document').item.json.proposalData.projectTitle }}

Email Type: HTML

Body:
Hi {{ $('Extract and Normalize Project Data').item.json.clientName }},

Thank you for considering me for your project. I've put together a detailed proposal based on our discussion and your requirements.

**Project:** {{ $('Format Proposal Document').item.json.proposalData.projectTitle }}
**Investment:** ${{ $('Format Proposal Document').item.json.proposalData.totalPrice.toLocaleString() }}
**Timeline:** {{ $('Format Proposal Document').item.json.proposalData.timeline.totalDuration }}

Please find the full proposal attached. I've outlined:
- Detailed scope of work with deliverables
- Project timeline and milestones
- Transparent pricing breakdown
- Expected outcomes and value

I'm excited about the opportunity to help {{ $('Extract and Normalize Project Data').item.json.companyName }} with this project.

If you have any questions or would like to discuss anything in the proposal, I'm happy to schedule a call at your convenience.

To move forward, simply reply to this email with your confirmation.

Looking forward to hearing from you!

Best regards,
{{ $env.YOUR_NAME }}

Attachments:
- {{ $json.pdfFileName }}
```

### 11. Save Proposal to Database (Google Sheets node)

```
Operation: Append Row
Sheet ID: {{ $env.PROPOSALS_DATABASE_SHEET_ID }}
Sheet: Proposals

Columns:
  Proposal ID: {{ $json.proposalData.proposalId }}
  Client Name: {{ $json.proposalData.clientName }}
  Client Email: {{ $json.proposalData.clientEmail }}
  Company: {{ $json.proposalData.companyName }}
  Project Title: {{ $json.proposalData.projectTitle }}
  Total Price: {{ $json.proposalData.totalPrice }}
  Timeline: {{ $json.proposalData.timeline.totalDuration }}
  Status: sent
  Sent At: {{ $json.proposalData.sentAt }}
  Follow-up 1 Due: {{ new Date(Date.now() + 3*24*60*60*1000).toISOString() }}
  Follow-up 2 Due: {{ new Date(Date.now() + 7*24*60*60*1000).toISOString() }}
  Follow-up 3 Due: {{ new Date(Date.now() + 14*24*60*60*1000).toISOString() }}
  Document URL: {{ $('Create Proposal Document').item.json.documentUrl }}
```

### 12. Schedule Follow-up Workflow (Wait node + Schedule)

**Create a separate workflow for follow-ups that runs daily:**

```
Workflow: Proposal Follow-ups (separate)

Trigger: Schedule - daily at 9am
   ↓
Fetch proposals where:
  - Status = "sent" or "follow-up-1-sent"
  - Follow-up due date = today
  - Not responded
   ↓
For each proposal:
  - Check follow-up stage
  - Send appropriate follow-up email
  - Update database
```

**Follow-up Email Templates:**

**Follow-up 1 (Day 3):**
```
Subject: Following up: {{ $json.projectTitle }}

Hi {{ $json.clientName }},

I wanted to follow up on the proposal I sent a few days ago for {{ $json.projectTitle }}.

Have you had a chance to review it? I'm happy to answer any questions or discuss any aspects in more detail.

Best,
{{ $env.YOUR_NAME }}
```

**Follow-up 2 (Day 7):**
```
Subject: Checking in on {{ $json.projectTitle }}

Hi {{ $json.clientName }},

I know you're busy, so I wanted to check in on the proposal I sent last week.

If timing isn't right or if there are any concerns, I'd appreciate hearing that too. Either way, I'd love to know where things stand.

Would a quick 10-minute call this week be helpful to discuss?

Best,
{{ $env.YOUR_NAME }}
```

**Follow-up 3 (Day 14):**
```
Subject: Final follow-up: {{ $json.projectTitle }}

Hi {{ $json.clientName }},

I haven't heard back, so I'm assuming the timing might not be right for this project at the moment.

I'll close out this proposal, but please don't hesitate to reach out if circumstances change or if you'd like to discuss this (or anything else) in the future.

Wishing you all the best!

{{ $env.YOUR_NAME }}
```

### 13. Track Email Opens (Optional - using tracking service)

If using email tracking service like Mailtrack or similar:

```javascript
// Add tracking pixel to email
const trackingPixel = `<img src="${process.env.EMAIL_TRACKING_URL}/track/${project.proposalId}.png" width="1" height="1" />`;

emailBody += trackingPixel;
```

### 14. Handle Response - Webhook Endpoint

Create webhook for client responses:

```
Path: /webhook/proposal-response/{{ $json.proposalId }}

Expected payload:
{
  "proposalId": "PROP-xxx",
  "response": "accepted|rejected|questions",
  "message": "optional message from client"
}
```

Update database:

```
Operation: Update Row
Sheet ID: {{ $env.PROPOSALS_DATABASE_SHEET_ID }}
Find row where: Proposal ID = {{ $json.proposalId }}

Update:
  Status: {{ $json.response }}
  Responded At: {{ new Date().toISOString() }}
  Response Message: {{ $json.message }}
```

### 15. If Accepted: Trigger Onboarding (IF node + Webhook)

```
Condition: {{ $json.response === 'accepted' }}
True: Trigger client onboarding workflow
```

```
Method: POST
URL: {{ $env.N8N_WEBHOOK_URL }}/webhook/client-onboarding
Body:
{
  "proposalId": "{{ $json.proposalId }}",
  "clientName": "{{ $json.clientName }}",
  "clientEmail": "{{ $json.clientEmail }}",
  "projectTitle": "{{ $json.projectTitle }}",
  "totalPrice": {{ $json.totalPrice }}
}
```

### 16. Send Internal Notification (Notification Dispatch subflow)

```json
{
  "severity": "info",
  "message": "New proposal sent",
  "context": {
    "Client": "={{ $('Extract and Normalize Project Data').item.json.clientName }}",
    "Company": "={{ $('Extract and Normalize Project Data').item.json.companyName }}",
    "Project": "={{ $('Format Proposal Document').item.json.proposalData.projectTitle }}",
    "Value": "$={{ $('Format Proposal Document').item.json.proposalData.totalPrice.toLocaleString() }}",
    "Proposal": "={{ $('Create Proposal Document').item.json.documentUrl }}"
  },
  "workflowName": "proposal-and-followup-automation",
  "channels": ["slack"]
}
```

### 17. Log Completion (Audit Log subflow)

```json
{
  "workflowName": "proposal-and-followup-automation",
  "eventType": "success",
  "payloadSummary": "Proposal sent: {{ $('Format Proposal Document').item.json.proposalData.projectTitle }}",
  "details": {
    "proposalId": "={{ $('Format Proposal Document').item.json.proposalData.proposalId }}",
    "clientEmail": "={{ $('Extract and Normalize Project Data').item.json.clientEmail }}",
    "value": "={{ $('Format Proposal Document').item.json.proposalData.totalPrice }}"
  }
}
```

## Subflows Used

- **ai-call-wrapper:** For proposal generation (step 5)
- **notification-dispatch:** For internal notification (step 16)
- **audit-log:** For workflow logging (step 17)

## Customization Tips

### Add pricing tiers/packages

Pre-define service packages:

```javascript
const packages = {
  basic: {
    name: 'Starter Automation Package',
    price: 5000,
    includes: ['3 automated workflows', '2 integrations', '1 training session']
  },
  standard: {
    name: 'Growth Automation Package',
    price: 12000,
    includes: ['10 workflows', '5 integrations', '3 training sessions', 'Monthly support']
  },
  premium: {
    name: 'Enterprise Automation Package',
    price: 25000,
    includes: ['Unlimited workflows', 'Full integration', 'Weekly support', 'Custom AI models']
  }
};

// Match client requirements to package
const suggestedPackage = matchToPackage(projectDescription);
```

### Dynamic pricing based on complexity

```javascript
// Calculate complexity score
let complexityMultiplier = 1.0;

if (projectDescription.includes('AI') || projectDescription.includes('machine learning')) {
  complexityMultiplier += 0.3;
}

if (projectDescription.includes('integration') && projectDescription.includes('legacy')) {
  complexityMultiplier += 0.2;
}

const adjustedPrice = basePrice * complexityMultiplier;
```

### Include case studies/testimonials

```javascript
// Add relevant case studies to proposal
const relevantCaseStudies = findCaseStudies(project.companyName, project.projectDescription);

proposalDoc += `\n\n## Relevant Experience\n\n${relevantCaseStudies.map(cs => `
### ${cs.title}
${cs.summary}
**Result:** ${cs.result}
`).join('\n')}`;
```

### E-signature integration

Add DocuSign or HelloSign:

```javascript
// Instead of email confirmation, send for e-signature
const docusignEnvelope = await createEnvelope({
  document: proposalPDF,
  signers: [
    { email: clientEmail, name: clientName },
    { email: yourEmail, name: yourName }
  ]
});
```

### Proposal expiration reminders

```javascript
// 3 days before expiration
if (daysUntilExpiration === 3 && status === 'sent') {
  sendEmail({
    subject: 'Proposal expires soon',
    body: `This proposal expires in 3 days. Let me know if you need more time.`
  });
}
```

### A/B test proposal formats

Track which formats convert better:

```javascript
const variant = Math.random() < 0.5 ? 'detailed' : 'concise';

if (variant === 'concise') {
  // Shorter proposal, more bullet points
} else {
  // Detailed proposal, more explanations
}

// Track in database for analysis
proposal.variant = variant;
```

## Environment Variables Needed

```bash
# Your business info
YOUR_NAME=Your Full Name
YOUR_EMAIL=your@email.com
YOUR_PHONE=your-phone

# Pricing
HOURLY_RATE=150
DAY_RATE=1200
PROJECT_MINIMUM=3000

# Form/CRM
PROPOSAL_REQUEST_FORM_ID=typeform-form-id
HUBSPOT_API_KEY=your-hubspot-key

# AI
AI_MODEL=gpt-4-turbo
AI_TEMPERATURE=0.4

# Documents
PROPOSALS_FOLDER_ID=google-drive-folder-id

# Database
PROPOSALS_DATABASE_SHEET_ID=your-sheet-id

# Email
EMAIL_FROM=your@email.com
EMAIL_TRACKING_URL=your-tracking-service (optional)

# Workflows
N8N_WEBHOOK_URL=https://your-n8n-instance.com

# Notifications
SLACK_WEBHOOK_URL=your-slack-webhook

# Audit
AUDIT_LOG_SHEET_ID=your-audit-sheet-id
```

## Performance Notes

- **Processing time:** 30-90 seconds per proposal (AI generation)
- **AI cost:** ~$0.15-0.30 per proposal (GPT-4)
- **Follow-up workflow:** Runs daily, processes proposals due for follow-up
- **Conversion tracking:** Monitor proposal → client conversion rate

**Metrics to track:**
- Proposals sent
- Response rate
- Acceptance rate
- Average deal value
- Time to response
- Follow-up effectiveness

## Common Issues & Solutions

**Issue: AI-generated pricing too high/low**
- Solution: Adjust hourly rates, add complexity multipliers, review and edit before sending

**Issue: Proposals too generic**
- Solution: Add more client-specific context to AI prompt, include custom research

**Issue: Follow-ups feel spam my**
- Solution: Personalize each follow-up, reduce frequency, add value in each email

**Issue: Clients don't respond to any follow-ups**
- Solution: Try phone call after follow-up 2, or ask if timing is off

**Issue: Scope creep during project**
- Solution: Reference original proposal, clear change order process

**Issue: PDF formatting issues**
- Solution: Use consistent Google Docs template, test PDF conversion

This workflow transforms proposal creation from a time-consuming chore into a streamlined, consistent process that frees you to focus on client relationships and delivery.
