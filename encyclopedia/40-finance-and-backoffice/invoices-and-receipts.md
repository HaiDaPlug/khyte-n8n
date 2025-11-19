# Invoices and Receipts

This document contains 9 automation patterns for invoice creation, delivery, payment tracking, and receipt management. Focus: accuracy and timeliness.

---

## 1. Automated Invoice Generation from Closed Deals

**Trigger**: Deal marked "Closed Won" in CRM
**Steps**: Extract deal details → generate invoice from template → populate line items, amounts, terms → send draft to sales/finance for review → on approval, send to customer → log in accounting system
**Outputs**: Instant invoicing, no manual data entry, consistent format
**Time Saved**: 15-30 min per invoice
**Risks**: Wrong amounts if deal data incomplete (validate before sending)
**Variants**: Recurring invoices, milestone-based billing, retainer invoices

---

## 2. Recurring Invoice Automation

**Trigger**: Monthly/quarterly billing date
**Steps**: Identify active subscription customers → generate invoice from template → adjust for usage/overages if applicable → send to customer automatically → log payment expected date → track payment status
**Outputs**: Never forget to invoice, consistent cash flow, reduced DSO
**Time Saved**: 1-2 hours per billing cycle
**Risks**: Customer changes (cancellations, plan changes) need manual intervention
**Variants**: Annual upfront invoices, usage-based adjustments, auto-payment integration

---

## 3. Payment Reminder Sequence

**Trigger**: Invoice due date approaching or passed
**Steps**: 7 days before due → friendly reminder email → due date → payment due notice → 3 days overdue → gentle follow-up → 7 days overdue → firmer reminder and alert to AR team → 14 days → escalate to collections process
**Outputs**: Faster payments, reduced overdue invoices
**Time Saved**: 10-15 min per invoice vs manual follow-up
**Risks**: Annoying good customers with spam (segment by payment history)
**Variants**: Different cadences by customer tier, auto-pause service for severe overdue

---

## 4. Payment Receipt and Reconciliation

**Trigger**: Payment received in bank or payment processor
**Steps**: Detect incoming payment → match to open invoice by amount/customer → mark invoice as paid in accounting system → send payment confirmation to customer → update CRM deal status → flag unmatched payments for review
**Outputs**: Real-time payment tracking, automated reconciliation
**Time Saved**: 20-30 min per day on reconciliation
**Risks**: Partial payments or wrong amounts won't auto-match (manual review queue)
**Variants**: Multi-currency handling, payment plan tracking

---

## 5. Receipt Capture and Categorization

**Trigger**: Receipt photo uploaded or emailed to designated address
**Steps**: Extract text from receipt image (OCR) → identify vendor, amount, date, items → use AI to categorize expense (meals, travel, supplies, etc.) → match to employee or project → create expense record in accounting system → flag unusual items for review
**Outputs**: No lost receipts, auto-categorization, tax-ready records
**Time Saved**: 5-10 min per receipt
**Risks**: OCR errors on poor quality images, wrong categorization needs review
**Variants**: Corporate card auto-categorization, per diem meal tracking

---

## 6. Vendor Bill Processing

**Trigger**: Vendor bill received via email
**Steps**: Extract bill details (vendor, amount, due date, terms) → match to purchase order if applicable → route for approval if required → schedule payment based on due date and cash flow → mark in accounting system → send payment on due date → reconcile against bank statement
**Outputs**: Never miss vendor payments, optimize payment timing
**Time Saved**: 15-20 min per bill
**Risks**: Duplicate bills if vendor sends multiple times (duplicate detection needed)
**Variants**: Early payment discounts automation, payment batching

---

## 7. Invoice Dispute Handling

**Trigger**: Customer replies to invoice with dispute
**Steps**: Detect dispute keywords in email → create dispute case → route to AR specialist or account manager → pause payment reminders → provide dispute resolution workflow → track resolution time → adjust invoice if needed → send corrected invoice → resume payment tracking
**Outputs**: Professional dispute handling, no spam to disputing customers
**Time Saved**: 10 min per dispute on coordination
**Risks**: AI dispute detection might miss some (manual forwarding option)
**Variants**: Automatic credit memo issuance for valid disputes

---

## 8. Month-End Invoice Review

**Trigger**: Last business day of month
**Steps**: Pull all invoices sent this month → identify unbilled work (completed projects without invoices) → flag missed invoices → check invoice aging (outstanding invoices by age) → generate AR aging report → send to finance team and leadership → create action items for overdue collections
**Outputs**: Complete monthly billing, visibility into collections needs
**Time Saved**: 1-2 hours per month
**Risks**: None (purely reporting, low risk)
**Variants**: Weekly mini-reviews, client-specific AR reports

---

## 9. Tax Document Preparation

**Trigger**: End of tax year or quarterly deadline approaching
**Steps**: Compile all receipts and expense records → categorize by tax category → generate expense summary by category → identify missing documentation → flag unusual items for accountant review → export to accountant's preferred format → archive backup copies → send to accountant with 2 weeks before deadline
**Outputs**: Tax-ready documentation, reduced accountant fees, no scrambling
**Time Saved**: 3-5 hours per tax period
**Risks**: Categorization errors could affect tax deductions (accountant should review)
**Variants**: Multi-entity handling, international tax compliance

---

## Implementation Priority

1. **Automated Invoice Generation** (#1) - Immediate cash flow impact
2. **Payment Reminder Sequence** (#3) - Reduce DSO
3. **Receipt Capture** (#5) - Daily pain point
4. **Payment Receipt and Reconciliation** (#4) - Accuracy critical

Start with invoice generation and payment reminders for immediate cash flow improvement.
