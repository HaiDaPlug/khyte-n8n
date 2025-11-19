# Recurring Tasks and Reminders

This document contains 10 automation patterns for recurring operational tasks, deadline reminders, and scheduled reviews. Focus: preventing "oops, we forgot" failures.

---

## 1. Monthly Recurring Task Generation

**Trigger**: First day of month (or specific date like 15th)

**Core Steps**:
1. Create recurring tasks in PM tool: monthly reports, invoicing reviews, backup verifications
2. Assign to appropriate team members
3. Set due dates (e.g., create on 1st, due by 10th)
4. Include instructions and templates in task description
5. Link to previous month's completion for reference
6. Send notification to assignee
7. Track completion and send reminders at 50%, 75%, 100% of deadline
8. Escalate to manager if overdue by 2 days

**Outputs**: Nothing forgotten, consistent execution, accountability

**Time Saved**: 15-30 min per month creating tasks manually

**Risks**: Task fatigue if too many recur ring items (audit quarterly)

**Variants**: Weekly, quarterly, or annual frequencies; team-based rotations

---

## 2. Deadline Reminder Escalation

**Trigger**: Task approaching due date

**Core Steps**:
1. Monitor all tasks with due dates
2. Send first reminder at 3 days before due
3. Second reminder at 1 day before due
4. Final reminder on due date morning
5. If still not complete by end of due date, escalate:
   - Notify assignee and manager
   - Mark as overdue in system
   - Add to manager's "needs attention" list
6. Daily overdue reminders until complete
7. Track overdue patterns by person and task type

**Outputs**: Fewer missed deadlines, visibility into chronic issues

**Time Saved**: Eliminates manual deadline tracking (30-60 min/week)

**Risks**: Reminder fatigue (people tune out), unrealistic deadlines cause noise

**Variants**: Custom reminder timing per task type, Slack vs email preferences

---

## 3. Quarterly Business Review Automation

**Trigger**: Last week of each quarter

**Core Steps**:
1. Generate QBR agenda from template
2. Pull data for review: revenue, pipeline, team metrics, OKRs
3. Create slides or doc with auto-populated charts
4. Identify topics needing discussion (missed goals, budget variances)
5. Schedule QBR meeting with leadership
6. Send pre-read materials 3 days before meeting
7. After meeting, collect action items and assign owners
8. Track action item completion through next quarter

**Outputs**: Structured quarterly planning, data-driven discussions

**Time Saved**: 2-3 hours per quarter on data gathering and formatting

**Risks**: Data auto-pull might be wrong (review before meeting)

**Variants**: Monthly or annual reviews, department-specific QBRs

---

## 4. Subscription and License Renewal Tracking

**Trigger**: 60, 30, 14, 7 days before renewal date

**Core Steps**:
1. Maintain database of all subscriptions with renewal dates
2. Send early warning (60 days): "Salesforce renews in 60 days. Review usage and budget."
3. At 30 days: "Confirm renewal or cancel Salesforce. Decide by [date]."
4. At 14 days: "Last call: Salesforce renews in 2 weeks."
5. At 7 days: Escalate to decision-maker if no action taken
6. Track renewal decisions (renewed, canceled, modified)
7. Update contract database with new terms
8. Monitor for unwanted auto-renewals

**Outputs**: No surprise renewals, budget control, vendor negotiation time

**Time Saved**: 30-60 min per renewal

**Risks**: Calendar sprawl (too many reminders), missing manual contract additions

**Variants**: Vendor-specific workflows, multi-year contracts

---

## 5. Backup Verification Automation

**Trigger**: Daily or weekly schedule

**Core Steps**:
1. Run automated backup test (restore sample file)
2. Verify backup completed successfully
3. Check backup size and completion time (flag anomalies)
4. Test restore process weekly
5. Alert if backup failed or incomplete
6. Escalate immediately (backups are critical)
7. Log verification results
8. Monthly report of backup health and trends

**Outputs**: Confidence in disaster recovery, early failure detection

**Time Saved**: 15-20 min per verification

**Risks**: False positives (investigate carefully), backup failures need immediate response

**Variants**: Different schedules for different systems, cloud vs local backups

---

## 6. Team Capacity Monitoring and Alerts

**Trigger**: Daily or weekly check of team workload

**Core Steps**:
1. Query project management system for assigned tasks
2. Calculate workload per person (hours assigned vs hours available)
3. Flag overallocated individuals (>100% capacity)
4. Flag underutilized individuals (<50% capacity)
5. Alert manager with capacity report
6. Suggest rebalancing: move tasks from overloaded to underloaded
7. Track capacity trends (is team consistently over/under?)
8. Feed into hiring or project scoping decisions

**Outputs**: Balanced workload, burnout prevention, resource visibility

**Time Saved**: 1-2 hours per week vs manual spreadsheet tracking

**Risks**: Workload estimates might be wrong (garbage in = garbage out)

**Variants**: Real-time vs weekly snapshots, team vs individual views

---

## 7. Compliance Deadline Tracker

**Trigger**: Upcoming compliance deadline (tax filings, certifications, audits)

**Core Steps**:
1. Maintain compliance calendar (annual, quarterly, monthly requirements)
2. Create tasks 90 days before deadline
3. Assign to compliance owner
4. Provide checklist of required documents and actions
5. Send escalating reminders (90, 60, 30, 14, 7, 3 days)
6. At 30 days: Alert leadership if not on track
7. Track completion and store proof of filing
8. Generate compliance status report monthly

**Outputs**: No missed compliance deadlines (penalties avoided), audit readiness

**Time Saved**: 30-60 min per deadline

**Risks**: Compliance requirements change (review calendar annually)

**Variants**: Industry-specific (GDPR, SOC2, HIPAA), multi-jurisdiction

---

## 8. Performance Review Cycle Automation

**Trigger**: Quarterly or annual review period

**Core Steps**:
1. Send review template to managers 30 days before due
2. Collect self-assessments from employees
3. Remind managers to complete reviews at 15 and 7 days
4. Flag incomplete reviews on due date
5. Schedule review meetings once forms complete
6. Track action items from reviews (goals, development plans)
7. Follow up on action items monthly
8. Compile review data for compensation/promotion decisions

**Outputs**: Timely reviews, structured feedback, development tracking

**Time Saved**: 2-3 hours per review cycle for HR

**Risks**: Rushed reviews if reminders ignored (start earlier)

**Variants**: 360-degree feedback, continuous vs annual reviews

---

## 9. System Health Check Automation

**Trigger**: Daily or weekly scheduled scan

**Core Steps**:
1. Check critical systems: website uptime, database performance, API response times
2. Run automated tests (login flows, key user journeys)
3. Monitor error rates and slow pages
4. Compare to baselines (normal performance)
5. Alert if anomalies detected:
   - Website down → Immediate page to on-call
   - Slow performance → Investigation ticket
   - Rising error rates → Alert engineering
6. Generate weekly health report
7. Track trends (improving or degrading?)

**Outputs**: Proactive issue detection, system reliability, uptime accountability

**Time Saved**: 30-60 min per day on manual checks

**Risks**: False alarms from normal fluctuations (tune thresholds)

**Variants**: Different monitoring for different systems, third-party monitoring integration

---

## 10. Meeting Agenda and Notes Automation

**Trigger**: Recurring meeting scheduled (weekly stand-up, monthly all-hands)

**Core Steps**:
1. Create agenda doc from template 2 days before meeting
2. Pull relevant data (metrics, project status updates)
3. Invite attendees to add agenda items
4. Send reminder 1 day before: "Add your items to tomorrow's agenda"
5. During meeting, take live notes in doc
6. After meeting, extract action items using AI
7. Create tasks for action items and assign owners
8. Send meeting summary to all attendees
9. Archive notes in team knowledge base

**Outputs**: Structured meetings, documented decisions, tracked actions

**Time Saved**: 15-20 min per meeting on prep and follow-up

**Risks**: AI action item extraction isn't perfect (review before sending)

**Variants**: Role-rotation for agenda creation, async vs live meetings

---

## Implementation Priority

1. **Deadline Reminder Escalation** (#2) - Immediate impact, prevents dropped balls
2. **Monthly Recurring Task Generation** (#1) - Consistency in routine operations
3. **Subscription Renewal Tracking** (#4) - Budget control and surprises avoided
4. **Compliance Deadline Tracker** (#7) - Critical for regulated businesses
5. Others based on specific operational pain points

Start with deadline tracking to prevent current failures, then add recurring task generation for proactive operations.
