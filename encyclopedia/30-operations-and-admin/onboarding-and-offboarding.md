# Onboarding and Offboarding

This document contains 10 automation patterns for employee onboarding, client onboarding, and graceful offboarding. Focus: consistency and completeness.

---

## 1. New Employee Onboarding Workflow

**Trigger**: New hire added to HR system or start date in calendar

**Core Steps**:
1. Create onboarding project in PM tool with phase-based tasks
2. Week before start: Order equipment, create email account, schedule orientation
3. Day 1: Send welcome email with credentials, first day schedule, team intro
4. Days 1-7: Daily check-ins via Slack with links to resources
5. Week 1-4: Stage-gated tasks (complete week 1 before week 2 unlocks)
6. Assign buddy, schedule 1-on-1s, enroll in training
7. 30/60/90 day check-in reminders to manager
8. Track completion and blockers

**Outputs**: Consistent onboarding, nothing forgotten, new hire feels supported

**Time Saved**: 2-3 hours per hire vs manual coordination

**Risks**: Rigid checklist might not fit all roles (allow customization)

**Variants**: Role-specific checklists (sales vs engineering vs ops), remote vs in-office variants

---

## 2. Tool Provisioning Automation

**Trigger**: New hire confirmed or role change requires new tool access

**Core Steps**:
1. Receive provisioning request (from HR system or form)
2. Determine tools needed based on role (sales = CRM, engineering = GitHub, etc.)
3. Create accounts in each system via API
4. Set appropriate permissions and licenses
5. Add to relevant Slack channels and email groups
6. Generate temporary passwords and send secure setup instructions
7. Track provisioning status (pending, completed, failed)
8. Alert IT if manual intervention needed
9. 30-day review: Remove unused tool access

**Outputs**: Fast access (hours not days), consistent permissions, license tracking

**Time Saved**: 30-60 min per new hire

**Risks**: Over-provisioning wastes licenses, under-provisioning blocks work

**Variants**: Just-in-time provisioning (tools added as needed), guest/contractor workflows

---

## 3. Client Onboarding Kickoff

**Trigger**: Deal marked "Closed Won" in CRM

**Core Steps**:
1. Create client project in PM tool from template
2. Generate welcome packet (contract, invoices, getting-started guide)
3. Send welcome email to client with next steps and timeline
4. Schedule kickoff call within 3 business days
5. Create client Slack channel and invite team members
6. Assign account manager and notify client
7. Set up recurring check-ins (weekly for first month, then monthly)
8. Populate client CRM record with project details
9. Create 30-day milestone review task

**Outputs**: Professional start, clear expectations, smooth handoff from sales

**Time Saved**: 45-60 min per new client

**Risks**: Generic onboarding feels impersonal (personalize welcome message)

**Variants**: Tiered onboarding by client size, product-specific workflows

---

## 4. Training Assignment and Tracking

**Trigger**: New hire starts or role changes

**Core Steps**:
1. Assign role-appropriate training modules
2. Send training links with deadlines
3. Track completion via LMS or manual check-ins
4. Send reminders at 50%, 75%, 90% of deadline
5. Escalate to manager if deadline missed
6. Quiz/assessment after completion
7. Certificate generation for compliance training
8. Track training history in HR system

**Outputs**: Compliant, trained employees; audit trail for certifications

**Time Saved**: 1 hour per training cycle

**Risks**: Training completion without actual learning (monitor quality)

**Variants**: Self-paced vs cohort-based, mandatory vs optional tracking

---

## 5. First Week Check-In Automation

**Trigger**: Day 1, 3, 5 of employment

**Core Steps**:
1. Send daily check-in message via Slack/email
2. Day 1: "How was your first day? Any blockers?"
3. Day 3: "Getting settled? Need anything?"
4. Day 5: "End of week 1! Quick survey on onboarding experience"
5. Collect responses and flag issues
6. Alert manager if new hire reports problems
7. Compile feedback for onboarding improvement
8. Follow up on any requests or blockers

**Outputs**: Early problem detection, better new hire experience, onboarding insights

**Time Saved**: 20-30 min per new hire

**Risks**: Can feel micromanaging (keep tone friendly and optional)

**Variants**: Manager-led check-ins, peer buddy check-ins, anonymous surveys

---

## 6. Equipment Ordering and Tracking

**Trigger**: New hire start date set (2 weeks before)

**Core Steps**:
1. Create equipment order based on role (laptop, monitor, phone, etc.)
2. Submit purchase request or select from inventory
3. Track order status via vendor API
4. Alert if order delayed
5. Schedule delivery to office or new hire home
6. Create asset tracking record (serial numbers, assigned to employee)
7. Send setup instructions to new hire
8. Track returns on offboarding

**Outputs**: Equipment ready day 1, inventory tracking, cost visibility

**Time Saved**: 30 min per new hire

**Risks**: Supply chain delays (order early), shipping issues (track closely)

**Variants**: Bring-your-own-device policies, rental vs purchase workflows

---

## 7. Buddy Assignment and Introduction

**Trigger**: New hire confirmed, 1 week before start

**Core Steps**:
1. Select buddy based on role, seniority, availability
2. Notify buddy with new hire details and expectations
3. Send introduction email connecting new hire and buddy
4. Provide buddy guide (what to cover, how to help)
5. Schedule buddy check-ins (day 1, end of week 1, end of month 1)
6. Remind buddy before each check-in
7. Collect buddy feedback on new hire progress
8. Thank buddy after 30 days

**Outputs**: Better new hire integration, faster ramp-up, cultural connection

**Time Saved**: 15-20 min per assignment

**Risks**: Bad buddy match hurts experience (monitor and swap if needed)

**Variants**: Peer buddy vs manager, rotating buddies, buddy networks

---

## 8. Offboarding Workflow

**Trigger**: Employee resignation or termination

**Core Steps**:
1. Create offboarding checklist with deadlines
2. Schedule exit interview
3. Disable system access on departure date (not before)
4. Collect company property (laptop, keys, etc.)
5. Transfer knowledge and projects to team
6. Final paycheck and benefits offboarding coordination
7. Remove from email lists, Slack, tools
8. Archive employee records
9. Send goodbye message to team (if appropriate)

**Outputs**: Secure exit, no lingering access, knowledge transfer complete

**Time Saved**: 1-2 hours per departure

**Risks**: Early access removal disrupts final days (time carefully)

**Variants**: Voluntary vs involuntary workflows, contractor vs employee

---

## 9. 30-60-90 Day Review Automation

**Trigger**: New hire day 30, 60, 90

**Core Steps**:
1. Send review form to manager and new hire
2. Questions cover: progress, challenges, satisfaction, goals
3. Collect responses and compile report
4. Schedule review meeting between manager and employee
5. Track action items from review
6. Alert HR if serious issues flagged
7. Update employee record with review notes
8. Set next review reminder

**Outputs**: Structured feedback, course correction, retention insight

**Time Saved**: 30 min per review cycle

**Risks**: Pro forma reviews (ensure meaningful conversation)

**Variants**: Peer feedback inclusion, skip-level reviews, self-assessment

---

## 10. Contractor Onboarding and Compliance

**Trigger**: Contractor agreement signed

**Core Steps**:
1. Collect required documents (W-9, insurance, SOW)
2. Verify compliance requirements met
3. Create contractor record in system
4. Provide limited tool access (no full employee access)
5. Assign project and point of contact
6. Send contractor handbook/guidelines
7. Track contract end date and renewal
8. 30 days before end: Remind manager to extend or close

**Outputs**: Compliant contractor management, clear scope, timely renewals

**Time Saved**: 45 min per contractor

**Risks**: Misclassification (ensure proper contractor vs employee status)

**Variants**: International contractors (tax implications), NDA workflows

---

## Implementation Priority

1. **New Employee Onboarding Workflow** (#1) - Foundation for everything else
2. **Tool Provisioning Automation** (#2) - Immediate pain point
3. **Offboarding Workflow** (#8) - Security critical
4. Others as needed based on volume and pain

Start with employee onboarding, add provisioning, then tackle offboarding for security.
