# Approvals and Handovers

This document contains 8 automation patterns for routing approval requests, managing handoffs between teams, and ensuring accountability. Focus: speed and visibility.

---

## 1. Purchase Approval Workflow

**Trigger**: Purchase request submitted via form

**Core Steps**:
1. Employee submits purchase request (item, cost, justification)
2. Route based on amount:
   - <$500 → Direct manager approval
   - $500-$5k → Manager + department head
   - >$5k → Manager + CFO
3. Send approval request to first approver
4. Approver gets email/Slack with details and approve/reject buttons
5. If approved by first, route to next approver
6. If rejected, notify requester with reason
7. Track time in each approval stage
8. Escalate if no response in 48 hours
9. On final approval, create purchase order and notify procurement

**Outputs**: Fast approvals, clear audit trail, budget control

**Time Saved**: 30-60 min per request vs email chains

**Risks**: Approvers might auto-approve without review (spot-check compliance)

**Variants**: Different thresholds by department, pre-approved vendor lists, recurring purchase fast-track

---

## 2. PTO Request and Calendar Integration

**Trigger**: PTO request submitted

**Core Steps**:
1. Employee submits PTO request with dates
2. Check PTO balance (enough days available?)
3. Check team calendar (conflicts with other PTO?)
4. Route to manager for approval
5. If approved:
   - Deduct from PTO balance
   - Add to team calendar
   - Set Slack status to "Out of office"
   - Create email auto-responder
   - Notify team of upcoming absence
6. If denied, notify employee with reason
7. Track approval time and patterns

**Outputs**: Self-service PTO, automatic coverage planning, no double-booking

**Time Saved**: 15-20 min per request

**Risks**: Last-minute requests might be auto-denied (allow override)

**Variants**: Blackout dates for busy seasons, coverage assignment automation

---

## 3. Content Approval Pipeline

**Trigger**: Content ready for review (blog post, social media, email campaign)

**Core Steps**:
1. Creator marks content as "ready for review"
2. Route to appropriate reviewers based on content type:
   - Blog → Editor → Legal (if needed) → Publish
   - Social → Brand manager → Publish
   - Email → Marketing lead → Publish
3. Each reviewer gets notification with preview
4. Reviewers can approve, request changes, or reject
5. If changes requested, return to creator with feedback
6. Track revision rounds and time in review
7. On final approval, automatically publish or add to schedule
8. Log approval chain for compliance

**Outputs**: Consistent review process, faster publishing, quality control

**Time Saved**: 20-30 min per piece on coordination

**Risks**: Bottleneck if single approver is slow (allow delegation)

**Variants**: Parallel review (multiple reviewers at once), tiered approval by content risk

---

## 4. Project Handoff from Sales to Delivery

**Trigger**: Deal marked "Closed Won"

**Core Steps**:
1. Create handoff document from template
2. Auto-populate with deal details, customer info, scope
3. Sales rep fills in customer context and expectations
4. Route to delivery team lead for review
5. Delivery lead accepts handoff and assigns project manager
6. Schedule kickoff call with sales, delivery, and customer
7. Transfer all documents and notes to delivery workspace
8. Update CRM with delivery team assignments
9. Create project in PM tool with initial tasks
10. Close sales loop: mark as "successfully handed off"

**Outputs**: Clean handoffs, no lost information, customer continuity

**Time Saved**: 45-60 min per handoff

**Risks**: Sales might skip context steps (make required fields)

**Variants**: Different workflows for different service types, client size tiers

---

## 5. Support Ticket Escalation

**Trigger**: Support ticket meets escalation criteria

**Core Steps**:
1. Monitor support tickets for escalation signals:
   - Customer marks as urgent
   - No response in X hours
   - Third reply from customer (indicates unresolved)
   - Specific keywords (angry, lawsuit, cancel)
2. Auto-escalate to senior support or manager
3. Notify customer: "We've escalated your request to ensure fast resolution"
4. Set shorter SLA for escalated tickets
5. Alert if escalated ticket still not resolved
6. Track escalation rates and reasons
7. Feed into product/support improvement

**Outputs**: Faster resolution of critical issues, customer satisfaction

**Time Saved**: Immediate escalation vs waiting for manual review

**Risks**: Over-escalation if criteria too broad (tune carefully)

**Variants**: VIP customer fast-track, product-specific escalation paths

---

## 6. Document Review and Sign-Off

**Trigger**: Document ready for stakeholder sign-off (policy, contract, process doc)

**Core Steps**:
1. Creator marks document as "ready for sign-off"
2. List required approvers (legal, finance, executive, etc.)
3. Route document to approvers in sequence or parallel
4. Track who has reviewed and who hasn't
5. Send reminders every 48 hours if not reviewed
6. Collect digital signatures or approval confirmations
7. Version control: lock document after final approval
8. Distribute approved document to relevant parties
9. Set review date for future updates

**Outputs**: Formal approval process, version control, compliance documentation

**Time Saved**: 30-45 min per document on coordination

**Risks**: Document changes during review (lock editing after submission)

**Variants**: Emergency fast-track for urgent docs, multi-stage review for complex docs

---

## 7. Expense Report Approval

**Trigger**: Expense report submitted

**Core Steps**:
1. Employee submits expenses with receipts
2. Auto-validate against policy:
   - Receipts attached for all items >$25?
   - Within category limits?
   - Duplicate expenses?
3. Flag policy violations for manual review
4. Route to manager for approval
5. Manager sees expenses, receipts, policy flags
6. Approve, reject, or request clarification
7. On approval, route to finance for payment
8. Track reimbursement status
9. Alert if payment delayed >10 days

**Outputs**: Policy compliance, fast reimbursement, expense tracking

**Time Saved**: 20-30 min per report on back-and-forth

**Risks**: Employees might not upload receipts (block submission without)

**Variants**: Per diem workflows, corporate card vs reimbursement, international expenses

---

## 8. Cross-Department Handoff Template

**Trigger**: Work needs to move from one department to another

**Core Steps**:
1. Completing team uses handoff form:
   - What was done
   - What's next
   - Important context or gotchas
   - Attachments and links
2. Form routes to receiving team
3. Receiving team acknowledges receipt and reviews
4. Questions/clarifications handled via threaded discussion
5. Receiving team accepts ownership (formal handoff complete)
6. Notify original team of acceptance
7. Track handoff time and completion
8. Flag handoffs that stall (no acceptance in 48 hours)

**Outputs**: Clear ownership transfer, documented context, accountability

**Time Saved**: 30 min per handoff vs email confusion

**Risks**: Receiving team might not read thoroughly (require acceptance confirmation)

**Variants**: Specific templates for common handoffs (design → dev, support → engineering)

---

## Implementation Priority

1. **Purchase Approval Workflow** (#1) - High frequency, clear ROI
2. **Project Handoff from Sales to Delivery** (#4) - Critical for customer success
3. **Support Ticket Escalation** (#5) - Customer satisfaction impact
4. **PTO Request and Calendar Integration** (#2) - Employee self-service win
5. Others based on specific bottlenecks

Start with purchase approvals (high volume, immediate impact), then tackle critical handoffs (sales to delivery, support escalation).
