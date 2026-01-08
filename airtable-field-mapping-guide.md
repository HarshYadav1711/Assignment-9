# Airtable Field Mapping Guide

## Overview

This guide provides exact field mappings from AI JSON output to Airtable fields, ensuring proper data type handling and avoiding common insertion errors.

---

## Field Mapping Table

| Airtable Field | Field Type | JSON Source Field | Data Type | n8n Expression | Notes |
|----------------|------------|-------------------|-----------|----------------|-------|
| **Property Title** | Single line text | `property_title` | string | `{{ $json.data.property_title }}` | Direct string mapping |
| **Asking Price** | Currency (USD) | `asking_price` | number \| null | `{{ $json.data.asking_price ?? null }}` | Must be number, not string |
| **Renovation Estimate** | Currency (USD) | `renovation_estimate` | number \| null | `{{ $json.data.renovation_estimate ?? null }}` | Must be number, not string |
| **Is Airbnb Legal?** | Checkbox | `airbnb_legal` | boolean \| null | `{{ $json.data.airbnb_legal ?? false }}` | Must be boolean true/false |
| **AI Verdict** | Single select | `ai_verdict` | string | `{{ $json.data.ai_verdict }}` | Must match exactly (emojis included) |
| **AI Summary** | Long text | `ai_summary` | string | `{{ $json.data.ai_summary }}` | Direct string mapping |

---

## Detailed Field Mappings

### 1. Property Title
**Airtable Type:** Single line text  
**JSON Field:** `property_title`  
**Data Type:** string

**n8n Expression:**
```javascript
{{ $json.data.property_title }}
```

**Notes:**
- Direct string mapping
- No conversion needed
- Handles null/undefined gracefully (Airtable accepts empty strings)

---

### 2. Asking Price
**Airtable Type:** Currency (USD)  
**JSON Field:** `asking_price`  
**Data Type:** number | null

**n8n Expression:**
```javascript
{{ $json.data.asking_price ?? null }}
```

**Critical Requirements:**
- ✅ Must be a number type (250000)
- ❌ NOT a string ("250000")
- ✅ null is acceptable (represents unknown/missing)

**Example Values:**
- Valid: `250000` (number)
- Valid: `null`
- Invalid: `"250000"` (string - will cause error)
- Invalid: `"$250,000"` (string with formatting)

**Why This Matters:**
Airtable Currency fields expect numeric values. Passing a string like `"250000"` will cause the insert to fail with a type mismatch error.

---

### 3. Renovation Estimate
**Airtable Type:** Currency (USD)  
**JSON Field:** `renovation_estimate`  
**Data Type:** number | null

**n8n Expression:**
```javascript
{{ $json.data.renovation_estimate ?? null }}
```

**Critical Requirements:**
- ✅ Must be a number type (50000)
- ❌ NOT a string ("50000")
- ✅ null is acceptable

**Example Values:**
- Valid: `50000` (number)
- Valid: `null`
- Invalid: `"50000"` (string)

---

### 4. Is Airbnb Legal?
**Airtable Type:** Checkbox  
**JSON Field:** `airbnb_legal`  
**Data Type:** boolean | null

**n8n Expression:**
```javascript
{{ $json.data.airbnb_legal ?? false }}
```

**Critical Requirements:**
- ✅ Must be boolean: `true` or `false`
- ❌ NOT string: `"true"` or `"false"`
- ❌ NOT number: `1` or `0`
- ✅ Default to `false` if null (Airtable checkbox defaults to unchecked)

**Example Values:**
- Valid: `true` (boolean)
- Valid: `false` (boolean)
- Valid: `null` (defaults to false)
- Invalid: `"true"` (string - will cause error)
- Invalid: `1` (number - will cause error)
- Invalid: `"yes"` (string - will cause error)

**Why This Matters:**
Airtable Checkbox fields expect boolean values. Passing strings or numbers will cause type mismatch errors.

---

### 5. AI Verdict
**Airtable Type:** Single select  
**JSON Field:** `ai_verdict`  
**Data Type:** string

**n8n Expression:**
```javascript
{{ $json.data.ai_verdict }}
```

**Critical Requirements:**
- ✅ Must match EXACTLY (case-sensitive, emoji-sensitive, whitespace-sensitive)
- ✅ Valid values: `"🟢 BUY"`, `"🔴 PASS"`, `"🟡 HUMAN REVIEW"`
- ❌ NOT `"BUY"` (missing emoji)
- ❌ NOT `"🟢BUY"` (missing space)
- ❌ NOT `"🟢 BUY "` (trailing space)
- ❌ NOT `"buy"` (wrong case)

**Example Values:**
- Valid: `"🟢 BUY"` (exact match)
- Valid: `"🔴 PASS"` (exact match)
- Valid: `"🟡 HUMAN REVIEW"` (exact match)
- Invalid: `"BUY"` (missing emoji - will fail insert)
- Invalid: `"🟢BUY"` (missing space - will fail insert)
- Invalid: `"buy"` (wrong case - will fail insert)

**Why This Matters:**
Airtable Single select fields only accept values that exactly match the predefined options. Any mismatch (including whitespace or emoji differences) will cause the insert to fail with an "Invalid value" error.

---

### 6. AI Summary
**Airtable Type:** Long text  
**JSON Field:** `ai_summary`  
**Data Type:** string

**n8n Expression:**
```javascript
{{ $json.data.ai_summary }}
```

**Notes:**
- Direct string mapping
- No conversion needed
- Supports multi-line text
- Handles null/undefined gracefully

---

## Complete n8n Airtable Node Configuration

### Valid Path (Success Case)

**Node:** `Airtable - Save Deal Opportunity`

**Field Mappings:**
```
Property Title: {{ $json.data.property_title }}
Asking Price: {{ $json.data.asking_price ?? null }}
Renovation Estimate: {{ $json.data.renovation_estimate ?? null }}
Is Airbnb Legal?: {{ $json.data.airbnb_legal ?? false }}
AI Verdict: {{ $json.data.ai_verdict }}
AI Summary: {{ $json.data.ai_summary }}
```

---

## Data Type Conversion (if needed)

If your data source provides strings for numbers or booleans, use these conversions:

### Convert String to Number
```javascript
// Convert string to number (if JSON has "250000" as string)
{{ parseInt($json.data.asking_price) || null }}
```

### Convert String to Boolean
```javascript
// Convert string "true"/"false" to boolean
{{ $json.data.airbnb_legal === "true" || $json.data.airbnb_legal === true }}
```

**Note:** With OpenAI structured output (JSON mode), these conversions should NOT be necessary if the prompt is correctly configured.

---

## Common Mistakes & Solutions

### ❌ Mistake #1: Passing Strings for Currency Fields

**Wrong:**
```javascript
Asking Price: {{ $json.data.asking_price }}  // If this is "250000" (string)
```

**Why It Fails:**
- Airtable Currency field expects number type
- String `"250000"` causes type mismatch error
- Insert fails with: "Invalid type for field 'Asking Price'"

**Correct:**
```javascript
Asking Price: {{ $json.data.asking_price ?? null }}  // Ensures number type
```

**How to Avoid:**
- Ensure OpenAI returns numbers (not strings) in JSON
- Use strict JSON schema in prompt
- Verify data type in Parse node before mapping

---

### ❌ Mistake #2: Single Select Value Mismatch

**Wrong:**
```javascript
AI Verdict: {{ $json.data.ai_verdict }}  // If this is "BUY" (missing emoji)
```

**Why It Fails:**
- Airtable Single select only accepts exact option matches
- `"BUY"` doesn't match `"🟢 BUY"` (missing emoji)
- Insert fails with: "Invalid value for field 'AI Verdict'"

**Correct:**
```javascript
AI Verdict: {{ $json.data.ai_verdict }}  // Must be "🟢 BUY" exactly
```

**How to Avoid:**
- Ensure OpenAI prompt specifies exact verdict format
- Validate verdict in Parse node before mapping
- Use exact match validation (already implemented in workflow)

---

### ❌ Mistake #3: Passing Strings/Numbers for Checkbox

**Wrong:**
```javascript
Is Airbnb Legal?: {{ $json.data.airbnb_legal }}  // If this is "true" (string) or 1 (number)
```

**Why It Fails:**
- Airtable Checkbox expects boolean type
- String `"true"` or number `1` causes type mismatch
- Insert fails with: "Invalid type for field 'Is Airbnb Legal?'"

**Correct:**
```javascript
Is Airbnb Legal?: {{ $json.data.airbnb_legal ?? false }}  // Ensures boolean type
```

**How to Avoid:**
- Ensure OpenAI returns boolean (not string/number) in JSON
- Use strict JSON schema with boolean type
- Default to false if null

---

### ❌ Mistake #4: Not Handling Null Values

**Wrong:**
```javascript
Asking Price: {{ $json.data.asking_price }}  // If null, may cause issues
```

**Why It Might Fail:**
- Some Airtable field configurations don't accept null
- Better to handle explicitly

**Correct:**
```javascript
Asking Price: {{ $json.data.asking_price ?? null }}  // Explicit null handling
```

---

## Validation Checklist

Before deploying, verify:

- [ ] Currency fields receive numbers (not strings)
- [ ] Checkbox fields receive booleans (not strings/numbers)
- [ ] Single select values match exactly (including emojis)
- [ ] Null values are handled appropriately
- [ ] Field names match Airtable exactly (case-sensitive)
- [ ] No extra whitespace in single select values
- [ ] Emojis are preserved correctly in verdict field

---

## Testing Field Mappings

### Test Case 1: Valid Data
```json
{
  "property_title": "123 Main St",
  "asking_price": 250000,
  "renovation_estimate": 50000,
  "airbnb_legal": true,
  "ai_verdict": "🟢 BUY",
  "ai_summary": "Good deal"
}
```
**Expected:** Record creates successfully

### Test Case 2: Null Values
```json
{
  "property_title": "123 Main St",
  "asking_price": null,
  "renovation_estimate": null,
  "airbnb_legal": null,
  "ai_verdict": "🟡 HUMAN REVIEW",
  "ai_summary": "Missing data"
}
```
**Expected:** Record creates successfully (nulls handled)

### Test Case 3: Invalid Types (Should Not Occur with Strict Prompt)
```json
{
  "asking_price": "250000",  // String instead of number
  "airbnb_legal": "true",    // String instead of boolean
  "ai_verdict": "BUY"        // Missing emoji
}
```
**Expected:** Insert fails (type/value mismatch) - This is why strict prompt is critical

---

## Summary

**Key Principles:**
1. ✅ Currency fields = numbers (not strings)
2. ✅ Checkbox fields = booleans (not strings/numbers)
3. ✅ Single select = exact string match (emojis included)
4. ✅ Handle null values explicitly
5. ✅ Use strict OpenAI prompt to ensure correct types from source

---

## Common Mistake That Breaks Airtable Inserts

### ❌ **The Most Common Mistake: Passing Strings for Currency Fields**

**Problem:**
Many developers pass currency values as strings (e.g., `"250000"`) instead of numbers (e.g., `250000`). This causes Airtable inserts to fail with type mismatch errors.

**Why It Happens:**
- JSON parsing may convert numbers to strings
- String interpolation can create strings: `"Price: " + price` results in string
- OpenAI might return strings if prompt isn't strict enough
- Manual data manipulation can introduce strings

**Example of Broken Code:**
```javascript
// ❌ WRONG - String instead of number
Asking Price: {{ $json.data.asking_price }}  // If this is "250000" (string)
```

**Error Message:**
```
Invalid type for field 'Asking Price'. Expected number, received string.
```

**How This Guide Avoids It:**

1. **Strict OpenAI Prompt:**
   - Uses `response_format: json_object` to enforce JSON structure
   - Prompt explicitly specifies `"asking_price": number` in schema
   - Ensures OpenAI returns numbers, not strings

2. **Proper Field Mapping:**
   - Uses `{{ $json.data.asking_price ?? null }}` which preserves number type
   - No string conversion or interpolation
   - Direct field access maintains data types

3. **Validation in Parse Node:**
   - Parse & Validate JSON node checks data types
   - Can add type coercion if needed (though shouldn't be necessary)

4. **Documentation:**
   - Clear mapping guide shows exact expressions
   - Examples show correct vs incorrect types
   - Testing checklist verifies types before deployment

**Correct Implementation:**
```javascript
// ✅ CORRECT - Number type preserved
Asking Price: {{ $json.data.asking_price ?? null }}  // Ensures number type
```

**Additional Safeguards:**
- Use strict JSON schema in OpenAI prompt
- Validate data types in Parse node
- Test with sample data before production
- Monitor Airtable insert errors for type mismatches

**Result:** Currency fields receive numbers, inserts succeed, data is properly formatted in Airtable.
