# Document Intake and Tagging

10 automation patterns for capturing, categorizing, and organizing documents. Focus: findability and metadata.

---

## 1. Email Attachment Auto-Filing
**Trigger**: Email with attachment received
**Steps**: Detect attachment → extract metadata → determine file type and sender → auto-file in appropriate folder structure → tag with date, sender, project → notify recipient of saved location
**Time Saved**: 5-10 min/day | **Variants**: Client-specific folders, contract auto-detection

## 2. Document OCR and Searchable PDF Creation
**Trigger**: Scanned document uploaded
**Steps**: Run OCR to extract text → create searchable PDF → extract key data (dates, amounts, parties) → store metadata → make fully text-searchable
**Time Saved**: 3-5 min per document | **Variants**: Multi-language OCR, handwriting recognition

## 3. Contract Data Extraction
**Trigger**: Contract document uploaded
**Steps**: Use AI to extract: parties, effective date, termination date, value, renewal terms → populate contract database → set renewal reminders → flag unusual terms → route for legal review if needed
**Time Saved**: 15-20 min per contract | **Variants**: NDA tracking, MSA management

## 4. Auto-Tagging with AI
**Trigger**: New document added to system
**Steps**: Analyze content with AI → extract topics and themes → assign category tags (client, project, document type) → detect related documents → suggest filing location → create backlinks
**Time Saved**: 2-3 min per doc | **Variants**: Custom taxonomies, multi-tag support

## 5. Invoice and Receipt OCR Processing
**Trigger**: Invoice/receipt image uploaded
**Steps**: Extract vendor, amount, date, line items → match to PO if applicable → create expense or AP record → file with tags → flag discrepancies
**Time Saved**: 5-10 min per doc | **Variants**: Multi-currency, tax extraction

## 6. Form Submission to Document Generation
**Trigger**: Form submitted (proposal request, questionnaire)
**Steps**: Capture responses → populate document template → generate formatted PDF → save to client folder → send copy to submitter → create follow-up tasks
**Time Saved**: 15-20 min per submission | **Variants**: Contract generation, SOW creation

## 7. Duplicate Document Detection
**Trigger**: Document uploaded
**Steps**: Calculate file hash → compare to existing docs → check filename similarity → flag exact duplicates → show near-duplicates for review → prevent redundant storage
**Time Saved**: Storage savings + confusion reduction | **Variants**: Version control integration

## 8. Document Expiration Tracking
**Trigger**: Document with expiration date filed (insurance, certifications, contracts)
**Steps**: Extract expiration date → set reminders at 90/60/30 days before → alert document owner → track renewal actions → archive expired docs
**Time Saved**: 30 min/month on manual tracking | **Variants**: Compliance documents, licenses

## 9. Client Document Portal Auto-Population
**Trigger**: New client onboarded or project milestone
**Steps**: Identify required client documents → generate from templates → populate client-specific data → upload to client portal → notify client of availability → track client access
**Time Saved**: 30-45 min per client | **Variants**: Project-specific portals

## 10. Document Version Control Automation
**Trigger**: Document edited
**Steps**: Detect changes → create version snapshot → log who changed what → maintain version history → allow rollback → notify stakeholders of major changes
**Time Saved**: 10-15 min per edit cycle | **Variants**: Approval workflows, change tracking

---

## Implementation Priority
1. **Email Attachment Auto-Filing** (#1) - Daily volume
2. **Auto-Tagging with AI** (#4) - Improves findability
3. **Contract Data Extraction** (#3) - High value, risk reduction
4. **Document OCR** (#2) - Makes everything searchable
