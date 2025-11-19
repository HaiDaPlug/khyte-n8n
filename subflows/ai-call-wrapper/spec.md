# Subflow: AI Call Wrapper

## Purpose

A standardized, safe way to call AI APIs (OpenAI, Anthropic Claude, etc.) with consistent error handling, JSON parsing, and retry logic.

Use this subflow for all AI API calls to ensure:
- Reliable API interactions
- Consistent output formats
- Graceful handling of JSON parsing failures
- Normalized error responses

## When to Use

Use `ai-call-wrapper` for:

- Calling LLM APIs (OpenAI GPT, Anthropic Claude, etc.)
- Structured data extraction with JSON mode
- Text generation and summarization
- Classification and tagging tasks
- Any AI operation requiring reliable error handling

**Don't use for:**

- Simple static text transformations (use Function nodes)
- Operations that don't need AI

## Inputs

The subflow expects these parameters:

| Parameter | Type | Required | Description | Default |
|-----------|------|----------|-------------|---------|
| `prompt` | string | Yes | The user prompt/input for the AI | - |
| `mode` | string | No | Output mode: "json" or "text" | "text" |
| `systemInstructions` | string | No | System prompt/instructions | "" |
| `model` | string | No | AI model to use | From env: `AI_MODEL` |
| `temperature` | number | No | Temperature (0.0-1.0) | 0.3 |
| `maxTokens` | number | No | Maximum tokens to generate | 2000 |
| `apiProvider` | string | No | "openai" or "anthropic" | "openai" |

## Behavior

### 1. Request Building

The subflow constructs an API request based on the provider:

**For OpenAI (GPT):**
```json
{
  "model": "gpt-4-turbo",
  "messages": [
    {"role": "system", "content": "{{ systemInstructions }}"},
    {"role": "user", "content": "{{ prompt }}"}
  ],
  "temperature": 0.3,
  "max_tokens": 2000,
  "response_format": {"type": "json_object"}  // if mode === "json"
}
```

**For Anthropic (Claude):**
```json
{
  "model": "claude-3-5-sonnet-20241022",
  "system": "{{ systemInstructions }}",
  "messages": [
    {"role": "user", "content": "{{ prompt }}"}
  ],
  "temperature": 0.3,
  "max_tokens": 2000
}
```

### 2. Safe API Call

Internally uses `safe-http-call` to:
- Execute the request with retry logic
- Handle network errors
- Handle rate limits
- Normalize HTTP errors

### 3. Response Processing

**For mode: "text"**
- Extract text from response
- Return as string

**For mode: "json"**
- Extract JSON from response
- Attempt to parse JSON
- On parse failure:
  - Retry with stricter prompt (append "You MUST return valid JSON")
  - If retry fails, return normalized error

### 4. Output Normalization

All outputs follow a consistent structure:

**Success (text mode):**
```json
{
  "success": true,
  "mode": "text",
  "outputText": "The generated text...",
  "tokensUsed": 150,
  "model": "gpt-4-turbo"
}
```

**Success (json mode):**
```json
{
  "success": true,
  "mode": "json",
  "outputJson": {
    "extractedField1": "value",
    "extractedField2": "value"
  },
  "tokensUsed": 200,
  "model": "gpt-4-turbo"
}
```

**Error:**
```json
{
  "success": false,
  "error": {
    "type": "APIError" | "ParseError" | "NetworkError",
    "message": "Human-readable description",
    "details": { /* original error */ }
  }
}
```

## Outputs

The subflow returns a single normalized object (see Output Normalization above).

**Key fields to check:**

- `success` (boolean) — True if call succeeded, false otherwise
- `outputText` (string) — Generated text (if mode = "text")
- `outputJson` (object) — Parsed JSON (if mode = "json")
- `error` (object) — Error details on failure

## Node-by-Node Implementation

### 1. Set Input Parameters (Set node)

Normalize inputs with defaults:

```javascript
{
  "prompt": "={{ $json.prompt }}",
  "mode": "={{ $json.mode || 'text' }}",
  "systemInstructions": "={{ $json.systemInstructions || '' }}",
  "model": "={{ $json.model || $env.AI_MODEL || 'gpt-4-turbo' }}",
  "temperature": "={{ $json.temperature || 0.3 }}",
  "maxTokens": "={{ $json.maxTokens || 2000 }}",
  "apiProvider": "={{ $json.apiProvider || 'openai' }}",
  "retryCount": 0
}
```

### 2. Build API Request (Function node)

Create provider-specific request payload:

```javascript
const params = $input.item.json;
const isOpenAI = params.apiProvider === 'openai';
const isJSON = params.mode === 'json';

let payload, url, headers;

if (isOpenAI) {
  payload = {
    model: params.model,
    messages: [
      ...(params.systemInstructions ? [{role: 'system', content: params.systemInstructions}] : []),
      {role: 'user', content: params.prompt}
    ],
    temperature: params.temperature,
    max_tokens: params.maxTokens
  };

  if (isJSON) {
    payload.response_format = {type: 'json_object'};
    // Ensure system prompt mentions JSON
    if (!payload.messages[0] || payload.messages[0].role !== 'system') {
      payload.messages.unshift({role: 'system', content: 'You must respond with valid JSON.'});
    }
  }

  url = 'https://api.openai.com/v1/chat/completions';
  headers = {
    'Content-Type': 'application/json',
    'Authorization': `Bearer {{ $env.OPENAI_API_KEY }}`
  };

} else { // Anthropic
  payload = {
    model: params.model,
    messages: [{role: 'user', content: params.prompt}],
    temperature: params.temperature,
    max_tokens: params.maxTokens
  };

  if (params.systemInstructions) {
    payload.system = params.systemInstructions;
  }

  if (isJSON && payload.system) {
    payload.system += '\n\nYou must respond with valid JSON only.';
  }

  url = 'https://api.anthropic.com/v1/messages';
  headers = {
    'Content-Type': 'application/json',
    'x-api-key': `{{ $env.ANTHROPIC_API_KEY }}`,
    'anthropic-version': '2023-06-01'
  };
}

return {
  url,
  method: 'POST',
  headers,
  body: payload,
  originalParams: params
};
```

### 3. Call Safe HTTP (Execute Workflow or node group)

Call `safe-http-call` subflow with:

```json
{
  "url": "{{ $json.url }}",
  "method": "POST",
  "headers": "={{ $json.headers }}",
  "body": "={{ $json.body }}",
  "maxRetries": 3,
  "backoffFactor": 2,
  "timeoutMs": 60000
}
```

### 4. Check HTTP Success (IF node)

Condition: `{{ $json.success === true }}`

- **True:** Go to "Extract AI Response"
- **False:** Go to "Normalize API Error"

### 5. Extract AI Response (Function node)

Extract content from provider-specific response:

```javascript
const response = $input.item.json.data;
const params = $('Build API Request').item.json.originalParams;
const isOpenAI = params.apiProvider === 'openai';

let content, tokensUsed;

if (isOpenAI) {
  content = response.choices[0].message.content;
  tokensUsed = response.usage?.total_tokens || 0;
} else { // Anthropic
  content = response.content[0].text;
  tokensUsed = response.usage?.input_tokens + response.usage?.output_tokens || 0;
}

return {
  content,
  tokensUsed,
  model: params.model,
  mode: params.mode,
  retryCount: params.retryCount
};
```

### 6. Process by Mode (Switch node)

Route by `{{ $json.mode }}`:

- **Case "text":** Go to "Return Text Response"
- **Case "json":** Go to "Parse JSON Response"

### 7. Return Text Response (Function node)

```javascript
return {
  success: true,
  mode: 'text',
  outputText: $input.item.json.content,
  tokensUsed: $input.item.json.tokensUsed,
  model: $input.item.json.model
};
```

### 8. Parse JSON Response (Function node)

```javascript
const content = $input.item.json.content;
const retryCount = $input.item.json.retryCount;

try {
  const parsed = JSON.parse(content);
  return {
    success: true,
    mode: 'json',
    outputJson: parsed,
    tokensUsed: $input.item.json.tokensUsed,
    model: $input.item.json.model
  };
} catch (error) {
  // Parse failed
  return {
    parseSuccess: false,
    content,
    retryCount,
    originalParams: $('Build API Request').item.json.originalParams,
    error: error.message
  };
}
```

### 9. Check Parse Success (IF node)

Condition: `{{ $json.parseSuccess === false }}`

- **True:** Go to "Should Retry Parse?"
- **False:** Already succeeded (output is final)

### 10. Should Retry Parse? (IF node)

Condition: `{{ $json.retryCount < 1 }}`

Only retry once for JSON parse failures.

- **True:** Go to "Build Retry with Stricter Prompt"
- **False:** Go to "Return Parse Error"

### 11. Build Retry with Stricter Prompt (Set node)

```javascript
{
  "prompt": "={{ $json.originalParams.prompt }}\n\nIMPORTANT: You MUST return ONLY valid JSON. No explanations, no markdown, no extra text.",
  "mode": "json",
  "systemInstructions": "={{ $json.originalParams.systemInstructions }}\n\nYou must respond with valid JSON only. No other text.",
  "model": "={{ $json.originalParams.model }}",
  "temperature": "={{ $json.originalParams.temperature }}",
  "maxTokens": "={{ $json.originalParams.maxTokens }}",
  "apiProvider": "={{ $json.originalParams.apiProvider }}",
  "retryCount": "={{ $json.retryCount + 1 }}"
}
```

Loop back to "Build API Request".

### 12. Return Parse Error (Function node)

```javascript
return {
  success: false,
  error: {
    type: 'ParseError',
    message: 'Failed to parse AI response as JSON after retry',
    details: {
      rawContent: $input.item.json.content,
      parseError: $input.item.json.error
    }
  }
};
```

### 13. Normalize API Error (Function node)

For HTTP/API failures from safe-http-call:

```javascript
const httpError = $input.item.json.error;

return {
  success: false,
  error: {
    type: httpError.type === 'RateLimitError' ? 'RateLimitError' : 'APIError',
    message: `AI API call failed: ${httpError.message}`,
    details: httpError
  }
};
```

## Usage Example

### Extracting structured data (JSON mode)

```json
{
  "prompt": "Extract key information from this lead:\nName: Jane Doe\nCompany: Acme Marketing\nWebsite: acme-marketing.se\nRole: Marketing Director",
  "mode": "json",
  "systemInstructions": "Extract lead information into JSON with fields: name, company, website, role, industry (guess), companySize (guess: small/medium/large)",
  "temperature": 0.2,
  "model": "gpt-4-turbo"
}
```

**Expected output:**

```json
{
  "success": true,
  "mode": "json",
  "outputJson": {
    "name": "Jane Doe",
    "company": "Acme Marketing",
    "website": "acme-marketing.se",
    "role": "Marketing Director",
    "industry": "Marketing/Advertising",
    "companySize": "small"
  },
  "tokensUsed": 180,
  "model": "gpt-4-turbo"
}
```

### Generating text (text mode)

```json
{
  "prompt": "Write a friendly follow-up email for this proposal sent 5 days ago to Acme Corp. Keep it brief and professional.",
  "mode": "text",
  "systemInstructions": "You are a professional but friendly business consultant. Write concise, warm emails without being pushy.",
  "temperature": 0.7,
  "model": "gpt-4-turbo"
}
```

**Expected output:**

```json
{
  "success": true,
  "mode": "text",
  "outputText": "Subject: Following up on our proposal\n\nHi there,\n\nI wanted to check in regarding the proposal we sent over last week...",
  "tokensUsed": 120,
  "model": "gpt-4-turbo"
}
```

## Customization Tips

### Use cheaper models for simple tasks

For basic classification or extraction:

```json
{
  "model": "gpt-3.5-turbo",
  "temperature": 0.1
}
```

Or Claude Haiku:

```json
{
  "model": "claude-3-haiku-20240307",
  "apiProvider": "anthropic",
  "temperature": 0.1
}
```

### Use better models for complex reasoning

For nuanced writing or complex extraction:

```json
{
  "model": "gpt-4-turbo",
  "temperature": 0.5
}
```

Or Claude Sonnet:

```json
{
  "model": "claude-3-5-sonnet-20241022",
  "apiProvider": "anthropic",
  "temperature": 0.5
}
```

### Adjust temperature for creativity vs consistency

**Structured extraction (want consistency):**
```json
{
  "temperature": 0.0
}
```

**Creative writing (want variety):**
```json
{
  "temperature": 0.7
}
```

### Provide JSON schemas in system instructions

For complex JSON outputs:

```json
{
  "systemInstructions": "Return JSON matching this schema:\n{\n  \"name\": string,\n  \"tags\": string[],\n  \"priority\": number (1-10)\n}"
}
```

### Handle specific error types

After calling the subflow:

```javascript
if (!$json.success) {
  if ($json.error.type === 'RateLimitError') {
    // Wait longer or use different API key
  } else if ($json.error.type === 'ParseError') {
    // JSON parsing failed - maybe simplify the prompt
  } else {
    // General API error
  }
}
```

## Notes

- Always check `success` field before using output
- JSON mode works best with clear schemas in system instructions
- Parse errors trigger one automatic retry with stricter prompt
- Token usage is returned for cost tracking
- Supports both OpenAI and Anthropic (easily extensible to others)

## Related Patterns

- **safe-http-call:** Used internally for API requests
- **Lead enrichment workflows:** Heavy users of this subflow
- **Document processing workflows:** Use for extraction

This subflow is the standard way to interact with AI across all `khyte-n8n` workflows.
