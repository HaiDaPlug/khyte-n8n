# Sales and Prospecting: Overview

## Scope of This Section

Sales teams live and die by follow-through. A lead that sits uncontacted for 24 hours is half as likely to convert as one contacted in 5 minutes. A deal that stalls in the pipeline for weeks without activity quietly dies. A CRM full of outdated contact information makes every sales call harder than it should be.

This section contains approximately 23 automation patterns designed to address these specific challenges:
- **Lead handling and routing**: Getting new leads to the right rep instantly, with all context
- **Follow-ups and nurturing**: Ensuring no opportunity falls through the cracks due to forgotten follow-ups
- **CRM hygiene and updates**: Keeping data clean, complete, and current without manual data entry

These patterns are particularly valuable for sales teams of 3-20 people, though they scale well for solo sellers and larger teams alike.

## Common Pain Points Addressed

### Pain Point 1: Leads Sit Too Long Before Contact
**Reality**: Lead comes in at 2pm. Sales rep is in a meeting. By the time they check their email at 4pm, the lead has already talked to three competitors.

**Automations that help**:
- Instant lead routing with mobile notification
- Auto-scheduled first call or email
- SLA tracking and escalation

### Pain Point 2: Follow-Ups Get Forgotten
**Reality**: "I'll follow up with them next week" rarely happens. Deals stall because no one remembered to check in.

**Automations that help**:
- Automatic follow-up sequences triggered by stages
- Reminder systems based on last activity
- Deal stagnation detection and alerts

### Pain Point 3: CRM is a Mess
**Reality**: Missing phone numbers, outdated job titles, duplicate records, empty fields that should have data.

**Automations that help**:
- Automatic data enrichment on new contacts
- Duplicate detection and merging
- Mandatory field enforcement and prompts

### Pain Point 4: Reps Don't Update CRM
**Reality**: "I'll update it later" means "never." Data lives in rep's head or scattered notes.

**Automations that help**:
- Email-to-CRM logging (BCC or AI-powered)
- Meeting auto-logging from calendar
- Mobile-friendly quick-update prompts

### Pain Point 5: No Visibility Into Pipeline Health
**Reality**: Manager asks "what's going to close this month?" and gets shrugs.

**Automations that help**:
- Automatic pipeline reports
- Deal health scoring
- Stagnation and risk detection

## What Makes Sales Automation Different

Sales automation must be fast, unobtrusive, and supportive—not bureaucratic. Sales reps are measured on closed deals, not data entry accuracy. Any automation that slows them down or feels like extra work will be circumvented.

The best sales automations:
- **Save reps time** (less admin = more selling)
- **Surface information** (right context at right moment)
- **Prompt action** (gentle nudges, not nagging)
- **Stay invisible** (work in background, only notify when necessary)

These aren't "sales automation platforms" like Outreach or SalesLoft (which handle email sequences and cadences). These are operational automations that make your CRM and sales process smoother.

## Tool Landscape

Sales teams typically work with:

**Core systems**:
- CRM (HubSpot, Pipedrive, Salesforce, Copper, etc.)
- Email (Gmail, Outlook)
- Calendar (Google Calendar, Outlook Calendar)
- Communication (Slack, Teams)

**Prospecting and enrichment tools**:
- LinkedIn Sales Navigator
- Apollo, Hunter, Clearbit
- ZoomInfo, Lusha

**Automation platforms well-suited for sales**:
- **n8n**: Great for CRM integrations and complex data flows
- **Zapier**: Fast setup for common CRM workflows
- **Make**: Visual builder, good for multi-step sequences

Most patterns in this section can be built on any platform that connects to your CRM's API.

## Implementation Strategy for Sales Teams

### Week 1: Lead Routing
Start with getting new leads to reps instantly:
- Lead routing automation
- Mobile notifications
- Basic SLA tracking

**Why**: Immediate impact. Measurable improvement in response time and conversion.

### Week 2: Follow-Up Prompts
Add systems to prevent forgotten follow-ups:
- Stale deal detection
- Follow-up reminders
- Activity-triggered next steps

**Why**: Reduces lost opportunities. Reps appreciate the helpful nudges.

### Week 3: CRM Hygiene
Improve data quality:
- Contact enrichment
- Duplicate detection
- Missing field alerts

**Why**: Better data = more effective outreach. Compound benefits over time.

### Week 4: Reporting
Add visibility for managers:
- Pipeline snapshots
- Activity summaries
- Deal health scores

**Why**: Leadership sees the value. Proves ROI of automation efforts.

### Ongoing: Expand and Refine
Listen to rep feedback. Adjust notification frequency. Add automations for specific pain points.

## Measuring Success

For sales automation, track:

**Speed metrics**:
- Lead response time (target: under 5 minutes)
- Time from lead to first meeting (target: under 48 hours)
- Deal cycle length (opportunity created to closed/won)

**Activity metrics**:
- Calls/emails per rep per day (are they freed up to do more outreach?)
- Deals moving forward per week (velocity through pipeline)
- Follow-up compliance (% of deals with activity in last 7 days)

**Quality metrics**:
- CRM data completeness (% of contacts with email, phone, company)
- Duplicate rate (should decrease over time)
- Win rate (better data and follow-through should improve this)

**Rep satisfaction**:
- Are reps using CRM more consistently? (login frequency, data entered)
- Fewer complaints about "CRM is a pain"
- Positive feedback on automation helpfulness

## Risk Considerations for Sales

Sales automation has specific risks to watch:

**Over-automation of human relationships**: Don't automate the relationship-building. Automate the mechanics around it. A prospect should never feel like they're talking to a robot.

**Notification fatigue**: Too many reminders turn into noise that reps ignore. Start conservative, increase only if needed.

**Data privacy**: Sales touches personal contact info. Ensure compliance with GDPR, CCPA, and industry regulations. Be extra careful with enrichment data sources.

**Premature deal death**: Auto-closing stale deals might kill opportunities that just need patience. Always have human review before destructive actions.

**Gaming the system**: If reps are measured on activity, they might log fake calls to satisfy automation. Focus on outcomes, not just activities.

## What's in Each Subsection

### Lead Handling and Routing (10+ automations)
Getting new leads assigned instantly to the right rep, with full context, mobile notifications, and SLA tracking. Focus: speed and appropriate assignment.

### Follow-Ups and Nurturing (10+ automations)
Preventing deals from stalling, prompting reps to follow up at the right time, automating sequences for common scenarios. Focus: persistence without being pushy.

### CRM Hygiene and Updates (5-8 automations)
Keeping contact data clean, complete, and current. Auto-logging activities. Detecting and fixing data issues. Focus: quality without manual work.

## Getting Started

If you're new to sales automation:

1. Start with **Instant Lead Routing** (get leads to reps fast)
2. Add **Stale Deal Detection** (prevent forgotten follow-ups)
3. Then **Contact Enrichment** (improve data quality)

These three create immediate value and are relatively low risk.

If you're experienced with automation:

1. Review all patterns for gaps in your current setup
2. Focus on the more complex patterns around deal health and predictive scoring
3. Look for ways to combine patterns into comprehensive workflows

## Cross-References

Many patterns in this section connect to:
- **10-marketing-and-agencies**: Where leads come from before sales takes over
- **40-finance-and-backoffice**: Where deals go after closing (invoicing, onboarding)
- **90-patterns-and-building-blocks**: Reusable components like human-in-the-loop and notification patterns

Sales doesn't operate in isolation. These automations often connect to marketing (lead handoff), operations (customer onboarding), and finance (deal closure).

## Sales-Specific Considerations

### For Solo Sellers and Consultants
You don't have "routing" challenges, but you still need follow-up discipline and CRM hygiene. Focus on:
- Follow-up reminders and sequences
- Data enrichment
- Activity logging

### For Small Sales Teams (2-5 people)
You need basic routing and coordination. Focus on:
- Round-robin lead assignment
- Shared pipeline visibility
- Preventing duplicate outreach

### For Larger Teams (10+ people)
You have territory, product, and seniority complexity. Focus on:
- Sophisticated routing rules
- Manager oversight and coaching tools
- Pipeline forecasting

Scale the complexity to your actual needs. Don't implement enterprise-grade automation if you're a team of three.
