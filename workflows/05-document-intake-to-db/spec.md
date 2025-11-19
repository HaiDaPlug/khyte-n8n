# Workflow: Document Intake to Database

## Summary

Automatically processes uploaded documents (PDFs, invoices, contracts, forms) by extracting structured data using OCR and AI, validating key fields, and storing in a database or Google Sheets. Handles invoices, receipts, contracts, applications, and other business documents.

**Time saved:** 10-15 minutes per document → 30 seconds automated
**Best for:** Accountants, legal teams, HR departments, any business processing forms

## Use Cases

1. **Invoice/receipt processing**
   - Upload invoice PDFs to Dropbox
   - Extract vendor, amount, date, line items
   - Write to accounting spreadsheet
   - Flag for approval if over threshold

2. **Contract intake**
   - New contracts uploaded to shared folder
   - Extract parties, terms, dates, obligations
   - Store in contract database
   - Set renewal reminders

3. **Application/form processing**
   - Customer applications (loan, insurance, service request)
   - Extract all form fields
   - Validate required fields present
   - Route to appropriate team

## Inputs & Dependencies

**Required:**
- **Document source:** Dropbox, Google Drive, or email attachment
- **OCR/Document AI:** One of:
  - Google Cloud Vision API (best for simple extraction)
  - AWS Textract (best for forms/tables)
  - OpenAI GPT-4 Vision (best for complex documents)
- **AI API access:** OpenAI or Anthropic (for structured extraction)
- **Database:** Google Sheets, Airtable, PostgreSQL, or similar
- **n8n credentials:**
  - Storage provider OAuth2
  - OCR API credentials
  - OpenAI/Anthropic API key
  - Database credentials

**Document requirements:**
- PDF, PNG, JPG, or TIFF format
- Readable text (not handwritten, unless using advanced OCR)
- Consistent document type for best results

## High-Level Flow

```
1. Trigger (new file in folder / email attachment / webhook upload)
   ↓
2. Download document file
   ↓
3. Detect document type (invoice, contract, form, etc.)
   ↓
4. OCR/extract text from document
   ↓
5. Call AI to extract structured data:
   5a. Key fields based on document type
   5b. Validation checks
   5c. Confidence scores
   ↓
6. Validate extracted data:
   6a. Required fields present
   6b. Data types correct (dates, amounts)
   6c. Business rules met
   ↓
7. If validation passes:
   7a. Write to database/sheet
   7b. Move file to "processed" folder
   7c. Send confirmation
   ↓
8. If validation fails:
   8a. Flag for manual review
   8b. Send notification with issues
   ↓
9. Optional: Trigger downstream actions
   (e.g., approval workflow, payment scheduling)
   ↓
10. Log completion
```

## Node-by-Node Implementation

### 1. Trigger: New File in Folder (Dropbox/Google Drive Trigger)

**For Dropbox:**

```
Event: File Created
Folder: /Documents/Inbox
File Types: .pdf, .png, .jpg
```

**For Google Drive:**

```
Event: File Created
Drive: My Drive
Folder ID: {{ $env.DOCUMENT_INBOX_FOLDER_ID }}
```

**Alternative: Email Attachment Trigger:**

```
Trigger: Email (IMAP)
Folder: INBOX
Filter: Has attachment
Attachment Extensions: pdf, png, jpg
```

### 2. Extract File Metadata (Function node)

```javascript
const file = $input.item.json;

// Dropbox format
if (file.path_display) {
  return {
    fileName: file.name,
    filePath: file.path_display,
    fileId: file.id,
    uploadedAt: file.client_modified || file.server_modified,
    fileSize: file.size,
    mimeType: getMimeType(file.name),
    source: 'dropbox'
  };
}

// Google Drive format
return {
  fileName: file.name,
  fileId: file.id,
  uploadedAt: file.createdTime,
  fileSize: file.size,
  mimeType: file.mimeType,
  source: 'gdrive'
};

function getMimeType(fileName) {
  if (fileName.endsWith('.pdf')) return 'application/pdf';
  if (fileName.endsWith('.png')) return 'image/png';
  if (fileName.endsWith('.jpg') || fileName.endsWith('.jpeg')) return 'image/jpeg';
  return 'application/octet-stream';
}
```

### 3. Download File (HTTP Request node)

**For Dropbox:**

```
Method: POST
URL: https://content.dropboxapi.com/2/files/download
Headers:
  Authorization: Bearer {{ $credentials.dropboxApi.token }}
  Dropbox-API-Arg: {"path": "{{ $json.filePath }}"}
Response Format: File
Download File: Yes
```

**For Google Drive:**

```
Method: GET
URL: https://www.googleapis.com/drive/v3/files/{{ $json.fileId }}?alt=media
Headers:
  Authorization: Bearer {{ $credentials.googleDriveApi.token }}
Response Format: File
Download File: Yes
```

### 4. Detect Document Type (Function node)

```javascript
const fileName = $input.item.json.fileName.toLowerCase();
const metadata = $input.item.json;

// Simple heuristics based on filename
let documentType = 'unknown';
let extractionSchema = null;

if (fileName.includes('invoice') || fileName.includes('bill')) {
  documentType = 'invoice';
  extractionSchema = {
    vendor: 'string',
    invoiceNumber: 'string',
    invoiceDate: 'date',
    dueDate: 'date',
    totalAmount: 'number',
    currency: 'string',
    lineItems: 'array',
    paymentTerms: 'string'
  };
} else if (fileName.includes('receipt')) {
  documentType = 'receipt';
  extractionSchema = {
    merchant: 'string',
    date: 'date',
    total: 'number',
    paymentMethod: 'string',
    items: 'array'
  };
} else if (fileName.includes('contract') || fileName.includes('agreement')) {
  documentType = 'contract';
  extractionSchema = {
    parties: 'array',
    effectiveDate: 'date',
    expirationDate: 'date',
    contractValue: 'number',
    terms: 'string',
    obligations: 'array'
  };
} else if (fileName.includes('application') || fileName.includes('form')) {
  documentType = 'form';
  extractionSchema = {
    applicantName: 'string',
    contactEmail: 'string',
    contactPhone: 'string',
    submissionDate: 'date',
    formFields: 'object'
  };
}

return {
  ...metadata,
  documentType,
  extractionSchema
};
```

### 5. OCR/Extract Text from Document (Google Cloud Vision / AWS Textract)

**Option A: Google Cloud Vision API:**

```
Operation: Detect Text
Image Source: Binary File
File: {{ $json.binaryData }}
Features: DOCUMENT_TEXT_DETECTION
```

**Option B: AWS Textract (better for forms/tables):**

```javascript
// Using AWS SDK via Function node or HTTP Request
const AWS = require('aws-sdk');
const textract = new AWS.Textract({
  accessKeyId: process.env.AWS_ACCESS_KEY,
  secretAccessKey: process.env.AWS_SECRET_KEY,
  region: process.env.AWS_REGION
});

const params = {
  Document: {
    Bytes: $input.item.binary.data
  },
  FeatureTypes: ['FORMS', 'TABLES']
};

const result = await textract.analyzeDocument(params).promise();

return {
  fullText: extractFullText(result),
  formData: extractFormFields(result),
  tables: extractTables(result),
  rawOCR: result
};
```

**Option C: GPT-4 Vision (simplest, good for most documents):**

```javascript
// Convert PDF to image first, or use image directly
const base64Image = $input.item.binary.data.toString('base64');

return {
  imageData: base64Image,
  mimeType: $input.item.json.mimeType
};
```

Then call OpenAI:

```
Operation: Create Image Analysis
Model: gpt-4-vision-preview
Image: {{ $json.imageData }}
Prompt: Extract all text from this document accurately. Preserve structure and formatting.
Max Tokens: 4096
```

### 6. Build Structured Extraction Prompt (Function node)

```javascript
const ocrText = $input.item.json.fullText || $input.item.json.text;
const documentType = $('Detect Document Type').item.json.documentType;
const schema = $('Detect Document Type').item.json.extractionSchema;

let extractionPrompt = '';

if (documentType === 'invoice') {
  extractionPrompt = `Extract structured data from this invoice.

**OCR Text:**
${ocrText}

Return JSON matching this schema:
{
  "vendor": "Company name",
  "vendorAddress": "Full address",
  "invoiceNumber": "Invoice #",
  "invoiceDate": "YYYY-MM-DD",
  "dueDate": "YYYY-MM-DD",
  "totalAmount": numeric value only,
  "currency": "USD/EUR/SEK/etc",
  "taxAmount": numeric value,
  "lineItems": [
    {
      "description": "item description",
      "quantity": number,
      "unitPrice": number,
      "total": number
    }
  ],
  "paymentTerms": "e.g., Net 30",
  "confidence": "high/medium/low - your confidence in extraction accuracy"
}

Important:
- Dates in ISO format (YYYY-MM-DD)
- Amounts as numbers only (no currency symbols)
- If field not found, use null
- Be precise with numbers`;

} else if (documentType === 'contract') {
  extractionPrompt = `Extract key information from this contract/agreement.

**OCR Text:**
${ocrText}

Return JSON:
{
  "documentTitle": "Contract title",
  "parties": ["Party 1 legal name", "Party 2 legal name"],
  "effectiveDate": "YYYY-MM-DD",
  "expirationDate": "YYYY-MM-DD or null if perpetual",
  "autoRenewal": true/false,
  "noticeRequired": "days notice required for termination",
  "contractValue": numeric total value or null,
  "paymentTerms": "payment schedule",
  "keyObligations": [
    "Obligation 1",
    "Obligation 2"
  ],
  "confidence": "high/medium/low"
}`;

} else if (documentType === 'receipt') {
  extractionPrompt = `Extract data from this receipt.

**OCR Text:**
${ocrText}

Return JSON:
{
  "merchant": "Store/business name",
  "date": "YYYY-MM-DD",
  "time": "HH:MM",
  "total": numeric value,
  "tax": numeric value,
  "paymentMethod": "Cash/Card/etc",
  "items": [
    {"name": "item", "price": number}
  ],
  "confidence": "high/medium/low"
}`;

} else {
  // Generic extraction
  extractionPrompt = `Extract all key information from this document as structured JSON.

**OCR Text:**
${ocrText}

Identify document type and extract relevant fields. Return JSON.`;
}

return {
  extractionPrompt,
  ocrText,
  documentType,
  schema
};
```

### 7. Call AI for Structured Extraction (AI Call Wrapper subflow)

```json
{
  "prompt": "={{ $json.extractionPrompt }}",
  "mode": "json",
  "systemInstructions": "You are a document processing expert. Extract data accurately from OCR text. Follow the schema exactly. If you're uncertain about a field, mark confidence as 'low' and use your best judgment.",
  "temperature": 0.1,
  "model": "gpt-4-turbo"
}
```

### 8. Check Extraction Success (IF node)

```
Condition: {{ $json.success === true && $json.outputJson.confidence !== 'low' }}
True: Proceed to validation
False: Flag for manual review
```

### 9. Validate Extracted Data (Function node)

```javascript
const extracted = $input.item.json.outputJson;
const documentType = $('Build Structured Extraction Prompt').item.json.documentType;

const validationErrors = [];
const warnings = [];

if (documentType === 'invoice') {
  // Required fields
  if (!extracted.vendor) validationErrors.push('Missing vendor name');
  if (!extracted.invoiceNumber) validationErrors.push('Missing invoice number');
  if (!extracted.totalAmount) validationErrors.push('Missing total amount');
  if (!extracted.invoiceDate) validationErrors.push('Missing invoice date');

  // Data type validation
  if (extracted.totalAmount && isNaN(extracted.totalAmount)) {
    validationErrors.push('Total amount is not a valid number');
  }

  // Date validation
  if (extracted.invoiceDate && !isValidDate(extracted.invoiceDate)) {
    validationErrors.push('Invalid invoice date format');
  }

  // Business rules
  if (extracted.totalAmount > 10000) {
    warnings.push('High-value invoice (>$10,000) - requires approval');
  }

  if (extracted.dueDate) {
    const due = new Date(extracted.dueDate);
    const today = new Date();
    const daysUntilDue = Math.ceil((due - today) / (1000 * 60 * 60 * 24));

    if (daysUntilDue < 0) {
      warnings.push('Invoice is overdue');
    } else if (daysUntilDue < 7) {
      warnings.push('Invoice due within 7 days');
    }
  }
}

const isValid = validationErrors.length === 0;

return {
  ...extracted,
  isValid,
  validationErrors,
  warnings,
  validatedAt: new Date().toISOString()
};

function isValidDate(dateString) {
  const date = new Date(dateString);
  return date instanceof Date && !isNaN(date);
}
```

### 10. Route by Validation (IF node)

```
Condition: {{ $json.isValid === true }}
True: Write to database
False: Flag for manual review
```

### 11. Prepare Database Record (Function node)

```javascript
const extracted = $input.item.json;
const fileMetadata = $('Extract File Metadata').item.json;
const documentType = $('Build Structured Extraction Prompt').item.json.documentType;

// Map to database schema
let record = {
  // Common fields
  documentId: `${documentType}-${Date.now()}`,
  documentType,
  fileName: fileMetadata.fileName,
  fileId: fileMetadata.fileId,
  uploadedAt: fileMetadata.uploadedAt,
  processedAt: new Date().toISOString(),
  status: extracted.warnings.length > 0 ? 'needs-review' : 'processed',

  // Extracted data (varies by document type)
  ...extracted,

  // Metadata
  warnings: extracted.warnings.join('; '),
  extractionConfidence: extracted.confidence
};

// Remove internal fields
delete record.validationErrors;
delete record.isValid;

return record;
```

### 12. Write to Database (Google Sheets / Airtable / PostgreSQL node)

**For Google Sheets:**

```
Operation: Append Row
Sheet ID: {{ $env.DOCUMENTS_DATABASE_SHEET_ID }}
Sheet Name: {{ $json.documentType }}s

Values:
  Document ID: {{ $json.documentId }}
  Vendor/Party: {{ $json.vendor || $json.merchant || $json.parties?.[0] }}
  Date: {{ $json.invoiceDate || $json.date || $json.effectiveDate }}
  Amount: {{ $json.totalAmount || $json.total || $json.contractValue }}
  Status: {{ $json.status }}
  File Name: {{ $json.fileName }}
  Uploaded At: {{ $json.uploadedAt }}
  Warnings: {{ $json.warnings }}
```

**For Airtable:**

```
Operation: Create Record
Base ID: {{ $env.AIRTABLE_BASE_ID }}
Table: Documents
Fields:
  {
    "Document ID": "{{ $json.documentId }}",
    "Type": "{{ $json.documentType }}",
    "Vendor": "{{ $json.vendor }}",
    "Amount": {{ $json.totalAmount }},
    "Date": "{{ $json.invoiceDate }}",
    "Status": "{{ $json.status }}",
    "File": [{"url": "{{ $json.fileUrl }}"}],
    "Warnings": "{{ $json.warnings }}"
  }
```

**For PostgreSQL:**

```sql
INSERT INTO documents (
  document_id, document_type, vendor, invoice_number,
  invoice_date, due_date, total_amount, currency,
  status, file_name, file_id, warnings, created_at
) VALUES (
  $1, $2, $3, $4, $5, $6, $7, $8, $9, $10, $11, $12, NOW()
)
```

### 13. Move File to Processed Folder (Dropbox/Google Drive node)

```
Operation: Move
File ID: {{ $('Extract File Metadata').item.json.fileId }}
Destination Folder: /Documents/Processed/{{ $json.documentType }}s
New Name: {{ $json.documentId }}_{{ $('Extract File Metadata').item.json.fileName }}
```

### 14. Handle Validation Failures (Function node)

```javascript
const extracted = $input.item.json;
const fileMetadata = $('Extract File Metadata').item.json;

return {
  fileName: fileMetadata.fileName,
  documentType: $('Build Structured Extraction Prompt').item.json.documentType,
  errors: extracted.validationErrors,
  extractedData: extracted,
  fileUrl: fileMetadata.fileUrl || `File ID: ${fileMetadata.fileId}`,
  needsManualReview: true
};
```

### 15. Send Failure Notification (Notification Dispatch subflow)

```json
{
  "severity": "warning",
  "message": "Document processing failed - manual review needed",
  "context": {
    "File": "={{ $json.fileName }}",
    "Document type": "={{ $json.documentType }}",
    "Errors": "={{ $json.errors.join(', ') }}",
    "File link": "={{ $json.fileUrl }}"
  },
  "workflowName": "document-intake-to-db",
  "channels": ["email", "slack"]
}
```

### 16. Optional: Trigger Approval Workflow (Webhook node)

For high-value invoices or contracts:

```javascript
if ($json.totalAmount > 5000 || $json.documentType === 'contract') {
  return {
    triggerApproval: true,
    documentId: $json.documentId,
    approvalType: $json.totalAmount > 5000 ? 'financial' : 'legal',
    approvalAmount: $json.totalAmount,
    metadata: $json
  };
}
return null;
```

Call approval workflow:

```
Method: POST
URL: {{ $env.N8N_WEBHOOK_URL }}/webhook/approval-request
Body:
{
  "documentId": "{{ $json.documentId }}",
  "type": "{{ $json.approvalType }}",
  "amount": {{ $json.approvalAmount }},
  "metadata": "={{ JSON.stringify($json.metadata) }}"
}
```

### 17. Send Success Notification (Notification Dispatch subflow)

```json
{
  "severity": "info",
  "message": "Document processed successfully",
  "context": {
    "File": "={{ $('Prepare Database Record').item.json.fileName }}",
    "Type": "={{ $('Prepare Database Record').item.json.documentType }}",
    "Vendor/Party": "={{ $('Prepare Database Record').item.json.vendor || $('Prepare Database Record').item.json.merchant }}",
    "Amount": "={{ $('Prepare Database Record').item.json.totalAmount || $('Prepare Database Record').item.json.total }}",
    "Status": "={{ $('Prepare Database Record').item.json.status }}"
  },
  "workflowName": "document-intake-to-db",
  "channels": ["email"]
}
```

### 18. Log Completion (Audit Log subflow)

```json
{
  "workflowName": "document-intake-to-db",
  "eventType": "success",
  "payloadSummary": "Processed {{ $('Prepare Database Record').item.json.documentType }}: {{ $('Prepare Database Record').item.json.fileName }}",
  "details": {
    "documentId": "={{ $('Prepare Database Record').item.json.documentId }}",
    "status": "={{ $('Prepare Database Record').item.json.status }}",
    "warnings": "={{ $('Prepare Database Record').item.json.warnings }}"
  }
}
```

## Subflows Used

- **ai-call-wrapper:** For structured data extraction (step 7)
- **notification-dispatch:** For success/failure notifications (steps 15 & 17)
- **audit-log:** For workflow logging (step 18)

Optional:
- **safe-http-call:** For downloading files (step 3)

## Customization Tips

### Add document classification ML

Train a classifier for document types:

```javascript
// Use first page image + filename to classify
const classification = await classifyDocument(imageData, fileName);
// Returns: { type: 'invoice', confidence: 0.95 }
```

### Multi-page PDF handling

```javascript
// Split PDF into pages
const pages = await splitPDF(pdfBuffer);

// OCR each page
const ocrResults = await Promise.all(
  pages.map(page => ocrPage(page))
);

// Combine text
const fullText = ocrResults.join('\n\n--- PAGE BREAK ---\n\n');
```

### Table extraction for line items

Use specialized table extraction:

```javascript
// AWS Textract returns structured tables
const tables = ocrResult.Blocks.filter(b => b.BlockType === 'TABLE');

// Convert to structured data
const lineItems = parseTableToLineItems(tables[0]);
```

### Duplicate detection

Check if document already processed:

```javascript
// Query database for matching invoice number
const existing = await checkDuplicate(extracted.invoiceNumber);

if (existing) {
  return {
    isDuplicate: true,
    existingRecordId: existing.id,
    action: 'skip'
  };
}
```

### Auto-categorization for accounting

Map vendors to expense categories:

```javascript
const vendorCategoryMap = {
  'AWS': 'Cloud Services',
  'Google Workspace': 'Software',
  'Office Depot': 'Office Supplies'
};

const category = vendorCategoryMap[extracted.vendor] || 'Uncategorized';
```

### Currency conversion

For multi-currency invoices:

```javascript
if (extracted.currency !== 'USD') {
  const rate = await getExchangeRate(extracted.currency, 'USD');
  extracted.totalAmountUSD = extracted.totalAmount * rate;
  extracted.conversionRate = rate;
}
```

## Environment Variables Needed

```bash
# Document source
DROPBOX_ACCESS_TOKEN=your-dropbox-token
DOCUMENT_INBOX_FOLDER_ID=folder-id
GDRIVE_FOLDER_ID=google-drive-folder-id

# OCR
GOOGLE_CLOUD_VISION_KEY=your-gcp-key
AWS_ACCESS_KEY=your-aws-key
AWS_SECRET_KEY=your-aws-secret
AWS_REGION=us-east-1

# AI
OPENAI_API_KEY=your-openai-key
AI_MODEL=gpt-4-turbo
AI_TEMPERATURE=0.1

# Database
DOCUMENTS_DATABASE_SHEET_ID=your-sheet-id
AIRTABLE_API_KEY=your-airtable-key
AIRTABLE_BASE_ID=your-base-id
POSTGRES_CONNECTION_STRING=postgresql://...

# Business rules
HIGH_VALUE_THRESHOLD=10000
APPROVAL_REQUIRED_AMOUNT=5000

# Notifications
SLACK_WEBHOOK_URL=your-slack-webhook
NOTIFICATION_EMAIL=accounting@company.com

# Workflows
N8N_WEBHOOK_URL=https://your-n8n-instance.com

# Audit
AUDIT_LOG_SHEET_ID=your-audit-sheet-id
```

## Performance Notes

- **Processing time:** 30-90 seconds per document (OCR is slowest part)
- **OCR costs:**
  - Google Cloud Vision: ~$1.50 per 1000 pages
  - AWS Textract: ~$1.50-15 per 1000 pages (depending on features)
  - GPT-4 Vision: ~$0.01-0.05 per page
- **AI extraction cost:** ~$0.05-0.15 per document (GPT-4)
- **Accuracy:** 95-99% for clean, typed documents; 70-90% for poor quality

**Optimization:**
- Use GPT-4 Vision for OCR+extraction in one call (simplest, good quality)
- Batch process during off-hours
- Cache vendor mappings to reduce AI calls

## Common Issues & Solutions

**Issue: OCR misreads numbers (0 vs O, 1 vs l)**
- Solution: Add post-processing validation, check if amounts sum correctly

**Issue: Multi-page documents only process first page**
- Solution: Ensure OCR returns all pages, or split PDF and process each

**Issue: Handwritten documents fail**
- Solution: Use advanced OCR (AWS Textract with handwriting) or flag for manual entry

**Issue: Table/line item extraction inaccurate**
- Solution: Use structured OCR (Textract FORMS), or ask AI to validate totals match

**Issue: Date formats vary (MM/DD vs DD/MM)**
- Solution: Ask AI to normalize to ISO format, add validation logic

**Issue: Duplicate documents processed**
- Solution: Add duplicate check by invoice number or file hash before processing

**Issue: Foreign language documents**
- Solution: Use Google Cloud Vision (supports 50+ languages), set language in AI prompt

This workflow eliminates manual data entry and turns document chaos into structured, searchable data.
