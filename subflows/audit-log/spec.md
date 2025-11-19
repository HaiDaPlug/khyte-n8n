# Subflow: Audit Log

## Purpose

A simple, standardized way to log workflow events for tracking, debugging, and reporting.

Use this subflow to:
- Track workflow executions (start, success, error)
- Log processing statistics
- Create audit trails for compliance
- Generate data for operational reports

## When to Use

Use `audit-log` for:

- Logging workflow start/completion
- Recording batch processing statistics
- Creating audit trails (who did what when)
- Debugging and troubleshooting
- Operational reporting data

**Don't use for:**

- User-facing notifications (use `notification-dispatch`)
- Every single item in a batch (too verbose)
- Sensitive data (PII, credentials, etc.)

## Inputs

The subflow expects these parameters:

| Parameter | Type | Required | Description | Default |
|-----------|------|----------|-------------|---------|
| `workflowName` | string | Yes | Name of the workflow | - |
| `eventType` | string | Yes | Event type: "start", "success", "error", "warning" | - |
| `payloadSummary` | string | No | Brief summary of what was processed | "" |
| `details` | object | No | Additional structured details | {} |
| `timestamp` | string | No | ISO timestamp | Auto-generated |

## Behavior

### 1. Log Entry Creation

Creates a standardized log entry:

```json
{
  "timestamp": "2025-11-19T10:30:00Z",
  "workflowName": "lead-enrichment-pipeline",
  "eventType": "success",
  "payloadSummary": "Processed 47 leads",
  "details": {
    "itemsProcessed": 47,
    "itemsFailed": 2,
    "duration": "3m 12s"
  },
  "environment": "production"
}
```

### 2. Storage

Logs can be stored in multiple destinations (configure one or more):

- **Google Sheets** — Simple, easy to view and query
- **Database table** — PostgreSQL, MySQL, Supabase, etc.
- **Notion database** — For teams already using Notion
- **External logging service** — Datadog, LogDNA, etc. (via webhook)

The default implementation uses Google Sheets for simplicity.

### 3. Event Types

**start:**
- Workflow beginning execution
- Useful for tracking duration (compare start to success timestamp)

**success:**
- Workflow completed successfully
- Include summary stats in details

**error:**
- Workflow failed
- Include error message and context in details

**warning:**
- Workflow completed but with issues
- Include what went wrong in details

## Outputs

The subflow returns a simple confirmation:

```json
{
  "logged": true,
  "logId": "row-123",
  "destination": "google-sheets",
  "timestamp": "2025-11-19T10:30:00Z"
}
```

**Key fields:**

- `logged` (boolean) — True if log entry was written successfully
- `logId` (string) — Reference ID (row number, database ID, etc.)
- `destination` (string) — Where it was logged

## Node-by-Node Implementation

### 1. Set Input Parameters (Set node)

```javascript
{
  "workflowName": "={{ $json.workflowName }}",
  "eventType": "={{ $json.eventType }}",
  "payloadSummary": "={{ $json.payloadSummary || '' }}",
  "details": "={{ $json.details || {} }}",
  "timestamp": "={{ $json.timestamp || $now.toISO() }}",
  "environment": "={{ $env.ENVIRONMENT || 'production' }}"
}
```

### 2. Build Log Entry (Function node)

```javascript
const { workflowName, eventType, payloadSummary, details, timestamp, environment } = $input.item.json;

return {
  timestamp,
  workflowName,
  eventType,
  payloadSummary,
  details: JSON.stringify(details),  // Stringify for sheet storage
  environment,
  loggedBy: 'n8n-automation'
};
```

### 3. Write to Google Sheets (Google Sheets node)

```
Operation: Append
Sheet ID: {{ $env.AUDIT_LOG_SHEET_ID }}
Range: Sheet1!A:G

Columns:
- timestamp: {{ $json.timestamp }}
- workflowName: {{ $json.workflowName }}
- eventType: {{ $json.eventType }}
- payloadSummary: {{ $json.payloadSummary }}
- details: {{ $json.details }}
- environment: {{ $json.environment }}
- loggedBy: {{ $json.loggedBy }}

Continue On Fail: true
```

### 4. Check Write Success (IF node)

Condition: `{{ $json.error === undefined }}`

- **True:** Go to "Return Success"
- **False:** Go to "Return Failure"

### 5. Return Success (Function node)

```javascript
return {
  logged: true,
  logId: `row-${Date.now()}`,
  destination: 'google-sheets',
  timestamp: $input.item.json.timestamp
};
```

### 6. Return Failure (Function node)

```javascript
return {
  logged: false,
  error: $input.item.json.error || 'Unknown error',
  destination: 'google-sheets',
  timestamp: $input.item.json.timestamp
};
```

## Usage Example

### Log workflow start

```json
{
  "workflowName": "lead-enrichment-pipeline",
  "eventType": "start",
  "payloadSummary": "Starting batch of 47 leads"
}
```

### Log workflow success

```json
{
  "workflowName": "lead-enrichment-pipeline",
  "eventType": "success",
  "payloadSummary": "Successfully enriched 45 of 47 leads",
  "details": {
    "itemsProcessed": 47,
    "itemsSucceeded": 45,
    "itemsFailed": 2,
    "duration": "3m 12s",
    "source": "Google Sheets"
  }
}
```

### Log workflow error

```json
{
  "workflowName": "lead-enrichment-pipeline",
  "eventType": "error",
  "payloadSummary": "Workflow failed: API authentication error",
  "details": {
    "error": "Invalid API key",
    "itemsProcessed": 15,
    "itemsFailed": 1,
    "failedAt": "AI enrichment step"
  }
}
```

### Log warning

```json
{
  "workflowName": "lead-enrichment-pipeline",
  "eventType": "warning",
  "payloadSummary": "Completed with rate limit warnings",
  "details": {
    "itemsProcessed": 47,
    "rateLimitHits": 3,
    "delayAdded": "15 seconds total"
  }
}
```

## Customization Tips

### Use database instead of sheets

Replace Google Sheets node with PostgreSQL/MySQL node:

```sql
INSERT INTO audit_logs (
  timestamp,
  workflow_name,
  event_type,
  payload_summary,
  details,
  environment
) VALUES (
  {{ $json.timestamp }},
  {{ $json.workflowName }},
  {{ $json.eventType }},
  {{ $json.payloadSummary }},
  {{ $json.details }},
  {{ $json.environment }}
)
```

### Add user tracking

If workflows are triggered by users, add a `userId` field:

```javascript
{
  "userId": "{{ $json.userId || $env.DEFAULT_USER_ID }}",
  // ... other fields
}
```

### Add execution ID for tracing

Link log entries to n8n execution:

```javascript
{
  "executionId": "{{ $execution.id }}",
  // ... other fields
}
```

### Batch logging for high-volume workflows

Instead of logging every workflow run, batch multiple runs:

```javascript
// In workflow: collect stats
const stats = {
  totalRuns: 100,
  totalSuccess: 95,
  totalFailed: 5,
  period: "2025-11-19 10:00 - 11:00"
};

// Log once per hour instead of 100 times
{
  "workflowName": "high-frequency-workflow",
  "eventType": "success",
  "payloadSummary": "Hourly batch completed",
  "details": stats
}
```

### Log to multiple destinations

Duplicate the Write node for each destination:

- Google Sheets (for easy viewing)
- PostgreSQL (for querying/reporting)
- External service (for centralized logging)

Run them in parallel.

### Filter by environment

Only log production events, skip dev:

```javascript
if (process.env.ENVIRONMENT === 'development') {
  return { logged: false, skipped: true, reason: 'dev environment' };
}
// ... proceed with logging
```

### Calculate workflow duration

At workflow start:

```json
{
  "eventType": "start",
  "timestamp": "2025-11-19T10:30:00Z"
}
```

At workflow end:

```javascript
const startTime = $('Log Start').item.json.timestamp;
const endTime = new Date().toISOString();
const duration = (new Date(endTime) - new Date(startTime)) / 1000;  // seconds

{
  "eventType": "success",
  "timestamp": endTime,
  "details": {
    "duration": `${duration}s`
  }
}
```

## Notes

- Keep log entries concise — avoid dumping large payloads
- Don't log PII (personally identifiable information) or sensitive data
- Use `details` for structured data, `payloadSummary` for human-readable summary
- Consider retention: archive or delete old logs periodically
- For high-volume workflows, batch logs or sample (log 1% of runs)

## Sheet Setup (if using Google Sheets)

Create a sheet with these columns:

| timestamp | workflowName | eventType | payloadSummary | details | environment | loggedBy |
|-----------|--------------|-----------|----------------|---------|-------------|----------|
| 2025-11-19T10:30:00Z | lead-enrichment | success | Processed 47 leads | {"itemsProcessed": 47} | production | n8n-automation |

**Tips:**

- Freeze header row
- Apply filters to header
- Add conditional formatting (green for success, red for error, yellow for warning)
- Regularly export to CSV and archive old entries

## Related Patterns

- **notification-dispatch:** For user-facing notifications (audit-log is more internal)
- **All workflows:** Should log start/success/error events

This subflow provides a simple audit trail for all `khyte-n8n` workflow activity.
