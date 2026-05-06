# Automation Project Scoping: Avoiding Scope Creep

## Overview
Scope creep—when projects expand beyond original boundaries—is the #1 killer of automation profitability. This guide teaches you how to define and defend scope.

**Key Principle:** A clear scope statement prevents 90% of project problems.

---

## Part 1: What is Scope?

### Scope Definition
The work that WILL be done:
- "Automate invoice data extraction and validation"
- "Integrate with QuickBooks"
- "Train 3 users"

### Out of Scope (What WON'T be done)
- "Redesign invoice templates"
- "Migrate historical invoices"
- "Custom reporting dashboard"
- "Integration with 5+ other systems"

**The written scope is your contract.**

---

## Part 2: Scoping Discovery Questions

### Current Process
- Walk through the process step-by-step
- Document each step: input, output, decision point
- Identify pain points and bottlenecks
- Measure: time spent, volume, error rate

**Example: Invoice Processing**
- How many invoices/month? (500)
- How long does each take? (15 minutes)
- Which steps are most error-prone? (Data entry)
- Who approves invoices? (Manager)
- What happens with rejected invoices? (Manual review)

### System Landscape
- What systems are involved? (Email, Excel, QuickBooks)
- How do they connect now? (Manual data transfer)
- What data lives where? (Scattered across tools)
- What are integration constraints? (API limits, authentication)

### Requirements
- Must-haves: "Extracts data 100% accurately"
- Nice-to-haves: "Auto-categorizes by vendor"
- Out of scope: "Redesigns invoice format"

### Success Metrics
- How will you measure success?
- Time saved per week?
- Error reduction percentage?
- Cost savings?

---

## Part 3: Writing a Clear Scope Statement

### Scope Statement Template
```
PROJECT SCOPE STATEMENT

Project Name: Invoice Processing Automation
Client: ABC Corp Accounting Department

WHAT WILL BE DELIVERED:
1. Automated invoice extraction from emails
   - Extracts: Vendor name, amount, date, PO number
   - Accuracy target: 98%+
   - Processes 500 invoices/month

2. Integration with QuickBooks
   - Creates invoice records automatically
   - Matches invoices to POs
   - Logs all processed invoices

3. Approval workflow
   - Routes invoices to managers for approval
   - Rejects invalid invoices for manual review
   - Creates audit trail

4. Training
   - 2-hour training session for 3 staff
   - Written documentation
   - 30-day support included

WHAT IS NOT INCLUDED (OUT OF SCOPE):
- Data migration for historical invoices
- Custom invoice template design
- Integration with HR or expense systems
- Multi-language invoice support
- Vendor master file maintenance
- API access for external systems

SUCCESS CRITERIA:
- 95%+ process automation (minimal manual work)
- 98%+ data extraction accuracy
- Zero unintended duplicate invoices
- <5 minutes average processing time per invoice

TIMELINE:
- Week 1: Analysis & Setup
- Week 2-3: Development & Testing
- Week 4: Deployment & Training
- Completion: [DATE]

ASSUMPTIONS:
- Client provides test data by [DATE]
- Decision maker available for approval meetings
- Invoice format remains consistent
- Access to QuickBooks API provided
- Email server supports automation scripts
```

---

## Part 4: The Scope Change Process

### When Clients Ask for "Just One More Thing"
1. **Don't say yes immediately**
   - Example: "That's a great idea. Let me assess the impact."

2. **Document the change request**
   - What specifically? "Add automatic email notifications to vendors"
   - Why? "To acknowledge receipt faster"
   - Impact? "Requires new email templates and additional testing"

3. **Estimate the impact**
   - Hours to implement: 8 hours
   - Cost: 8 hours × $85/hour = $680
   - Delay: 3 days

4. **Present options**
   - Option A: "Add to scope for $680, delay delivery 3 days"
   - Option B: "Implement as Phase 2 after launch"
   - Option C: "Implement manually during transition period"

5. **Get written approval**
   - Email: "Confirmed: Adding vendor notifications for $680"
   - Sign change order
   - Update timeline

---

## Part 5: Common Scope Creep Scenarios

### Scenario 1: "Can you also automate the other workflow?"
**Client asks:** "While you're at it, can you also handle expense reports?"
**Your response:** "That's a great opportunity! That would be a separate phase. Let me scope it separately and give you a timeline and cost estimate."
**Action:** Document as Phase 2, don't let it grow Phase 1.

### Scenario 2: "We need it in 50% of the time"
**Client asks:** "Can you finish in 2 weeks instead of 4?"
**Your response:** "We can accelerate, but it costs extra for overtime. That's X hours × 1.5× = $Y extra, and we'd reduce testing time, increasing risk of bugs."
**Action:** Get written approval for faster timeline and cost increase.

### Scenario 3: "We need to support multiple formats"
**Client asks:** "We also receive invoices as images and PDFs, not just emails."
**Your response:** "That changes the complexity significantly—we'd need OCR. Let me assess: adds 12 hours, costs +$1,000, increases timeline 1 week."
**Action:** Treat as scope change. Get approval and cost increase.

### Scenario 4: "We need training for more people"
**Client asks:** "Actually, all 15 people need training, not just 3."
**Your response:** "That's great for adoption! Original scope included 3 people. Training 15 people is 2 additional hours. Cost: 2 hours × $85/hour = +$170."
**Action:** Get written approval for scope change.

---

## Part 6: Protecting Yourself

### In the Contract
```
SCOPE CHANGE PROCESS:
Any changes to the scope must be documented in writing
and approved before work begins. Scope changes will result in:
- Updated timeline
- Updated cost estimate
- Written change order signed by both parties

Changes requested but not approved will not be completed.
```

### During Project
- Weekly check-ins: "Are we still on scope?"
- Send weekly status: "We completed X from the original scope"
- Flag new requests immediately: "This is outside our scope statement"

### At Completion
- Scope review: "We delivered everything in the scope statement"
- Document any changes: "We completed 2 out-of-scope items for $1,500"

---

## Part 7: Scope vs. Timeline vs. Quality

### The Iron Triangle
You can pick 2 of 3:
- **Fast + Good = Expensive**
- **Fast + Cheap = Poor quality**
- **Cheap + Good = Slow**

**Example:**
- Client wants: "Fast, cheap, AND great"
- Your response: "Pick 2. I recommend good + reasonable timeline at fair price."

---

## Summary

**Scope Management:**
- ✓ Document what WILL and WON'T be delivered
- ✓ Get written agreement before starting
- ✓ Use a change control process
- ✓ Make scope changes visible and billable
- ✓ Protect your margins by defending scope

**Key Takeaway:**
Scope creep steals profit faster than anything else. Protect your scope like you protect your family.

---

*This guide was created by **Rework Digital** - Resources Department for automation professionals.*

Questions? Reach out: resource@reworkdigital.io | Follow on GitHub: https://github.com/Reworkdigital-io
