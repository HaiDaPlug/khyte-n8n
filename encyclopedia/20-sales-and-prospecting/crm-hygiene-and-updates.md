# CRM Hygiene and Updates

This document contains 8 automation patterns for keeping CRM data clean, complete, and current without manual data entry. The focus is on data quality that happens automatically, not through rep discipline.

---

## 1. Email-to-CRM Activity Logging

Automatically log emails sent to and from prospects/customers into CRM without manual copying.

**Trigger**: Email sent or received involving contact in CRM

**Inputs**:
- Email system (Gmail, Outlook)
- CRM integration or BCC address
- Contact email addresses from CRM
- Email content and metadata

**Core Steps**:
1. Rep sends email to prospect (or receives email from them)
2. System detects email via:
   - **Option A**: CRM email integration (HubSpot, Salesforce inbox)
   - **Option B**: BCC automation address on outgoing emails
   - **Option C**: AI email scanner (with permission)
3. Match email address to contact/lead in CRM
4. If match found, create activity record:
   - Email subject
   - Timestamp
   - Direction (sent or received)
   - Snippet or full content (depending on privacy settings)
5. Attach to contact and related deal (if applicable)
6. Use AI to categorize email:
   - Proposal sent
   - Follow-up
   - Meeting request
   - Question/answer
7. Extract action items or next steps mentioned in email
8. Create reminder if rep promised follow-up

**Outputs**:
- Complete email history in CRM
- No manual logging required
- Full context for any contact
- Searchable email archive

**Time Saved**: 2-3 minutes per email. At 20 emails/day = 40-60 min saved.

**Value Beyond Time**: Complete activity history. New reps can review past conversations. Managers can coach based on actual emails.

**Risks and Caveats**:
- Privacy: Be careful about logging sensitive emails
- Email volume: High-volume users might have thousands of logs (need filtering)
- False matches: Common names or personal emails might log to wrong contact
- BCC method requires rep discipline (remember to BCC)

**Variants**:
- **Selective**: Only log emails with specific tag or BCC address (rep chooses what to log)
- **AI-powered**: Scan all emails, intelligently decide what's worth logging
- **Draft review**: Create draft activity logs, rep approves before saving

---

## 2. Calendar Meeting Auto-Logging

Automatically create CRM activity when meetings are scheduled or completed with contacts.

**Trigger**: Calendar meeting scheduled or completed involving CRM contact

**Inputs**:
- Calendar system (Google Calendar, Outlook)
- Meeting attendees
- CRM contact email addresses
- Meeting details (title, time, duration, notes)

**Core Steps**:
1. Meeting scheduled in calendar with prospect/customer email
2. System matches attendee email to CRM contact
3. When match found, create CRM activity:
   - Meeting type (discovery call, demo, check-in)
   - Date/time scheduled
   - Duration
   - Attendees list
   - Meeting notes or agenda (if in calendar)
4. Link to related deal
5. When meeting time arrives, send reminder to log outcome
6. After meeting completes:
   - Prompt rep: "Your meeting with [contact] ended. Add notes?"
   - Simple form: Outcome, next steps, date for follow-up
7. Update CRM activity with outcome notes
8. Create next action reminder based on outcome

**Outputs**:
- All meetings logged in CRM automatically
- Meeting history visible at contact/deal level
- Outcome tracking with gentle prompts
- Follow-up reminders created

**Time Saved**: 3-5 minutes per meeting. At 5 meetings/week = 15-25 min saved.

**Value Beyond Time**: Complete meeting history. Transition between reps is smoother (full context available).

**Risks and Caveats**:
- Internal meetings might get logged (filter by external attendee domains)
- Outcome prompt might be ignored (follow up if no notes added)
- Calendar integration permissions needed
- Time zone confusion for global teams

**Variants**:
- **Recording integration**: Link call recordings to CRM activity (Zoom, Gong)
- **AI notes**: Auto-generate meeting summary from recording
- **Selective**: Only log meetings with specific tags or calendar categories

---

## 3. Contact Data Enrichment and Updates

Automatically enrich contact records with updated job titles, company info, and social profiles when data changes.

**Trigger**: Scheduled monthly scan or when contact email domain changes

**Inputs**:
- CRM contact records
- Enrichment APIs (Clearbit, Apollo, LinkedIn)
- Change detection (previous vs current data)

**Core Steps**:
1. Select contacts for enrichment:
   - New contacts (created in last 30 days)
   - Contacts with incomplete data
   - Contacts not enriched in 6+ months
2. Call enrichment API with email or LinkedIn URL
3. Receive updated data:
   - Current job title
   - Company (in case they changed jobs)
   - Location
   - LinkedIn profile
   - Phone number (if available)
4. Compare to existing CRM data
5. If changes detected:
   - Job title changed → Update and flag rep ("John Smith is now VP at NewCompany!")
   - Company changed → Update company association, create note
   - Other updates → Append to contact record
6. Log enrichment timestamp
7. For significant changes (job change), create task for rep to reach out

**Outputs**:
- Current, accurate contact data
- Job change alerts (sales opportunity)
- Reduced manual data entry and research
- Higher connect rates (correct info = successful outreach)

**Time Saved**: 5-10 minutes per contact enriched. At 20 contacts/month = 1.5-3 hours saved.

**Value Beyond Time**: Sales opportunities from job change alerts. Better data = better targeting.

**Risks and Caveats**:
- Enrichment APIs cost money (budget accordingly)
- Data might be outdated or wrong (verify important changes)
- Over-writing manually entered data can cause issues (compare before replacing)
- Rate limits on free tiers (scale based on volume needs)

**Variants**:
- **On-demand**: Rep triggers enrichment when needed, not automatic
- **New contacts only**: Enrich at creation, don't re-enrich existing
- **Job change monitoring**: Focus only on detecting job changes, not full enrichment

---

## 4. Duplicate Contact Detection and Merging

Automatically detect duplicate contact records and either auto-merge or flag for manual review.

**Trigger**: New contact created or scheduled weekly scan

**Inputs**:
- All contact records in CRM
- Matching criteria (email, phone, first+last name+company)
- Merge rules (which record takes precedence)

**Core Steps**:
1. When new contact created, search for potential duplicates:
   - Exact email match
   - Phone number match (normalized)
   - Fuzzy name match + same company
2. If exact match found (same email), check data completeness:
   - Keep record with more complete data
   - Merge activities and notes from both
   - Mark one as master, archive other
3. If fuzzy match (similar names), flag for manual review:
   - Send to data admin: "Possible duplicate: John Smith at Acme vs Jon Smith at Acme Corp"
   - Provide merge or dismiss options
4. For auto-merged records:
   - Consolidate all activities
   - Keep most recent data
   - Log merge action
5. Notify record owner of merge
6. Weekly: scan full database for duplicates missed at creation

**Outputs**:
- Cleaner database with fewer duplicates
- No split activity history across multiple records
- Less confusion about which record is current
- Reduced wasted effort on duplicate outreach

**Time Saved**: 1-2 hours per week (vs manual deduplication).

**Value Beyond Time**: Better data quality. Prevent embarrassing duplicate outreach to same person.

**Risks and Caveats**:
- Auto-merge can be destructive if logic is wrong (conservative approach)
- Common names create false positives ("John Smith" at different companies)
- People change jobs → old and new contact records might both be valid
- Merging loses some history if not carefully designed

**Variants**:
- **Manual-only**: Flag all potential duplicates, human reviews all before merging
- **Email-only**: Only auto-merge on exact email match, everything else manual
- **Preventive**: Block duplicate creation at source (warn user before saving)

---

## 5. Missing Data Alert and Prompts

Alert reps when critical fields are missing on contacts or deals and prompt them to fill in.

**Trigger**: Contact or deal updated/created with missing critical fields, or weekly scan

**Inputs**:
- Contact/deal records
- Required fields by stage or type
- Data quality rules

**Core Steps**:
1. Define critical fields by context:
   - New contact: Email, company, title (minimum)
   - Qualified lead: Phone, decision-maker status
   - Opportunity: Deal size, close date, next steps
2. When record is created/updated, check for missing fields
3. If critical fields missing:
   - Send gentle prompt to rep: "Contact [name] is missing phone number. Can you add it?"
   - Show prompt in CRM when rep opens record
   - Add to weekly "data cleanup" task list
4. Weekly scan: Generate report of incomplete records by rep
5. Gamify: Leaderboard of "most complete records" by rep
6. Block stage progression if critical fields missing (e.g., can't move to "Proposal" without deal size)

**Outputs**:
- More complete CRM data
- Reps prompted at point of need (not after the fact)
- Data quality improves over time
- Better reporting (no missing values)

**Time Saved**: Prevents later scrambling for missing info (30-60 min/week).

**Value Beyond Time**: Complete data enables better segmentation, reporting, and automation downstream.

**Risks and Caveats**:
- Too many prompts = ignored (prioritize truly critical fields)
- Blocking progression can frustrate reps (use sparingly)
- Some data might be genuinely unavailable (allow "not applicable" option)
- Manual data entry is still required (this just reminds, doesn't auto-fill)

**Variants**:
- **Soft prompts**: Suggest but don't require (gentle nudges)
- **Hard blocks**: Can't save record without required fields
- **Smart suggestions**: Use AI to suggest values based on context (title from email signature)

---

## 6. Bounced Email Detection and Flagging

Automatically detect bounced emails and flag contacts with invalid email addresses for cleanup.

**Trigger**: Email bounce notification from email system

**Inputs**:
- Email bounce events (hard bounce, soft bounce)
- Contact email addresses
- Bounce reason (invalid address, mailbox full, etc.)

**Core Steps**:
1. Email sent to contact bounces
2. Capture bounce type:
   - **Hard bounce**: Email address doesn't exist (permanent)
   - **Soft bounce**: Temporary issue (mailbox full, server down)
3. For hard bounces:
   - Flag contact record: "Email invalid"
   - Remove from active email campaigns
   - Create task for rep: "Find updated email for [contact]"
   - Optionally use enrichment API to find new email
4. For soft bounces:
   - Log bounce but don't flag (might be temporary)
   - If 3+ soft bounces in a row, flag for review
5. Weekly: Report contacts with invalid emails to data admin
6. Track bounce rate by source (identify sources providing bad data)

**Outputs**:
- No wasted emails to invalid addresses
- Improved deliverability (lower bounce rate = better sender reputation)
- Reps know which contacts need updated info
- Cleaner contact list

**Time Saved**: Prevents wasted outreach attempts (10-15 min/week).

**Value Beyond Time**: Better sender reputation. Higher email deliverability overall.

**Risks and Caveats**:
- Bounces can be false positives (temporary server issues)
- Flagging might not get resolved (rep ignores task)
- Some contacts are truly lost (no new email available)
- Bounce detection requires email integration

**Variants**:
- **Auto-enrichment**: Automatically try to find new email via API when bounce detected
- **Quarantine**: Move bounced contacts to separate list until fixed
- **Validation-first**: Validate emails before first send (prevent bounces entirely)

---

## 7. Deal Stage Progression Logic and Enforcement

Enforce required actions or data before deals can move to next stage.

**Trigger**: Rep attempts to move deal to new stage

**Inputs**:
- Deal stage change event
- Stage requirements (activities, fields, approval)
- Deal data

**Core Steps**:
1. Rep tries to move deal from "Discovery" to "Proposal"
2. System checks stage requirements:
   - Required activities: Has discovery call been logged?
   - Required fields: Is deal size entered? Close date set?
   - Required assets: Is proposal document attached?
3. If requirements met: Allow stage change
4. If requirements not met:
   - Block stage change
   - Show checklist: "Complete these before moving to Proposal:"
     - [ ] Log discovery call
     - [ ] Enter estimated deal size
     - [ ] Set expected close date
5. Track stage progression velocity
6. Alert if deal jumps stages (skipping steps)
7. For final stages (Closed Won/Lost), require:
   - Loss reason (if lost)
   - Deal size confirmation (if won)
   - Handoff tasks created (if won)

**Outputs**:
- Consistent, high-quality pipeline data
- Reps can't skip important steps
- Better forecasting (deals in "Proposal" actually have proposals)
- Audit trail of progression

**Time Saved**: Not about time saved—about data integrity and process compliance.

**Value Beyond Time**: Reliable pipeline reporting. Better stage-to-close rates. Process discipline.

**Risks and Caveats**:
- Too strict = frustration (reps circumvent system)
- Some deals don't fit standard process (allow exceptions)
- Requirements must be reasonable and truly necessary
- Initial setup takes thought (define good requirements)

**Variants**:
- **Soft enforcement**: Warn but allow progression anyway
- **Role-based**: Managers can override, reps cannot
- **Deal-size based**: Enterprise deals have stricter requirements

---

## 8. CRM Data Cleanup Campaigns

Run periodic campaigns to clean up old, stale, or incorrect data in CRM.

**Trigger**: Scheduled monthly or quarterly

**Inputs**:
- Full CRM database
- Cleanup rules (what constitutes "stale" or "bad" data)
- Rep assignments

**Core Steps**:
1. Identify data issues:
   - Contacts not touched in 2+ years
   - Deals in "Proposal" stage for 6+ months
   - Contacts missing critical fields
   - Outdated company information
   - Duplicate records
2. Segment by issue type and assign to reps:
   - "You have 15 contacts that haven't been contacted in 2 years. Review and archive if appropriate."
   - "You have 3 deals stalled in Proposal stage. Update status."
3. Provide easy bulk actions:
   - Archive inactive contacts
   - Mark deals as lost
   - Enrich missing data
   - Delete obvious junk
4. Set deadline for cleanup (e.g., 2 weeks)
5. Track completion rate
6. Auto-archive anything not reviewed by deadline
7. Generate "before and after" report showing data quality improvement

**Outputs**:
- Cleaner CRM database
- More usable, relevant data
- Faster CRM performance (less junk)
- Better reporting accuracy

**Time Saved**: Not directly time-saving (creates work), but prevents long-term data rot.

**Value Beyond Time**: Usable CRM. Reps don't wade through dead contacts. Reports reflect reality.

**Risks and Caveats**:
- Cleanup is work (reps might ignore)
- Auto-archiving might delete valuable old contacts
- Quarterly is frequent enough for most teams (monthly might be overkill)
- Needs management buy-in to prioritize

**Variants**:
- **Targeted**: Focus on one issue per campaign (duplicates this month, stale contacts next month)
- **Gamified**: Rewards for most improved data quality
- **Automated**: Auto-archive/delete based on rules, no human review (risky but efficient)

---

## Implementation Priorities

For CRM hygiene and updates, recommend this order:

1. **Email-to-CRM Activity Logging** (automation #1) - Foundation of automatic data capture
2. **Duplicate Contact Detection** (automation #4) - Prevent data quality degradation
3. **Missing Data Alert and Prompts** (automation #5) - Improve completeness
4. **Contact Data Enrichment** (automation #3) - Keep data current
5. Others based on specific data quality pain points

Start with automatic activity logging (emails + meetings), then tackle duplicates, then improve completeness through prompts and enrichment.
