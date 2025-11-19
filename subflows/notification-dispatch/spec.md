# Subflow: Notification Dispatch

## Purpose

A standardized way to send notifications (Slack, email, or both) with severity levels and consistent formatting.

Use this subflow to:
- Notify users about workflow completion
- Alert on errors and failures
- Surface high-priority items for human attention
- Send operational summaries

## When to Use

Use `notification-dispatch` for:

- Workflow completion notifications
- Error and failure alerts
- High-priority item notifications (e.g. important leads)
- Daily/weekly digest summaries
- Any user-facing notification

**Don't use for:**

- Verbose debug logging (use `audit-log` instead)
- Every single item processed (too noisy)
- Internal data passing between nodes

## Inputs

The subflow expects these parameters:

| Parameter | Type | Required | Description | Default |
|-----------|------|----------|-------------|---------|
| `severity` | string | Yes | Notification level: "info", "warning", or "error" | - |
| `message` | string | Yes | The main notification message | - |
| `context` | object | No | Additional context fields | {} |
| `channels` | array | No | Where to send: ["slack"], ["email"], or ["slack", "email"] | ["slack"] |
| `workflowName` | string | No | Name of the workflow sending this notification | "" |

## Behavior

### 1. Message Formatting

Formats the message based on severity and channel:

**Slack format:**
```
🔵 INFO: Workflow completed
✉️ lead-enrichment-pipeline
───────────────
Processed 47 leads
Failed: 2
Duration: 3m 12s
```

**Email format:**
```
Subject: [INFO] Workflow completed - lead-enrichment-pipeline

Workflow: lead-enrichment-pipeline
Severity: info
Time: 2025-11-19 10:30:00

Message:
Processed 47 leads

Details:
- Failed: 2
- Duration: 3m 12s
```

### 2. Severity Indicators

**info:**
- Icon: 🔵 (Slack) / [INFO] (Email subject)
- Color: Blue (Slack attachments)
- Use for: Normal operations, completions, summaries

**warning:**
- Icon: ⚠️ (Slack) / [WARNING] (Email subject)
- Color: Orange (Slack attachments)
- Use for: Attention needed, unusual conditions, approaching limits

**error:**
- Icon: 🔴 (Slack) / [ERROR] (Email subject)
- Color: Red (Slack attachments)
- Use for: Failures, exceptions, critical issues

### 3. Channel Routing

- **Slack:** Uses webhook URL from `SLACK_WEBHOOK_URL` environment variable
- **Email:** Uses SMTP credentials configured in n8n
- **Both:** Sends to both channels in parallel

### 4. Delivery Tracking

Returns delivery status for each channel:

```json
{
  "notified": true,
  "channels": ["slack", "email"],
  "deliveryResults": {
    "slack": { "success": true, "timestamp": "2025-11-19T10:30:00Z" },
    "email": { "success": true, "timestamp": "2025-11-19T10:30:01Z" }
  }
}
```

## Outputs

The subflow returns a delivery summary:

```json
{
  "notified": true,
  "channels": ["slack"],
  "deliveryResults": {
    "slack": { "success": true, "timestamp": "2025-11-19T10:30:00Z" }
  },
  "failedChannels": []
}
```

**Key fields:**

- `notified` (boolean) — True if at least one channel succeeded
- `channels` (array) — Channels attempted
- `deliveryResults` (object) — Status per channel
- `failedChannels` (array) — Channels that failed

## Node-by-Node Implementation

### 1. Set Input Parameters (Set node)

```javascript
{
  "severity": "={{ $json.severity }}",
  "message": "={{ $json.message }}",
  "context": "={{ $json.context || {} }}",
  "channels": "={{ $json.channels || ['slack'] }}",
  "workflowName": "={{ $json.workflowName || '' }}",
  "timestamp": "={{ $now.toISO() }}"
}
```

### 2. Format Slack Message (Function node)

```javascript
const { severity, message, context, workflowName, timestamp } = $input.item.json;

const severityIcons = {
  info: '🔵',
  warning: '⚠️',
  error: '🔴'
};

const colors = {
  info: '#36a64f',
  warning: '#ff9900',
  error: '#ff0000'
};

let text = `${severityIcons[severity] || '🔵'} ${severity.toUpperCase()}: ${message}`;

if (workflowName) {
  text += `\n✉️ ${workflowName}`;
}

text += '\n───────────────';

// Add context fields
if (context && Object.keys(context).length > 0) {
  for (const [key, value] of Object.entries(context)) {
    text += `\n${key}: ${value}`;
  }
}

return {
  text,
  attachments: [{
    color: colors[severity] || colors.info,
    footer: `Sent at ${timestamp}`,
    ts: Math.floor(Date.now() / 1000)
  }]
};
```

### 3. Format Email Message (Function node)

```javascript
const { severity, message, context, workflowName, timestamp } = $input.item.json;

const subject = `[${severity.toUpperCase()}] ${message.substring(0, 50)}${message.length > 50 ? '...' : ''}${workflowName ? ' - ' + workflowName : ''}`;

let body = `Workflow: ${workflowName || 'N/A'}\n`;
body += `Severity: ${severity}\n`;
body += `Time: ${timestamp}\n\n`;
body += `Message:\n${message}\n\n`;

if (context && Object.keys(context).length > 0) {
  body += `Details:\n`;
  for (const [key, value] of Object.entries(context)) {
    body += `- ${key}: ${value}\n`;
  }
}

return {
  subject,
  body
};
```

### 4. Route to Channels (Switch node)

Based on `{{ $json.channels }}`, route to appropriate branches:

- If includes "slack" → go to Slack branch
- If includes "email" → go to Email branch
- If includes both → split to both branches

### 5. Send Slack Notification (HTTP Request node)

```
Method: POST
URL: {{ $env.SLACK_WEBHOOK_URL }}
Headers:
  Content-Type: application/json
Body:
  {
    "text": "={{ $('Format Slack Message').item.json.text }}",
    "attachments": "={{ $('Format Slack Message').item.json.attachments }}"
  }
Continue On Fail: true
```

### 6. Send Email Notification (Send Email node)

```
To: {{ $env.NOTIFICATION_EMAIL }}
Subject: {{ $('Format Email Message').item.json.subject }}
Text: {{ $('Format Email Message').item.json.body }}
Continue On Fail: true
```

### 7. Collect Results (Function node)

```javascript
// Gather results from Slack and Email nodes
const slackResult = $('Send Slack Notification').item?.json || null;
const emailResult = $('Send Email Notification').item?.json || null;

const deliveryResults = {};
const failedChannels = [];

if (slackResult) {
  const success = slackResult.statusCode === 200 || slackResult.ok === true;
  deliveryResults.slack = {
    success,
    timestamp: new Date().toISOString()
  };
  if (!success) failedChannels.push('slack');
}

if (emailResult) {
  const success = !emailResult.error;
  deliveryResults.email = {
    success,
    timestamp: new Date().toISOString()
  };
  if (!success) failedChannels.push('email');
}

const notified = Object.values(deliveryResults).some(r => r.success);

return {
  notified,
  channels: Object.keys(deliveryResults),
  deliveryResults,
  failedChannels
};
```

## Usage Example

### Info notification (workflow completion)

```json
{
  "severity": "info",
  "message": "Lead enrichment completed",
  "context": {
    "Processed": "47 leads",
    "Failed": "2",
    "Duration": "3m 12s"
  },
  "workflowName": "lead-enrichment-pipeline",
  "channels": ["slack"]
}
```

### Warning notification (approaching limit)

```json
{
  "severity": "warning",
  "message": "API rate limit at 80%",
  "context": {
    "Remaining": "200 calls",
    "Reset time": "2025-11-19 11:00:00"
  },
  "workflowName": "crm-sync",
  "channels": ["slack", "email"]
}
```

### Error notification (workflow failure)

```json
{
  "severity": "error",
  "message": "Workflow failed: Authentication error",
  "context": {
    "Error": "Invalid API key",
    "Workflow": "lead-enrichment-pipeline",
    "Items processed": "15 of 47"
  },
  "workflowName": "lead-enrichment-pipeline",
  "channels": ["email"]
}
```

## Customization Tips

### Add custom Slack channels per severity

Modify Format Slack Message node:

```javascript
const webhookUrls = {
  info: process.env.SLACK_WEBHOOK_INFO,
  warning: process.env.SLACK_WEBHOOK_ALERTS,
  error: process.env.SLACK_WEBHOOK_ERRORS
};

return {
  text,
  attachments,
  webhookUrl: webhookUrls[severity] || process.env.SLACK_WEBHOOK_URL
};
```

### Add @mentions for errors

In Format Slack Message:

```javascript
if (severity === 'error') {
  text = `<!channel> ${text}`;  // Mention @channel for errors
}
```

### Rich email formatting (HTML)

Change Email node to HTML mode and update Format Email Message:

```javascript
let htmlBody = `<h2>${severity.toUpperCase()}: ${message}</h2>`;
htmlBody += `<p><strong>Workflow:</strong> ${workflowName || 'N/A'}</p>`;
// ... etc
return { subject, body: htmlBody };
```

### Rate limit notifications

To avoid notification spam, add a rate limit check before sending:

```javascript
// Check if we've sent a notification for this workflow in the last N minutes
const recentNotifications = []; // Fetch from a cache or database
const cooldownMinutes = 15;

const lastNotification = recentNotifications.find(n =>
  n.workflowName === workflowName &&
  (Date.now() - new Date(n.timestamp)) < cooldownMinutes * 60 * 1000
);

if (lastNotification && severity === 'info') {
  return { skip: true };  // Skip info notifications if recently sent
}
```

### Different email recipients per severity

```javascript
const emailRecipients = {
  info: process.env.NOTIFICATION_EMAIL_INFO,
  warning: process.env.NOTIFICATION_EMAIL_TEAM,
  error: process.env.NOTIFICATION_EMAIL_ONCALL
};
```

## Notes

- Always use consistent severity levels: `info`, `warning`, `error`
- Keep messages concise but informative
- Use `context` for structured details, not in the main message
- For high-volume workflows, send batch notifications (e.g. "100 items processed" not "item processed" x100)
- Consider rate limiting to avoid notification spam
- Test notifications in a dev channel first

## Related Patterns

- **audit-log:** For detailed logging (less user-facing)
- **All workflows:** Should use this for completion/error notifications

This subflow ensures consistent, professional notifications across all `khyte-n8n` workflows.
