# Lead Handling and Routing

This document contains 10 automation patterns for capturing leads, routing them to the right sales rep, tracking response times, and ensuring nothing falls through the cracks. The focus is on speed (under 5 minutes from lead to first contact) and appropriate assignment (right rep for each opportunity).

---

## 1. Instant Lead Routing with Mobile Push

Route new leads to assigned sales rep within seconds with mobile push notification containing key lead details and immediate action options.

**Trigger**: New lead created in CRM (from any source: form, marketing automation, manual entry)

**Inputs**:
- Lead data (name, company, email, phone, source, score)
- Routing rules (territory, product, round-robin)
- Rep availability and contact preferences
- Mobile push integration (Slack, dedicated app, SMS)

**Core Steps**:
1. New lead enters CRM
2. Apply routing logic (see "Lead Routing by Criteria" in marketing section for details)
3. Assign to appropriate rep
4. Send immediate mobile push notification with:
   - Lead name and company
   - Lead score and source
   - Key qualifying details (budget, timeline, pain point)
   - Quick actions: "Call Now" (click-to-dial), "Email" (draft template), "Snooze 30min"
5. Log notification delivery time
6. Start response time clock
7. If no action within 5 minutes, send reminder
8. If no action within 15 minutes, escalate to manager
9. Log first contact time when action taken

**Outputs**:
- Rep notified instantly (even if away from desk)
- One-tap action reduces friction to respond
- Response time tracked automatically
- Escalation ensures no lead ignored

**Time Saved**: Not about time saved—about speed gained (minutes vs hours).

**Value Beyond Time**: 5-minute response converts 2-4x better than 30-minute response. This automation makes 5-minute response the norm, not the exception.

**Risks and Caveats**:
- Push notifications can be disruptive (allow reps to set "focus time" when no pushes)
- Mobile actions require good click-to-dial and email integration
- After-hours leads need different handling (queue for morning vs push at midnight)
- Over-reliance on mobile can reduce thoughtful, researched outreach

**Variants**:
- **SMS fallback**: If push not delivered within 60 seconds, send SMS
- **Team escalation**: If assigned rep unavailable, auto-reassign to next available rep
- **VIP fast-track**: High-score leads get phone call trigger, not just push

---

## 2. Lead Qualification Scoring Automation

Automatically score new leads based on firmographic data, behavioral signals, and fit criteria to prioritize sales effort.

**Trigger**: New lead created or existing lead updated

**Inputs**:
- Lead data (title, company size, industry, location)
- Behavioral data (website pages visited, content downloaded, email engagement)
- Fit criteria (ideal customer profile definition)
- Scoring rules and weights

**Core Steps**:
1. Collect lead data from CRM and enrichment sources
2. Score demographic fit:
   - Company size (1-10 employees = 0, 11-50 = 5, 51-200 = 8, 200+ = 10)
   - Industry match (target industry = 10, adjacent = 5, other = 0)
   - Job title (decision-maker = 10, influencer = 7, end-user = 3)
   - Geography (primary market = 8, secondary = 5, other = 2)
3. Score behavioral engagement:
   - Pricing page visit = +15
   - Case study download = +10
   - Multiple email opens = +5
   - Webinar attendance = +12
4. Calculate total score (0-100 scale)
5. Assign category:
   - Hot lead (80-100): Immediate contact, assign to senior rep
   - Warm lead (50-79): Contact within 24 hours
   - Cool lead (20-49): Nurture sequence, contact when engagement increases
   - Cold lead (0-19): Long-term nurture only
6. Update lead record with score and category
7. Trigger appropriate workflow for each category
8. Re-score when new behavioral data arrives

**Outputs**:
- Every lead has score and priority
- Reps focus effort on highest-potential opportunities
- Marketing knows which lead sources drive quality
- Automated routing uses score for assignment

**Time Saved**: 5-10 minutes per lead (vs manual qualification research).

**Value Beyond Time**: Better win rates from focus on qualified leads. Less time wasted on poor-fit prospects.

**Risks and Caveats**:
- Scoring model needs calibration (test and refine based on actual conversion data)
- Over-reliance on score can miss unconventional but valuable opportunities
- Behavioral data might reflect researcher not decision-maker
- Score decay over time (hot lead that goes cold should be re-scored)

**Variants**:
- **Predictive**: Use machine learning on historical win/loss data to predict likelihood to close
- **Simple**: Just demographic fit, skip behavioral scoring
- **Real-time adjustment**: Score changes dynamically as lead engages

---

## 3. Round-Robin Lead Distribution

Distribute leads evenly among sales team using round-robin rotation to prevent assignment disputes and ensure fair opportunity allocation.

**Trigger**: New lead enters system without territory-based assignment rule

**Inputs**:
- List of eligible reps and their status (active, on PTO, at capacity)
- Current rotation position
- Rep performance data (optional: weighted distribution)

**Core Steps**:
1. New lead needs assignment
2. Check if any territory/specialty rules apply (if yes, use those instead)
3. If no specific rules, use round-robin:
   - Query current position in rotation
   - Check next rep's status (active and available?)
   - If available, assign lead to that rep
   - If unavailable (PTO, sick, over capacity), skip to next in rotation
   - Update rotation position
4. Log assignment with timestamp and reason
5. Notify assigned rep
6. Track assignments to ensure balance (alert if one rep getting disproportionate share)
7. Reset rotation periodically (daily, weekly) to prevent long-term imbalances

**Outputs**:
- Fair lead distribution across team
- No arguments about who gets which lead
- Visibility into assignment patterns
- Automatic skip-over for unavailable reps

**Time Saved**: Eliminates manual assignment decisions (5-10 min/day for managers).

**Value Beyond Time**: Team morale (fair distribution). Manager time freed for coaching.

**Risks and Caveats**:
- True random rotation might not account for rep skill levels (senior vs junior)
- Some reps might work leads faster than others (same volume ≠ fair if close rates differ)
- Doesn't account for existing relationships (new lead from existing customer should go to account owner)
- Weekend/holiday leads need queue management (don't assign at midnight)

**Variants**:
- **Weighted**: Give more leads to top performers or full-time vs part-time reps
- **Skill-based**: Rotate within skill tier (senior reps get complex deals, juniors get simpler)
- **Workload-based**: Assign to rep with fewest open opportunities, not strict rotation

---

## 4. Lead Response Time Tracking and Leaderboard

Track how quickly each rep responds to assigned leads and display on leaderboard to encourage speed and accountability.

**Trigger**: Lead assigned to rep (continuation from routing automation)

**Inputs**:
- Lead assignment timestamp
- First contact activity (call, email, meeting booked)
- Rep assignment
- Target SLA (e.g., 5 minutes)

**Core Steps**:
1. Record lead assignment time
2. Monitor for first outbound activity:
   - Call logged in CRM
   - Email sent to lead
   - Meeting scheduled
   - Any note added with "contacted" tag
3. When activity detected, calculate elapsed time (assignment to first contact)
4. Log response time in lead record
5. Update rep's metrics:
   - Average response time
   - % within SLA (under 5 min)
   - Fastest response this week
   - Slowest response this week
6. Update leaderboard (visible to team):
   - Rank by average response time
   - Highlight "Speed Champion" (fastest average)
   - Show trends (improving or declining)
7. Weekly email summary to team and manager
8. Flag chronic slow responders for coaching

**Outputs**:
- Friendly competition drives faster response
- Visibility into who's responsive vs who's lagging
- Data for coaching conversations
- Trend analysis (are we getting faster or slower?)

**Time Saved**: Eliminates manual tracking (30-60 min/week for managers).

**Value Beyond Time**: Cultural change toward speed. Improved conversion from faster contact.

**Risks and Caveats**:
- Can create pressure/stress if used punitively
- Reps might log fake activity to game the leaderboard
- Speed without quality can backfire (rushed calls that don't connect)
- After-hours and weekend leads need adjusted expectations
- Solo sellers don't need leaderboards (just personal tracking)

**Variants**:
- **Private**: Show reps only their own stats, not full leaderboard
- **Team average**: Focus on team average, not individual rankings
- **Quality-adjusted**: Weight by conversion rate (fast contact that converts > fast contact that doesn't)

---

## 5. Inbound Call Capture and Logging

Automatically log inbound calls from prospects and customers to CRM with recording link, duration, and auto-generated summary.

**Trigger**: Inbound call received on business line

**Inputs**:
- Call data from phone system (caller number, duration, recording)
- CRM contact data (match caller to contact record)
- AI transcription service

**Core Steps**:
1. Inbound call received and answered
2. Capture caller phone number
3. Look up number in CRM to find matching contact/lead
4. If match found, create activity log on that record
5. If no match, create new lead from phone number (prompt rep to fill in details after call)
6. Record call duration and timestamp
7. If recording available, upload to CRM and link to activity
8. Use AI to transcribe call and generate summary (optional)
9. Extract key points: reason for call, outcome, next steps
10. Update contact record with summary and next action
11. If call was missed, create task for rep to call back

**Outputs**:
- Complete call history in CRM without manual logging
- Searchable transcripts for reference
- Matched to correct contact automatically
- Missed calls become follow-up tasks

**Time Saved**: 2-3 minutes per call (vs manual logging). At 10 calls/day = 20-30 min saved.

**Value Beyond Time**: Complete activity history. New reps can review calls for context. Manager can review for quality.

**Risks and Caveats**:
- Call recording regulations vary by location (two-party consent in some states)
- Phone number matching fails if customer calls from different number
- AI transcription isn't perfect (technical jargon, accents, background noise)
- Recording storage costs money (audio files add up)
- Privacy: call recordings contain sensitive information

**Variants**:
- **Manual trigger**: Rep clicks "log call" button, automation fills in details
- **Selective recording**: Only record calls to specific numbers or departments
- **Integration-heavy**: Deep integration with phone system for screen-pop of caller info during ring

---

## 6. Lead Enrichment on Assignment

Automatically enrich assigned lead with additional data (company info, social profiles, recent news) and present to rep before first contact.

**Trigger**: Lead assigned to rep (before or during first contact attempt)

**Inputs**:
- Basic lead data (email, company name)
- Enrichment API access (Clearbit, Apollo, LinkedIn, etc.)
- News API access (Google News, company-specific sources)

**Core Steps**:
1. Lead assigned to rep
2. Extract email domain and company name
3. Call enrichment APIs to gather:
   - Company details (size, industry, revenue, HQ location, tech stack)
   - Contact details (verified email, phone, LinkedIn profile, title)
   - Company news (recent funding, leadership changes, expansion)
   - Social presence (LinkedIn, Twitter)
4. Use AI to summarize: "Acme Corp is a 50-person marketing agency in Boston. They recently hired a new CMO and are expanding their digital services. They use HubSpot and Slack."
5. Append all data to lead record in CRM
6. Format as "rep briefing" with:
   - Key facts at a glance
   - Talking points (congratulations on new CMO, saw you're expanding digital...)
   - Potential pain points (based on company stage and industry)
7. Send briefing to rep via Slack or email: "Your new lead is ready to contact. Here's what you need to know."
8. Flag if critical data is missing (can't find company, email bounces)

**Outputs**:
- Rep has full context before calling (no blind outreach)
- Personalized conversation starters
- Higher connect and conversion rates
- Professional, informed first impression

**Time Saved**: 10-15 minutes per lead (vs manual research).

**Value Beyond Time**: Better first conversations. Higher conversion from personalized outreach.

**Risks and Caveats**:
- Enrichment APIs cost money ($0.10-0.50 per lookup)
- Data might be outdated or wrong (always verify key claims)
- Over-reliance on data can make reps lazy (still need to think and customize)
- Personal emails (Gmail, Yahoo) won't enrich with company data

**Variants**:
- **Free version**: Use just LinkedIn and Google search, skip paid APIs
- **On-demand**: Rep triggers enrichment when needed, not automatic for every lead
- **Selective**: Only enrich leads above certain score threshold (save API costs)

---

## 7. Meeting Scheduling Link Auto-Send

Automatically send calendar scheduling link to leads after qualification, reducing back-and-forth email to find meeting times.

**Trigger**: Lead marked as "qualified" or reaches specific stage in CRM

**Inputs**:
- Lead contact information
- Rep's calendar availability (via Calendly, Cal.com, or similar)
- Email template with scheduling link
- Follow-up sequence if not scheduled

**Core Steps**:
1. Lead reaches qualified stage (manual or automated)
2. Generate or retrieve rep's personal scheduling link
3. Send personalized email:
   - "Thanks for your interest. I'd love to discuss how we can help. Please grab a time that works for you: [link]"
   - Include context about what the meeting will cover
   - Suggest optimal meeting length (15min intro call, 30min discovery, etc.)
4. Track email open and link clicks
5. When meeting scheduled, update CRM with meeting details
6. Send confirmation to rep and lead
7. If no meeting scheduled within 48 hours, send gentle reminder
8. If still no meeting after 7 days, alert rep for manual follow-up

**Outputs**:
- Meetings scheduled faster (1 email instead of 3-5 back-and-forth)
- Rep's calendar automatically blocked when prospect books
- Reduced scheduling friction
- Tracking of conversion from qualified → meeting

**Time Saved**: 10-15 minutes per lead (eliminating scheduling tennis).

**Value Beyond Time**: Higher meeting conversion (easy scheduling = more bookings). Better prospect experience.

**Risks and Caveats**:
- Impersonal feel (some prospects expect human interaction)
- Calendar tools sometimes double-book if not careful with integrations
- Rep needs to keep calendar current (garbage in = garbage out)
- Different cultures have different scheduling norms (some prefer phone call)

**Variants**:
- **Rep-triggered**: Rep manually sends link when appropriate, not automatic
- **Conditional**: High-value leads get personal outreach, others get auto-link
- **Multi-step**: Send link in second email after initial introduction

---

## 8. Lead Source Performance Tracking

Track which lead sources (Google Ads, referral, content download, etc.) produce the highest-quality opportunities and inform marketing budget allocation.

**Trigger**: Deal closed (won or lost) with lead source attribution

**Inputs**:
- Lead source from initial capture
- Deal outcome (won, lost, abandoned)
- Deal value and cycle length
- All leads and outcomes over time

**Core Steps**:
1. When deal reaches closed status, record outcome
2. Look up original lead source attribution
3. Update source metrics:
   - Total leads from this source
   - Leads that became opportunities (qualification rate)
   - Opportunities that closed (close rate)
   - Average deal size from this source
   - Average sales cycle length
   - Total revenue from this source
4. Calculate source quality score:
   - (Qualification rate × Close rate × Avg deal size) / Avg cost per lead
5. Rank sources by quality score
6. Generate monthly report showing:
   - Best sources (high quality, high volume)
   - Emerging sources (new, promising)
   - Underperforming sources (high cost, low quality)
   - Trends (improving or declining sources)
7. Share with marketing team for budget optimization
8. Alert if a previously good source quality declines significantly

**Outputs**:
- Know which lead sources drive actual revenue, not just volume
- Data-driven marketing budget decisions
- Close the loop from marketing to sales to revenue
- Identify declining source quality early

**Time Saved**: 2-3 hours per month (vs manual spreadsheet analysis).

**Value Beyond Time**: Optimized marketing spend. More budget to winners, less to losers. Higher ROI.

**Risks and Caveats**:
- Attribution is hard (especially multi-touch journeys)
- Small sample sizes lead to false conclusions (need meaningful volume)
- Long sales cycles delay feedback (B2B might take 6 months to know if source is good)
- Correlation ≠ causation (source change might coincide with other factors)

**Variants**:
- **Real-time dashboard**: Live view instead of monthly report
- **Multi-touch attribution**: Credit all touchpoints, not just first or last
- **Cost-integrated**: Pull ad spend automatically and calculate true ROI

---

## 9. Territory-Based Lead Assignment

Route leads to reps based on geographic territory, company size tier, or industry specialization.

**Trigger**: New lead created with location or firmographic data

**Inputs**:
- Lead location (country, state, city, zip code)
- Lead company data (size, industry, revenue)
- Territory assignment rules (which rep owns which areas)
- Rep specializations and capacity

**Core Steps**:
1. New lead enters system
2. Extract territory indicators:
   - Geographic (headquarters location, billing address)
   - Company size (employee count or revenue)
   - Industry vertical
3. Match to territory assignment table:
   - "California + 50-200 employees = Rep A"
   - "Northeast + manufacturing = Rep B"
   - "Enterprise (200+ employees) = Senior Rep C"
4. Check for conflicts or overlaps (lead matches multiple rules)
5. If conflict, use priority hierarchy (e.g., industry > geography > size)
6. If no match, use round-robin fallback
7. Check for existing relationship (this company already has an account owner?)
8. If existing relationship found, override and assign to account owner
9. Assign to selected rep and notify
10. Log assignment reason for auditability

**Outputs**:
- Leads go to reps with local knowledge or industry expertise
- Existing relationships preserved
- Clear ownership (no disputes)
- Scalable as team grows

**Time Saved**: Eliminates manual assignment decisions (10-20 min/day).

**Value Beyond Time**: Better rep-lead fit = higher conversion. Fewer territory disputes.

**Risks and Caveats**:
- Territory rules become complex as team grows (needs governance)
- Location data might be wrong (HQ vs where contact actually works)
- Remote/distributed companies don't fit traditional geo territories
- Frequent territory changes confuse customers (multiple reps reaching out)

**Variants**:
- **Hybrid**: Use territory for new leads, round-robin for inbound inquiries
- **Account-based**: Assign by named account list, not geography
- **Team-based**: Assign to pods or teams, not individuals

---

## 10. Lead Handoff from Marketing to Sales

Formalize the lead handoff process with data validation, context transfer, and acceptance confirmation.

**Trigger**: Marketing-qualified lead (MQL) reaches threshold for sales handoff

**Inputs**:
- Lead data from marketing automation
- Qualification criteria (score, activities, explicit request)
- Sales team routing rules
- Handoff checklist

**Core Steps**:
1. Marketing automation marks lead as MQL
2. Validate required data before handoff:
   - Email (verified)
   - Phone (if available)
   - Company name
   - Pain point or interest area
   - Source and campaign attribution
3. If data incomplete, hold in marketing and request additional info
4. If complete, prepare handoff package:
   - Lead summary (score, key attributes)
   - Engagement history (emails opened, content downloaded, pages visited)
   - Recent activity (last 7 days)
   - Suggested talking points
5. Create opportunity in sales CRM (or update lead record)
6. Assign to sales rep based on routing rules
7. Send handoff notification to rep with full context
8. Request acceptance: "Please confirm you've received this lead and will contact within 24 hours"
9. If rep doesn't accept within 24 hours, alert manager and consider reassignment
10. Track lead fate (contacted, qualified, disqualified, converted)

**Outputs**:
- Clean handoff with full context (no information loss)
- Sales knows why this lead is worth their time
- Accountability (rep must accept and commit to contact)
- Feedback loop to marketing (what happened to the leads we sent?)

**Time Saved**: 10-15 minutes per MQL handoff (vs ad-hoc process).

**Value Beyond Time**: Higher contact rates. Better alignment between marketing and sales. Fewer "this lead sucks" complaints.

**Risks and Caveats**:
- Acceptance requirement can feel bureaucratic (keep lightweight)
- Sales might reject leads marketing thinks are qualified (need alignment on MQL definition)
- Data validation might slow handoff (balance quality vs speed)
- Conflict when lead isn't contacted within SLA (need escalation path)

**Variants**:
- **Auto-accept**: Skip acceptance step, assume rep will handle
- **Two-tier**: Different process for hot leads (immediate) vs warm (can wait)
- **Reverse flow**: Sales can send leads back to marketing if not truly qualified

---

## Implementation Priorities

For lead handling and routing, recommend this order:

1. **Instant Lead Routing with Mobile Push** (automation #1) - Immediate impact on response time
2. **Round-Robin Lead Distribution** (automation #3) - Fair allocation, reduces manual work
3. **Lead Enrichment on Assignment** (automation #6) - Better conversations from first contact
4. **Lead Response Time Tracking** (automation #4) - Visibility and accountability
5. Others based on team size and complexity (territory rules, source tracking, etc.)

Start with speed (routing + push), then fairness (round-robin), then quality (enrichment), then accountability (tracking).
