# Automation Scoring Model

## Purpose

When you're looking at 100+ potential automations, you need a way to decide which ones to implement first. This scoring model helps you evaluate and prioritize based on four factors: Impact, Frequency, Complexity, and Risk.

The goal isn't mathematical precision. It's structured thinking that prevents you from building the wrong automation just because it seems interesting.

## The Four Factors

### Impact (1-5 points)

**What it measures**: How much time, money, or quality improvement this automation delivers per run.

**Scoring guidance**:

- **5 points - Transformative**: Saves hours per run, prevents costly errors, or unlocks new revenue. Example: Automated proposal generation that saves 3 hours and increases win rate.

- **4 points - Significant**: Saves 30-60 minutes per run or eliminates a major pain point. Example: Lead routing that ensures no leads wait more than 5 minutes.

- **3 points - Moderate**: Saves 10-30 minutes per run or improves consistency. Example: Auto-tagging emails by client for easier searching.

- **2 points - Minor**: Saves 5-10 minutes or reduces minor annoyances. Example: Slack notification when someone fills out a form.

- **1 point - Minimal**: Saves a few minutes or provides slight convenience. Example: Auto-filing receipts you could file manually in 2 minutes.

**What to consider**:
- Direct time savings
- Error prevention value
- Opportunity cost (faster response = more deals won)
- Quality improvements
- Team morale impact (eliminating soul-crushing work)

**Red flags**:
- If you can't articulate the value in concrete terms, score it 1
- If it saves time but creates confusion, lower the score
- If it only helps one person rarely, probably 1-2

### Frequency (1-5 points)

**What it measures**: How often this automation runs.

**Scoring guidance**:

- **5 points - Constant**: Multiple times per day, every day. Example: Lead capture form submissions.

- **4 points - Daily**: At least once per business day. Example: Daily digest of new support tickets.

- **3 points - Weekly**: Once to several times per week. Example: Friday client status reports.

- **2 points - Monthly**: A few times per month. Example: Monthly invoice generation.

- **1 point - Rare**: Less than monthly or only during specific seasons. Example: Annual contract renewal reminders.

**What to consider**:
- Current frequency (not hoped-for future frequency)
- Seasonal variations
- Growth trajectory (if you're doubling clients quarterly, frequency will increase)

**Red flags**:
- Don't score based on how often you *wish* it would run
- Be honest about actual current volume
- If it's seasonal, use the busy-season frequency

**Frequency multiplier effect**: A 5-minute automation that runs 20 times per day (100 minutes saved) is more valuable than a 30-minute automation that runs once per month. Impact × Frequency gives you total time saved.

### Complexity (1-5 points, inverse scoring)

**What it measures**: How hard this automation is to build and maintain. Lower scores are better here.

**Scoring guidance**:

- **1 point - Trivial**: Connect two tools with a direct integration, no transformation needed. Example: New Typeform submission → add row to Google Sheets.

- **2 points - Simple**: Basic data transformation or one decision point. Example: New lead → check if email is valid → add to CRM or send error notification.

- **3 points - Moderate**: Multiple steps, some AI processing, or integration with 3+ tools. Example: New document → extract key data with AI → create project in PM tool → notify team in Slack.

- **4 points - Complex**: Custom logic, error handling for many edge cases, or requires new infrastructure. Example: Monitor competitor pricing across multiple sites → normalize data → compare to your pricing → flag anomalies → update dashboard.

- **5 points - Very Complex**: Requires custom code, multiple external services, or deep integration work. Example: Full document processing pipeline with OCR, classification, data extraction, validation, and multi-system updates.

**What to consider**:
- Number of tools involved
- Data transformation complexity
- Error handling requirements
- Need for custom code
- Availability of existing integrations
- Your team's skill level

**Red flags**:
- If you're not sure how you'd build it, it's probably 4-5
- If it requires tools you don't have, add 1 point
- If it needs ongoing manual tuning, add 1 point

### Risk (1-5 points, inverse scoring)

**What it measures**: What could go wrong and how bad it would be. Lower scores are better here.

**Scoring guidance**:

- **1 point - Negligible**: Failures are obvious and have no consequences. Example: Daily digest email fails → you notice immediately, no harm done.

- **2 points - Low**: Failures might not be noticed immediately but cause only minor issues. Example: Auto-tagging mistakes → slightly harder to find things, easily fixed.

- **3 points - Moderate**: Failures could cause customer confusion or wasted work. Example: Calendar invite automation sends to wrong people → awkward but recoverable.

- **4 points - High**: Failures could damage customer relationships or cost significant money. Example: Automated invoicing sends wrong amounts → angry clients, accounting headaches.

- **5 points - Critical**: Failures could cause legal issues, data breaches, or major financial loss. Example: Automated contract signing without human review → legal liability.

**What to consider**:
- Visibility of failures (will you notice?)
- Consequence of errors
- Reversibility (can you undo it?)
- Compliance requirements
- Customer-facing vs internal
- Financial impact

**Risk types**:
- **Data risk**: Wrong information sent to wrong people
- **Financial risk**: Incorrect invoices, pricing, or payments
- **Relationship risk**: Spam, inappropriate messages, missed opportunities
- **Compliance risk**: GDPR violations, industry regulations
- **Operational risk**: Automation fails and no one notices for days

**Mitigation strategies** (if you can add these, reduce risk score by 1):
- Human review step before critical actions
- Automatic rollback capability
- Monitoring and alerting for failures
- Sandbox testing environment
- Gradual rollout to subset of cases

## Calculating Priority Score

**Formula**: `(Impact × Frequency) - (Complexity + Risk)`

This gives you a score typically ranging from -8 to +23.

**Interpretation**:

- **15+**: Implement this immediately. High value, low downside.
- **10-14**: Strong candidates, prioritize by business need.
- **5-9**: Solid automations, implement after quick wins.
- **1-4**: Marginal value, only implement if you have specific reasons.
- **0 or negative**: Don't build this. The effort and risk outweigh the benefits.

## Worked Examples

### Example 1: Lead Routing from Website Form

- **Impact**: 4 (ensures 5-minute response time, increases conversion)
- **Frequency**: 5 (10-15 leads per day)
- **Complexity**: 2 (webhook → check criteria → assign to rep → notify)
- **Risk**: 2 (worst case: lead goes to wrong person, they'll forward it)

**Score**: (4 × 5) - (2 + 2) = 20 - 4 = **16**

**Decision**: High priority. Implement immediately.

### Example 2: Monthly Expense Report Compilation

- **Impact**: 3 (saves 30 minutes of copy-paste work)
- **Frequency**: 1 (once per month)
- **Complexity**: 3 (gather from multiple sources, format, send)
- **Risk**: 2 (errors are caught in review, no customer impact)

**Score**: (3 × 1) - (3 + 2) = 3 - 5 = **-2**

**Decision**: Don't automate. 30 minutes per month isn't worth the setup and maintenance. Do it manually.

### Example 3: AI-Powered Meeting Notes Summary

- **Impact**: 4 (saves 20 minutes, improves team alignment)
- **Frequency**: 4 (one meeting per day)
- **Complexity**: 3 (recording → transcription → AI summary → formatting → send)
- **Risk**: 2 (summaries might miss context, but it's for internal use)

**Score**: (4 × 4) - (3 + 2) = 16 - 5 = **11**

**Decision**: Strong candidate. Implement after critical routing/notification automations.

### Example 4: Automated Contract Generation with E-Signature

- **Impact**: 5 (saves 2 hours, speeds up deal closure)
- **Frequency**: 3 (2-3 contracts per week)
- **Complexity**: 4 (data gathering, template population, legal review logic, signature routing)
- **Risk**: 4 (wrong terms or missing clauses could be legally problematic)

**Score**: (5 × 3) - (4 + 4) = 15 - 8 = **7**

**Decision**: Moderate priority. High value but needs careful implementation with legal review. Start with draft-only version to reduce risk.

### Example 5: Daily Competitor Social Media Monitoring

- **Impact**: 2 (interesting intel, but rarely actionable immediately)
- **Frequency**: 5 (daily scan)
- **Complexity**: 4 (scraping, filtering, analysis, summarization)
- **Risk**: 1 (low risk, just informational)

**Score**: (2 × 5) - (4 + 1) = 10 - 5 = **5**

**Decision**: Marginal. Only build if you have spare capacity or specific competitive pressure. Consider a weekly manual review instead.

## Using the Model in Practice

### Step 1: Quick Assessment

When you read an automation pattern, do a rough mental scoring:
- "Does this save meaningful time?" (Impact)
- "How often would it run?" (Frequency)
- "Could I build this in a day?" (Complexity)
- "What's the worst-case failure?" (Risk)

This takes 30 seconds and filters out obvious nos.

### Step 2: Detailed Scoring (Top Candidates Only)

For automations that pass the quick filter:
1. Write down the four scores
2. Calculate the priority score
3. Note any special considerations

Don't score all 100+ automations in detail. Just the ones you're seriously considering.

### Step 3: Portfolio View

Look at your top 5-10 scored automations:
- Do they cluster in one area? (Maybe focus there first)
- Do you have a mix of quick wins (high score, low complexity) and strategic bets?
- Are you being realistic about complexity given your team's skills?

### Step 4: Adjust for Context

The model doesn't know your business. Adjust scores based on:

- **Current priorities**: If you're focused on client retention, boost Impact for automations that improve customer experience
- **Available resources**: If you have a developer for two weeks, you can tackle higher Complexity
- **Compliance requirements**: Some industries need higher standards (adjust Risk accordingly)
- **Team skill**: If your team is new to automation, lower Complexity tolerance

### Step 5: Review After Implementation

After building an automation, revisit your scores:
- Was the Impact what you expected?
- Was Complexity higher or lower than estimated?
- Did any Risks materialize?

This calibrates your scoring for future decisions.

## Common Scoring Mistakes

### Mistake 1: Overestimating Impact
**Symptom**: "This will save so much time!" but you can't name a specific hour amount.
**Fix**: Track actual time spent on the manual task for one week before scoring.

### Mistake 2: Underestimating Complexity
**Symptom**: "How hard can it be?" followed by three weeks of debugging.
**Fix**: If you're not sure how to build a step, assume Complexity is at least 4.

### Mistake 3: Ignoring Risk
**Symptom**: Building automations that send customer emails without review.
**Fix**: Anything customer-facing starts at Risk 3 minimum. Add review steps or adjust.

### Mistake 4: Not Considering Frequency Realistically
**Symptom**: Building for scale you don't have yet.
**Fix**: Score based on current frequency, not projected. Re-evaluate in 6 months.

### Mistake 5: Forgetting Maintenance
**Symptom**: Automation works for 3 months, then breaks when a vendor changes their API.
**Fix**: Add 1 to Complexity for automations that depend on external services you don't control.

## Scoring for Different Automation Types

### Quick Wins (High Impact × Frequency, Low Complexity + Risk)
Target these first. They build momentum and prove value. Examples:
- Simple notifications
- Basic data routing
- Scheduled reports

### Strategic Investments (High Impact, Lower Frequency, Moderate Complexity)
Worth building if you have capacity after quick wins. Examples:
- Monthly reporting dashboards
- Quarterly review compilations
- Contract generation

### Efficiency Plays (Moderate Everything)
Solid value, but not urgent. Build these to round out your automation portfolio. Examples:
- Auto-filing and tagging
- Data enrichment
- Quality checks

### Avoid (Low Impact × Frequency or Very High Complexity + Risk)
Be honest: some automations aren't worth it. Examples:
- Rarely-run complex transformations (do them manually)
- High-risk processes that need human judgment
- Automations that require tools you don't have

## Building Your Automation Roadmap

Use the scoring model to create a 90-day roadmap:

**Month 1: Quick Wins (Score 15+)**
- Build 3-5 simple, high-impact automations
- Focus on different business areas to spread the value
- Get people used to automation being helpful

**Month 2: Strategic Plays (Score 10-14)**
- Tackle 2-3 more complex automations
- Include at least one that required custom logic or AI
- Start seeing compounding effects

**Month 3: Polish and Expand (Score 5-9)**
- Add variants to successful automations
- Build supporting automations that leverage earlier work
- Document patterns for team use

**Ongoing: Maintain and Optimize**
- Review what's working monthly
- Deprecate automations that aren't delivering value
- Adjust scoring based on learned experience

## Final Thoughts on Scoring

This model is a tool for thinking, not a scientific instrument. Two people might score the same automation differently based on their context, and both could be right.

The value is in the structured evaluation. By forcing yourself to consider Impact, Frequency, Complexity, and Risk, you make better decisions than "this seems cool, let's build it."

Use the model as a starting point. Adjust it to your business. The best scoring model is the one you actually use.
