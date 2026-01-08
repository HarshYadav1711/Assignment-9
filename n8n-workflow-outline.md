# n8n Workflow Outline: Instant Real Estate Deal Engine

## Workflow Name
**Instant Real Estate Deal Engine**

---

## Workflow Overview

This workflow processes raw real estate listing text, extracts structured data using AI, validates the verdict, and stores results in Airtable.

**Data Flow:**
```
Webhook Trigger → Input Validation (IF) → OpenAI Extraction → Verdict Validation (IF) → Airtable Create Record
                    ↓ (invalid input)        ↓ (error output)     ↓ (invalid verdict/JSON)
              Airtable (HUMAN REVIEW) → Format Error Record → Airtable (HUMAN REVIEW)
                                                  ↑
                                    (OpenAI errors/timeouts/invalid JSON)
```

---

## Nodes Configuration

### 1. Webhook Trigger
**Node Name:** `Webhook - Receive Listing`

**Configuration:**
- **HTTP Method:** POST
- **Path:** `real-estate-deal`
- **Response Mode:** Last Node
- **Options:** Default settings

**Input:** 
- Raw JSON payload with `listing_text` field containing real estate listing
- Expected format: `{ "listing_text": "<raw property description>" }`

**Output:**
- `$json.body.listing_text` - The listing text to process
- `$json.body` - Full request body

**Purpose:** Receives incoming real estate listing data via HTTP POST request.

---

### 2. Input Validation Node
**Node Name:** `Validate Listing Text Input`

**Type:** IF Node

**Condition:**
```
{{ $json.body.listing_text && $json.body.listing_text.trim().length > 0 }}
```

**True Path (Valid Input):**
- Output 0 (true branch)
- Continues to OpenAI Extraction node
- Data passes through unchanged

**False Path (Invalid Input - Missing or Empty):**
- Output 1 (false branch)
- Routes to Airtable Create Record (HUMAN REVIEW path)
- Creates record with verdict: 🟡 HUMAN REVIEW
- Skips OpenAI call to save costs and latency

**Purpose:** Validates that `listing_text` is present and non-empty before processing, preventing wasted API calls and ensuring data quality.

---

### 3. Airtable Create Record (HUMAN REVIEW Path)
**Node Name:** `Airtable - Save HUMAN REVIEW (Invalid Input)`

**Configuration:**
- **Operation:** Create Record
- **Base:** [Airtable Base ID - from credentials/environment]
- **Table:** `Deal Opportunities`

**Field Mappings (for invalid input path):**
| Airtable Field | n8n Expression |
|----------------|----------------|
| Property Title | `{{ "Input Validation Required - " + ($json.body.listing_text || "No listing text provided") }}` |
| Asking Price | `{{ 0 }}` |
| Renovation Estimate | `{{ null }}` |
| Is Airbnb Legal? | `{{ false }}` |
| AI Verdict | `{{ "🟡 HUMAN REVIEW" }}` |
| AI Summary | `{{ "Input validation failed: listing_text is missing or empty. Manual review required to process this submission." }}` |

**When Used:**
- Called directly from Input Validation node (false path)
- Creates record immediately without OpenAI processing
- Ensures no data is lost even for invalid inputs

**Purpose:** Captures invalid submissions in Airtable with HUMAN REVIEW verdict, ensuring all inputs are tracked and nothing is silently dropped.

---

### 4. OpenAI Node
**Node Name:** `OpenAI - Extract Deal Data`

**Configuration:**
- **Model:** gpt-4o (or gpt-4o-mini for faster/lower cost)
- **Temperature:** 0 (deterministic output)
- **Response Format:** JSON Object (structured output)
- **Max Tokens:** 1000
- **Continue on Fail:** Enabled (routes errors to error output instead of failing workflow)

**System Message:**
```
You are a JSON extraction system. Extract real estate data and return ONLY valid JSON. No markdown, no explanations, no additional text. Output must be parseable JSON only.
```

**User Message:**
```
[Use the strict prompt from openai-prompt-strict.md]
```

**Output (Success Path):**
- OpenAI API response structure: `$json.choices[0].message.content` (contains JSON string)
- Parse with: `JSON.parse($json.choices[0].message.content)` to access fields

**Error Output:**
- Connected to "Format Error Record" node
- Contains error information: `$json.error.message`, `$json.error.name`
- Also receives original input data from webhook

**Purpose:** Uses AI to extract structured deal data and generate verdict from raw listing text. Errors are routed to error handling path.

---

### 5. Parse & Validate JSON Node
**Node Name:** `Parse JSON and Validate Verdict`

**Type:** Code Node (with error handling)

**Configuration:**
- Wraps JSON parsing in try-catch
- Validates verdict value
- Routes to appropriate path

**JavaScript Code:**
```javascript
// Get OpenAI response
const openaiResponse = $input.item.json;
let parsedData;
let errorMessage = null;

try {
  // Parse JSON from OpenAI response
  const jsonString = openaiResponse.choices?.[0]?.message?.content;
  
  if (!jsonString) {
    throw new Error('OpenAI response missing content');
  }
  
  parsedData = JSON.parse(jsonString);
  
  // Validate required fields
  if (!parsedData.hasOwnProperty('ai_verdict')) {
    throw new Error('Missing ai_verdict field in response');
  }
  
  // Validate verdict value
  const validVerdicts = ['🟢 BUY', '🔴 PASS', '🟡 HUMAN REVIEW'];
  if (!validVerdicts.includes(parsedData.ai_verdict)) {
    throw new Error(`Invalid verdict: ${parsedData.ai_verdict}`);
  }
  
  // Success - return parsed data
  return {
    success: true,
    data: parsedData,
    originalResponse: openaiResponse
  };
  
} catch (error) {
  // JSON parsing or validation failed
  errorMessage = error.message;
  
  // Return error data for HUMAN REVIEW path
  return {
    success: false,
    error: errorMessage,
    errorType: 'JSON_PARSE_OR_VALIDATION_ERROR',
    originalResponse: openaiResponse,
    listingText: $('Webhook - Receive Listing').item.json.body.listing_text
  };
}
```

**Output:**
- Success case: `{ success: true, data: {...}, originalResponse: {...} }`
- Error case: `{ success: false, error: "...", errorType: "...", listingText: "..." }`

**Purpose:** Safely parses JSON and validates verdict, catching parsing errors and routing them to HUMAN REVIEW path.

---

### 6. Route Based on Parse Result Node
**Node Name:** `Route Parse Result`

**Type:** IF Node

**Condition:**
```
{{ $json.success === true }}
```

**True Path (Parse Success):**
- Output 0 (true branch)
- Continues to Airtable Create Record (Valid Path)
- Contains parsed data in `$json.data`

**False Path (Parse Error):**
- Output 1 (false branch)
- Routes to Format Error Record node
- Contains error information in `$json.error`

**Purpose:** Routes successful parses to normal flow, errors to HUMAN REVIEW path.

---

### 7. Format Error Record Node
**Node Name:** `Format Error Record for HUMAN REVIEW`

**Type:** Code Node

**Configuration:**
- Formats error data into Airtable record structure
- Handles both OpenAI API errors and JSON parsing errors

**JavaScript Code:**
```javascript
const inputData = $input.item.json;

// Determine error source and message
let errorType = 'Unknown Error';
let errorMessage = 'AI processing failed';
let listingText = '';

if (inputData.errorType === 'JSON_PARSE_OR_VALIDATION_ERROR') {
  // JSON parsing/validation error from Parse node
  errorType = 'JSON Parse/Validation Error';
  errorMessage = inputData.error || 'Failed to parse or validate OpenAI response';
  listingText = inputData.listingText || '';
} else if (inputData.error) {
  // OpenAI API error (from error output)
  errorType = 'OpenAI API Error';
  errorMessage = inputData.error.message || inputData.error.name || 'OpenAI API call failed';
  listingText = $('Webhook - Receive Listing').item.json.body.listing_text || '';
} else {
  // Fallback
  listingText = $('Webhook - Receive Listing').item.json.body.listing_text || '';
}

// Extract property title from listing text (first 50 chars or "Unknown")
const propertyTitle = listingText 
  ? (listingText.substring(0, 50).trim() + (listingText.length > 50 ? '...' : ''))
  : 'Unknown Property';

// Format summary
const summary = `AI processing failed: ${errorMessage}. Error type: ${errorType}. Manual review required.`;

// Return formatted record with correct data types
return {
  property_title: propertyTitle,  // String
  asking_price: null,              // Number (null) - Currency field
  renovation_estimate: null,       // Number (null) - Currency field
  airbnb_legal: false,             // Boolean (false) - Checkbox field (not null)
  ai_verdict: '🟡 HUMAN REVIEW',  // String (exact match for single select)
  ai_summary: summary,             // String
  original_listing_text: listingText
};
```

**Output:**
- Formatted object matching Airtable schema
- All fields populated with defaults/error info
- Verdict always: 🟡 HUMAN REVIEW

**Purpose:** Formats error information into Airtable record structure for HUMAN REVIEW path.

---

### 8. Airtable Create Record Node (Valid Path)
**Node Name:** `Airtable - Save Deal Opportunity`

**Configuration:**
- **Operation:** Create Record
- **Base:** [Airtable Base ID - from credentials/environment]
- **Table:** `Deal Opportunities`

**Field Mappings (from Parse node success path):**
| Airtable Field | Airtable Type | n8n Expression | Data Type Notes |
|----------------|---------------|----------------|-----------------|
| Property Title | Single line text | `{{ $json.data.property_title }}` | String - direct mapping |
| Asking Price | Currency (USD) | `{{ $json.data.asking_price ?? null }}` | **Number** (not string) - null allowed |
| Renovation Estimate | Currency (USD) | `{{ $json.data.renovation_estimate ?? null }}` | **Number** (not string) - null allowed |
| Is Airbnb Legal? | Checkbox | `{{ $json.data.airbnb_legal ?? false }}` | **Boolean** (not string/number) - defaults to false |
| AI Verdict | Single select | `{{ $json.data.ai_verdict }}` | **Exact match required** (emojis included: 🟢 BUY, 🔴 PASS, 🟡 HUMAN REVIEW) |
| AI Summary | Long text | `{{ $json.data.ai_summary }}` | String - direct mapping |

**Critical Type Requirements:**
- **Currency fields:** Must be numbers (e.g., `250000`), NOT strings (e.g., `"250000"`)
- **Checkbox field:** Must be boolean (`true`/`false`), NOT strings (`"true"`/`"false"`) or numbers (`1`/`0`)
- **Single select:** Must match exactly including emojis and spacing (e.g., `"🟢 BUY"` not `"BUY"` or `"🟢BUY"`)

See `airtable-field-mapping-guide.md` for detailed mapping documentation and common mistakes.

**Output:**
- Created record with Airtable record ID
- All field values as stored

**Purpose:** Saves the validated deal opportunity to Airtable base for dashboard viewing and further analysis.

---

### 9. Airtable Create Record Node (Error Path)
**Node Name:** `Airtable - Save HUMAN REVIEW (Error)`

**Configuration:**
- **Operation:** Create Record
- **Base:** [Airtable Base ID - from credentials/environment]
- **Table:** `Deal Opportunities`

**Field Mappings (from Format Error Record node):**
| Airtable Field | Airtable Type | n8n Expression | Data Type Notes |
|----------------|---------------|----------------|-----------------|
| Property Title | Single line text | `{{ $json.property_title }}` | String - direct mapping |
| Asking Price | Currency (USD) | `{{ $json.asking_price }}` | Number (null) - Currency field accepts null |
| Renovation Estimate | Currency (USD) | `{{ $json.renovation_estimate }}` | Number (null) - Currency field accepts null |
| Is Airbnb Legal? | Checkbox | `{{ $json.airbnb_legal }}` | Boolean (false) - Checkbox defaults to unchecked |
| AI Verdict | Single select | `{{ $json.ai_verdict }}` | Exact match: `"🟡 HUMAN REVIEW"` (emojis included) |
| AI Summary | Long text | `{{ $json.ai_summary }}` | String - error description |

**When Used:**
- Called from Format Error Record node
- Receives formatted error data
- Always creates record with HUMAN REVIEW verdict

**Purpose:** Saves error cases to Airtable with HUMAN REVIEW verdict, ensuring no data is lost when OpenAI fails.

---

## Complete Data Flow

1. **Webhook receives POST request** with `listing_text` in `body.listing_text`
2. **Input Validation checks** if `listing_text` exists and is non-empty
3. **If input is invalid (missing/empty):**
   - Routes to Airtable Create Record (HUMAN REVIEW path - Invalid Input)
   - Creates record with verdict: 🟡 HUMAN REVIEW
   - Summary explains validation failure
   - Workflow completes gracefully (no errors)

4. **If input is valid:**
   - Continues to OpenAI node for data extraction
   
5. **OpenAI Processing:**
   - **Success path:** Returns structured JSON in `choices[0].message.content`
   - **Error path (timeout/API failure):** Routes to error output → Format Error Record node
   
6. **Parse & Validate JSON:**
   - Safely parses JSON with try-catch
   - Validates verdict value
   - **Success:** Returns parsed data
   - **Error (invalid JSON/parse failure):** Returns error info

7. **Route Parse Result:**
   - **Success path:** Routes to Airtable Create Record (Valid Path)
   - **Error path:** Routes to Format Error Record node

8. **Format Error Record (if error occurred):**
   - Formats error information into Airtable record structure
   - Sets verdict to: 🟡 HUMAN REVIEW
   - Creates descriptive summary explaining the error

9. **Airtable Create Record:**
   - **Valid path:** Creates record with AI-extracted data
   - **Error path:** Creates record with HUMAN REVIEW verdict and error summary
   - Both paths complete successfully (workflow never fails)

---

## Error Handling Strategy

- **Invalid Input (Missing/Empty listing_text):** Caught by Input Validation node, routed to Airtable with HUMAN REVIEW verdict (graceful handling, no error)
- **OpenAI API Errors (timeout/failure/rate limit):** Caught by OpenAI node error output, routed to Format Error Record → Airtable with HUMAN REVIEW verdict
- **JSON Parse Errors:** Caught by Parse & Validate JSON node try-catch, routed to Format Error Record → Airtable with HUMAN REVIEW verdict
- **Invalid Verdict/JSON Structure:** Caught by Parse & Validate JSON node validation, routed to Format Error Record → Airtable with HUMAN REVIEW verdict
- **Airtable Errors:** n8n native error handling (connection/field errors) - workflow may fail, but all prior errors are handled gracefully

**Key Principle:** All OpenAI-related errors create Airtable records with HUMAN REVIEW verdict, ensuring workflow completion and data visibility.

---

## Error Handling & Reliability Protection

### How Error Handling Prevents Workflow Failure While Preserving Visibility

1. **Graceful Degradation:**
   - OpenAI failures (timeouts, API errors, rate limits) don't crash the workflow
   - Errors are captured and routed to Airtable with HUMAN REVIEW verdict
   - Workflow execution completes successfully (marked as "Success" in n8n)
   - No manual intervention needed to recover from errors

2. **Zero Data Loss:**
   - Every submission creates an Airtable record, even when OpenAI fails
   - Error cases are visible in Airtable with clear error descriptions
   - Operations team can review and manually process failed cases
   - All submissions are tracked and auditable

3. **Operational Visibility:**
   - Error records appear in "Human Review Queue" view in Airtable
   - AI Summary field clearly explains what failed (e.g., "AI processing failed: OpenAI API timeout")
   - Error types are distinguishable (API error vs JSON parse error)
   - Team can identify patterns (e.g., frequent timeouts indicate need for optimization)

4. **Workflow Stability:**
   - Workflow execution history shows "Success" status (no error states)
   - No need to manually restart failed workflows
   - Monitoring systems see consistent success rates
   - Debugging is easier (all cases appear in Airtable, not just successful ones)

5. **Cost Efficiency:**
   - Failed OpenAI calls don't waste retries if error is permanent
   - Errors are handled once, then routed to human review
   - No duplicate processing attempts for known failures

6. **User Experience:**
   - Webhook requests always return successfully (record created)
   - No 500 errors or timeouts returned to API clients
   - Clients receive confirmation that submission was received
   - Processing status visible in Airtable dashboard

### Error Handling Architecture

**Three Layers of Error Handling:**

1. **Input Validation Layer:** Catches invalid inputs before API calls
2. **OpenAI Error Output:** Catches API failures/timeouts at source
3. **JSON Parse/Validation Layer:** Catches malformed responses and invalid data

**All layers route to same destination:** Airtable record with HUMAN REVIEW verdict

**Result:** Workflow never fails due to OpenAI issues, and all errors are visible and actionable.

---

## Input Validation & Reliability Protection

### How Input Validation Protects Reliability

1. **Prevents Wasted API Calls:**
   - Invalid inputs (missing/empty `listing_text`) are caught before OpenAI call
   - Saves API costs and reduces latency
   - Prevents OpenAI from processing meaningless data

2. **Ensures No Data Loss:**
   - Invalid inputs are still captured in Airtable with HUMAN REVIEW verdict
   - All submissions are tracked and visible in dashboard
   - Operations team can review and manually process invalid inputs

3. **Graceful Failure Handling:**
   - Workflow does not crash or throw errors for invalid input
   - Returns successful response (record created)
   - Maintains workflow execution history and debugging capability

4. **Maintains Data Integrity:**
   - Invalid inputs don't corrupt database with malformed records
   - HUMAN REVIEW verdict clearly flags records needing attention
   - Consistent data structure across all records (valid and invalid)

5. **Operational Visibility:**
   - Invalid inputs appear in "Human Review Queue" view in Airtable
   - Operations team can identify and fix data quality issues
   - Metrics on input validation failure rate are trackable

6. **Cost Optimization:**
   - Skips expensive OpenAI API call for invalid inputs
   - Reduces unnecessary processing
   - Maintains < 30 second latency requirement (invalid inputs complete in ~2 seconds)

### Reliability Benefits Summary

- ✅ **No silent failures** - All inputs create records
- ✅ **No workflow crashes** - Invalid inputs handled gracefully  
- ✅ **Cost efficient** - No wasted API calls
- ✅ **Fast response** - Invalid inputs complete quickly
- ✅ **Audit trail** - All submissions tracked
- ✅ **Clear visibility** - HUMAN REVIEW verdict signals issues

---

## Workflow Settings

- **Execution Timeout:** 30 seconds (meets latency requirement)
- **Error Workflow:** Optional - can trigger separate error workflow
- **Save Data:** Enable for debugging
- **Save Execution Data:** Enabled

---

## Integration Points

- **Credentials Required:**
  - OpenAI API (configured in OpenAI node)
  - Airtable API (configured in Airtable node)

- **Environment Variables (Optional):**
  - `AIRTABLE_BASE_ID` - Base identifier
  - `OPENAI_MODEL` - Model override (default: gpt-4o)

---

## Performance Considerations

- **Target Latency:** < 30 seconds end-to-end
- **OpenAI Call:** ~2-5 seconds (gpt-4o-mini faster)
- **Airtable Write:** ~1-2 seconds
- **Total Expected:** < 10 seconds (well under limit)
