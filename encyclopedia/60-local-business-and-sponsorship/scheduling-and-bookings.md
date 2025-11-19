# Scheduling and Bookings

7 automation patterns for appointment management, capacity planning, and reducing no-shows. Focus: maximize revenue and minimize wasted time.

---

## 1. Automated Appointment Reminders
**Trigger**: 24 hours and 2 hours before appointment
**Steps**: Send reminder via SMS and email → include appointment details (date, time, service, provider) → provide easy reschedule/cancel link → track confirmations → flag unconfirmed appointments → follow up on no-response
**Time Saved**: 1-2 hours/day on phone reminders | **Variants**: Multi-touch reminders, custom timing by service type

## 2. Waitlist and Cancellation Filling
**Trigger**: Appointment canceled
**Steps**: Identify waitlist customers for that service/time → send immediate notification: "Spot opened for [time]" → first to respond gets slot → auto-book confirmed customer → remove from waitlist → repeat until slot filled
**Time Saved**: 30 min per cancellation | **Variants**: VIP priority, deposit requirements

## 3. Dynamic Pricing for Peak Times
**Trigger**: Capacity thresholds reached or time-based
**Steps**: Monitor booking rates by time slot → identify high-demand periods → adjust pricing dynamically (surge pricing) → promote off-peak with discounts → track price elasticity → optimize revenue
**Time Saved**: Revenue optimization vs fixed pricing | **Variants**: Happy hour pricing, seasonal adjustments

## 4. Staff Capacity and Skill Matching
**Trigger**: Booking request received
**Steps**: Check service requirements → identify qualified staff → check availability → match to least-utilized qualified person → book appointment → balance workload across team → track staff utilization
**Time Saved**: 15-20 min per booking | **Variants**: Customer preferences, skill-level tiers

## 5. No-Show Tracking and Deposit Requirements
**Trigger**: Appointment time passes without check-in
**Steps**: Mark as no-show → track no-show history by customer → calculate no-show cost → require deposit for repeat offenders → send follow-up: "We missed you, rebook?" → flag serial no-shows
**Time Saved**: Prevents revenue loss | **Variants**: Cancellation fees, blacklist policies

## 6. Group Booking and Event Coordination
**Trigger**: Group booking request (parties, events, classes)
**Steps**: Check capacity for requested date/time → block resources → send group coordinator details and requirements → collect attendee info → send group reminders → track RSVPs → adjust setup based on confirmed count
**Time Saved**: 1-2 hours per group booking | **Variants**: Package deals, minimum attendee requirements

## 7. Recurring Appointment Auto-Scheduling
**Trigger**: Recurring service customer checks out
**Steps**: Offer to book next appointment (weekly hair cut, monthly massage, etc.) → suggest date based on frequency → send confirmation → add to calendar → send regular reminders → track recurring customer retention
**Time Saved**: Guaranteed future revenue | **Variants**: Subscription packages, pre-pay discounts

---

## Implementation Priority
1. **Automated Appointment Reminders** (#1) - Reduce no-shows immediately
2. **Waitlist and Cancellation Filling** (#2) - Maximize capacity
3. **Recurring Appointment Auto-Scheduling** (#7) - Build predictable revenue
4. **No-Show Tracking** (#5) - Protect against repeat offenders
