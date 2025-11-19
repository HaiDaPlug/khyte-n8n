# Subflow: Safe HTTP Call

## Purpose

A reusable pattern for making HTTP API calls with built-in retry logic, timeout handling, and normalized error responses.

Use this subflow whenever you need to call an external API to ensure consistent error handling and resilience against transient failures.

## When to Use

Use `safe-http-call` for:

- Calling external REST APIs
- Fetching data from third-party services
- Webhook calls
- Any HTTP request that might fail due to network issues or rate limits

**Don't use for:**

- Simple, one-off HTTP calls where you don't need retry logic (though it doesn't hurt)
- Internal n8n workflow calls (use Execute Workflow node)

## Inputs

The subflow expects these parameters:

| Parameter | Type | Required | Description | Default |
|-----------|------|----------|-------------|---------|
| `url` | string | Yes | The full URL to call | - |
| `method` | string | Yes | HTTP method (GET, POST, PUT, DELETE, etc.) | GET |
| `headers` | object | No | HTTP headers as key-value pairs | {} |
| `body` | object/string | No | Request body (for POST/PUT/PATCH) | null |
| `maxRetries` | number | No | Maximum number of retry attempts | 3 |
| `backoffFactor` | number | No | Exponential backoff multiplier (seconds) | 2 |
| `timeoutMs` | number | No | Request timeout in milliseconds | 30000 |

## Behavior

### 1. Request Execution

The subflow performs an HTTP request with the provided parameters.

### 2. Error Detection

Detects and categorizes errors:

- **Network errors:** Connection refused, timeout, DNS failure
- **Server errors:** HTTP 5xx status codes
- **Rate limiting:** HTTP 429 status code
- **Client errors:** HTTP 4xx status codes (except 429)

### 3. Retry Logic

**Retries on:**
- Network errors (ECONNREFUSED, ETIMEDOUT, ENOTFOUND, etc.)
- HTTP 5xx errors (500, 502, 503, 504)
- HTTP 429 (Too Many Requests)

**Does NOT retry on:**
- HTTP 4xx errors (except 429) — client errors like 400, 401, 403, 404
- Success responses (2xx, 3xx)

**Retry strategy:**
- Exponential backoff: wait time = `backoffFactor ^ attemptNumber` seconds
- Default: 2s, 4s, 8s for retries 1, 2, 3
- Max retries: configurable (default 3)

### 4. Timeout Handling

Each request has a timeout (default 30 seconds). If the request doesn't complete within this time, it's treated as a network error and retried.

### 5. Response Normalization

All responses are normalized to a consistent structure:

**Success response:**
```json
{
  "success": true,
  "statusCode": 200,
  "data": { /* response body */ },
  "headers": { /* response headers */ }
}
```

**Error response:**
```json
{
  "success": false,
  "error": {
    "type": "NetworkError" | "ServerError" | "RateLimitError" | "ClientError",
    "message": "Human-readable error description",
    "statusCode": 500,
    "responseBody": { /* error response if available */ },
    "attemptsMade": 3
  }
}
```

## Outputs

The subflow returns a single normalized object (see Response Normalization above).

**Key fields to check:**

- `success` (boolean) — True if request succeeded, false otherwise
- `data` (object) — Response data on success
- `error` (object) — Error details on failure

## Node-by-Node Implementation

### 1. Set Input Parameters (Set node)

Prepare input variables:

```javascript
{
  "url": "{{ $json.url }}",
  "method": "{{ $json.method || 'GET' }}",
  "headers": "{{ $json.headers || {} }}",
  "body": "{{ $json.body || null }}",
  "maxRetries": "{{ $json.maxRetries || 3 }}",
  "backoffFactor": "{{ $json.backoffFactor || 2 }}",
  "timeoutMs": "{{ $json.timeoutMs || 30000 }}",
  "currentAttempt": 0
}
```

### 2. Build Request (Function node)

Prepare the HTTP request configuration:

```javascript
const config = {
  url: $input.item.json.url,
  method: $input.item.json.method,
  headers: $input.item.json.headers,
  timeout: $input.item.json.timeoutMs,
  maxRetries: $input.item.json.maxRetries,
  backoffFactor: $input.item.json.backoffFactor,
  currentAttempt: $input.item.json.currentAttempt
};

if ($input.item.json.body) {
  config.body = $input.item.json.body;
}

return config;
```

### 3. Execute HTTP Request (HTTP Request node)

Configure the HTTP Request node:

- **Method:** `{{ $json.method }}`
- **URL:** `{{ $json.url }}`
- **Headers:** `{{ $json.headers }}`
- **Body:** `{{ $json.body }}`
- **Timeout:** `{{ $json.timeout }}`
- **Continue On Fail:** Enabled (important!)

### 4. Check Response Status (IF node)

Condition: `{{ $json.statusCode >= 200 && $json.statusCode < 400 }}`

- **True branch:** Success → go to "Normalize Success Response"
- **False branch:** Error → go to "Check If Should Retry"

### 5. Normalize Success Response (Function node)

```javascript
return {
  success: true,
  statusCode: $input.item.json.statusCode || 200,
  data: $input.item.json,
  headers: $input.item.json.headers || {}
};
```

### 6. Check If Should Retry (Function node)

Determine if the error is retryable:

```javascript
const statusCode = $input.item.json.statusCode;
const currentAttempt = $input.item.json.currentAttempt || 0;
const maxRetries = $input.item.json.maxRetries || 3;

// Check if error is retryable
const isNetworkError = !statusCode; // No status code = network error
const isServerError = statusCode >= 500;
const isRateLimitError = statusCode === 429;
const shouldRetry = (isNetworkError || isServerError || isRateLimitError) && currentAttempt < maxRetries;

return {
  shouldRetry,
  currentAttempt,
  maxRetries,
  statusCode: statusCode || 0,
  errorType: isNetworkError ? 'NetworkError' :
             isServerError ? 'ServerError' :
             isRateLimitError ? 'RateLimitError' :
             'ClientError'
};
```

### 7. Retry Branch (IF node)

Condition: `{{ $json.shouldRetry === true }}`

- **True:** Go to "Calculate Backoff"
- **False:** Go to "Normalize Error Response"

### 8. Calculate Backoff (Function node)

```javascript
const backoffFactor = $input.item.json.backoffFactor || 2;
const currentAttempt = $input.item.json.currentAttempt || 0;
const waitSeconds = Math.pow(backoffFactor, currentAttempt + 1);

return {
  waitSeconds,
  nextAttempt: currentAttempt + 1
};
```

### 9. Wait (Wait node)

Wait for `{{ $json.waitSeconds }}` seconds.

### 10. Increment Attempt and Retry (Set node)

```javascript
{
  "currentAttempt": "{{ $json.nextAttempt }}",
  "url": "{{ $('Set Input Parameters').item.json.url }}",
  "method": "{{ $('Set Input Parameters').item.json.method }}",
  // ... copy other parameters
}
```

Loop back to "Build Request" node.

### 11. Normalize Error Response (Function node)

```javascript
const errorType = $input.item.json.errorType || 'UnknownError';
const statusCode = $input.item.json.statusCode || 0;
const attemptsMade = $input.item.json.currentAttempt || 0;

let message = 'Request failed';
if (errorType === 'NetworkError') {
  message = 'Network error: unable to reach the server';
} else if (errorType === 'ServerError') {
  message = `Server error: ${statusCode}`;
} else if (errorType === 'RateLimitError') {
  message = 'Rate limit exceeded (429)';
} else if (errorType === 'ClientError') {
  message = `Client error: ${statusCode}`;
}

return {
  success: false,
  error: {
    type: errorType,
    message,
    statusCode,
    responseBody: $input.item.json.body || $input.item.json,
    attemptsMade
  }
};
```

## Usage Example

### In a parent workflow

**Set node preparing the call:**

```json
{
  "url": "https://api.example.com/leads",
  "method": "POST",
  "headers": {
    "Content-Type": "application/json",
    "Authorization": "Bearer {{ $env.API_KEY }}"
  },
  "body": {
    "name": "{{ $json.leadName }}",
    "email": "{{ $json.leadEmail }}"
  },
  "maxRetries": 3,
  "backoffFactor": 2,
  "timeoutMs": 30000
}
```

**Execute Workflow node (or call the subflow nodes):**

Call `safe-http-call` subflow.

**IF node checking result:**

Condition: `{{ $json.success === true }}`

- **True:** Process `$json.data`
- **False:** Handle error from `$json.error`

## Customization Tips

### Adjust retry count for critical calls

For critical operations, increase `maxRetries`:

```json
{
  "maxRetries": 5,
  "backoffFactor": 2
}
```

### Faster retries for lightweight APIs

For fast APIs where you want quicker retries:

```json
{
  "maxRetries": 3,
  "backoffFactor": 1.5
}
```

This gives shorter wait times: 1.5s, 2.25s, 3.375s.

### Disable retries for testing

During development:

```json
{
  "maxRetries": 0
}
```

Fails immediately without retries, making it easier to see errors.

### Custom timeout for slow endpoints

For APIs known to be slow:

```json
{
  "timeoutMs": 60000
}
```

60-second timeout instead of default 30.

### Add custom error handling

After calling the subflow, add custom logic for specific error types:

```javascript
if (!$json.success) {
  if ($json.error.type === 'RateLimitError') {
    // Notify team, pause workflow, etc.
  } else if ($json.error.statusCode === 401) {
    // Invalid credentials - notify immediately
  }
}
```

## Notes

- This subflow is **stateless** — each call is independent
- Suitable for most REST APIs
- For GraphQL, adjust the body structure but logic remains the same
- For APIs with different retry requirements (e.g. exponential backoff with jitter), modify the backoff calculation
- Always check the `success` field before using `data`

## Related Patterns

- **ai-call-wrapper:** Uses `safe-http-call` internally for AI API requests
- **notification-dispatch:** May use `safe-http-call` for webhook notifications

This subflow is the foundation of reliable API integrations across all `khyte-n8n` workflows.
