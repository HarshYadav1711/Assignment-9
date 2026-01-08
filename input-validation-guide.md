# Input Validation Guide

## Overview

The workflow includes input validation to handle missing or empty `listing_text` fields gracefully, routing them to HUMAN REVIEW instead of causing errors.

---

## Expected Webhook Payload

**Format:**
```json
{
  "listing_text": "<raw property description>"
}
```

**Example:**
```json
{
  "listing_text": "Beautiful 3-bedroom house in Austin, TX. $250,000. 1,500 sqft. 2 bathrooms. Perfect for families."
}
```

---

## Validation Rules

### Valid Input
- ✅ `listing_text` field exists in JSON payload
- ✅ `listing_text` is a string (not null)
- ✅ `listing_text` is not empty (after trimming whitespace)
- ✅ `listing_text` has length > 0

### Invalid Input (Triggers HUMAN REVIEW)
- ❌ `listing_text` field is missing from payload
- ❌ `listing_text` is `null` or `undefined`
- ❌ `listing_text` is empty string `""`
- ❌ `listing_text` contains only whitespace (e.g., `"   "`)

---

## Validation Logic

**IF Node Condition:**
```javascript
{{ $json.body.listing_text && $json.body.listing_text.trim().length > 0 }}
```

**How it works:**
1. Checks if `$json.body.listing_text` exists (truthy check)
2. Calls `.trim()` to remove leading/trailing whitespace
3. Checks if length is greater than 0
4. Returns `true` only if all conditions pass

---

## Behavior for Invalid Inputs

When validation fails, the workflow:

1. **Skips OpenAI call** (saves cost and time)
2. **Creates Airtable record** with:
   - **AI Verdict:** 🟡 HUMAN REVIEW
   - **Property Title:** "Input Validation Required - [input value or 'No listing text provided']"
   - **Asking Price:** 0
   - **Renovation Estimate:** null
   - **Is Airbnb Legal?:** false
   - **AI Summary:** "Input validation failed: listing_text is missing or empty. Manual review required to process this submission."

3. **Completes successfully** (no error thrown)
4. **Returns normal response** (workflow execution succeeds)

---

## Example Scenarios

### Scenario 1: Valid Input
**Request:**
```json
{
  "listing_text": "3-bedroom house, $300k, Austin TX"
}
```

**Result:**
- ✅ Passes validation
- ✅ Sent to OpenAI for processing
- ✅ Record created with AI-extracted data

---

### Scenario 2: Missing Field
**Request:**
```json
{
  "some_other_field": "value"
}
```

**Result:**
- ❌ Fails validation (listing_text missing)
- ✅ Record created with HUMAN REVIEW verdict
- ✅ Summary explains validation failure
- ✅ No OpenAI call made
- ✅ Workflow completes successfully

---

### Scenario 3: Empty String
**Request:**
```json
{
  "listing_text": ""
}
```

**Result:**
- ❌ Fails validation (empty string)
- ✅ Record created with HUMAN REVIEW verdict
- ✅ Summary explains validation failure
- ✅ No OpenAI call made
- ✅ Workflow completes successfully

---

### Scenario 4: Whitespace Only
**Request:**
```json
{
  "listing_text": "   "
}
```

**Result:**
- ❌ Fails validation (trimmed length = 0)
- ✅ Record created with HUMAN REVIEW verdict
- ✅ Summary explains validation failure
- ✅ No OpenAI call made
- ✅ Workflow completes successfully

---

## Reliability Benefits

### 1. Prevents API Waste
- Invalid inputs don't consume OpenAI API quota
- Saves costs on malformed requests
- Reduces latency (no API wait time)

### 2. Ensures Data Capture
- All submissions create records (no silent failures)
- Invalid inputs are visible in Airtable
- Operations team can review and correct

### 3. Maintains Workflow Stability
- No errors thrown for invalid input
- Workflow execution always succeeds
- Clean execution logs for debugging

### 4. Clear Operational Visibility
- HUMAN REVIEW verdict flags issues
- Summary explains what went wrong
- Easy to filter and find invalid inputs

### 5. Cost & Performance
- Invalid inputs complete in ~2 seconds (vs ~10 seconds with OpenAI)
- No unnecessary API costs
- Better resource utilization

---

## Testing Invalid Inputs

### Using cURL:
```bash
# Missing field
curl -X POST https://your-n8n-instance.com/webhook/real-estate-deal \
  -H "Content-Type: application/json" \
  -d '{}'

# Empty string
curl -X POST https://your-n8n-instance.com/webhook/real-estate-deal \
  -H "Content-Type: application/json" \
  -d '{"listing_text": ""}'

# Whitespace only
curl -X POST https://your-n8n-instance.com/webhook/real-estate-deal \
  -H "Content-Type: application/json" \
  -d '{"listing_text": "   "}'
```

### Expected Result:
- Workflow executes successfully
- Record created in Airtable with HUMAN REVIEW verdict
- No errors in execution log
- Response indicates success

---

## Monitoring & Metrics

### Key Metrics to Track:
1. **Validation Failure Rate:** % of requests with invalid input
2. **HUMAN REVIEW Queue Size:** Number of records needing manual review
3. **Input Quality Trends:** Track improvement over time

### Airtable Views for Monitoring:
- **Human Review Queue:** Filter by `AI Verdict = 🟡 HUMAN REVIEW`
- **Input Validation Failures:** Filter by `AI Summary` contains "Input validation failed"

---

## Best Practices

1. **Client-Side Validation:** Validate input before sending to webhook (improves UX)
2. **Clear Error Messages:** Use descriptive summaries in HUMAN REVIEW records
3. **Regular Review:** Monitor HUMAN REVIEW queue and address patterns
4. **Documentation:** Document expected format for API consumers
5. **Monitoring:** Set up alerts for high validation failure rates
