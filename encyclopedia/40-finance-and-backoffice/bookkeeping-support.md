# Bookkeeping Support

This document contains 5 automation patterns to assist with categorization, reconciliation, and accounting preparation. Focus: accuracy and reduced manual entry.

---

## 1. Transaction Auto-Categorization

**Trigger**: New transaction appears in bank feed
**Steps**: Import transaction from bank → identify vendor from description → use AI/rules to categorize (travel, software subscription, office supplies, etc.) → assign to GL account → flag unusual transactions for review → learn from manual corrections → track categorization confidence score → send weekly review queue of low-confidence items
**Outputs**: 80-90% of transactions auto-categorized, less manual work
**Time Saved**: 2-3 hours per week vs manual categorization
**Risks**: Wrong categories affect reporting (review regularly)
**Variants**: Client-specific categorization for agencies, project-based allocation

---

## 2. Bank Reconciliation Assistance

**Trigger**: Monthly or on-demand
**Steps**: Pull transactions from bank → pull transactions from accounting system → match by date, amount, description → flag unmatched items from each side (bank has it, books don't; books have it, bank doesn't) → identify likely matches for human review → calculate reconciliation difference → generate reconciliation report → guide through resolving discrepancies
**Outputs**: Faster reconciliation, fewer errors, audit trail
**Time Saved**: 1-2 hours per month
**Risks**: Complex matching scenarios still need human judgment
**Variants**: Multi-account reconciliation, credit card reconciliation

---

## 3. Missing Documentation Alerts

**Trigger**: Weekly review or before month-end close
**Steps**: Scan all expenses in accounting system → identify expenses without receipts/documentation → check against documentation requirements by category → flag missing items → send alert to expense owners → track resolution → escalate if documentation not provided within 1 week → required for month-end close
**Outputs**: Complete documentation, audit readiness, tax compliance
**Time Saved**: 30-60 min per week chasing receipts
**Risks**: Some old expenses might be impossible to document (set cutoff rules)
**Variants**: Real-time alerts as expenses are entered

---

## 4. Duplicate Transaction Detection

**Trigger**: New transaction entered (manual entry or import)
**Steps**: Check for potential duplicates: same vendor + similar amount + similar date (within 7 days) → flag suspected duplicates → show side-by-side comparison → prompt user: "Is this a duplicate or separate transaction?" → if duplicate, merge or delete → track duplicate patterns to prevent at source
**Outputs**: Prevented duplicate expenses, accurate books
**Time Saved**: 15-30 min per occurrence
**Risks**: Legitimate similar transactions might be flagged (easy to dismiss)
**Variants**: Credit card vs expense report cross-checking

---

## 5. Journal Entry Template Automation

**Trigger**: Recurring event (month-end accruals, depreciation, etc.)
**Steps**: Identify recurring journal entries (rent, depreciation, amortization) → use saved templates → auto-populate date and standard amounts → adjust for any variables (usage-based allocations) → generate draft journal entries → send to accountant for review → on approval, post to ledger → track posting history
**Outputs**: Consistent month-end close, time savings, reduced errors
**Time Saved**: 30-60 min per month-end close
**Risks**: Wrong amounts if variables change (review before posting)
**Variants**: Quarter-end and year-end specific entries, project-based allocations

---

## Implementation Priority

1. **Transaction Auto-Categorization** (#1) - Daily time savings
2. **Bank Reconciliation Assistance** (#2) - Monthly accuracy
3. **Duplicate Transaction Detection** (#4) - Prevent errors
4. **Missing Documentation Alerts** (#3) - Compliance

Start with auto-categorization for immediate daily relief, then tackle monthly reconciliation.
