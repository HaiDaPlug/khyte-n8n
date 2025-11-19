# Architecture & Conventions

This document outlines the design patterns, naming conventions, and architectural principles used across all workflows and subflows in `khyte-n8n`.

## General Architecture Pattern

Most workflows follow this high-level structure:

```
Trigger
  ↓
Data Fetch / Input
  ↓
Validation / Normalization
  ↓
Processing (logic, transforms, branching)
  ↓
External Calls (APIs, AI, databases)
  ↓
Output / Write
  ↓
Notifications
  ↓
Audit Logging
```

Not every workflow needs every stage, but this is the general flow.

## Naming Conventions

### Workflow names

- Use descriptive, action-oriented names
- Format: `NN-kebab-case-name` (where NN is a number for ordering)
- Examples:
  - `01-lead-enrichment-pipeline`
  - `09-proposal-and-followup-automation`

### Node names in n8n

Use clear, descriptive names that indicate what each node does:

**Good examples:**
- `Fetch New Leads from Sheet`
- `Normalize Company Name`
- `Call AI: Classify Industry`
- `Write Enriched Data Back`
- `Send Slack Notification`
- `Log to Audit Sheet`

**Bad examples:**
- `HTTP Request` (too generic)
- `Function` (says nothing)
- `Code 1`, `Code 2` (meaningless)

**Pattern:**
- Start with a verb when possible: Fetch, Normalize, Call, Write, Send, Check, Filter
- Include context: what data, which system
- For subflow nodes, prefix with the subflow name: `SafeHTTP: Call External API`

### Variable names

In Function nodes and expressions:

- Use `camelCase` for JavaScript variables
- Use descriptive names: `leadEmail`, `enrichedData`, `apiResponse`
- Avoid single letters except for standard loop variables (`i`, `j`) or common abbreviations (`url`, `id`)

### Environment variable names

- Use `SCREAMING_SNAKE_CASE`
- Be specific: `OPENAI_API_KEY`, not just `API_KEY`
- Group by service:
  - `SLACK_WEBHOOK_URL`
  - `SLACK_BOT_TOKEN`
  - `CRM_API_KEY`
  - `CRM_BASE_URL`

## Subflow Usage Patterns

### When to use subflows

Use subflows for:

- **Repeated patterns** — Any logic you use in 2+ workflows
- **Error-prone operations** — HTTP calls, API parsing, etc.
- **Standard outputs** — Notifications, logging, etc.

### How to call subflows

In n8n, subflows can be:

1. **Execute Workflow nodes** — Call another workflow
2. **Copied node groups** — Copy/paste a standard set of nodes
3. **Function/Code nodes** — Encapsulate logic in reusable functions

For `khyte-n8n`, we recommend **node groups** or **Execute Workflow** depending on your n8n version and preferences.

### Standard subflows

Every workflow should consider using:

1. **safe-http-call** — For all external HTTP requests
2. **ai-call-wrapper** — For all AI API calls
3. **notification-dispatch** — For user-facing notifications
4. **audit-log** — For tracking workflow execution (optional but recommended)

## Error Handling Strategy

### Normalize errors

All errors should be caught and normalized to a consistent structure:

```json
{
  "success": false,
  "error": {
    "type": "NetworkError" | "ValidationError" | "APIError" | "ParseError",
    "message": "Human-readable description",
    "statusCode": 500,
    "details": { /* any additional context */ }
  }
}
```

### Where to handle errors

**At the subflow level:**
- HTTP errors (network, timeouts, 5xx)
- JSON parse errors
- API-specific errors

**At the workflow level:**
- Business logic errors (invalid data, missing required fields)
- Downstream failures (couldn't write to CRM, couldn't send notification)

### Error recovery strategies

1. **Retry with backoff** — For network/transient errors (use in `safe-http-call`)
2. **Fail gracefully** — Log the error, notify, continue with other items
3. **Stop workflow** — For critical failures (bad credentials, missing required data source)

### Error notifications

For production workflows:

- Send errors to `notification-dispatch` with `severity: "error"`
- Include context: which workflow, which item failed, what was the error
- Don't spam: batch errors or rate-limit notifications

## Retry Logic

### Standard retry pattern (HTTP calls)

```
Max retries: 3
Backoff: exponential (2s, 4s, 8s)
Retry on:
  - Network errors (ECONNREFUSED, ETIMEDOUT, etc.)
  - HTTP 5xx errors
  - Specific 429 (rate limit) errors

Do NOT retry on:
  - HTTP 4xx errors (except 429)
  - Authentication failures (401, 403)
  - Validation errors (400)
```

Implemented in: `subflows/safe-http-call`

### When NOT to retry

- User input errors (bad email, invalid format)
- Authorization failures (wrong API key)
- Resource not found (404)

## Data Flow Conventions

### Input validation

At the start of every workflow:

1. Check required fields exist
2. Validate formats (email, URL, date, etc.)
3. Set defaults for optional fields
4. Fail fast if critical data is missing

Use a `Function` or `Set` node named "Validate Inputs" near the top.

### Data normalization

Before processing, normalize:

- **Dates** — Convert to ISO 8601 (`YYYY-MM-DD` or `YYYY-MM-DDTHH:mm:ssZ`)
- **Emails** — Lowercase, trim whitespace
- **Names** — Trim, proper case if needed
- **URLs** — Remove trailing slashes, ensure protocol
- **Phone numbers** — Remove formatting, keep digits only (or use a standard format)

### Item processing

When processing lists (e.g. multiple leads, multiple emails):

- Use n8n's built-in item looping (most nodes process items automatically)
- For custom logic, use `Function` nodes with `items.map()` or `for` loops
- Consider rate limits: add delays between API calls if needed (`Wait` node)

### Output structure

Outputs should be consistent and predictable:

**For data workflows:**
```json
{
  "id": "unique-identifier",
  "source": "where this came from",
  "processed": true,
  "result": { /* actual data */ },
  "meta": {
    "processedAt": "2025-11-19T10:30:00Z",
    "workflowName": "lead-enrichment-pipeline"
  }
}
```

**For notification workflows:**
```json
{
  "notified": true,
  "channels": ["slack", "email"],
  "sentAt": "2025-11-19T10:30:00Z"
}
```

## Branching and Conditional Logic

### Use IF nodes for clarity

For simple true/false decisions, use `IF` nodes with clear condition names:

- "Is Email Valid?"
- "Is Lead Score > 7?"
- "Has Follow-up Been Sent?"

### Use Switch nodes for multiple paths

For 3+ outcomes, use `Switch` nodes:

- Email classification: `lead`, `support`, `internal`, `noise`
- Priority levels: `urgent`, `high`, `medium`, `low`
- Document types: `invoice`, `contract`, `brief`, `other`

### Handle all branches

Every IF or Switch should handle:

- True/False (or all switch cases)
- What happens on each path
- Where branches merge back together (if applicable)

Don't leave "hanging" branches that do nothing.

## AI Call Patterns

### Use consistent prompts

Structure AI prompts as:

```
System context:
[Role and constraints]

Task:
[What you want the AI to do]

Input:
[The data]

Output format:
[Expected structure, especially for JSON mode]
```

### Always specify output format

For structured data, use JSON mode and provide a schema:

```
Output format (JSON):
{
  "industry": "string",
  "size": "small" | "medium" | "large",
  "tags": ["array", "of", "strings"]
}
```

### Handle parse failures

Even in JSON mode, AI can return invalid JSON. Use `ai-call-wrapper` which:

- Tries to parse
- On failure, retries with stricter instructions
- On repeated failure, returns a normalized error

### Temperature and model selection

**For structured extraction (low creativity):**
- Temperature: 0.0 - 0.3
- Model: Cheaper/faster (Claude Haiku, GPT-3.5)

**For creative writing (emails, summaries):**
- Temperature: 0.5 - 0.8
- Model: Can use better models if needed (Claude Sonnet, GPT-4)

## Notification Patterns

### Severity levels

Use `notification-dispatch` with three levels:

1. **info** — Normal operations ("100 leads processed", "Weekly report ready")
2. **warning** — Attention needed ("Rate limit approaching", "Unusual data detected")
3. **error** — Something failed ("Workflow failed", "API returned 500", "Invalid credentials")

### When to notify

**Do notify for:**
- Workflow completion (especially long-running ones)
- Errors and failures
- Items requiring human attention (e.g. high-priority leads)

**Don't notify for:**
- Every single item processed (too noisy)
- Routine operations (unless explicitly requested)
- Verbose debug info (use audit logs instead)

### Message format

Good notification messages:

- **Start with the key info:** "3 new high-priority leads detected"
- **Include context:** "from LinkedIn export processed at 10:30"
- **Provide action if needed:** "Review in CRM: [link]"

Bad notification messages:

- "Workflow completed" (too vague)
- "Error" (no context)
- Long dumps of JSON or stack traces

## Audit Logging Patterns

### What to log

Log key events:

- Workflow start/completion
- Number of items processed
- Errors and failures
- Important decisions (e.g. "Classified as high-priority")

### What NOT to log

- Full item data (too much noise, potential PII issues)
- Every single step (use n8n's execution history for that)
- Sensitive info (API keys, passwords, etc.)

### Log structure

Standard log entry:

```json
{
  "timestamp": "2025-11-19T10:30:00Z",
  "workflowName": "lead-enrichment-pipeline",
  "eventType": "success" | "error" | "start",
  "summary": "Processed 47 leads",
  "details": {
    "itemsProcessed": 47,
    "itemsFailed": 2,
    "duration": "3m 12s"
  }
}
```

## Secrets and Credentials

### Never in the workflow

**NEVER hardcode:**
- API keys
- Passwords
- Webhook URLs (if they contain secrets)
- Database credentials
- Email addresses (if sensitive)

### Use n8n credentials

For each external service:

1. Create a credential in n8n's credential manager
2. Reference it in nodes (HTTP Request, API nodes, etc.)
3. Use environment variables for values n8n doesn't have built-in credential types for

### Environment variables

For values that change per deployment (but aren't secret):

- `SHEET_ID` (which sheet to read from)
- `CRM_BASE_URL` (which CRM instance)
- `NOTIFICATION_CHANNEL` (which Slack channel)

Use n8n's environment variable syntax: `{{ $env.SHEET_ID }}`

## Testing and Validation

### Test with sample data first

Before running on real data:

1. Create a small test dataset (2-5 items)
2. Run the workflow manually
3. Check outputs at each step
4. Verify error handling (intentionally cause errors)

### Common test scenarios

- **Empty input** — What if the sheet is empty?
- **Malformed data** — What if email is missing or invalid?
- **API failures** — What if the API is down?
- **Rate limits** — What if you hit rate limits?
- **Duplicate data** — What if the same item is processed twice?

### Validation checklist

Before deploying a workflow:

- [ ] All credentials configured and tested
- [ ] Error handling in place (try/catch, IF nodes for errors)
- [ ] Notifications configured (at least for errors)
- [ ] Audit logging in place (optional but recommended)
- [ ] Tested with sample data
- [ ] Tested error scenarios
- [ ] Documentation updated (if you changed the workflow)

## Performance Considerations

### Rate limiting

Many APIs have rate limits. Strategies:

1. **Batch processing** — Process items in groups, not all at once
2. **Add delays** — Use `Wait` nodes between API calls (e.g. 100ms - 1s)
3. **Respect headers** — Check `X-RateLimit-Remaining` headers
4. **Queue items** — For large datasets, use a queue system (or process in smaller batches)

### Avoid unnecessary API calls

- Cache results where possible (e.g. fetch all leads once, not once per item)
- Skip processing if data hasn't changed (check timestamps, checksums)
- Use batch API endpoints if available

### Large datasets

For 100+ items:

- Consider breaking into multiple workflow runs
- Use n8n's queue mode (if available in your version)
- Add progress notifications (e.g. "Processed 50/200 items")

## Workflow Composition

### Modular design

Break large workflows into logical sections:

1. **Input & Validation**
2. **Core Processing**
3. **External Calls** (AI, APIs)
4. **Output & Write**
5. **Notifications & Logging**

Use n8n's `Sticky Note` nodes to label sections.

### Reuse subflows

Don't copy/paste the same 10 nodes across workflows. Extract into a subflow and call it.

### One responsibility per workflow

Each workflow should do **one thing well**:

- Lead enrichment (don't also send follow-up emails in the same workflow)
- Email triage (don't also draft responses)

Chain workflows together using triggers (e.g. "When lead is enriched → trigger follow-up workflow") instead of making monolithic mega-workflows.

## Documentation Standards

Every workflow `spec.md` should include:

1. **Summary** — What it does in 2-3 sentences
2. **Use cases** — 2-3 concrete examples
3. **Inputs** — What data/systems it needs
4. **High-level flow** — Text diagram or bullet list
5. **Node-by-node outline** — Step-by-step implementation
6. **Subflows used** — Which building blocks it calls
7. **Customization tips** — What to change for different clients

Every subflow `spec.md` should include:

1. **Purpose** — What problem it solves
2. **Inputs** — Parameters and their types
3. **Behavior** — What it does step-by-step
4. **Outputs** — What it returns
5. **Usage example** — How to call it from a workflow

## Platform-Specific Notes

### n8n versions

These patterns are designed for:

- n8n 1.0+ (cloud or self-hosted)
- May require adjustments for older versions

### Credentials in self-hosted vs cloud

**Self-hosted:**
- Environment variables in `.env` file or docker-compose
- Credentials stored in n8n's encrypted database

**Cloud:**
- Credentials configured in n8n's UI
- Environment variables in cloud instance settings

Both work the same way from the workflow's perspective.

## Summary: The Khyte Way

These conventions reflect a simple philosophy:

- **Clear over clever** — Readable names, obvious logic
- **Safe over fast** — Retries, error handling, validation
- **Reusable over repeated** — Subflows, not copy/paste
- **Generic over specific** — Patterns, not hardcoded tools
- **Documented over implicit** — Specs, not tribal knowledge

Follow these patterns and your workflows will be easier to build, maintain, and hand off to clients.
