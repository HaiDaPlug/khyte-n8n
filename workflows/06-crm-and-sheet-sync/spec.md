# Workflow: CRM and Sheet Sync

## Summary

Bidirectional synchronization between your CRM (HubSpot, Pipedrive, Salesforce) and Google Sheets or Airtable. Ensures data stays consistent across systems, allows non-technical team members to work in spreadsheets while keeping CRM updated, and prevents data silos.

**Time saved:** 2-3 hours weekly on manual data entry → Fully automated
**Best for:** Sales teams, agencies managing client data, teams transitioning to/from CRM

## Use Cases

1. **Sales pipeline in Sheets + CRM**
   - Sales team updates deal stages in Google Sheets
   - Changes sync to HubSpot/Pipedrive automatically
   - New deals from CRM appear in sheet for reporting

2. **Contact database maintenance**
   - Marketing team enriches contacts in Airtable
   - Updates push to CRM for sales team
   - New leads from CRM auto-populate Airtable

3. **Project tracking sync**
   - Client success team tracks projects in Sheets
   - Links to CRM deals for unified view
   - Status changes update CRM automatically

## Inputs & Dependencies

**Required:**
- **CRM system:** HubSpot, Pipedrive, Salesforce, or similar
- **Spreadsheet:** Google Sheets or Airtable
- **n8n credentials:**
  - CRM API credentials
  - Google Sheets OAuth2 or Airtable API key
- **Sync configuration:** Field mappings defined

**Data requirements:**
- Unique identifier field in both systems (email, deal ID, etc.)
- Consistent field types (text, number, date)
- Clear sync direction rules (bi-directional or one-way)

## High-Level Flow

```
1. Trigger (scheduled every 15-30 minutes OR webhook on changes)
   ↓
2. Determine sync direction(s) for this run:
   2a. CRM → Sheet
   2b. Sheet → CRM
   2c. Both (bi-directional)
   ↓
3. For CRM → Sheet sync:
   3a. Fetch recently modified records from CRM
   3b. For each record:
       - Check if exists in Sheet (by unique ID)
       - If exists: Compare timestamps, update if CRM newer
       - If new: Append to Sheet
   ↓
4. For Sheet → CRM sync:
   4a. Fetch recently modified rows from Sheet
   4b. For each row:
       - Check if exists in CRM (by unique ID)
       - If exists: Compare timestamps, update if Sheet newer
       - If new: Create in CRM
   ↓
5. Handle conflicts (same record updated in both):
   5a. Apply conflict resolution rule (CRM wins, Sheet wins, or manual)
   5b. Log conflicts for review
   ↓
6. Update "last synced" timestamps
   ↓
7. Send sync summary notification
   ↓
8. Log completion
```

## Node-by-Node Implementation

### 1. Trigger: Schedule Sync (Schedule Trigger node)

```
Trigger: Cron - every 30 minutes
Expression: */30 * * * *
```

**Alternative: Webhook Trigger (for real-time sync):**

Set up webhooks in CRM to trigger on record changes:

```
HTTP Method: POST
Path: /webhook/crm-sync
Authentication: Header Auth
```

### 2. Load Sync Configuration (Function node)

```javascript
// Define field mappings and sync rules
const syncConfig = {
  // Sync direction: 'crm_to_sheet', 'sheet_to_crm', 'bidirectional'
  direction: process.env.SYNC_DIRECTION || 'bidirectional',

  // Conflict resolution: 'crm_wins', 'sheet_wins', 'newest_wins', 'manual'
  conflictResolution: process.env.CONFLICT_RESOLUTION || 'newest_wins',

  // Unique identifier field
  uniqueIdField: {
    crm: 'email',  // or 'id' for deals
    sheet: 'Email'  // column name in sheet
  },

  // Field mappings: CRM field → Sheet column
  fieldMappings: {
    // Contact fields
    'email': 'Email',
    'firstname': 'First Name',
    'lastname': 'Last Name',
    'company': 'Company',
    'phone': 'Phone',
    'lifecyclestage': 'Stage',
    'hs_lead_status': 'Status',

    // Deal fields (if syncing deals)
    'dealname': 'Deal Name',
    'amount': 'Amount',
    'dealstage': 'Deal Stage',
    'closedate': 'Close Date',
    'pipeline': 'Pipeline',

    // Custom fields
    'custom_field_1': 'Custom Column 1'
  },

  // Timestamp fields for conflict detection
  timestampFields: {
    crm: 'hs_lastmodifieddate',  // HubSpot format
    sheet: 'Last Modified'
  },

  // Filters
  filters: {
    crm: {
      // Only sync active contacts
      lifecyclestage: ['lead', 'opportunity', 'customer']
    },
    sheet: {
      // Only sync rows where Status != 'Archived'
      statusColumn: 'Status',
      excludeValues: ['Archived', 'Deleted']
    }
  }
};

return {
  syncConfig,
  syncRunId: `sync-${Date.now()}`,
  startTime: new Date().toISOString()
};
```

### 3. Get Last Sync Timestamp (Google Sheets node)

Read from a "sync_metadata" sheet that tracks last sync:

```
Operation: Get All
Sheet ID: {{ $env.SYNC_METADATA_SHEET_ID }}
Range: sync_log!A:C
Filter: Last row only
```

Parse result:

```javascript
const lastSyncData = $input.item.json;

const lastSyncTimestamp = lastSyncData[0]?.timestamp || new Date(Date.now() - 86400000).toISOString();  // Default: 24 hours ago

return {
  lastSyncTimestamp,
  isFirstSync: !lastSyncData[0]
};
```

### 4. Branch: CRM to Sheet Sync

#### 4a. Fetch Updated CRM Records (HubSpot/Pipedrive node)

**For HubSpot:**

```
Operation: Get All
Resource: Contacts (or Deals)
Filters:
  hs_lastmodifieddate__gte: {{ $('Get Last Sync Timestamp').item.json.lastSyncTimestamp }}
Properties: {{ Object.keys($('Load Sync Configuration').item.json.syncConfig.fieldMappings) }}
Limit: 100
```

**For Pipedrive:**

```
Operation: Get All
Resource: Persons (or Deals)
Filters:
  update_time: {{ $('Get Last Sync Timestamp').item.json.lastSyncTimestamp }}
```

#### 4b. Fetch All Sheet Rows (Google Sheets node)

```
Operation: Get All
Sheet ID: {{ $env.CONTACTS_SHEET_ID }}
Range: Sheet1!A:Z
Options:
  - Data starts on row: 2 (skip header)
  - Data location on sheet: Columns A-Z
```

#### 4c. Map CRM Data to Sheet Format (Function node)

```javascript
const crmRecords = $('Fetch Updated CRM Records').all();
const sheetRows = $('Fetch All Sheet Rows').all();
const syncConfig = $('Load Sync Configuration').item.json.syncConfig;

const mappedRecords = crmRecords.map(record => {
  const crmData = record.json;

  // Map CRM fields to Sheet columns
  const mappedRow = {};

  Object.entries(syncConfig.fieldMappings).forEach(([crmField, sheetColumn]) => {
    mappedRow[sheetColumn] = crmData.properties?.[crmField] || crmData[crmField] || '';
  });

  // Add metadata
  mappedRow['Last Modified'] = crmData.properties?.hs_lastmodifieddate || crmData.update_time || new Date().toISOString();
  mappedRow['CRM ID'] = crmData.id;
  mappedRow['Synced At'] = new Date().toISOString();

  return {
    crmId: crmData.id,
    uniqueId: crmData.properties?.email || crmData.email,
    mappedRow,
    crmTimestamp: mappedRow['Last Modified']
  };
});

// Build lookup map of existing sheet rows
const sheetLookup = {};
sheetRows.forEach(row => {
  const uniqueId = row.json[syncConfig.uniqueIdField.sheet];
  if (uniqueId) {
    sheetLookup[uniqueId] = {
      rowNumber: row.json.rowNumber,
      data: row.json,
      timestamp: row.json[syncConfig.timestampFields.sheet]
    };
  }
});

return {
  mappedRecords,
  sheetLookup
};
```

#### 4d. Determine Actions for Each Record (Function node)

```javascript
const { mappedRecords, sheetLookup } = $input.item.json;
const syncConfig = $('Load Sync Configuration').item.json.syncConfig;

const toCreate = [];
const toUpdate = [];
const conflicts = [];

mappedRecords.forEach(record => {
  const existingRow = sheetLookup[record.uniqueId];

  if (!existingRow) {
    // New record - create in sheet
    toCreate.push(record);
  } else {
    // Existing record - check timestamps
    const crmTime = new Date(record.crmTimestamp);
    const sheetTime = new Date(existingRow.timestamp);

    if (crmTime > sheetTime) {
      // CRM is newer - update sheet
      toUpdate.push({
        ...record,
        rowNumber: existingRow.rowNumber
      });
    } else if (sheetTime > crmTime) {
      // Sheet is newer - potential conflict
      if (syncConfig.conflictResolution === 'crm_wins') {
        toUpdate.push({
          ...record,
          rowNumber: existingRow.rowNumber
        });
      } else {
        conflicts.push({
          uniqueId: record.uniqueId,
          crmData: record.mappedRow,
          sheetData: existingRow.data,
          crmTime,
          sheetTime
        });
      }
    }
    // else: timestamps equal, no action needed
  }
});

return {
  toCreate,
  toUpdate,
  conflicts,
  summary: {
    toCreate: toCreate.length,
    toUpdate: toUpdate.length,
    conflicts: conflicts.length
  }
};
```

#### 4e. Create New Rows in Sheet (Google Sheets node)

Loop through `toCreate` array:

```
Operation: Append Row
Sheet ID: {{ $env.CONTACTS_SHEET_ID }}
Columns: Auto-map from {{ $json.mappedRow }}
Values: {{ $json.mappedRow }}
```

#### 4f. Update Existing Rows in Sheet (Google Sheets node)

Loop through `toUpdate` array:

```
Operation: Update
Sheet ID: {{ $env.CONTACTS_SHEET_ID }}
Row Number: {{ $json.rowNumber }}
Columns: Auto-map from {{ $json.mappedRow }}
Values: {{ $json.mappedRow }}
```

### 5. Branch: Sheet to CRM Sync

#### 5a. Fetch Updated Sheet Rows (Google Sheets node)

```
Operation: Get All
Sheet ID: {{ $env.CONTACTS_SHEET_ID }}
Range: Sheet1!A:Z
Filter: Where 'Last Modified' > {{ $('Get Last Sync Timestamp').item.json.lastSyncTimestamp }}
```

#### 5b. Fetch All CRM Records (for lookup)

```
Operation: Get All
Resource: Contacts
Properties: email, id, hs_lastmodifieddate
Limit: 1000
```

#### 5c. Map Sheet Data to CRM Format (Function node)

```javascript
const sheetRows = $('Fetch Updated Sheet Rows').all();
const crmRecords = $('Fetch All CRM Records').all();
const syncConfig = $('Load Sync Configuration').item.json.syncConfig;

// Reverse the field mappings (Sheet column → CRM field)
const reverseMappings = {};
Object.entries(syncConfig.fieldMappings).forEach(([crmField, sheetColumn]) => {
  reverseMappings[sheetColumn] = crmField;
});

const mappedRows = sheetRows.map(row => {
  const sheetData = row.json;

  // Map Sheet columns to CRM fields
  const crmProperties = {};

  Object.entries(reverseMappings).forEach(([sheetColumn, crmField]) => {
    if (sheetData[sheetColumn] !== undefined && sheetData[sheetColumn] !== '') {
      crmProperties[crmField] = sheetData[sheetColumn];
    }
  });

  return {
    uniqueId: sheetData[syncConfig.uniqueIdField.sheet],
    crmProperties,
    sheetTimestamp: sheetData[syncConfig.timestampFields.sheet],
    rowNumber: row.json.rowNumber
  };
});

// Build CRM lookup
const crmLookup = {};
crmRecords.forEach(record => {
  const uniqueId = record.json.properties?.email || record.json.email;
  if (uniqueId) {
    crmLookup[uniqueId] = {
      crmId: record.json.id,
      timestamp: record.json.properties?.hs_lastmodifieddate || record.json.update_time
    };
  }
});

return {
  mappedRows,
  crmLookup
};
```

#### 5d. Determine Actions for Each Row (Function node)

```javascript
const { mappedRows, crmLookup } = $input.item.json;
const syncConfig = $('Load Sync Configuration').item.json.syncConfig;

const toCreateInCRM = [];
const toUpdateInCRM = [];

mappedRows.forEach(row => {
  const existingCRM = crmLookup[row.uniqueId];

  if (!existingCRM) {
    // New row - create in CRM
    toCreateInCRM.push(row);
  } else {
    // Existing record - update in CRM
    toUpdateInCRM.push({
      ...row,
      crmId: existingCRM.crmId
    });
  }
});

return {
  toCreateInCRM,
  toUpdateInCRM,
  summary: {
    toCreateInCRM: toCreateInCRM.length,
    toUpdateInCRM: toUpdateInCRM.length
  }
};
```

#### 5e. Create New Contacts in CRM (HubSpot/Pipedrive node)

Loop through `toCreateInCRM` array:

**For HubSpot:**

```
Operation: Create
Resource: Contact
Properties: {{ $json.crmProperties }}
```

**For Pipedrive:**

```
Operation: Create
Resource: Person
Fields: {{ $json.crmProperties }}
```

#### 5f. Update Existing Contacts in CRM (HubSpot/Pipedrive node)

Loop through `toUpdateInCRM` array:

**For HubSpot:**

```
Operation: Update
Resource: Contact
Contact ID: {{ $json.crmId }}
Properties: {{ $json.crmProperties }}
```

### 6. Handle Conflicts (Function node)

```javascript
const conflicts = $('Determine Actions for Each Record').item.json.conflicts || [];
const syncConfig = $('Load Sync Configuration').item.json.syncConfig;

if (conflicts.length === 0) {
  return null;  // No conflicts
}

// Log conflicts for manual review
const conflictReport = conflicts.map(conflict => ({
  id: conflict.uniqueId,
  issue: 'Both CRM and Sheet modified since last sync',
  crmModified: conflict.crmTime,
  sheetModified: conflict.sheetTime,
  resolution: syncConfig.conflictResolution,
  crmData: JSON.stringify(conflict.crmData),
  sheetData: JSON.stringify(conflict.sheetData)
}));

return {
  conflicts: conflictReport,
  count: conflicts.length
};
```

Write conflicts to a log sheet:

```
Operation: Append Rows
Sheet ID: {{ $env.SYNC_CONFLICTS_SHEET_ID }}
Rows: {{ $json.conflicts }}
```

### 7. Update Last Sync Timestamp (Google Sheets node)

```
Operation: Append Row
Sheet ID: {{ $env.SYNC_METADATA_SHEET_ID }}
Sheet: sync_log
Values:
  Timestamp: {{ $('Load Sync Configuration').item.json.startTime }}
  Sync Run ID: {{ $('Load Sync Configuration').item.json.syncRunId }}
  CRM to Sheet: {{ $('Determine Actions for Each Record').item.json.summary.toCreate + $('Determine Actions for Each Record').item.json.summary.toUpdate }}
  Sheet to CRM: {{ $('Determine Actions for Each Row').item.json.summary.toCreateInCRM + $('Determine Actions for Each Row').item.json.summary.toUpdateInCRM }}
  Conflicts: {{ $('Handle Conflicts').item.json.count || 0 }}
```

### 8. Build Sync Summary (Function node)

```javascript
const crmToSheet = $('Determine Actions for Each Record').item?.json.summary || { toCreate: 0, toUpdate: 0, conflicts: 0 };
const sheetToCRM = $('Determine Actions for Each Row').item?.json.summary || { toCreateInCRM: 0, toUpdateInCRM: 0 };
const conflicts = $('Handle Conflicts').item?.json.count || 0;

const totalChanges = crmToSheet.toCreate + crmToSheet.toUpdate +
                     sheetToCRM.toCreateInCRM + sheetToCRM.toUpdateInCRM;

return {
  totalChanges,
  crmToSheet: {
    created: crmToSheet.toCreate,
    updated: crmToSheet.toUpdate
  },
  sheetToCRM: {
    created: sheetToCRM.toCreateInCRM,
    updated: sheetToCRM.toUpdateInCRM
  },
  conflicts,
  syncRunId: $('Load Sync Configuration').item.json.syncRunId,
  completedAt: new Date().toISOString()
};
```

### 9. Send Sync Summary (Notification Dispatch subflow)

Only send if there were changes or conflicts:

```javascript
if ($json.totalChanges === 0 && $json.conflicts === 0) {
  return null;  // Skip notification
}
return $json;
```

```json
{
  "severity": "={{ $json.conflicts > 0 ? 'warning' : 'info' }}",
  "message": "CRM-Sheet sync completed",
  "context": {
    "Total changes": "={{ $json.totalChanges }}",
    "CRM → Sheet": "={{ $json.crmToSheet.created }} created, {{ $json.crmToSheet.updated }} updated",
    "Sheet → CRM": "={{ $json.sheetToCRM.created }} created, {{ $json.sheetToCRM.updated }} updated",
    "Conflicts": "={{ $json.conflicts }}"
  },
  "workflowName": "crm-and-sheet-sync",
  "channels": ["slack"]
}
```

### 10. Log Completion (Audit Log subflow)

```json
{
  "workflowName": "crm-and-sheet-sync",
  "eventType": "success",
  "payloadSummary": "Synced {{ $('Build Sync Summary').item.json.totalChanges }} records",
  "details": "={{ $('Build Sync Summary').item.json }}"
}
```

## Subflows Used

- **notification-dispatch:** For sync summary (step 9)
- **audit-log:** For workflow logging (step 10)

## Customization Tips

### Selective field sync

Only sync certain fields based on conditions:

```javascript
// Don't overwrite certain fields in CRM
const protectedCRMFields = ['owner', 'created_date', 'source'];

Object.keys(crmProperties).forEach(field => {
  if (protectedCRMFields.includes(field)) {
    delete crmProperties[field];
  }
});
```

### Sync different record types

Sync contacts, deals, and companies separately:

```javascript
const recordTypes = ['contacts', 'deals', 'companies'];

recordTypes.forEach(type => {
  // Run sync for this record type
  // Use different field mappings for each
});
```

### Data transformation during sync

Apply transformations:

```javascript
// Normalize phone numbers
if (crmProperties.phone) {
  crmProperties.phone = normalizePhoneNumber(crmProperties.phone);
}

// Convert currency
if (sheetData['Amount (SEK)']) {
  crmProperties.amount = convertToUSD(sheetData['Amount (SEK)']);
}

// Parse multi-select fields
if (sheetData['Tags']) {
  crmProperties.tags = sheetData['Tags'].split(',').map(t => t.trim());
}
```

### Add data validation

Validate before syncing:

```javascript
const errors = [];

if (!crmProperties.email || !isValidEmail(crmProperties.email)) {
  errors.push('Invalid email');
}

if (crmProperties.amount && isNaN(crmProperties.amount)) {
  errors.push('Amount must be a number');
}

if (errors.length > 0) {
  // Skip this record, log error
  return null;
}
```

### Archive deleted records

Track deletions:

```javascript
// In Sheet: mark row as "Deleted" instead of removing
// In CRM: move to archived stage

if (sheetData.Status === 'Deleted') {
  crmProperties.lifecyclestage = 'archived';
}
```

### Multi-directional sync priority

Different rules for different fields:

```javascript
const syncRules = {
  email: 'crm_wins',  // Email always from CRM
  phone: 'sheet_wins',  // Phone updated more often in sheet
  notes: 'append',  // Combine both versions
  stage: 'newest_wins'  // Use most recent
};
```

## Environment Variables Needed

```bash
# CRM
HUBSPOT_API_KEY=your-hubspot-key
PIPEDRIVE_API_TOKEN=your-pipedrive-token

# Spreadsheet
CONTACTS_SHEET_ID=your-google-sheet-id
AIRTABLE_BASE_ID=your-airtable-base-id
AIRTABLE_API_KEY=your-airtable-key

# Sync configuration
SYNC_DIRECTION=bidirectional
CONFLICT_RESOLUTION=newest_wins
SYNC_METADATA_SHEET_ID=sheet-for-tracking-syncs
SYNC_CONFLICTS_SHEET_ID=sheet-for-logging-conflicts

# Notifications
SLACK_WEBHOOK_URL=your-slack-webhook

# Audit
AUDIT_LOG_SHEET_ID=your-audit-sheet-id
```

## Performance Notes

- **Processing time:** 30-120 seconds for 100 records
- **API rate limits:**
  - HubSpot: 100 requests/10 seconds
  - Pipedrive: 600 requests/10 seconds
  - Google Sheets: 60 requests/minute per user
- **Recommended sync frequency:** Every 30 minutes (balance freshness vs. API limits)
- **Batch size:** Process 50-100 records at a time

**Optimization:**
- Only sync modified records (use timestamps)
- Cache lookups to reduce API calls
- Use batch API endpoints when available

## Common Issues & Solutions

**Issue: Sync conflicts increase over time**
- Solution: Review conflict resolution rules, may need to be more specific per field

**Issue: Duplicates created in CRM or Sheet**
- Solution: Ensure unique ID field is consistently populated and correctly mapped

**Issue: Some fields don't sync**
- Solution: Check field type compatibility (CRM multi-select vs. Sheet text), add transformation

**Issue: Sync is slow (>5 minutes)**
- Solution: Implement incremental sync (only changed records), use batch operations

**Issue: Data gets overwritten unexpectedly**
- Solution: Add "last modified by" tracking to identify sync vs. manual changes

**Issue: Date fields sync incorrectly**
- Solution: Normalize date formats to ISO 8601 before syncing

**Issue: CRM API rate limit exceeded**
- Solution: Reduce sync frequency, implement exponential backoff, or use batch endpoints

This workflow eliminates data silos and keeps teams in sync regardless of which tool they prefer to use.
