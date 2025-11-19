# Human-in-the-Loop Patterns

When and how to add human judgment to automated workflows.

## Why Human-in-the-Loop?

**Not everything should be fully automated**. Human judgment is valuable for:
- High-stakes decisions (contracts, large purchases, customer-facing communication)
- Edge cases and ambiguity (AI isn't sure, data is weird)
- Learning and improvement (humans train automation by reviewing)
- Trust and accountability (someone owns the outcome)
- Regulatory compliance (some decisions legally require human approval)

**The goal**: Automate the mechanics, preserve human judgment where it matters.

---

## Pattern 1: Draft-Only Mode

**Concept**: Automation creates draft, human reviews and approves before sending/executing

**When to use**:
- Customer-facing emails (proposals, invoices, contracts)
- Social media posts (brand reputation risk)
- Large financial transactions
- Legal documents
- Hiring decisions

**Implementation**:
1. Automation runs and generates output
2. Output saved as draft (not sent/executed)
3. Human notified: "Draft ready for review"
4. Human reviews, edits if needed, approves
5. On approval, automation completes (sends email, posts content, etc.)

**Example: Invoice generation**
```
1. Deal marked "Closed Won"
2. Automation generates invoice from template
3. Invoice saved as draft in accounting system
4. Finance team notified: "Review invoice for Acme Corp"
5. Team reviews amount, line items, terms
6. On approval, invoice sent to customer
```

**Best practices**:
- Make draft easily editable (don't lock content)
- Show what will happen when approved (preview)
- Allow bulk approval (50 drafts that are all correct)
- Track review time (if taking days, automate more)

**Graduation path**: After 100 drafts with <5% edits, consider auto-sending low-risk cases

---

## Pattern 2: Approve/Reject Workflow

**Concept**: Automation proposes action, human decides yes/no

**When to use**:
- Expense approvals
- Purchase requests
- Content publication
- Data deletion or archival
- Access grants

**Implementation**:
1. Automation detects need for approval (expense submitted, purchase requested)
2. Route to appropriate approver(s)
3. Present context and request decision
4. Approver chooses: Approve, Reject, Request Changes
5. Automation proceeds based on decision
6. Log decision and rationale

**Example: Expense approval**
```
1. Employee submits $500 expense report
2. Automation routes to manager
3. Manager sees: expenses, receipts, policy compliance check
4. Manager decides: [Approve] [Reject] [Request clarification]
5. If approved → send to finance for payment
6. If rejected → notify employee with reason
7. If clarification needed → return to employee
```

**Best practices**:
- Default action if no response (usually: escalate, don't auto-approve)
- Escalation path (if manager doesn't respond in 48hrs, route to their manager)
- Bulk operations (approve multiple at once)
- Audit trail (who approved, when, why)

**Delegation**: Allow approvers to delegate ("Sarah approves in my absence")

---

## Pattern 3: Confidence Threshold Routing

**Concept**: If automation is confident, proceed; if uncertain, ask human

**When to use**:
- AI classification or categorization
- Lead scoring
- Document processing
- Data matching and deduplication

**Implementation**:
1. Automation makes decision (categorize this email, match this contact)
2. Calculate confidence score (0-100%)
3. If confidence >80%: Proceed automatically
4. If confidence 50-80%: Flag for human review
5. If confidence <50%: Require human decision
6. Learn from human corrections

**Example: Email categorization**
```
1. Email arrives
2. AI categorizes: "Support ticket" (confidence: 95%)
3. Confidence >80% → Auto-route to support queue
4. Another email: "Sales or support?" (confidence: 60%)
5. Confidence 50-80% → Ask human: "Is this sales or support?"
6. Human answers: "Support"
7. System learns: similar emails in future → higher confidence for support
```

**Best practices**:
- Start with low auto-threshold (require review initially)
- Raise threshold as accuracy improves
- Track accuracy by confidence band (are 80% items really 80% accurate?)
- Provide context for human review (why was automation uncertain?)

**Feedback loop**: Human corrections improve the model

---

## Pattern 4: Spot-Check Sampling

**Concept**: Automation handles most, human reviews random sample

**When to use**:
- High-volume, low-risk tasks
- Quality assurance
- Compliance monitoring
- Process improvement

**Implementation**:
1. Automation processes all items
2. Randomly select X% for human review (typically 5-10%)
3. Human reviews sample for accuracy and quality
4. If issues found, review more extensively
5. If sample is clean, continue with current automation
6. Track accuracy over time

**Example: Receipt categorization**
```
1. Automation categorizes 200 receipts this week
2. Randomly select 10 (5%) for review
3. Accountant reviews 10: finds 1 miscategorization
4. 90% accuracy in sample
5. Continue automation but tune categorization rules
6. Next week: retest with new sample
```

**Best practices**:
- True random selection (not just first 10)
- Representative sample (cover all edge cases)
- Regular cadence (weekly or monthly reviews)
- Feedback to improve automation

**Risk management**: Increase sample rate for high-stakes or new processes

---

## Pattern 5: Escalation Path

**Concept**: Automation handles routine cases, complex ones go to humans

**When to use**:
- Customer support routing
- Approval workflows
- Exception handling
- Risk assessment

**Implementation**:
1. Define escalation criteria (complexity, value, risk, sentiment)
2. Automation assesses each case
3. Routine cases → automated handling
4. Complex cases → human expert
5. Track escalation rate
6. Reduce over time as automation improves

**Example: Support ticket routing**
```
1. Customer submits ticket
2. Automation checks:
   - Is customer VIP? → Escalate to senior support
   - Is issue in knowledge base? → Send KB article
   - Negative sentiment detected? → Escalate to human
   - Standard request? → Auto-resolve with template
3. Escalated tickets go to appropriate human
4. Routine tickets handled automatically
```

**Escalation triggers**:
- High value (VIP customer, large deal, big purchase)
- High risk (legal issue, security breach, compliance)
- High emotion (angry customer, sensitive topic)
- High complexity (multiple interconnected issues)
- Repeated attempts (automation tried and failed)

**Best practices**:
- Clear escalation criteria (not subjective)
- Fast escalation (don't make customer wait)
- Context transfer (human gets full history)
- Learn from escalations (why couldn't automation handle it?)

---

## Pattern 6: Human Expert Library

**Concept**: Automation learns from how humans handle cases

**When to use**:
- Email response templates
- Decision-making workflows
- Customer service
- Content creation

**Implementation**:
1. Humans handle cases initially (no automation)
2. Collect human decisions and actions
3. Identify patterns in human responses
4. Automate common patterns
5. Route new patterns back to humans
6. Continuously learn and expand automation coverage

**Example: Customer inquiry responses**
```
Phase 1: Humans answer all customer emails
Phase 2: Categorize human responses by inquiry type
Phase 3: Create templates from most common responses
Phase 4: Automation suggests response, human approves
Phase 5: Auto-send for common cases, human handles unique
```

**Best practices**:
- Start with observation, not automation
- Document expert decision-making process
- Codify rules gradually
- Preserve edge case handling

**Timeline**: Expect 3-6 months of human handling before meaningful automation

---

## Pattern 7: Progressive Automation

**Concept**: Gradually increase automation level as confidence grows

**Stages**:

**Stage 1: Manual with tracking**
- Humans do everything
- Track time, decisions, outcomes
- Identify automation opportunities

**Stage 2: Automation suggests**
- Automation proposes next step
- Human reviews and decides
- Track suggestion acceptance rate

**Stage 3: Draft and review**
- Automation creates draft
- Human reviews before executing
- Track edit rate

**Stage 4: Auto-execute with oversight**
- Automation executes
- Human reviews results after
- Spot-check sample

**Stage 5: Fully automated**
- Automation handles end-to-end
- Exception-only human involvement
- Periodic quality audits

**Example progression: Invoice sending**
```
Stage 1: Manually create and send each invoice (baseline)
Stage 2: Automation suggests invoice amount, human creates and sends
Stage 3: Automation creates draft invoice, human reviews and sends
Stage 4: Automation sends invoice, human reviews sent invoices weekly
Stage 5: Automation sends all invoices, human reviews exceptions only
```

**Move to next stage when**:
- Current stage running smoothly for 4+ weeks
- Error rate <5%
- Team comfortable with automation
- Proper monitoring in place

---

## Pattern 8: Veto Power

**Concept**: Automation proceeds unless human intervenes

**When to use**:
- Time-sensitive actions
- Routine but important tasks
- High-confidence automation
- Trust-building phase

**Implementation**:
1. Automation announces intended action
2. Provides window for human veto (30 min, 2 hours, 24 hours)
3. Human can cancel, modify, or approve
4. If no veto, automation proceeds
5. Log all actions and vetos

**Example: Social media posting**
```
1. Automation creates social post from blog article
2. Posts to Slack: "Publishing this to LinkedIn in 30 minutes: [preview]"
3. Team has 30 min to say "don't post" or edit
4. If no veto, auto-publish at scheduled time
5. Track veto rate (should decrease over time)
```

**Best practices**:
- Clear deadline for veto (don't wait indefinitely)
- Easy veto mechanism (single button, not complex process)
- Notification that veto period is ending (5 min warning)
- Default to proceed (not to cancel)

**When to use**: Medium-risk actions where delay is acceptable

---

## Pattern 9: Collaborative Refinement

**Concept**: Automation and human work together iteratively

**When to use**:
- Complex decision-making
- Creative work
- Strategic planning
- Custom solutions

**Implementation**:
1. Automation provides initial output (80% complete)
2. Human refines and improves
3. Automation learns from human edits
4. Next iteration is better (85% complete)
5. Repeat until human edits minimal

**Example: Report generation**
```
Round 1: Automation generates report (human edits 50%)
Round 2: Automation incorporates feedback (human edits 30%)
Round 3: Automation improves (human edits 10%)
Round 4: Automation nearly perfect (human edits 5%)
```

**Best practices**:
- Track what humans change (not just that they changed)
- Use changes to improve automation
- Set improvement goals (reduce edit rate by 20% per quarter)
- Celebrate milestones (90% auto-generated quality)

---

## Choosing the Right Pattern

**Decision matrix**:

| Risk Level | Volume | Pattern |
|------------|--------|---------|
| High | Low | Approve/Reject |
| High | High | Draft-Only → Progressive Automation |
| Medium | Low | Escalation Path |
| Medium | High | Confidence Threshold → Spot-Check |
| Low | Any | Veto Power → Fully Automated |

**Questions to ask**:
1. What's the cost of error? (High → more human involvement)
2. How often does it happen? (High volume → worth automating)
3. How consistent are cases? (Consistent → easier to automate)
4. How time-sensitive? (Urgent → veto pattern)
5. How much trust in automation? (Low → start with draft-only)

---

## Common Mistakes

**1. All-or-nothing thinking**
"We'll automate this completely or not at all"
→ Start with draft-only, progress over time

**2. No measurement**
Not tracking how often humans intervene or why
→ Log every human decision and reason

**3. Permanent review mode**
"Automation creates drafts" forever, never graduating to auto-send
→ Set criteria for moving to next stage

**4. Unclear authority**
Who can override automation? Everyone? No one?
→ Define approval authority clearly

**5. No feedback loop**
Humans fix automation mistakes but automation never improves
→ Build learning into system

---

## Measuring Success

**Metrics to track**:
- **Automation rate**: % handled without human (goal: increase over time)
- **Review time**: How long humans spend reviewing (goal: decrease)
- **Intervention rate**: % where human changes automation output (goal: <10%)
- **Escalation rate**: % escalated to humans (goal: stable or decreasing)
- **Accuracy**: % of automated actions that are correct (goal: >95%)
- **Time savings**: Hours saved vs fully manual (goal: quantify ROI)

**Healthy patterns**:
- Automation rate increasing month-over-month
- Intervention rate decreasing
- Time savings growing
- Team trusts automation (positive feedback)

**Warning signs**:
- High intervention rate not improving
- Team routinely overrides automation
- Escalation rate increasing
- Errors not being fixed

The best human-in-the-loop automation makes humans feel supported, not surveilled. They handle what they're good at (judgment, edge cases, relationships) while automation handles repetitive mechanics.
