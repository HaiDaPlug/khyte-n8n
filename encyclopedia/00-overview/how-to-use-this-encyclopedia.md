# How to Use This Encyclopedia

## Navigation Strategy

This encyclopedia contains over 100 automation patterns organized by business function. You don't need to read it linearly. Instead, use one of these approaches:

### Approach 1: Pain-Point Driven

1. Identify your current biggest time sink or error source
2. Navigate to the relevant section (e.g., if leads get lost, check Sales → Lead Handling)
3. Scan the automation titles for something close to your situation
4. Read that pattern in detail
5. Adapt it to your specific tools and processes

**When to use this**: You have a specific problem and need a solution now.

### Approach 2: Domain Survey

1. Open the overview for your primary business function
2. Read through all automation titles in that section
3. Mark 3-5 that would have immediate impact
4. Score them using the Automation Scoring Model
5. Implement the highest-scoring automation first

**When to use this**: You know automation would help but aren't sure where to start.

### Approach 3: Tool-First

1. Decide which automation platform you'll use (n8n, Zapier, Make, etc.)
2. Scan the encyclopedia for patterns that match your tool's strengths
3. Look for patterns that connect tools you already use
4. Start with automations that require no new subscriptions

**When to use this**: You've already committed to a tool and want to maximize its value.

### Approach 4: Quick Wins Portfolio

1. Read only the "Time Saved" sections across multiple patterns
2. Find automations with high time savings and low complexity
3. Implement 5-10 simple automations in the first month
4. Use the momentum to tackle more complex patterns

**When to use this**: You need to demonstrate ROI quickly to get buy-in for bigger projects.

## Understanding Each Automation Pattern

Every automation in this encyclopedia follows a consistent structure:

### Name/Title
A clear, descriptive name that tells you what it does. No marketing fluff.

### Description
2-3 sentences explaining the business problem and how this automation solves it.

### Trigger
What event starts this automation running. Could be:
- Schedule (every Monday at 9am)
- Incoming data (new form submission, email received)
- Status change (deal moved to "Closed Won")
- Manual trigger (someone clicks a button)

Understanding triggers is critical. A badly chosen trigger causes spam, missed events, or unnecessary runs.

### Inputs
What data the automation needs to function:
- Where it comes from
- What format it's in
- What happens if data is missing

If you don't have these inputs available in a structured way, you'll need to create them first.

### Core Steps
4-10 bullets describing what the automation does between trigger and output. These typically include:
- Data retrieval or enrichment
- Transformation or calculation
- AI-powered analysis (where relevant)
- Decision points or branching logic
- Writing results somewhere

This is the "recipe" portion. You'll adapt these steps to your specific tools.

### Outputs
Where the results go:
- Updated records in a CRM
- Messages in Slack
- Rows in a spreadsheet
- Emails to specific people
- Files in a document system

Good outputs are obvious, timely, and actionable.

### Time Saved
An honest estimate of time saved per run and frequency. We try to be conservative here. If we say "15 minutes per week," it's probably accurate, not aspirational.

Also includes broader value beyond time: fewer errors, faster response time, better data quality.

### Risks and Caveats
What could go wrong:
- Data quality issues
- Integration limitations
- Edge cases that break the automation
- Compliance or privacy concerns
- Over-automation risks (losing human judgment)

Read this section carefully. Every automation has tradeoffs.

### Variants
2-3 adaptations of the core pattern for different situations:
- Simpler version for small teams
- More complex version for scale
- Industry-specific twists
- Alternative tools or approaches

If the main pattern doesn't fit exactly, check the variants.

## Adapting Patterns to Your Context

**No pattern will work exactly as written.** That's intentional. These are patterns, not scripts. Here's how to adapt them:

### Step 1: Map Your Current Process
Before automating, document your manual process:
1. What triggers you to start this task?
2. What information do you gather?
3. What decisions do you make along the way?
4. Where do the results go?
5. Who needs to know it's done?

### Step 2: Identify Deviations
Compare your process to the pattern:
- What steps don't apply to you?
- What steps are you missing?
- Where do your tools differ?
- What business rules are specific to you?

### Step 3: Simplify First
Your first version should be simpler than the documented pattern:
- Remove edge case handling
- Skip optional enrichments
- Use manual steps where automation is complex
- Limit scope (one team, one use case)

### Step 4: Test with Real Data
Run it with actual scenarios from last week:
- Did it trigger correctly?
- Did it handle the data properly?
- Did the output go to the right place?
- Would the result have been useful?

### Step 5: Iterate Based on Failures
Watch where it breaks:
- Add error handling for those cases
- Improve data validation
- Add notification for manual review
- Adjust trigger conditions

## Using This as a Consulting Tool

If you're a consultant or agency helping clients with automation:

### Discovery Phase
- Share the relevant section with your client
- Ask them to mark patterns that resonate
- Use their selections to understand pain points
- Estimate implementation effort for each

### Scoping Projects
- Use the scoring model to prioritize
- Group related automations into phases
- Set expectations based on "Time Saved" estimates
- Flag risks early

### Implementation
- Show clients the pattern you're following
- Explain deviations and why
- Use the structure to document what you build
- Reference variants for future enhancements

### Handover
- Give clients the relevant encyclopedia sections
- Mark which patterns you've implemented
- Highlight variants they could explore next
- Use this as ongoing reference documentation

## Common Mistakes to Avoid

### Mistake 1: Trying to Automate Everything at Once
**Fix**: Pick one pattern, implement it fully, let it run for two weeks, then pick another.

### Mistake 2: Automating a Broken Process
**Fix**: Fix the process first, then automate it. Automation makes bad processes fail faster.

### Mistake 3: Over-Engineering the First Version
**Fix**: Ship a simple version that works. Add complexity only when you hit real limitations.

### Mistake 4: Ignoring the "Risks" Section
**Fix**: Read risks first. Decide if you can live with them. Add mitigation steps if needed.

### Mistake 5: Setting Triggers Too Broadly
**Fix**: Start with a narrow trigger (one specific form, one tag, one status). Expand later.

### Mistake 6: No Human Review Step
**Fix**: For first implementations, add a "draft-only" step where humans review before sending.

### Mistake 7: Not Measuring Impact
**Fix**: Track one metric before and after. Even rough numbers help justify the next automation.

## Integration with Your Existing Stack

Most patterns assume you have:
- A way to trigger workflows (webhooks, schedules, watchers)
- A way to transform data (scripting, built-in functions, AI)
- A way to connect to your other tools (APIs, native integrations)

If you're using n8n, Zapier, Make, or similar platforms, you have all of this. If you're building custom scripts, you'll need to handle these yourself.

### Tool Translation Guide

When a pattern mentions a specific tool:

| Pattern Says | Your Alternative |
|-------------|------------------|
| "Send to Slack" | Teams, Discord, email, or any notification channel |
| "Update in Airtable" | Your database, spreadsheet, or CRM |
| "Run through AI summarizer" | Claude, GPT-4, or manual summary |
| "Store in Google Drive" | SharePoint, Dropbox, or any file system |
| "Add to ClickUp" | Your project management tool |

The *what* matters more than the *how*. Focus on the data flow and decisions, not the specific buttons to click.

## Ongoing Reference

Bookmark these sections for regular reference:

- **90-patterns-and-building-blocks**: Reusable components you'll use across many automations
- **triggers-and-events.md**: When you're deciding how to start an automation
- **human-in-the-loop-patterns.md**: When you need approval steps or safety checks
- **notification-patterns.md**: When people complain about spam or missing alerts

## Getting Help

If you're stuck implementing a pattern:

1. Check the Variants section for simpler alternatives
2. Review the Patterns and Building Blocks section for component-level help
3. Break the automation into smaller pieces and test each one
4. Reach out to your automation platform's community
5. Consider whether this automation is actually the right starting point

Not every pattern will be right for every business. If something feels too complex or doesn't quite fit, that's valuable information. Pick a different pattern and come back to this one later.

## What Success Looks Like

You'll know you're using this encyclopedia effectively when:

- You implement 1-2 new automations per month consistently
- Each automation saves measurable time or improves measurable quality
- You're adapting patterns rather than copying them exactly
- You're combining building blocks from section 90 into custom workflows
- Your team asks "can we automate this?" before doing repetitive work
- You're contributing your own variants back to the knowledge base

This is a living reference. Treat it like a cookbook you return to regularly, not a novel you read once.
