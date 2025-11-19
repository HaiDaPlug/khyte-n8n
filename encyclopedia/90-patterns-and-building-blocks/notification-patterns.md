# Notification Patterns

How to notify humans without creating spam or alert fatigue.

## The Notification Spectrum

**Immediate → Real-time → Digest → On-demand**

Choose based on urgency and frequency:
- **Immediate (push, SMS)**: System down, security breach, VIP customer issue
- **Real-time (Slack, email)**: New lead, payment received, approval needed
- **Digest (batched email)**: Daily summary, weekly review, monthly report
- **On-demand (dashboard, manual check)**: Metrics, historical data, reference info

**Rule of thumb**: The more notifications you send, the less attention each gets. Be selective.

---

## Immediate Alerts

**When to use**: Critical issues requiring immediate action

**Characteristics**:
- Interrupts current work (push notification, SMS, phone call)
- Short, urgent message
- Clear action required
- Escalates if not acknowledged

**Example scenarios**:
- Server down, customers affected
- Security breach detected
- High-value customer submitted cancellation request
- Payment failure on day contract expires

**Best practices**:
- Reserve for true emergencies (cry wolf = ignored)
- Include context in notification (not just "ERROR")
- Provide direct action link when possible
- Track acknowledgment and response time
- Escalate if no response in X minutes

**Message template**:
```
[URGENT] Server prod-01 is down
Status: 503 errors on checkout flow
Impact: Customers cannot complete purchases
Action: Investigate immediately
Dashboard: [link]
Escalating to manager in 10 min if not acknowledged
```

**Anti-pattern**: Everything is urgent (so nothing is)

---

## Real-Time Notifications

**When to use**: Time-sensitive but not emergency

**Characteristics**:
- Sent within seconds/minutes of event
- Recipient can respond within hours
- Contains enough context to take action
- Multiple channels (Slack, email, mobile app)

**Example scenarios**:
- New lead assigned to you
- Contract signed, ready for delivery
- Approval request waiting
- Customer left review

**Best practices**:
- Send to appropriate channel (work hours = Slack, after hours = email)
- Group rapid-fire notifications (5 leads in 10 min → send one notification with all 5)
- Include "snooze" or "dismiss" options
- Allow channel preferences per person
- Don't repeat across too many channels

**Message template**:
```
New lead assigned: Acme Corp
Contact: John Smith (VP Sales)
Source: Website form | Score: 85/100
Last touch: 2 minutes ago
Actions: [Call now] [Schedule meeting] [View details]
```

**Notification fatigue prevention**:
- Max notifications per hour per person (cap at 10-20)
- Quiet hours (no notifications 10pm-7am unless critical)
- Vacation/focus mode (hold non-urgent notifications)

---

## Digest Notifications

**When to use**: Regular updates that don't need immediate action

**Characteristics**:
- Batched (daily, weekly, monthly)
- Summarizes multiple items
- Predictable timing (every Monday at 8am)
- Scannable format (bullets, tables)

**Example scenarios**:
- Daily sales summary
- Weekly content performance
- Monthly financial report
- Overdue tasks list

**Best practices**:
- Consistent schedule (same time, same day)
- Start with summary/highlights
- Group by category or priority
- Include trends (up/down vs previous period)
- Link to details, don't overwhelm with data
- Allow customization (what to include)

**Daily digest template**:
```
Good morning! Here's your daily summary for [date]

📈 HIGHLIGHTS
- 12 new leads (up 20% vs yesterday)
- $25k in closed deals
- 1 critical support ticket needs attention

📊 METRICS
Leads: 12 new | 8 contacted | 3 qualified
Deals: 5 progressed | 2 won | 1 lost
Tasks: 18 completed | 7 overdue

⚠️ NEEDS ATTENTION
- 3 deals stalled >14 days
- 1 high-priority support ticket open 48+ hours
- Monthly invoice reminder for Acme Corp

[View full dashboard]
```

**Common mistakes**:
- Too long (should read in <2 min)
- Too much detail (link to details instead)
- Irregular timing (hard to build habit)
- All data, no insight (what should I care about?)

---

## Exception-Only Notifications

**When to use**: Reduce noise, only notify when something is wrong

**Characteristics**:
- Silent when everything is normal
- Alert when threshold crossed or anomaly detected
- Requires good baseline understanding (what is "normal"?)

**Example scenarios**:
- Campaign spend 2x over daily budget
- Server response time >3 seconds
- No leads received in 24 hours (usually get 10-20/day)
- Inventory below minimum threshold

**Best practices**:
- Define "normal" clearly (baseline + acceptable variance)
- Tune thresholds to avoid false alarms
- Include context (how far from normal?)
- Suggest resolution actions
- Send "all clear" when back to normal

**Anomaly alert template**:
```
⚠️ Anomaly detected: Campaign spend

Current: $1,200 (last 6 hours)
Normal: ~$200 per 6-hour period
Status: 500% above baseline

Possible causes:
- Budget cap not applied
- Bid adjustment triggered surge
- Unusually high competition

Action: Review campaign settings
[Pause campaign] [View analytics]
```

**Threshold tuning**:
Start conservative (only alert on severe anomalies), gradually make more sensitive based on false positive rate.

---

## Progress and Status Updates

**When to use**: Long-running processes

**Characteristics**:
- Periodic updates showing progress
- Estimated completion time
- Ability to cancel if needed
- Final notification on completion

**Example scenarios**:
- Large data import (10,000 records)
- Video rendering or file processing
- Monthly report generation
- Multi-step approval workflow

**Best practices**:
- Update at meaningful milestones (25%, 50%, 75%, complete)
- Don't spam with every 1% increment
- Show estimated time remaining
- Allow cancellation
- Include summary on completion

**Progress template**:
```
Invoice generation: 50% complete
Processed: 250 of 500 invoices
Estimated completion: 15 minutes
[View details] [Cancel]

---

Invoice generation: Complete
Generated: 500 invoices | Total: $125,000
Failed: 3 (see error log)
Next step: Review and approve for sending
[Review invoices] [Error details]
```

---

## Smart Notification Routing

**Pattern**: Send to the right person, via the right channel

**Routing logic**:
1. **By role**: Sales lead → sales team, support ticket → support team
2. **By geography**: Lead in Sweden → Nordic rep
3. **By availability**: On-call rotation, business hours vs after-hours
4. **By expertise**: Technical issue → senior engineer, billing → finance
5. **By workload**: Round-robin to distribute evenly

**Channel selection**:
- **Desktop/laptop hours (9am-6pm)**: Slack preferred
- **Mobile hours (before 9am, after 6pm)**: Email or app notification
- **Urgent**: SMS or phone
- **Summary/digest**: Email
- **Collaborative**: Shared Slack channel
- **Private**: DM or email

**User preferences**:
Allow customization:
- "No Slack notifications after 7pm"
- "Email me daily digest, never real-time"
- "Only urgent issues via SMS"
- "I check dashboard myself, don't notify"

---

## Reducing Notification Fatigue

**Symptoms**:
- People ignore notifications
- Important alerts get missed in noise
- Complaints about too many messages
- Notifications disabled/muted

**Solutions**:

**1. Consolidate**
Instead of: 5 separate "New lead" notifications in 10 minutes
Send: One notification: "5 new leads in the last 10 minutes"

**2. Prioritize**
Tag notifications:
- 🔴 Urgent (act within hour)
- 🟡 Important (act today)
- 🔵 FYI (can wait)

**3. Digest by default**
Real-time opt-in, not opt-out

**4. Quiet hours**
No notifications 10pm-7am (except true emergencies)

**5. Snooze and defer**
"Remind me in 1 hour" option

**6. Smart grouping**
Group related notifications (all deal updates together, not scattered)

**7. Feedback loop**
Track: What % of notifications lead to action?
If <10%, you're over-notifying

---

## Notification Content Best Practices

**Good notification**:
- Subject clearly states what happened
- First line summarizes action needed
- Includes relevant context (who, what, when)
- Provides quick action links
- States what happens if no action taken

**Bad notification**:
- Generic subject: "Update"
- Requires clicking through to understand
- Missing context (who is this about?)
- No clear action
- Unclear urgency

**Before**:
```
Subject: Action Required
Body: There's something you need to review. Click here for details.
```

**After**:
```
Subject: Approval needed: $15k purchase request from Marketing
Body: Sarah requested approval to purchase Webinar platform license.
Amount: $15,000/year
Justification: Replace current tool (saves $5k/yr) + needed features
Action required by: End of day Friday
[Approve] [Reject] [Request more info]
If no action: Auto-escalates to CFO on Saturday
```

---

## Testing Notification Patterns

Before rolling out to team:

1. **Send to yourself first**: Live with it for a week. Too much? Too little?
2. **A/B test**: Try different frequencies with small group
3. **Measure action rate**: What % of notifications lead to action?
4. **Collect feedback**: "Was this notification helpful?"
5. **Iterate**: Adjust thresholds, frequency, content based on data

**Success metrics**:
- Notification → Action rate >25%
- Time to acknowledge/respond <1 hour
- Low opt-out rate (<5%)
- Positive feedback on surveys

The best notification is the one that prevents a problem or enables fast action. Every other notification is noise.
