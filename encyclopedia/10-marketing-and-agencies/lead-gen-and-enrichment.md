# Lead Generation and Enrichment

This document contains 11 automation patterns for capturing, enriching, qualifying, and routing leads from multiple sources. The focus is on speed (respond within minutes, not hours) and completeness (all leads in one system, fully enriched).

---

## 1. Multi-Channel Lead Capture Hub

Consolidate lead submissions from website forms, LinkedIn ads, Facebook lead forms, webinar signups, and manual entries into a single system with standardized fields.

**Trigger**: New submission in any connected lead source (webhook, API polling, email forward)

**Inputs**:
- Form submissions from various platforms (Typeform, Google Forms, Unbounce, etc.)
- LinkedIn Lead Gen Forms via API
- Facebook Lead Ads via API
- Manual entries forwarded to a dedicated email address
- Webinar platform attendee lists

**Core Steps**:
1. Detect source of lead (which form or platform)
2. Map fields to standardized schema (handle different field names like "email" vs "email_address")
3. Add source tag and timestamp
4. Check for duplicates by email address
5. If duplicate, append notes and update "last contacted" date
6. If new, create record in central database (Airtable, Google Sheets, or CRM)
7. Send confirmation to lead source (if appropriate)
8. Trigger enrichment workflow (see next automation)

**Outputs**:
- Standardized lead record in central system
- Slack notification to marketing team with source and key details
- Lead enters enrichment queue

**Time Saved**: 30-45 minutes per day (no manual checking of 5+ platforms)

**Value Beyond Time**: Leads no longer fall through cracks. Response time drops from hours to minutes.

**Risks and Caveats**:
- Duplicate detection by email only won't catch people who use different emails
- Some platforms have API rate limits (check quotas)
- Manual email forwards need consistent formatting (provide template)
- GDPR: Ensure you have consent to process data from each source

**Variants**:
- **Simplified**: Start with just your top 3 lead sources, add others later
- **Enhanced**: Add lead scoring calculation in step 4 based on source, company size, or form responses
- **Industry-specific**: For B2B, prioritize enrichment of company data; for B2C, prioritize contact preferences

---

## 2. Lead Enrichment Pipeline

Automatically enrich new leads with firmographic data (company size, industry, revenue), social profiles, and contact information using APIs or AI-powered research.

**Trigger**: New lead created in central database (continuation from capture hub)

**Inputs**:
- Email address (minimum required)
- Company name (if available)
- Any other data from initial capture

**Core Steps**:
1. Use email domain to identify company (extract from email address)
2. Call enrichment API (Clearbit, Apollo, Hunter, or similar) with email or domain
3. Receive data: company name, size, industry, revenue range, location, tech stack
4. For missing data, run secondary lookup (LinkedIn company page scrape or Google search)
5. Store enriched data in lead record with source attribution
6. Calculate lead score based on ICP match (ideal customer profile)
7. Flag for manual review if key data is missing
8. Trigger routing automation if lead score exceeds threshold

**Outputs**:
- Lead record updated with 5-15 additional data points
- Lead score calculated
- High-value leads flagged for immediate outreach

**Time Saved**: 5-10 minutes per lead (vs manual research). At 20 leads/week = 2-3 hours saved.

**Value Beyond Time**: More consistent data quality, better qualification, faster response to high-value leads.

**Risks and Caveats**:
- Enrichment APIs cost money (typically $0.10-0.50 per lookup)
- Data quality varies; some records will be incomplete or wrong
- Personal email addresses (Gmail, Outlook) won't enrich well
- Rate limits on APIs can slow processing during high-volume periods
- Privacy: Only enrich data you're legally allowed to collect

**Variants**:
- **Budget version**: Skip paid APIs, use free lookup tools and accept lower coverage
- **Enhanced**: Add AI-powered web research for leads where API returns no data (LLM searches LinkedIn, company website, recent news)
- **B2C version**: Focus on demographic and behavioral data instead of firmographics

---

## 3. Lead Routing by Criteria

Automatically assign leads to the right salesperson based on geography, company size, industry, or round-robin rotation to ensure fast, appropriate follow-up.

**Trigger**: New lead created with enrichment complete, or lead score updated above threshold

**Inputs**:
- Lead record with location, company size, industry
- Routing rules table (which rep handles which territories/segments)
- Rep availability status (optional: check calendar or status in CRM)

**Core Steps**:
1. Check lead location (country, state, city) and match to territory assignments
2. Check company size and industry for segment assignment
3. If multiple reps qualify, use round-robin or workload balancing
4. Check rep availability (if integrated with calendar or CRM status)
5. Assign lead to selected rep in CRM
6. Send notification to rep via Slack/email with lead details and suggested talking points
7. Set reminder for rep if no action taken within 2 hours
8. Log assignment in activity history

**Outputs**:
- Lead assigned in CRM with timestamp
- Rep notified with actionable information
- Follow-up reminder scheduled
- Dashboard updated with routing metrics

**Time Saved**: 5 minutes per lead (vs manual assignment). At 50 leads/week = 4 hours saved.

**Value Beyond Time**: Response time drops from hours/days to minutes. Leads get the right rep, not whoever happened to see them first.

**Risks and Caveats**:
- Routing rules need maintenance as territories change
- Reps may game the system (mark themselves unavailable to avoid tough leads)
- Complex rules can lead to edge cases where no rep matches (have default fallback)
- Notification fatigue if too aggressive (see notification patterns in section 90)

**Variants**:
- **Simple version**: Geography-only routing, no complex scoring
- **Enhanced**: Include lead score in routing (high-value leads go to senior reps)
- **Account-based**: Check if lead's company already has an assigned account manager

---

## 4. Lead Response Time Tracker

Monitor how long it takes reps to respond to new leads and alert managers when leads are approaching or exceeding SLA.

**Trigger**: Lead assigned to rep (continuation of routing automation)

**Inputs**:
- Lead assignment timestamp
- Rep assigned
- Target response SLA (e.g., 5 minutes, 1 hour, 4 hours)
- Lead priority/score

**Core Steps**:
1. Record lead assignment time
2. Start monitoring for first outbound activity (email sent, call logged, meeting booked)
3. At 50% of SLA time, send gentle reminder to rep (Slack DM)
4. At 90% of SLA time, send urgent reminder to rep and notify manager
5. When rep takes action, record actual response time
6. Calculate SLA compliance (met or missed)
7. Update rep's response time metrics in dashboard
8. Weekly rollup: send summary to sales manager

**Outputs**:
- Real-time alerts to reps and managers
- Response time logged for each lead
- Weekly summary report with SLA compliance by rep
- Dashboard showing average response times and trends

**Time Saved**: Eliminates manual follow-up and tracking (15-30 min/day for managers).

**Value Beyond Time**: Improved conversion from faster response. Accountability without micromanagement.

**Risks and Caveats**:
- Can create pressure/stress if SLAs are unrealistic
- Reps might log fake activity to stop notifications
- Doesn't measure quality of response, just speed
- Weekend/after-hours leads need different SLA rules

**Variants**:
- **Simplified**: Just track and report, no real-time alerts
- **Enhanced**: Integrate with call/email systems to auto-detect first contact
- **Gamified**: Leaderboard of fastest responders, monthly recognition

---

## 5. Duplicate Lead Detection and Merge

Identify duplicate leads entering the system (same person from different sources) and either auto-merge or flag for manual review.

**Trigger**: New lead created, or scheduled daily scan of all leads

**Inputs**:
- All lead records in database
- Matching criteria (email, phone, company + first/last name)
- Merge rules (which record takes precedence)

**Core Steps**:
1. When new lead enters, search existing records for matches on email (exact match)
2. If no email match, search on phone number normalization
3. If no phone match, fuzzy search on first name + last name + company name
4. If potential match found, compare creation dates and data completeness
5. Determine which record is "primary" (usually older or more complete)
6. If high-confidence match (email or phone exact), auto-merge: append notes, keep all tags, update "last seen" date
7. If medium-confidence match (name fuzzy), flag for manual review
8. Send notification of merge/flag to data owner

**Outputs**:
- Cleaner database with fewer duplicates
- Merged lead records with consolidated history
- Manual review queue for uncertain matches
- Weekly report on duplicates found and resolved

**Time Saved**: 1-2 hours per week (vs manual deduplication).

**Value Beyond Time**: Better data quality, prevents reps from contacting the same lead twice, more accurate reporting.

**Risks and Caveats**:
- Auto-merge can be destructive if logic is wrong (test extensively)
- Common names create false positives (two John Smiths at different companies)
- People change jobs; old company data might get merged incorrectly
- Merging loses some historical context if not carefully structured

**Variants**:
- **Conservative**: Flag all potential duplicates for manual review, no auto-merge
- **Aggressive**: Auto-merge on any reasonable match, trust the algorithm
- **Selective**: Auto-merge only on exact email match, manual review for everything else

---

## 6. Form Abandonment Recovery

Track users who start filling out a form but don't submit, and send follow-up email or notification to encourage completion.

**Trigger**: User interacts with form fields but doesn't submit within session (tracked via analytics or form platform)

**Inputs**:
- Partial form data (fields filled before abandonment)
- Email address (if captured in first field)
- Form URL and page context
- Timestamp of abandonment

**Core Steps**:
1. Detect form interaction without submission (via analytics event or form platform)
2. Capture any data entered before abandonment (if email was entered)
3. Wait 30-60 minutes to see if user returns and completes
4. If still incomplete, send follow-up email with unique link to pre-filled form
5. Email says: "We noticed you started our [X] form. Here's a link to finish—your info is saved."
6. Track click-through and completion rates
7. If completed after follow-up, tag lead as "recovered" for attribution
8. If no completion after 48 hours, stop following up

**Outputs**:
- Follow-up email sent to partial submitters
- Increased form completion rate (recovered leads)
- Dashboard showing abandonment rate and recovery rate

**Time Saved**: Minimal direct time savings (automation is small), but recovers leads that would otherwise be lost.

**Value Beyond Time**: 10-30% of form abandoners complete after reminder. Direct revenue impact.

**Risks and Caveats**:
- Email needs to be captured first (ask for it early in form)
- Can feel intrusive if too aggressive
- GDPR: User must have consented to contact (checkbox in form)
- Some abandonment is intentional (just browsing, not ready)
- Don't spam: only send one follow-up

**Variants**:
- **Multi-step**: Send reminder at 1 hour, then again at 24 hours if still incomplete
- **Exit-intent**: Show popup when user tries to leave page, offering help or incentive
- **SMS-based**: For mobile users, send SMS reminder if phone number was captured

---

## 7. Lead Source Attribution Tracking

Automatically track which marketing channels and campaigns are generating leads, and append attribution data to each lead for ROI analysis.

**Trigger**: New lead captured from any source

**Inputs**:
- Lead source field (from form or platform)
- UTM parameters from URL (if web form)
- Campaign IDs (from ad platforms)
- Referral data (where they came from before landing page)

**Core Steps**:
1. Extract UTM parameters from form submission (utm_source, utm_medium, utm_campaign, utm_content)
2. Map platform-specific campaign IDs to campaign names (e.g., Facebook Ad ID → campaign name)
3. Append full attribution chain to lead record (first touch, last touch, and all touches if available)
4. Store source channel (organic search, paid social, referral, direct, email)
5. Link to specific ad creative or content asset if available
6. Calculate cost-per-lead if spend data is integrated
7. Tag lead with campaign name for easy filtering and reporting

**Outputs**:
- Lead records with full attribution data
- Reports showing leads by source, campaign, and channel
- Cost-per-lead and ROI metrics by campaign
- Insight into which channels drive highest-quality leads (if combined with deal closure data)

**Time Saved**: 2-3 hours per week (vs manual tagging and spreadsheet analysis).

**Value Beyond Time**: Know where to spend marketing budget. Optimize campaigns based on actual lead quality, not just volume.

**Risks and Caveats**:
- UTM parameters must be consistently applied (requires discipline from marketing team)
- Cross-device tracking is limited (user clicks ad on mobile, submits form on desktop)
- Attribution models are imperfect (multi-touch attribution is complex)
- Some sources don't pass UTM data (organic search, direct traffic)

**Variants**:
- **Simple**: First-touch attribution only (where did they first hear about us)
- **Advanced**: Multi-touch attribution with weighted models (first touch 40%, last touch 40%, middle touches 20%)
- **Integrated**: Pull ad spend from platforms and calculate ROI automatically

---

## 8. Lead Qualification Questionnaire Automation

Send an automated questionnaire to new leads to gather qualification information (budget, timeline, decision-maker status) before routing to sales.

**Trigger**: New lead captured, or lead reaches certain stage in nurture sequence

**Inputs**:
- Lead contact information
- Questionnaire template (questions specific to your qualification criteria)
- Qualification scoring rubric

**Core Steps**:
1. Send email with link to short questionnaire (3-7 questions)
2. Questions cover BANT or similar framework: Budget, Authority, Need, Timeline
3. Collect responses in form platform
4. Score responses (e.g., "budget over $10k" = +3 points, "decision-maker" = +5 points)
5. Calculate total qualification score
6. If score exceeds threshold, route to sales immediately with summary
7. If score is marginal, add to nurture sequence
8. If score is low, mark as unqualified and add to long-term nurture
9. Log qualification status in CRM

**Outputs**:
- Qualified leads routed to sales with context
- Unqualified leads filtered out or nurtured longer
- Qualification score and responses logged in CRM
- Sales team gets pre-qualified leads with key info

**Time Saved**: 10-15 minutes per lead for sales (no discovery calls with unqualified leads).

**Value Beyond Time**: Higher close rate because sales focuses on qualified opportunities. Faster sales cycle.

**Risks and Caveats**:
- Response rate to questionnaire will be low (30-50% typical)
- Some qualified leads won't respond and will be deprioritized unfairly
- Questions must be carefully worded to not scare off prospects
- Can feel impersonal if not positioned right ("Help us serve you better")

**Variants**:
- **Conversational**: Use chatbot instead of form for more engaging experience
- **Integrated**: Ask questions during initial form submission, not as separate step
- **Phone-based**: For high-value leads, trigger call from sales to ask questions personally

---

## 9. Webinar Attendee Follow-Up Sequence

Automatically enroll webinar attendees in tailored follow-up sequences based on attendance and engagement (attended live, watched replay, didn't attend).

**Trigger**: Webinar ends, or replay is watched

**Inputs**:
- List of registrants with attendance status (attended, no-show)
- Engagement data (poll responses, questions asked, watch duration)
- Webinar content and offers

**Core Steps**:
1. Pull attendee list from webinar platform (Zoom, WebinarJam, etc.) with attendance status
2. Segment attendees into groups:
   - Attended full session (90%+ watch time)
   - Partial attendance (30-90% watch time)
   - Registered but didn't attend
   - Watched replay later
3. For full attendees: Send thank-you email with slides, recording, and strong CTA (book demo, download resource)
4. For partial attendees: Send replay link and highlight what they missed
5. For no-shows: Send replay link with "Sorry we missed you" message
6. Track engagement with follow-up emails (opens, clicks)
7. Route highly engaged attendees to sales (clicked CTA, watched full replay)
8. Add others to appropriate nurture sequence

**Outputs**:
- Segmented follow-up emails sent within 2 hours of webinar ending
- Engaged leads routed to sales
- Replay viewers identified and nurtured
- Dashboard showing webinar ROI (registrations → attendees → MQLs → customers)

**Time Saved**: 1-2 hours per webinar (vs manual segmentation and email sending).

**Value Beyond Time**: Higher conversion from webinar leads. Personalized follow-up increases engagement.

**Risks and Caveats**:
- Webinar platforms vary in API quality (some don't provide engagement data)
- Timing matters: send too fast and recording isn't ready, too slow and interest fades
- Segmentation logic needs testing (what's the right watch time threshold?)
- Don't spam: limit follow-up sequence to 3-4 emails max

**Variants**:
- **Simple**: Single follow-up to all registrants, no segmentation
- **Enhanced**: Use AI to analyze questions asked and tailor follow-up based on topic interest
- **Multi-touch**: Combine email with LinkedIn connection requests and retargeting ads

---

## 10. Lead Import and Cleanup from List Purchases

Automate the cleanup, validation, and import of purchased or scraped lead lists to ensure data quality before adding to CRM.

**Trigger**: New spreadsheet uploaded to designated folder, or manual trigger

**Inputs**:
- Spreadsheet with lead data (names, titles, companies, emails, phones)
- Validation rules (email format, required fields, company domain checks)
- Existing CRM data for duplicate checking

**Core Steps**:
1. Ingest spreadsheet and parse data
2. Validate email addresses (format check, catch-all domain detection, MX record verification)
3. Validate phone numbers (format, country code)
4. Enrich missing data (company name from email domain, title standardization)
5. Check for duplicates against existing CRM records
6. Flag suspicious entries (role-based emails like info@, generic names, incomplete data)
7. Segment into: Ready to import, Needs manual review, Reject
8. Send "ready to import" batch to CRM with source tag
9. Create review spreadsheet for "needs manual review"
10. Log rejected records with reasons

**Outputs**:
- Clean, validated leads imported to CRM
- Review queue for uncertain records
- Rejection log for quality analysis
- Data quality report (X% valid, Y% duplicates, Z% rejected)

**Time Saved**: 2-4 hours per list (vs manual review and cleanup).

**Value Beyond Time**: Better data quality = higher deliverability and response rates. Fewer wasted sales efforts on bad data.

**Risks and Caveats**:
- Purchased lists are often low quality (expect 30-50% invalid or duplicate)
- Validation isn't perfect (some emails will bounce even after validation)
- GDPR/CCPA: Purchased lists may not have proper consent (legal risk)
- Over-reliance on lists instead of inbound leads can hurt brand
- Some validation APIs have costs (especially phone validation)

**Variants**:
- **Manual-first**: Auto-validation only, human reviews all before import
- **Enrichment-heavy**: Use paid APIs to fill in missing data (titles, company info, social profiles)
- **Compliance-focused**: Add consent re-confirmation step via email before adding to active marketing lists

---

## 11. Event Registration to CRM Sync

Automatically sync event registrations (trade shows, conferences, local meetups) to CRM with event-specific tags and follow-up tasks.

**Trigger**: New registration in event platform (Eventbrite, Luma, etc.) or manual CSV upload

**Inputs**:
- Event registration data (name, email, company, ticket type)
- Event details (name, date, location, your booth number)
- Registration timestamp and source

**Core Steps**:
1. Capture registration data from event platform via webhook or API
2. Map registration fields to CRM fields
3. Check for existing contact in CRM (by email)
4. If exists, append event registration to contact history and add event tag
5. If new, create contact record with event as source
6. Tag with event name, date, and ticket type (VIP, general admission, etc.)
7. Create follow-up task for assigned rep: "Contact within 3 days after event"
8. Add to event-specific nurture sequence (pre-event: logistics, post-event: recap and offer)
9. For VIP registrants or high-value companies, notify sales immediately

**Outputs**:
- All registrants in CRM with event context
- Follow-up tasks created for sales team
- Pre-event and post-event nurture emails scheduled
- Dashboard showing registrations and engagement by event

**Time Saved**: 30-60 minutes per event (vs manual CSV import and tagging).

**Value Beyond Time**: No leads lost. Timely follow-up after events (when memory is fresh). Better event ROI tracking.

**Risks and Caveats**:
- Event platforms have varying API quality (some require manual exports)
- Last-minute registrations might not sync in time for pre-event outreach
- Free events generate low-quality leads (lots of registrations, few serious buyers)
- Post-event follow-up timing is critical (within 3 days is ideal, week+ is too late)

**Variants**:
- **Badge scan integration**: Sync badge scans from your event booth, not just registrations
- **Enhanced**: Use AI to analyze registration questions/survey responses and prioritize leads
- **Multi-event**: Track which contacts attend multiple events over time (high intent signal)

---

## Implementation Priorities

For lead gen and enrichment, recommend this order:

1. **Multi-Channel Lead Capture Hub** (automation #1) - Foundation for everything else
2. **Lead Routing by Criteria** (automation #3) - Immediate impact on response time
3. **Lead Enrichment Pipeline** (automation #2) - Better qualification after routing is working
4. **Lead Source Attribution Tracking** (automation #7) - Know what's working
5. Others as needed based on your specific lead sources and pain points

Start with the hub and routing. Those two alone will transform lead handling.
