# Error Handling Guide: OpenAI Failures

## Overview

The workflow includes comprehensive error handling to ensure OpenAI failures (timeouts, API errors, invalid JSON) are gracefully handled by creating Airtable records with HUMAN REVIEW verdict instead of failing the workflow.

---

## Error Scenarios Handled

### 1. OpenAI API Errors
- **Timeout:** API call exceeds time limit
- **Rate Limit:** API quota exceeded
- **Authentication Error:** Invalid API key
- **Network Error:** Connection failure
- **Service Unavailable:** OpenAI service down

### 2. JSON Parsing Errors
- **Invalid JSON:** Response not valid JSON format
- **Malformed Structure:** JSON doesn't match expected schema
- **Missing Fields:** Required fields absent from response

### 3. Validation Errors
- **Invalid Verdict:** Verdict value not in allowed set
- **Type Mismatches:** Fields have wrong data types

---

## Error Handling Flow

```
OpenAI Node
  ├─ Success → Parse & Validate JSON
  │              ├─ Success → Airtable (Valid Record)
  │              └─ Error → Format Error Record → Airtable (HUMAN REVIEW)
  │
  └─ Error Output → Format Error Record → Airtable (HUMAN REVIEW)
```

---

## Error Record Structure

When an error occurs, the workflow creates an Airtable record with:

| Field | Value |
|-------|-------|
| **Property Title** | First 50 chars of listing text (or "Unknown Property") |
| **Asking Price** | `null` |
| **Renovation Estimate** | `null` |
| **Is Airbnb Legal?** | `null` |
| **AI Verdict** | `🟡 HUMAN REVIEW` |
| **AI Summary** | Error description (e.g., "AI processing failed: OpenAI API timeout. Error type: OpenAI API Error. Manual review required.") |

---

## Example Error Scenarios

### Scenario 1: OpenAI API Timeout

**Error:** OpenAI API call times out after 30 seconds

**Flow:**
1. OpenAI node error output triggers
2. Format Error Record node receives error
3. Creates Airtable record with:
   - Verdict: 🟡 HUMAN REVIEW
   - Summary: "AI processing failed: Request timeout. Error type: OpenAI API Error. Manual review required."

**Result:** Workflow completes successfully, record visible in Airtable for manual processing.

---

### Scenario 2: Invalid JSON Response

**Error:** OpenAI returns malformed JSON that cannot be parsed

**Flow:**
1. OpenAI returns response (no API error)
2. Parse & Validate JSON node attempts JSON.parse()
3. Parse fails, catch block executes
4. Routes to Format Error Record node
5. Creates Airtable record with:
   - Verdict: 🟡 HUMAN REVIEW
   - Summary: "AI processing failed: Unexpected token in JSON. Error type: JSON Parse/Validation Error. Manual review required."

**Result:** Workflow completes successfully, record visible in Airtable.

---

### Scenario 3: Missing Verdict Field

**Error:** OpenAI returns valid JSON but missing `ai_verdict` field

**Flow:**
1. OpenAI returns response
2. Parse & Validate JSON node parses JSON successfully
3. Validation checks for `ai_verdict` field
4. Field missing, validation fails
5. Routes to Format Error Record node
6. Creates Airtable record with error summary

**Result:** Workflow completes successfully, record visible in Airtable.

---

### Scenario 4: Rate Limit Exceeded

**Error:** OpenAI API returns 429 Rate Limit error

**Flow:**
1. OpenAI node receives 429 error
2. Error output triggers (if "Continue on Fail" enabled)
3. Routes to Format Error Record node
4. Creates Airtable record with:
   - Verdict: 🟡 HUMAN REVIEW
   - Summary: "AI processing failed: Rate limit exceeded. Error type: OpenAI API Error. Manual review required."

**Result:** Workflow completes successfully, operations team can retry manually or wait for rate limit reset.

---

## Benefits of This Approach

### 1. Workflow Never Fails
- All errors are caught and handled gracefully
- Workflow execution status: "Success"
- No manual intervention needed to restart workflows

### 2. Complete Visibility
- All errors are visible in Airtable
- Error descriptions explain what went wrong
- Operations team can identify patterns and issues

### 3. No Data Loss
- Every submission creates a record
- Failed cases can be manually processed
- Audit trail of all submissions

### 4. Operational Efficiency
- No need to monitor for workflow failures
- All cases appear in single Airtable view
- Human review queue shows all cases needing attention

### 5. Better User Experience
- API calls always return success (record created)
- Clients don't see 500 errors
- Processing status visible in dashboard

---

## Monitoring Error Rates

### Airtable Views for Monitoring

1. **Human Review Queue:**
   - Filter: `AI Verdict = 🟡 HUMAN REVIEW`
   - Sort by: Created date (newest first)
   - Shows all cases needing review

2. **Error Analysis:**
   - Filter: `AI Summary` contains "AI processing failed"
   - Group by: First 30 chars of AI Summary
   - Identifies common error patterns

3. **Error Rate Dashboard:**
   - Count records with HUMAN REVIEW verdict
   - Compare to total submissions
   - Track error rate trends

### Key Metrics to Track

- **Error Rate:** % of submissions with HUMAN REVIEW due to errors
- **Error Types:** Distribution of error types (API timeout, JSON parse, etc.)
- **Time to Resolution:** How long error cases sit in review queue
- **Retry Success Rate:** % of error cases successfully processed after manual review

---

## Configuration Requirements

### OpenAI Node Settings

1. **Continue on Fail:** Must be enabled
   - Allows error output to trigger
   - Prevents workflow from failing

2. **Timeout Settings:** Configure appropriate timeout
   - Default: 30 seconds
   - Adjust based on model and expected response time

3. **Retry Settings:** Optional
   - Can disable retries to fail fast
   - Or enable retries with limit (e.g., 1 retry)

### Error Output Connection

- **Required:** Connect OpenAI node error output to Format Error Record node
- **Critical:** This connection enables graceful error handling

---

## Testing Error Handling

### Test Cases

1. **Simulate API Timeout:**
   - Use invalid API key (will fail quickly)
   - Verify record created with HUMAN REVIEW

2. **Simulate Invalid JSON:**
   - Mock OpenAI response with invalid JSON
   - Verify parse error caught and recorded

3. **Simulate Rate Limit:**
   - Send multiple rapid requests
   - Verify rate limit errors handled gracefully

### Verification Checklist

- [ ] OpenAI timeout creates HUMAN REVIEW record
- [ ] Invalid JSON creates HUMAN REVIEW record
- [ ] Missing fields create HUMAN REVIEW record
- [ ] Workflow execution shows "Success" status
- [ ] Error records appear in Human Review Queue
- [ ] Error summaries are descriptive and actionable

---

## Troubleshooting

### Issue: Errors still cause workflow failure

**Solution:** Verify "Continue on Fail" is enabled on OpenAI node and error output is connected.

### Issue: Error records missing listing text

**Solution:** Ensure Format Error Record node accesses webhook data correctly using `$('Webhook - Receive Listing')`.

### Issue: Error summaries not descriptive

**Solution:** Check that error messages are properly extracted and formatted in Format Error Record node.

---

## Best Practices

1. **Monitor Error Rates:** Set up alerts for high error rates
2. **Review Error Patterns:** Identify common issues and address root causes
3. **Optimize Timeouts:** Adjust based on actual API performance
4. **Document Error Types:** Maintain list of known error scenarios
5. **Train Operations Team:** Ensure team knows how to process HUMAN REVIEW cases
