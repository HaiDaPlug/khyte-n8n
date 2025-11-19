# Follow-Ups and Nurturing

This document contains 10 automation patterns for ensuring consistent follow-up, preventing deals from stalling, and nurturing prospects through the sales cycle. The focus is on persistence without being pushy and systems that prevent "I forgot to follow up" failures.

---

## 1. Stale Deal Detection and Alerts

Automatically identify deals that haven't had activity in X days and alert rep and manager.

**Trigger**: Scheduled daily check of all open opportunities

**Inputs**:
- All open deals in CRM with last activity date
- Stale thresholds by stage (e.g., "Proposal sent" stage stale after 7 days, "Discovery" stale after 14 days)
- Rep assignments

**Core Steps**:
1. Query all open opportunities
2. Calculate days since last activity (call, email, meeting, note)
3. Compare to stage-specific staleness threshold
4. Flag deals exceeding threshold
5. Categorize by severity:
   - Yellow (approaching stale): 80% of threshold
   - Orange (stale): Past threshold
   - Red (very stale): 2x threshold
6. Send daily digest to each rep: "You have 3 deals that need attention"
7. Include suggested actions: "Last contact was 14 days ago. Call to check status?"
8. Send weekly rollup to manager: "Team has 12 stale deals total"
9. Track whether rep takes action after alert

**Outputs**:
- No deals forgotten
- Gentle daily reminders keep deals moving
- Manager visibility into pipeline health
- Reduced deal abandonment

**Time Saved**: Eliminates manual pipeline review (30-60 min/week per rep).

**Value Beyond Time**: Higher win rates from consistent follow-up. Deals don't die from neglect.

**Risks and Caveats**:
- Alert fatigue if too many deals flagged (adjust thresholds)
- Some deals need patience, not prodding (long enterprise sales cycles)
- Reps might log fake activity to clear alerts (track quality, not just quantity)
- Different deal types have different natural rhythms

**Variants**:
- **Tiered**: Different thresholds for deal size (large deals get more patience)
- **Smart**: Use AI to predict if deal is stalled vs naturally paused
- **Automated outreach**: Send automated "just checking in" email instead of just alerting rep

---

## 2. Next-Step Reminder Automation

After every meaningful activity (call, meeting, proposal sent), automatically create reminder for next step.

**Trigger**: Activity logged in CRM (call, meeting, email)

**Inputs**:
- Activity type and notes
- Deal stage
- Standard follow-up timelines (call → 3 days, meeting → 24 hours, proposal → 7 days)
- Rep calendar

**Core Steps**:
1. Rep logs activity (call, meeting, demo, proposal sent)
2. System detects activity type
3. Use AI to extract promised next step from notes: "Customer said they'd review and get back by Friday"
4. If explicit next step found, create task/reminder for that date
5. If no explicit next step, use default timeline:
   - After discovery call → "Follow up in 3 days"
   - After demo → "Check in tomorrow"
   - After proposal sent → "Follow up in 5 days if no response"
6. Create calendar reminder or CRM task
7. Set due date based on timeline
8. When due date arrives, notify rep
9. If rep misses reminder, escalate after 2 days

**Outputs**:
- Automatic follow-up schedule for every deal
- Reps never forget to follow up
- Promises to prospects are kept
- Pipeline keeps moving

**Time Saved**: Eliminates manual reminder creation (2-3 min per activity).

**Value Beyond Time**: Professional follow-through. Deals move faster. Higher close rates.

**Risks and Caveats**:
- AI extraction of next steps isn't perfect (verify in notes)
- Rigid timelines don't fit every situation (rep can adjust manually)
- Too many reminders = noise (consolidate into daily digest)
- Doesn't replace human judgment about when to follow up

**Variants**:
- **AI-powered**: Use AI to analyze deal stage and history to suggest optimal follow-up timing
- **Multi-channel**: Create reminders for email, call, and LinkedIn message options
- **Escalating**: First reminder = email, second = call, third = manager involved

---

## 3. No-Response Follow-Up Sequence

Automatically send follow-up emails when prospect doesn't respond to initial outreach.

**Trigger**: Email sent to prospect with no response after X days

**Inputs**:
- Original email sent timestamp
- Prospect contact info
- Follow-up email templates (3-4 variants with different angles)
- Stop conditions (unsubscribe, response received)

**Core Steps**:
1. Rep sends initial outreach email (or proposal, or any important email)
2. System tracks email sent event
3. Monitor for response (reply email, meeting scheduled, call logged)
4. If no response after 3 days, send Follow-up #1:
   - "Just wanted to make sure this didn't get lost. Still interested in discussing [topic]?"
5. If no response after 7 days, send Follow-up #2:
   - Different angle: "Saw you downloaded our [resource]. Have questions I can answer?"
6. If no response after 14 days, send Follow-up #3 (breakup email):
   - "Haven't heard back, so I'll assume timing isn't right. Reaching out one last time..."
7. If response at any point, stop sequence and notify rep
8. If no response after sequence, mark lead as "not interested" and move to long-term nurture
9. Track sequence performance (which email gets most responses)

**Outputs**:
- Consistent follow-up without manual effort
- Recover leads who were interested but got busy
- Professional persistence without being annoying
- Clear end point (breakup email, not infinite follow-ups)

**Time Saved**: 5-10 minutes per prospect (vs manual follow-up tracking).

**Value Beyond Time**: 15-30% of non-responders reply after 2nd or 3rd touch. Direct revenue impact.

**Risks and Caveats**:
- Can feel automated (personalize with merge fields and vary templates)
- Some people are annoyed by follow-ups (respect unsubscribes immediately)
- Doesn't replace genuine relationship-building
- Can damage brand if too aggressive or generic

**Variants**:
- **Multi-channel**: Mix email, LinkedIn, phone calls in sequence
- **Value-add**: Each follow-up includes useful content, not just "checking in"
- **Manual review**: Send drafts to rep for review before auto-sending

---

## 4. Meeting No-Show Follow-Up

Automatically reach out when prospect doesn't show up for scheduled meeting.

**Trigger**: Calendar meeting time passes with no activity logged

**Inputs**:
- Scheduled meeting in calendar
- Attendee list
- Meeting link (Zoom, Teams, etc.)
- No-show template

**Core Steps**:
1. Meeting scheduled for specific time
2. System monitors for:
   - Meeting link opened
   - Call logged during meeting time
   - Email exchange during meeting window
3. If no activity detected 10 minutes after scheduled start, flag as potential no-show
4. Wait until scheduled end time to confirm (prospect might be late)
5. If confirmed no-show, send immediate email:
   - "I think we may have had our wires crossed—I was on the call at [time]. Hope everything is okay!"
   - Offer to reschedule: "Here's my calendar link for another time that works better"
6. Create task for rep to follow up via phone later that day
7. Send rescheduling reminder after 24 hours if no response
8. Track no-show rate by prospect (serial no-shows indicate low interest)

**Outputs**:
- Immediate, professional response to no-shows
- Recovery of genuinely interested prospects who forgot
- Time not wasted waiting on calls
- Data on no-show patterns

**Time Saved**: 5 minutes per no-show (immediate template send vs manual follow-up).

**Value Beyond Time**: Recover 30-50% of no-shows with immediate, graceful follow-up.

**Risks and Caveats**:
- Detection isn't perfect (meeting might have happened off-system)
- Can seem passive-aggressive if prospect had emergency
- Tone matters (graceful, not accusatory)
- Serial no-shows might not be worth chasing (qualify interest level)

**Variants**:
- **Pre-meeting reminder**: Send reminder 1 hour before to prevent no-shows
- **Multi-step**: Email + SMS if high-value meeting
- **Intelligent**: Check if prospect sent cancellation email before flagging no-show

---

## 5. Win-Loss Follow-Up Automation

Automatically send appropriate follow-up after deals close (won: onboarding kickoff; lost: feedback request and long-term nurture).

**Trigger**: Deal status updated to "Closed Won" or "Closed Lost"

**Inputs**:
- Deal outcome and details
- Contact information
- Win/loss templates
- Next steps workflow

**Core Steps for Closed Won**:
1. Deal marked as won
2. Send immediate congratulations to customer
3. Trigger onboarding workflow:
   - Introduction to customer success team
   - Onboarding kickoff meeting link
   - Welcome packet with next steps
4. Create handoff tasks for implementation team
5. Add customer to success management system
6. Tag in CRM as "new customer"

**Core Steps for Closed Lost**:
1. Deal marked as lost with reason (price, timing, chose competitor, etc.)
2. Send gracious follow-up:
   - "Thanks for considering us. Hope we can work together in the future."
   - Offer to stay in touch (newsletter, content sharing)
3. Request feedback (if appropriate):
   - "Would you mind sharing what led to your decision? Helps us improve."
4. Add to long-term nurture sequence (quarterly check-ins)
5. Set reminder for 6 months: "Check if circumstances have changed"
6. Tag with loss reason for analysis

**Outputs for Won**:
- Smooth handoff to delivery/customer success
- Customer feels valued and excited
- No drop-off between sale and onboarding

**Outputs for Lost**:
- Professional, graceful close
- Feedback for improvement
- Door open for future opportunities

**Time Saved**: 10-15 minutes per closed deal (vs manual handoff/follow-up).

**Value Beyond Time**: Better customer experience (won). Future pipeline from lost deals that reopen.

**Risks and Caveats**:
- Lost deal follow-up can feel tone-deaf if too soon or sales-y
- Feedback requests often go unanswered (don't expect high response rate)
- Won deals need real human touch, not just automated emails
- Handoff automation doesn't replace rep introducing customer personally

**Variants**:
- **Delayed**: Wait 1-2 weeks after loss before nurture outreach
- **Tiered**: Different follow-up based on deal size (enterprise vs SMB)
- **Referral request**: Ask won customers for referrals after 30-60 days

---

## 6. Proposal Follow-Up Sequence

Automate follow-up after proposal or quote sent to maintain momentum and answer questions.

**Trigger**: Proposal/quote document sent or deal moved to "Proposal Sent" stage

**Inputs**:
- Proposal sent date
- Prospect contact info
- Decision timeline (if known)
- Follow-up templates

**Core Steps**:
1. Proposal sent (document emailed or deal stage updated)
2. Track proposal open/view (if using trackable link)
3. Next day: "Making sure you received the proposal. Any immediate questions?"
4. Day 3: "Have you had a chance to review? Happy to walk through any sections."
5. Day 5: Check decision timeline
   - If decision date approaching, remind: "I know you're planning to decide by [date]. Can I provide any additional info?"
   - If no timeline, ask: "What's your timeline for making a decision? Want to be respectful of your process."
6. Track proposal status:
   - Opened (engaged)
   - Not opened (send reminder or call)
   - Partially viewed (specific pages = interest areas)
7. Adjust follow-up based on engagement
8. Create task for rep to call if no response after 7 days

**Outputs**:
- Consistent proposal follow-up
- Higher conversion from proposals
- Prospects don't forget about outstanding proposals
- Data on proposal engagement

**Time Saved**: 10 minutes per proposal (automated sequence vs manual tracking).

**Value Beyond Time**: 20-30% higher proposal-to-close rate from consistent follow-up.

**Risks and Caveats**:
- Too frequent = pushy (space out appropriately)
- Generic follow-up less effective than customized (use merge fields)
- Some prospects need time to review internally (don't rush)
- Proposal tracking can feel invasive (use judiciously)

**Variants**:
- **Value-based**: Each follow-up adds value (case study, FAQ, calculator)
- **Multi-stakeholder**: Different follow-up to different decision-makers
- **Tiered**: Fast track for small deals, patient approach for enterprise

---

## 7. Re-Engagement Campaign for Cold Leads

Automatically re-engage leads that went cold months ago with new angle or offer.

**Trigger**: Scheduled quarterly scan for leads marked "cold" or "lost" 6+ months ago

**Inputs**:
- Cold leads from CRM (status = cold, lost, or no activity in 6+ months)
- Re-engagement email templates
- New content or offers to share
- Opt-out management

**Core Steps**:
1. Identify leads that have been cold for 6-12 months
2. Segment by original interest or pain point
3. Craft re-engagement message:
   - "It's been a while since we last connected..."
   - Share something new: new product, new content, case study, industry insight
   - Low-pressure ask: "Would you be open to a quick catch-up call?"
4. Send re-engagement email
5. Track responses:
   - Interested (schedule call, move back to active)
   - Not now (leave in cold status)
   - Unsubscribe (respect and remove)
6. For engaged leads, update status and assign to rep
7. For non-responders, mark as "re-engagement attempted" and check again in 6 months

**Outputs**:
- Recover deals that might be ready now
- Use dormant lead database productively
- Cost-effective lead gen (cheaper than new leads)
- Clean list (remove uninterested)

**Time Saved**: 1-2 hours per campaign (vs manual list building and outreach).

**Value Beyond Time**: 5-10% of cold leads re-engage. Direct pipeline impact at low cost.

**Risks and Caveats**:
- Can annoy people who already said no (keep frequency low, once or twice per year max)
- Must respect unsubscribes immediately
- Generic re-engagement gets low response (personalize by segment)
- Timing matters (don't reach out during busy season for their industry)

**Variants**:
- **Event-triggered**: Re-engage when prospect's company has news (funding, expansion)
- **Content-led**: Share valuable content, not sales pitch
- **Account-based**: Target companies, not just individual leads (might be new decision-maker)

---

## 8. Birthday and Milestone Outreach

Automatically send personalized messages on customer birthdays, work anniversaries, or company milestones.

**Trigger**: Date-based (birthday, work anniversary, company founding date)

**Inputs**:
- Contact birthday or work anniversary (from LinkedIn or manual entry)
- Company founding date (from enrichment data)
- Personalization data (interests, previous conversations)
- Template library

**Core Steps**:
1. Monitor calendar for upcoming milestones (7 days out)
2. For each milestone, determine appropriate outreach:
   - Contact birthday: Personal note or small gift
   - Work anniversary: Congratulations message
   - Company anniversary: "Congrats on 10 years!" note
3. Generate personalized message (not generic template):
   - Reference previous interactions
   - Genuine good wishes (not sales-y)
   - Optional: Offer value (resource, introduction, gift)
4. Send via email or LinkedIn message
5. For key accounts, notify account rep to send personally (not automated)
6. Track engagement and relationship warmth
7. For prospects, this becomes a soft touchpoint (relationship building, not selling)

**Outputs**:
- Genuine relationship touchpoints
- Stay top of mind without being sales-y
- Differentiation (most reps don't do this)
- Warmer relationships = easier re-engagement later

**Time Saved**: 5 minutes per milestone (automated vs manual tracking).

**Value Beyond Time**: Relationship depth. People remember who sent thoughtful messages.

**Risks and Caveats**:
- Feels creepy if you don't have permission or obvious source for data
- Can feel automated (personalize beyond "Dear {FirstName}")
- Not appropriate for all relationships (casual prospects might find it odd)
- Birthday data is often not in CRM (manual collection needed)

**Variants**:
- **Gift automation**: Send small gift (coffee card, book) on milestones for key accounts
- **Customers-only**: Focus on existing customers, skip prospects
- **Quarterly check-in**: Use milestones as excuse to check in, not just "happy birthday"

---

## 9. Deal Velocity Monitoring and Acceleration

Track how long deals spend in each stage and alert when deals are moving too slowly.

**Trigger**: Continuous monitoring, alerts when thresholds exceeded

**Inputs**:
- Deal stage and entry timestamps
- Historical averages for stage duration
- Deal size and complexity
- Alert thresholds

**Core Steps**:
1. Track when deal enters each stage
2. Calculate time in current stage
3. Compare to benchmarks:
   - Average time in this stage (from historical data)
   - Expected time based on deal size
4. Flag slow-moving deals:
   - Yellow: 1.5x average time
   - Red: 2x average time
5. Use AI to suggest acceleration tactics:
   - "Schedule executive meeting"
   - "Send case study from similar company"
   - "Offer discount for faster decision"
6. Alert rep with suggestion
7. For critical deals, alert manager
8. Track whether suggested action is taken and if it helps

**Outputs**:
- Visibility into deal pacing
- Proactive intervention on stalling deals
- Shorter sales cycles
- Higher close rates

**Time Saved**: Eliminates manual pipeline analysis (1-2 hours/week).

**Value Beyond Time**: Faster deals = more revenue. Early warning system for problems.

**Risks and Caveats**:
- Benchmarks must be accurate (bad data = bad alerts)
- Some deals naturally move slower (enterprise, complex)
- Pressure to accelerate can backfire (prospects need time)
- Suggestions might not fit specific situation (human judgment needed)

**Variants**:
- **Predictive**: Use AI to predict likelihood of close based on velocity patterns
- **Stage-specific**: Different alerts for different bottleneck stages
- **Comparative**: Compare rep's deal velocity to team average

---

## 10. Automated Check-In Sequence for Long Sales Cycles

For B2B enterprise sales with 6-12 month cycles, automate periodic check-ins to stay top of mind.

**Trigger**: Deal in long-cycle stage (e.g., "Evaluating", "Committee Review") for 30+ days

**Inputs**:
- Deal stage and timeline
- Contact preferences and engagement history
- Content library (case studies, industry reports, product updates)
- Check-in schedule

**Core Steps**:
1. Deal enters long-cycle stage
2. Establish check-in cadence (every 2-4 weeks)
3. For each check-in, vary the approach:
   - Week 2: Share relevant case study
   - Week 4: Industry trend article or report
   - Week 6: Product update or new feature announcement
   - Week 8: "Just checking in, anything I can clarify?"
   - Week 10: Invitation to webinar or event
4. Personalize based on prospect's role and interests
5. Track engagement (opens, clicks, replies)
6. If high engagement, notify rep to reach out personally
7. If no engagement for 3+ check-ins, flag for manual review
8. When deal moves forward or closes, stop sequence

**Outputs**:
- Consistent presence without being annoying
- Prospects remember you when decision time comes
- Value-added touchpoints (not just "checking in")
- Pipeline doesn't go cold during long evaluations

**Time Saved**: 15-20 minutes per check-in per deal.

**Value Beyond Time**: Stay in consideration during long buying cycles. Top-of-mind when budget approved.

**Risks and Caveats**:
- Can feel like drip marketing (make it genuinely valuable)
- Too frequent = annoying, too rare = forgettable
- Generic content gets ignored (tailor to their industry/role)
- Some prospects want space (respect signals)

**Variants**:
- **Multi-threaded**: Check in with different stakeholders on different schedules
- **Event-triggered**: Check in when their company has news, not just on calendar
- **Executive involvement**: Alternate between rep check-ins and executive touches

---

## Implementation Priorities

For follow-ups and nurturing, recommend this order:

1. **Stale Deal Detection and Alerts** (automation #1) - Immediate impact, prevents forgotten deals
2. **Next-Step Reminder Automation** (automation #2) - Systematic follow-up discipline
3. **No-Response Follow-Up Sequence** (automation #3) - Recover non-responders
4. **Proposal Follow-Up Sequence** (automation #6) - Higher proposal conversion
5. Others based on sales cycle complexity and team habits

Start with preventing forgotten deals (stale detection + next-step reminders), then add systematic sequences (no-response + proposal).
