# Triggers and Events

Common trigger patterns for starting automations, with pros, cons, and best practices.

## Schedule-Based Triggers

**When to use**: Recurring tasks, reports, reminders, batch processing

**Common patterns**:
- **Daily at specific time**: Morning reports, daily digests, end-of-day summaries
- **Weekly on specific day**: Monday planning, Friday reviews, weekend prep
- **Monthly on specific date**: Invoicing, financial close, subscription renewals
- **Custom intervals**: Every 4 hours, every 15 minutes, every quarter

**Pros**: Predictable, reliable, easy to understand
**Cons**: Rigid timing, might run when unnecessary, doesn't respond to real-time needs

**Best practices**:
- Choose times thoughtfully (8am report arrives before workday starts)
- Consider time zones for distributed teams
- Add "skip if" conditions (don't run on holidays)
- Log runs for debugging

**Common mistakes**:
- Too frequent (every 5 min when daily would suffice)
- Wrong timezone (UTC vs local time confusion)
- No failure handling (if it fails, when does it retry?)

---

## Data Change Triggers

**When to use**: Respond to status changes, new records, updates

**Common patterns**:
- **New record created**: New lead, new customer, new project
- **Status/stage change**: Deal moved to "Proposal Sent", support ticket closed
- **Field updated**: Email address changed, contract value increased
- **Record deleted**: Cleanup workflows, archival processes

**Pros**: Real-time response, only runs when needed, efficient
**Cons**: Depends on consistent data entry, can miss manual changes outside system

**Best practices**:
- Validate data before running automation (required fields present?)
- Handle partial data gracefully
- Add "changed from X to Y" logic (not just "is now Y")
- Prevent infinite loops (automation triggers itself)

**Common mistakes**:
- Triggering on every field change (too noisy)
- Not handling bulk updates (100 records at once breaks automation)
- Missing delayed triggers (status changed, then changed back quickly)

---

## Event-Based Triggers

**When to use**: External events, user actions, system events

**Common patterns**:
- **Webhook received**: Form submitted, payment received, API call from partner
- **Email received**: Support inquiry, contract signed, bounce notification
- **File uploaded**: Document added to folder, image ready to process
- **API event**: Social media mention, ad campaign status change, calendar event created

**Pros**: Immediate response, works across systems, flexible
**Cons**: Requires external system support, webhook reliability varies, security considerations

**Best practices**:
- Verify webhook signatures (prevent spoofing)
- Handle duplicate events (same event sent twice)
- Queue for processing (don't block webhook response)
- Log all incoming events for debugging

**Common mistakes**:
- No authentication on webhook endpoint (security risk)
- Processing synchronously (slow response times out sender)
- Not handling retries (sender retries if you don't respond fast)

---

## Manual Triggers

**When to use**: On-demand processing, admin tasks, one-time fixes

**Common patterns**:
- **Button in UI**: "Generate report now", "Sync data", "Send reminder"
- **Command in Slack**: `/invoice @client`, `/weekly-review`
- **Form submission**: "Request automation run"
- **API call**: Triggered by external script or dashboard

**Pros**: Full control, no accidental runs, safe for testing
**Cons**: Requires human action, can be forgotten, not truly automated

**Best practices**:
- Clear button labels and confirmation prompts
- Log who triggered and when
- Provide feedback (success/failure message)
- Set permissions (not everyone can trigger everything)

**Common mistakes**:
- No confirmation for destructive actions
- Unclear what the button does
- No feedback (did it work?)

---

## Conditional Triggers

**When to use**: Complex logic, multi-condition starts

**Common patterns**:
- **Threshold-based**: Bank balance < $10k, inventory < 50 units
- **Pattern detection**: 3 failed login attempts, 5 same questions asked
- **Time + condition**: Every Monday if pending tasks > 10
- **Aggregate triggers**: When total value of deals in stage > $100k

**Pros**: Intelligent, reduces noise, precise control
**Cons**: Complex to build, harder to debug, can miss edge cases

**Best practices**:
- Start simple, add complexity gradually
- Test edge cases thoroughly
- Log why trigger fired (which condition met)
- Provide manual override

**Common mistakes**:
- Over-complicated logic (simpler is better)
- No logging of condition evaluation
- Conditions that are never true

---

## Composite Triggers (Multiple Events)

**When to use**: Require multiple conditions or events

**Common patterns**:
- **AND triggers**: Form submitted AND email verified
- **OR triggers**: Payment received OR invoice manually marked paid
- **Sequence triggers**: Email sent → wait 3 days → no response → trigger
- **Aggregate triggers**: 5 positive reviews in one week → celebrate

**Pros**: Sophisticated workflows, handles complex scenarios
**Cons**: Difficult to debug, state management needed, can be fragile

**Best practices**:
- Document the logic clearly
- Track state between events
- Add timeouts (if second event doesn't happen in X time, reset)
- Test all paths

**Common mistakes**:
- State gets stuck (waiting forever for event that won't come)
- No timeout handling
- Can't restart or reset

---

## Choosing the Right Trigger

**Decision framework**:

1. **Does it need to happen exactly on time?** → Use schedule trigger
2. **Does it respond to data changing?** → Use data change trigger
3. **Does it respond to external system?** → Use webhook/event trigger
4. **Is it occasional/admin task?** → Use manual trigger
5. **Does it need multiple conditions?** → Use conditional or composite trigger

**Anti-patterns**:
- Polling when webhooks available (inefficient)
- Manual trigger for something that should be automatic
- Too-frequent scheduled checks when event-based would work
- Missing trigger entirely (automation just runs continuously)

**Testing triggers**:
- Set up test environment
- Trigger manually to verify behavior
- Monitor first few real triggers closely
- Add logging and error handling before going live
