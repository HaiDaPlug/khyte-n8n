# Cash Flow and Summaries

This document contains 6 automation patterns for monitoring cash position, forecasting, and financial reporting. Focus: visibility and proactive management.

---

## 1. Daily Cash Position Report

**Trigger**: Daily at 8am
**Steps**: Query bank balances across all accounts → calculate total cash on hand → list incoming payments expected today → list outgoing payments scheduled today → calculate end-of-day projected balance → compare to historical baseline → flag if below threshold → send digest to finance team and leadership
**Outputs**: Daily cash visibility, early warning of low balances
**Time Saved**: 15-20 min per day
**Risks**: Bank API delays might show stale data (note timestamp)
**Variants**: Real-time dashboard vs daily email, multi-currency consolidation

---

## 2. Cash Flow Forecast Automation

**Trigger**: Weekly or on-demand
**Steps**: Pull expected inflows (open invoices with expected payment dates, recurring revenue) → pull expected outflows (vendor bills, payroll, subscriptions) → calculate net cash flow by week for next 90 days → identify potential shortfall weeks → visualize as chart → send to CFO/leadership → flag if any week shows negative balance
**Outputs**: 90-day cash visibility, runway awareness, proactive planning
**Time Saved**: 2-3 hours per week vs manual spreadsheet
**Risks**: Forecast assumes payment dates are accurate (reality varies)
**Variants**: Scenario modeling (best/worst case), sensitivity analysis

---

## 3. Low Balance Alerts

**Trigger**: Bank balance falls below threshold
**Steps**: Monitor bank balance in real-time or hourly → compare to minimum threshold (e.g., $10k) → if below, send urgent alert to CFO and finance team → include context (what payments are pending, what's coming in) → suggest actions (delay payments, follow up on receivables, line of credit draw) → escalate if critically low (below absolute minimum)
**Outputs**: No surprise overdrafts, time to take action
**Time Saved**: Prevents crisis mode (value >> time)
**Risks**: False alarms from normal fluctuations (set appropriate threshold)
**Variants**: Tiered alerts (yellow/orange/red), predictive alerts (will be low tomorrow)

---

## 4. Revenue vs Target Tracking

**Trigger**: Daily or weekly
**Steps**: Pull actual revenue from accounting system → compare to budget/target for month → calculate variance (ahead or behind) → project month-end revenue based on current pace → flag if projected to miss target by >10% → send weekly summary to leadership → include trend chart (this month vs last month vs target)
**Outputs**: Know if you're on track for monthly goals, early warning of shortfalls
**Time Saved**: 1-2 hours per week on manual reporting
**Risks**: Revenue recognition timing can skew comparisons
**Variants**: Department-level tracking, product-line breakdowns

---

## 5. Expense Budget Monitoring

**Trigger**: New expense recorded or weekly rollup
**Steps**: Categorize expense by department and category → sum expenses by category for month/quarter → compare to budget → calculate % of budget used and % of time period elapsed → flag if overspending (>budget allocation for time period) → send alerts to budget owners → monthly rollup to finance team and leadership
**Outputs**: Budget compliance, overspend warnings, spending trends
**Time Saved**: 1-2 hours per month vs manual spreadsheet tracking
**Risks**: Budget might need adjustments mid-period (allow revisions)
**Variants**: Project-based budgets, client-specific cost tracking

---

## 6. Monthly Financial Summary Automation

**Trigger**: First business day of new month (for previous month)
**Steps**: Compile key metrics: revenue, expenses, profit, cash balance, AR aging, AP aging → calculate month-over-month and year-over-year changes → generate charts (revenue trend, expense breakdown, cash flow) → use AI to write 3-5 bullet summary of month → highlight notable items (large expenses, revenue spike/drop) → format as executive summary → send to leadership and board → archive for record-keeping
**Outputs**: Consistent monthly reporting, leadership visibility, historical record
**Time Saved**: 2-4 hours per month vs manual report creation
**Risks**: Automated commentary might miss nuance (human review before sending)
**Variants**: Weekly flash reports, quarterly board reports

---

## Implementation Priority

1. **Daily Cash Position Report** (#1) - Foundation for cash management
2. **Low Balance Alerts** (#3) - Crisis prevention
3. **Cash Flow Forecast** (#2) - Strategic planning
4. **Monthly Financial Summary** (#6) - Leadership communication

Start with daily cash visibility and alerts, then add forecasting for strategic planning.
