# Building Multi-Step Workflows with Error Handling

## Overview
Moving beyond simple workflows, this guide covers creating reliable, production-ready automations that handle failures gracefully and execute complex business logic.

**Core Principle:** Expect failures and plan for them.

---

## Part 1: Workflow Architecture

### Simple Workflow (Not Recommended)
```
Trigger → Action 1 → Action 2 → Done
```

**Problem:** If Action 1 fails, Action 2 never runs (silent failure)

### Robust Workflow (Recommended)
```
Trigger
  ↓
Try Action 1
  ├─ Success → Continue to Action 2
  └─ Failure → Error handler
      ├─ Retry 3 times
      ├─ If still failing → Alert admin
      ├─ Log error
      └─ Stop workflow
```

---

## Part 2: Conditional Logic

### If-Then Branching

**Scenario:** Different action based on data value

```
New order received
  ↓
Check order amount
  ├─ If amount > $1000:
  │   ├─ Send to manager approval
  │   └─ Set priority: HIGH
  ├─ If amount $100-1000:
  │   ├─ Send auto-approval
  │   └─ Set priority: MEDIUM
  └─ If amount < $100:
      ├─ Process automatically
      └─ Set priority: LOW
```

### Multiple Conditions

```
Process form submission
  ├─ If email is valid AND name is not empty:
  │   └─ Add to CRM
  └─ Else:
      └─ Send "Missing information" email
```

### Complex Logic

```
IF (customer_tier = "premium" AND order_value > 500 AND inventory > 10)
  THEN: Rush fulfillment
ELSE IF (customer_tier = "standard" AND order_value > 100)
  THEN: Standard fulfillment
ELSE:
  THEN: Send low-inventory message
```

---

## Part 3: Loops & Iteration

### Loop Through Multiple Records

**Scenario:** Process multiple items from a list

```
Trigger: Receive list of 10 customers

Loop (for each customer):
  ├─ Action 1: Validate email
  ├─ Action 2: Check if duplicate
  ├─ Action 3: Add to CRM
  └─ Continue to next customer

After loop: Send summary email
```

### Conditional Loop

```
While (attempts < 3):
  ├─ Try API call
  ├─ If success: Break out of loop
  ├─ If failure: Increment attempts
  └─ Wait 5 seconds before retry

If still failed after 3 attempts:
  └─ Send alert
```

---

## Part 4: Error Handling Strategies

### Error Types

**Temporary Errors (Retry-able):**
- Network timeout
- Service temporarily unavailable
- Rate limited
- Connection refused

**Permanent Errors (Not retry-able):**
- Invalid API key
- Resource not found (404)
- Invalid data format
- Access denied

**Unknown Errors (Need investigation):**
- Unexpected response format
- Unusual response content
- Strange side effects

---

### Error Detection

```
Make API call
  ↓
Check response:
  ├─ Status 200-299: Success
  ├─ Status 400-499: Likely permanent error
  ├─ Status 500-599: Likely temporary error
  ├─ No response: Timeout (temporary)
  └─ Other: Unknown error
```

---

## Part 5: Retry Strategies

### Simple Retry

```
Attempt 1: Try
  ├─ Success: Done
  └─ Failure: Wait 2 sec

Attempt 2: Try
  ├─ Success: Done
  └─ Failure: Wait 5 sec

Attempt 3: Try
  ├─ Success: Done
  └─ Failure: Alert admin
```

### Exponential Backoff

Wait longer between each retry:

```
Attempt 1 fails: Wait 2 seconds
Attempt 2 fails: Wait 4 seconds
Attempt 3 fails: Wait 8 seconds
Attempt 4 fails: Alert admin
```

**Why:** Gives server time to recover, prevents hammering.

### Circuit Breaker

Stop retrying if too many failures:

```
Failures in last hour:
  ├─ < 5: Keep retrying
  ├─ 5-10: Increase wait time
  └─ > 10: Stop, alert admin, stop workflow
```

**Why:** Prevents cascading failures.

---

## Part 6: Fallback Actions

### Fallback to Alternative Service

```
Try primary API
  ├─ Success: Use response
  └─ Failure:
      └─ Try backup API
          ├─ Success: Use response
          └─ Failure: Use cached data
                  └─ Failure: Alert + stop
```

### Fallback to Manual Process

```
Try automated processing
  ├─ Success: Continue
  └─ Failure:
      ├─ Create task for human review
      ├─ Assign to appropriate person
      └─ Notify them via email
```

### Fallback to Default Value

```
Try to get data
  ├─ Success: Use it
  └─ Failure: Use default
      ├─ Status: "pending"
      ├─ Priority: "medium"
      └─ Owner: "admin"
```

---

## Part 7: Logging & Monitoring

### What to Log

```json
{
  "timestamp": "2024-04-10T15:30:45Z",
  "workflow_id": "order_processing_v2",
  "execution_id": "exec_12345",
  "trigger": "new_order",
  "steps": [
    {"step": "validate", "status": "success", "duration": "245ms"},
    {"step": "check_inventory", "status": "success", "duration": "1200ms"},
    {"step": "process_payment", "status": "failed", "error": "timeout", "duration": "30000ms"}
  ],
  "final_status": "failed",
  "total_duration": "31445ms"
}
```

### Key Metrics

- **Execution time:** How long the workflow takes
- **Success rate:** % of runs that succeed
- **Failure types:** What errors occur most
- **Bottlenecks:** Which steps take longest
- **Error location:** Where failures happen

---

### Alerting

Send alerts for:
```
- Workflow failed after all retries
- Multiple consecutive failures (> 3 in a row)
- Unusually slow execution (> 5x normal)
- Unusual error type occurs
- System resources exhausted
```

---

## Part 8: Testing Workflows

### Before Production

**Unit Test:** Test each action independently
```
Does the API call work?
Does the conditional logic work?
Does the retry logic work?
```

**Integration Test:** Test workflow end-to-end
```
With real data (not production)
With all error scenarios
With edge cases
With high volume
```

**Load Test:** Can it handle volume?
```
Run 100 workflows simultaneously
Run 1000 workflows in rapid succession
Monitor system resources
```

---

### Test Scenarios

**Happy Path:**
```
Input: Valid data
Expected: Successful completion
Verify: Correct output
```

**Error Paths:**
```
Scenario 1: Invalid data → Should reject gracefully
Scenario 2: API down → Should retry and alert
Scenario 3: Timeout → Should retry
Scenario 4: Rate limit → Should back off
```

**Edge Cases:**
```
Empty input
Very large input
Special characters
Duplicate submissions
Concurrent execution
```

---

## Part 9: Debugging Workflow Failures

### Step 1: Check Logs
- Find the execution that failed
- Look at each step
- Which step failed first?
- What was the error message?

### Step 2: Reproduce
- Collect the input that caused failure
- Run the workflow with that input
- Does it fail consistently?
- Or was it intermittent?

### Step 3: Isolate
- Test each step independently
- Is it the API? The data? The logic?
- Test with simpler data
- Test with known-good API

### Step 4: Fix & Verify
- Apply fix
- Test with original failing data
- Test with 10 more similar inputs
- Deploy fix
- Monitor for recurrence

---

## Part 10: Production Best Practices

### Deployment Checklist
- [ ] Code reviewed
- [ ] Tests pass (unit + integration + load)
- [ ] Error handling comprehensive
- [ ] Logging in place
- [ ] Alerts configured
- [ ] Monitoring dashboard set up
- [ ] Runbook documented
- [ ] Team trained
- [ ] Rollback plan ready

### Monitoring Dashboard
Track:
- Success vs failure rate
- Execution time trends
- Error type distribution
- Throughput (workflows/hour)
- System resource usage
- Alert triggers

### Incident Response
```
1. Detect: Alert fires or user reports issue
2. Investigate: Check logs, reproduced problem
3. Mitigate: Stop problematic workflow or disable
4. Fix: Make code changes
5. Verify: Test thoroughly
6. Deploy: Roll out fix
7. Monitor: Watch for recurrence
8. Document: Update runbook
9. Review: Post-mortem with team
```

---

## Part 11: Example: Robust Order Processing

```
Trigger: New order received

Step 1: Validate Order
  ├─ Check: Email valid? Phone valid? Address valid?
  ├─ If invalid: Mark "needs review", alert operator
  └─ If valid: Continue

Step 2: Check Inventory
  Try:
    └─ Query inventory system
  Retry: 3 times with exponential backoff
  Fallback: Use cached inventory data
  Error: If all fail, alert manager

Step 3: Process Payment
  Try:
    └─ Charge customer
  Retry: 2 times (payments especially fail-safe)
  Fallback: Send manual payment request
  Error: If all fail, create "payment failed" task

Step 4: Ship Order
  Try:
    └─ Send to fulfillment
  Retry: 3 times
  Error: Create manual fulfillment task

Step 5: Notify Customer
  Email: Order confirmation
  Fallback: SMS if email fails
  Always: Log notification status

Step 6: Final Cleanup
  ├─ Log full workflow execution
  ├─ Record timing
  └─ Update metrics

Error throughout workflow:
  ├─ Log the error with context
  ├─ Alert operator if critical
  └─ Don't proceed beyond failure point
```

---

## Summary

**Building Robust Workflows:**
- Assume failures will occur
- Plan for each error type
- Use retry + fallback strategies
- Log everything
- Monitor closely
- Test thoroughly before production

**Error Handling:**
- Distinguish temporary vs permanent errors
- Implement exponential backoff for retries
- Provide fallback options
- Alert when human intervention needed

**Key Takeaway:**
Production-ready workflows handle failures gracefully, log comprehensively, and provide clear visibility into system behavior.

---

## Resources

- Zapier Error Handling: https://zapier.com/help/create/code-webhooks/handle-errors-in-zapier
- Make Error Handling: https://www.make.com/en/help/app-reference/tools/error-handler
- n8n Error Handling: https://docs.n8n.io/nodes/n8n-nodes-base.switch/
- Best Practices: https://www.sitepoint.com/error-handling-best-practices/

---

*This guide was created by **Rework Digital** - Resources Department for automation professionals.*

Questions? Reach out: resource@reworkdigital.io | Follow on GitHub: https://github.com/Reworkdigital-io
