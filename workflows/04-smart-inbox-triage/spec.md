# Workflow: Smart Inbox Triage

## Summary

Automatically monitors your inbox, categorizes incoming emails using AI, applies labels/filters, and takes action based on content type: urgent messages trigger notifications, low-priority emails get auto-archived, actionable emails create tasks, and questions get drafted AI responses for review.

**Time saved:** 30-60 minutes daily → 5 minutes reviewing actions
**Best for:** Consultants, founders, anyone drowning in email

## Use Cases

1. **Inbox zero maintenance**
   - Auto-label newsletters, receipts, notifications
   - Archive non-actionable emails
   - Surface only what needs attention

2. **Client email management**
   - Flag urgent client requests immediately
   - Create tasks for client deliverables mentioned
   - Draft response templates for common questions

3. **Sales/business development**
   - Identify warm leads in cold outreach responses
   - Auto-respond to meeting requests with calendar link
   - Track follow-ups that need replies

## Inputs & Dependencies

**Required:**
- **Email provider:** Gmail (via IMAP/API) or Outlook
- **AI API access:** OpenAI or Anthropic (for classification and drafts)
- **Task tool (optional):** Asana, ClickUp, Todoist
- **n8n credentials:**
  - Gmail OAuth2 or IMAP credentials
  - OpenAI/Anthropic API key
  - Optional: Task management tool credentials

**Email requirements:**
- Access to inbox via IMAP or API
- Ability to apply labels/move to folders
- Ability to mark as read/starred

## High-Level Flow

```
1. Trigger (new email arrives or scheduled check every 5-15 min)
   ↓
2. Fetch unprocessed emails
   ↓
3. For each email:
   3a. Extract metadata (from, subject, body, thread)
   3b. Call AI to classify:
       - Category (client/sales/admin/newsletter/spam)
       - Priority (urgent/normal/low)
       - Intent (question/request/info/notification)
       - Sentiment (positive/neutral/negative/angry)
   3c. Route based on classification:
       ├─ Urgent: Send immediate notification
       ├─ Newsletter/Promo: Auto-archive or label
       ├─ Question: Draft AI response for review
       ├─ Task request: Create task in PM tool
       └─ Follow-up needed: Add to follow-up list
   3d. Apply labels and update email status
   ↓
4. Aggregate actions taken
   ↓
5. Send daily digest of triage actions
   ↓
6. Log completion
```

## Node-by-Node Implementation

### 1. Trigger: New Email or Schedule (Email Trigger / Schedule node)

**Option A: IMAP Trigger (real-time-ish)**

```
Protocol: IMAP
Host: imap.gmail.com
Port: 993
User: {{ $env.EMAIL_ADDRESS }}
Password: {{ $env.EMAIL_PASSWORD }}

Mailbox: INBOX
Options:
  - Mark as read: No (we'll do this selectively)
  - Post-process action: Mark as seen (custom label)
Check interval: Every 5 minutes
```

**Option B: Schedule Trigger (batch processing)**

```
Trigger: Cron - every 15 minutes
Expression: */15 * * * *
```

### 2. Fetch Unread Emails (Gmail node or IMAP node)

**For Gmail API:**

```
Operation: Get All
Resource: Messages
Filters:
  - is:unread
  - in:inbox
  - NOT label:processed
  - NOT from:me
Max Results: 20
Format: Full (include body)
```

**For IMAP:**

```
Operation: Search
Criteria: UNSEEN
Mailbox: INBOX
Fetch: Full message with body
```

### 3. Check if Emails Exist (IF node)

```
Condition: {{ $json.length > 0 }}
True: Process emails
False: End (no new emails)
```

### 4. Parse Email Content (Function node)

```javascript
const email = $input.item.json;

// Extract body (handle both plain text and HTML)
let body = '';
if (email.payload?.body?.data) {
  // Gmail API format
  body = Buffer.from(email.payload.body.data, 'base64').toString();
} else if (email.payload?.parts) {
  // Find text/plain part
  const textPart = email.payload.parts.find(p => p.mimeType === 'text/plain');
  if (textPart?.body?.data) {
    body = Buffer.from(textPart.body.data, 'base64').toString();
  }
} else if (email.textPlain) {
  // IMAP format
  body = email.textPlain;
}

// Clean body (remove signatures, quoted text)
const cleanBody = body
  .split(/On .+ wrote:/)[0]  // Remove quoted replies
  .split(/_{3,}/)[0]  // Remove signature separators
  .split(/--\s*$/m)[0]  // Remove -- signature marker
  .trim();

// Extract headers
const from = email.from || email.headers?.from || '';
const subject = email.subject || email.headers?.subject || '';

// Check if part of thread
const threadId = email.threadId;
const isReply = subject.toLowerCase().startsWith('re:');

return {
  id: email.id,
  threadId,
  from,
  fromEmail: from.match(/<(.+?)>/)?.[1] || from,
  subject,
  body: cleanBody,
  fullBody: body,
  receivedAt: email.internalDate || email.date || new Date().toISOString(),
  isReply,
  labels: email.labelIds || []
};
```

### 5. Build AI Classification Prompt (Function node)

```javascript
const email = $input.item.json;

const prompt = `Analyze this email and classify it for automated triage.

**From:** ${email.from}
**Subject:** ${email.subject}
**Body:**
${email.body.substring(0, 2000)}  // Limit to first 2000 chars

Classify and return JSON:

{
  "category": "client|sales|internal|newsletter|receipt|notification|spam|other",
  "priority": "urgent|high|normal|low",
  "intent": "question|task_request|meeting_request|information|complaint|thank_you|follow_up|other",
  "sentiment": "positive|neutral|negative|angry",
  "actionable": true/false,
  "suggestedAction": "respond|create_task|schedule_meeting|archive|forward_to_team|escalate|none",
  "tags": ["tag1", "tag2"],
  "summary": "One sentence summary",
  "urgencyReason": "Why urgent (if priority is urgent/high)"
}

**Context for classification:**
- Client emails: From known client domains or containing project names
- Urgent: Mentions "urgent", "asap", "immediate", contains complaint, or meeting in <48hrs
- Sales: Cold outreach, partnership inquiries, lead responses
- Newsletter: Promotional content, marketing emails
- Actionable: Requires a response or action from me`;

return {
  prompt,
  email
};
```

### 6. Call AI for Classification (AI Call Wrapper subflow)

```json
{
  "prompt": "={{ $json.prompt }}",
  "mode": "json",
  "systemInstructions": "You are an email triage assistant. Classify emails accurately to help prioritize and automate responses. Be conservative with 'urgent' priority.",
  "temperature": 0.1,
  "model": "gpt-4-turbo"
}
```

### 7. Check AI Success (IF node)

```
Condition: {{ $json.success === true }}
True: Process classification
False: Use fallback (mark as normal priority for manual review)
```

### 8. Extract Classification (Function node)

```javascript
const classification = $input.item.json.outputJson;
const email = $('Build AI Classification Prompt').item.json.email;

return {
  ...email,
  category: classification.category,
  priority: classification.priority,
  intent: classification.intent,
  sentiment: classification.sentiment,
  actionable: classification.actionable,
  suggestedAction: classification.suggestedAction,
  tags: classification.tags,
  summary: classification.summary,
  urgencyReason: classification.urgencyReason
};
```

### 9. Route by Priority (Switch node)

```
Mode: Expression
Routes:
1. {{ $json.priority === 'urgent' }} → Urgent handler
2. {{ $json.category === 'newsletter' }} → Auto-archive
3. {{ $json.intent === 'question' && $json.actionable }} → Draft response
4. {{ $json.intent === 'task_request' }} → Create task
5. {{ $json.intent === 'meeting_request' }} → Schedule handler
Default: Apply labels only
```

### 10a. Handle Urgent Emails (Notification Dispatch subflow)

```json
{
  "severity": "urgent",
  "message": "Urgent email received",
  "context": {
    "From": "={{ $json.from }}",
    "Subject": "={{ $json.subject }}",
    "Summary": "={{ $json.summary }}",
    "Reason": "={{ $json.urgencyReason }}",
    "Link": "https://mail.google.com/mail/u/0/#inbox/{{ $json.id }}"
  },
  "workflowName": "smart-inbox-triage",
  "channels": ["slack", "sms"]
}
```

Also apply "URGENT" label:

```
Operation: Modify Message
Message ID: {{ $json.id }}
Add Labels: URGENT
Star: Yes
```

### 10b. Handle Newsletters/Low Priority (Gmail node)

```
Operation: Modify Message
Message ID: {{ $json.id }}
Add Labels: {{ $json.category }}
Remove From Inbox: Yes (archive)
Mark as Read: Yes
```

### 10c. Handle Questions - Draft Response (Function node)

```javascript
const email = $input.item.json;

const draftPrompt = `Draft a professional email response to this inquiry.

**Original email from ${email.from}:**
Subject: ${email.subject}

${email.body}

**Instructions:**
- Be professional but friendly
- Answer their question directly
- Keep it concise (2-3 paragraphs max)
- Sign off with my name
- If you need information I don't have, suggest next steps

**My context:**
- I'm a consultant specializing in automation and AI
- I typically respond within 24 hours
- I offer free 30-min discovery calls

Return the draft email body (no subject line needed).`;

return {
  draftPrompt,
  email
};
```

Call AI for draft:

```json
{
  "prompt": "={{ $json.draftPrompt }}",
  "mode": "text",
  "systemInstructions": "You are drafting email responses on behalf of a consultant. Be helpful, professional, and concise.",
  "temperature": 0.5,
  "model": "gpt-4-turbo"
}
```

Save draft to Gmail:

```
Operation: Create Draft
To: {{ $('Extract Classification').item.json.fromEmail }}
Subject: Re: {{ $('Extract Classification').item.json.subject }}
Message: {{ $json.outputText }}

Labels: needs-review-draft
```

### 10d. Handle Task Requests (Function node)

```javascript
const email = $input.item.json;

// Extract task details from email
const taskName = `Email request: ${email.subject}`;
const taskDescription = `From: ${email.from}
Received: ${email.receivedAt}

Summary: ${email.summary}

Original email:
${email.body}

Email link: https://mail.google.com/mail/u/0/#inbox/${email.id}`;

return {
  taskName,
  taskDescription,
  priority: email.priority === 'urgent' ? 'high' : 'normal',
  tags: ['email-request', ...email.tags],
  emailId: email.id
};
```

Create task in ClickUp/Asana:

```
Operation: Create Task
Name: {{ $json.taskName }}
Description: {{ $json.taskDescription }}
Priority: {{ $json.priority }}
Tags: {{ $json.tags.join(',') }}
```

Label email:

```
Operation: Modify Message
Message ID: {{ $json.emailId }}
Add Labels: task-created
```

### 10e. Handle Meeting Requests (Function node)

```javascript
const email = $input.item.json;

// Check if we should auto-respond with calendar link
const shouldAutoRespond = email.category === 'sales' ||
                          email.fromEmail.includes('@prospect.com');

if (!shouldAutoRespond) {
  return null;  // Skip auto-response
}

const response = `Hi ${email.from.split('<')[0].trim()},

Thanks for reaching out! I'd be happy to chat.

You can book a time on my calendar here: ${process.env.CALENDLY_LINK || 'https://calendly.com/yourlink'}

Looking forward to connecting!

Best,
${process.env.YOUR_NAME}`;

return {
  response,
  toEmail: email.fromEmail,
  subject: `Re: ${email.subject}`,
  emailId: email.id
};
```

Send response:

```
Operation: Send Email
To: {{ $json.toEmail }}
Subject: {{ $json.subject }}
Message: {{ $json.response }}
```

### 11. Apply Labels to All Emails (Gmail node)

```
Operation: Modify Message
Message ID: {{ $json.id }}
Add Labels:
  - ai-triaged
  - {{ $json.category }}
  - priority-{{ $json.priority }}
Mark as Read: {{ $json.priority === 'low' || $json.category === 'newsletter' }}
```

### 12. Aggregate Results (Aggregate node)

```
Aggregate All Items
```

### 13. Build Daily Summary (Function node)

```javascript
const emails = $input.all();

const byCategory = {};
const byPriority = {};
const actionsTaken = {
  draftsCreated: 0,
  tasksCreated: 0,
  archived: 0,
  flaggedUrgent: 0
};

emails.forEach(email => {
  const data = email.json;

  // Count by category
  byCategory[data.category] = (byCategory[data.category] || 0) + 1;

  // Count by priority
  byPriority[data.priority] = (byPriority[data.priority] || 0) + 1;

  // Count actions
  if (data.suggestedAction === 'respond') actionsTaken.draftsCreated++;
  if (data.suggestedAction === 'create_task') actionsTaken.tasksCreated++;
  if (data.suggestedAction === 'archive') actionsTaken.archived++;
  if (data.priority === 'urgent') actionsTaken.flaggedUrgent++;
});

return {
  total: emails.length,
  byCategory,
  byPriority,
  actionsTaken,
  timestamp: new Date().toISOString()
};
```

### 14. Send Daily Digest (Notification Dispatch subflow)

**Only send once daily (check if already sent today):**

```javascript
const lastDigest = $env.LAST_DIGEST_DATE;
const today = new Date().toDateString();

if (lastDigest === today) {
  return null;  // Already sent today
}

// Update environment variable (you'll need to store this in a database/sheet)
return $input.item.json;
```

```json
{
  "severity": "info",
  "message": "Daily inbox triage summary",
  "context": {
    "Emails processed": "={{ $json.total }}",
    "Urgent flagged": "={{ $json.actionsTaken.flaggedUrgent }}",
    "Drafts created": "={{ $json.actionsTaken.draftsCreated }}",
    "Tasks created": "={{ $json.actionsTaken.tasksCreated }}",
    "Auto-archived": "={{ $json.actionsTaken.archived }}"
  },
  "workflowName": "smart-inbox-triage",
  "channels": ["email"]
}
```

### 15. Log Completion (Audit Log subflow)

```json
{
  "workflowName": "smart-inbox-triage",
  "eventType": "success",
  "payloadSummary": "Processed {{ $('Build Daily Summary').item.json.total }} emails",
  "details": "={{ $('Build Daily Summary').item.json }}"
}
```

## Subflows Used

- **ai-call-wrapper:** For email classification and draft generation (steps 6 & 10c)
- **notification-dispatch:** For urgent alerts and daily digest (steps 10a & 14)
- **audit-log:** For workflow logging (step 15)

## Customization Tips

### Add VIP sender priority

```javascript
const vipSenders = [
  'importantclient@example.com',
  'boss@company.com',
  'investor@fund.com'
];

if (vipSenders.includes(email.fromEmail)) {
  classification.priority = 'urgent';
  classification.tags.push('vip');
}
```

### Train on your email history

Export labeled examples and include in prompt:

```javascript
const exampleClient = "Example client email: [subject] [body snippet]";
const exampleNewsletter = "Example newsletter: [subject] [body snippet]";

prompt += `\n\nExamples for reference:\n${exampleClient}\n${exampleNewsletter}`;
```

### Smart follow-up tracking

Track emails that need follow-up:

```javascript
if (email.isReply && email.sentiment === 'positive') {
  // Might need follow-up if no response in 3 days
  scheduledFollowUp = new Date();
  scheduledFollowUp.setDate(scheduledFollowUp.getDate() + 3);
}
```

### Integration with CRM

For sales emails, auto-log to CRM:

```javascript
if (classification.category === 'sales' && classification.actionable) {
  // Create/update contact in HubSpot
  // Log email as activity
  // Update deal stage if mentioned
}
```

### Custom rules override AI

Add manual rules that override AI:

```javascript
// Always prioritize certain subjects
if (email.subject.toLowerCase().includes('invoice')) {
  classification.category = 'receipt';
  classification.priority = 'high';
}

// Always archive certain senders
if (email.fromEmail.endsWith('@notifications.github.com')) {
  classification.suggestedAction = 'archive';
}
```

### Create summary email of archived items

Weekly digest of what was auto-archived:

```
Subject: This week's auto-archived emails
Body: FYI, these emails were automatically archived. Review if needed.
[List of archived emails with subject and sender]
```

## Environment Variables Needed

```bash
# Email
EMAIL_ADDRESS=your@email.com
EMAIL_PASSWORD=your-app-password  # or OAuth
IMAP_HOST=imap.gmail.com

# AI
AI_MODEL=gpt-4-turbo
AI_TEMPERATURE=0.1

# Calendar (for auto-responses)
CALENDLY_LINK=https://calendly.com/yourlink
YOUR_NAME=Your Name

# Task management
CLICKUP_API_KEY=your-clickup-key
CLICKUP_TASK_LIST_ID=default-list-id

# Notifications
SLACK_WEBHOOK_URL=your-slack-webhook
SMS_NOTIFY_NUMBER=your-phone  # for urgent emails
NOTIFICATION_EMAIL=your@email.com

# VIP senders (comma-separated)
VIP_SENDERS=client1@example.com,client2@example.com

# Audit
AUDIT_LOG_SHEET_ID=your-audit-sheet-id
```

## Performance Notes

- **Processing time:** 2-5 seconds per email
- **AI cost:** ~$0.02-0.05 per email (classification + drafting)
- **Recommended frequency:** Every 5-15 minutes (balance freshness vs. API costs)
- **Gmail API limits:** 250 quota units per user per second (well within limits)

**Optimization:**
- Use GPT-3.5-turbo for classification (5x cheaper, still accurate)
- Only draft responses for high-priority questions
- Batch process emails during off-hours

## Common Issues & Solutions

**Issue: AI classifies client emails as spam**
- Solution: Add known client domains to VIP list or provide examples in prompt

**Issue: Too many false urgent flags**
- Solution: Tighten urgency criteria in prompt, add "urgencyReason" field for transparency

**Issue: Drafts sound too robotic**
- Solution: Increase temperature to 0.7, provide example responses in your voice

**Issue: Newsletter detection fails**
- Solution: Add unsubscribe link detection: if body contains "unsubscribe", likely newsletter

**Issue: Gmail API quota exceeded**
- Solution: Reduce check frequency, or use IMAP (no quota)

**Issue: Email body truncated**
- Solution: Increase character limit in parse function, or summarize long emails first

**Issue: Thread handling - AI analyzes old quoted text**
- Solution: Improve quoted text removal regex, or pass only most recent message

This workflow transforms email from a time sink into a managed, automated system that surfaces only what matters.
