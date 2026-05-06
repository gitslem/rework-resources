# Zapier Starter Kit: 10 Essential Business Workflows

## Pre-Built Automation Templates to Get Started Immediately

---

## Quick Start Guide

Each workflow below is battle-tested and ready to customize for your business. Copy the step-by-step instructions and adapt the app names and fields to your setup.

---

## 1. Lead Capture & Auto-Response

**Apps Needed:** Web Form (or Email) → Zapier → Email + Spreadsheet

**What It Does:** Captures leads from your website form, sends auto-response, and logs to spreadsheet.

**Steps:**
1. Trigger: New form submission (Typeform, Gravity Forms, or Email)
2. Action 1: Send email to lead with templated response
3. Action 2: Append lead to Google Sheet (Name, Email, Phone, Company)
4. Action 3: Notify sales team via Slack #leads channel

**Customization:**
- Change trigger to different form tool
- Add conditional logic (e.g., "if company size > 50 employees, mark as enterprise")
- Add welcome email automation (first in series of 5)

**Result:** Zero-manual lead tracking, instant acknowledgment to prospects

---

## 2. Invoice Processing & Payment Alerts

**Apps Needed:** Email → Zapier → Accounting Software + Slack

**What It Does:** Monitors for invoices, logs them, and alerts finance team.

**Steps:**
1. Trigger: New email with invoice (Gmail, Outlook)
2. Action 1: Extract invoice data (amount, vendor, date)
3. Action 2: Create record in QuickBooks/Xero
4. Action 3: Post to Slack #accounting with amount and vendor
5. Optional: Create reminder for due date

**Customization:**
- Add OCR extraction to pull data from PDF attachments
- Filter by vendor (only certain suppliers trigger)
- Auto-pay small invoices under $500 (requires API)

**Result:** 30 min/week saved on invoice data entry

---

## 3. Customer Feedback Loop

**Apps Needed:** Spreadsheet → Zapier → Email/Slack + CRM

**What It Does:** Tracks customer feedback, logs it, and notifies relevant teams.

**Steps:**
1. Trigger: New row in feedback Google Sheet
2. Condition: If rating < 3 (unhappy), escalate
3. Action 1 (If unhappy): Email to customer success manager
4. Action 2 (If unhappy): Add high-priority task in Asana
5. Action 3: Log feedback in CRM contact record (HubSpot/Pipedrive)
6. Action 4: Post trending feedback to #feedback Slack channel (weekly digest)

**Customization:**
- Route feedback by category (product, support, billing)
- Auto-create support ticket for urgent issues
- Generate sentiment analysis (happy vs. unhappy)

**Result:** Faster response to upset customers, visibility into feedback trends

---

## 4. Employee Onboarding Workflow

**Apps Needed:** HR Platform → Zapier → Email + G Suite + Slack

**What It Does:** Automates new hire processes from day one.

**Steps:**
1. Trigger: New employee added to BambooHR (or similar)
2. Action 1: Create Google Workspace account
3. Action 2: Send welcome email with IT setup instructions
4. Action 3: Create calendar invite for 1-on-1 with manager
5. Action 4: Post intro to #general Slack channel
6. Action 5: Create checklist in Asana with 30-60-90 day tasks
7. Action 6: Add to relevant Slack channels by department

**Customization:**
- Multi-step welcome email sequence (5-7 emails over 2 weeks)
- Department-specific channels and resources
- Assign equipment request workflow
- Schedule 30/60/90-day check-in calendar reminders

**Result:** Consistent onboarding, reduced manual HR work, faster employee productivity

---

## 5. Social Media Scheduling from Spreadsheet

**Apps Needed:** Google Sheets → Zapier → Twitter/LinkedIn/Instagram

**What It Does:** Schedule social posts from a spreadsheet.

**Steps:**
1. Trigger: New row in "Social Posts" Google Sheet with columns (Date, Platform, Text, Image)
2. Action 1: If platform = "Twitter", post to Twitter
3. Action 2: If platform = "LinkedIn", post to LinkedIn
4. Action 3: Log posted status back to spreadsheet
5. Optional: Add 24-hour reminder to add analytics to another sheet

**Customization:**
- Add URL shortener for link tracking
- Schedule multiple platforms with different content
- Cross-post to multiple accounts
- Add image from Google Drive

**Result:** Batch social content creation, consistent posting schedule

---

## 6. Project Status Dashboard

**Apps Needed:** Project Tool (Monday, Asana) → Zapier → Slack + Spreadsheet

**What It Does:** Weekly project status sent to stakeholders.

**Steps:**
1. Trigger: Weekly digest (e.g., every Monday 9am)
2. Action 1: Pull incomplete tasks from Monday.com/Asana
3. Action 2: Summarize status by project
4. Action 3: Post formatted report to #project-updates Slack
5. Action 4: Email report to stakeholders

**Customization:**
- Filter by project or team
- Highlight overdue tasks in red
- Include velocity metrics
- Add week-over-week comparison

**Result:** Automated status updates, visibility without manual reporting

---

## 7. CRM Sync & Lead Scoring

**Apps Needed:** Form/Email → Zapier → CRM (HubSpot/Salesforce) + Spreadsheet

**What It Does:** Auto-sync leads and score them for sales priority.

**Steps:**
1. Trigger: New lead from website form
2. Action 1: Create contact in HubSpot
3. Action 2: Calculate lead score based on criteria (company size, engagement, industry)
4. Action 3: Assign to sales rep based on territory
5. Action 4: Log to "Hot Leads" spreadsheet if score > 70

**Customization:**
- Adjust lead scoring formula
- Route by geography or vertical
- Add enrichment (company info, technographics)
- Trigger Slack alert for high-scoring leads

**Result:** Faster sales follow-up on qualified leads, automatic lead routing

---

## 8. Expense Report Approval

**Apps Needed:** Email/Form → Zapier → Slack + Accounting Software

**What It Does:** Streamlines expense approval workflow.

**Steps:**
1. Trigger: New expense report submitted (email attachment or form)
2. Action 1: Extract expense data (amount, category, date)
3. Action 2: Route to manager via Slack button approve/reject
4. Action 3 (If approved): Log to accounting software
5. Action 4 (If approved): Email employee confirmation + reimbursement details
6. Action 4 (If rejected): Email employee with reason, ask for revision

**Customization:**
- Add expense limit escalation (>$500 to CFO approval)
- Attach receipt images
- Category-based auto-approval (under $100)
- Monthly expense summary for cost centers

**Result:** Faster approval cycle, automatic accounting entries

---

## 9. Customer Renewal Reminders

**Apps Needed:** Spreadsheet/CRM → Zapier → Email + Calendar

**What It Does:** Proactive renewal outreach before contracts end.

**Steps:**
1. Trigger: Daily check at 9am
2. Condition: Find customers with renewal date within 30 days
3. Action 1: Send renewal reminder email (templated, personalized)
4. Action 2: Create reminder task in Asana for account manager
5. Action 3: Log email sent to CRM
6. Action 4: Slack alert to sales team #renewals channel

**Customization:**
- Escalate if no renewal by day 14 (discount offer)
- Include success metrics from past year
- Multi-touch: email at 30, 14, 7 days
- Special handling for at-risk accounts

**Result:** Higher renewal rate, proactive revenue protection

---

## 10. Team Time-Off Coordination

**Apps Needed:** Calendar → Zapier → Slack + Spreadsheet

**What It Does:** Tracks team time off and prevents double-booking.

**Steps:**
1. Trigger: New calendar event with "Out of Office" label
2. Action 1: Post to #time-off Slack channel (searchable list)
3. Action 2: Add to "Team Availability" spreadsheet
4. Action 3: Send Slack reminder to manager
5. Optional: Block that person's slots in scheduling calendar (Calendly)

**Customization:**
- Different channels by team (engineering, sales, support)
- Quarterly PTO summary for planning
- Auto-out-of-office email responder
- Coverage checklist for critical roles

**Result:** No missed meetings, clear team availability, better coverage planning

---

## Implementation Checklist

For each workflow:

- [ ] Choose the apps you'll connect
- [ ] Create free trial accounts (if needed)
- [ ] Authenticate apps to Zapier
- [ ] Build the trigger step
- [ ] Add action steps one at a time
- [ ] Test with sample data
- [ ] Adjust field mappings
- [ ] Go live
- [ ] Monitor first 3 runs
- [ ] Refine as needed

---

## Common Customizations

### Add Email Delays
```
Insert a "Delay" step between actions to space out communications (e.g., 24 hours between emails)
```

### Add Conditions
```
Use "Conditions" to route based on criteria:
- If value > 100, send to finance
- If domain = "enterprise", flag as important
```

### Add Formatting
```
Use "Formatter" step to:
- Format dates (MM/DD/YYYY)
- Extract text (split "first.last@company" into first/last name)
- Convert case (uppercase, lowercase)
```

### Add Slack Rich Formatting
```
In Slack action, use blocks for rich formatting:
{
  "blocks": [
    {
      "type": "section",
      "text": {
        "type": "mrkdwn",
        "text": "New lead: *{Lead Name}*\n<{Link}|View in CRM>"
      }
    }
  ]
}
```

---

## Troubleshooting Tips

### "Authentication Failed"
- Ensure you have correct permissions in connected app
- Re-authenticate (disconnect & reconnect)
- Check that app requires no 2FA during Zapier auth

### "Field Not Found"
- Ensure correct app/action selected
- Fields may differ between versions (Zapier may cache)
- Disconnect and re-authenticate

### "Zap Turned Off"
- Zapier auto-disables zaps after 5 consecutive errors
- Check error message in zap history
- Fix issue and manually turn zap back on

### "Rate Limiting"
- If syncing high volume, add delays between actions
- Consider using native integrations instead (faster)
- Upgrade Zapier plan for higher volume

---

## Tips for Success

1. **Start Simple:** Pick one workflow, get it working, then expand
2. **Use Naming:** Name your zaps clearly ("Lead Capture" not "Untitled Zap 1")
3. **Test First:** Use test data before going live
4. **Monitor:** Check logs first week to catch issues
5. **Iterate:** Refine based on real-world use
6. **Document:** Keep notes on setup and customizations

---

## About Rework Digital

This starter kit was created by **Rework Digital** - Resources Department for automation professionals.

**Resources Department Contact:** resource@reworkdigital.io  
**Follow us on GitHub:** https://github.com/Reworkdigital-io

---

*Last Updated: 2026-04-10*  
*Version: 1.0*
