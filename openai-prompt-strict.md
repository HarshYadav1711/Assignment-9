# Strict OpenAI Prompt for Real Estate Deal Extraction

## System Message

```
You are a JSON extraction system. Extract real estate data and return ONLY valid JSON. No markdown, no explanations, no additional text. Output must be parseable JSON only.
```

## User Message Template

```
Extract real estate data from this listing and return ONLY valid JSON (no markdown, no explanations, no code blocks, no extra text):

JSON Schema:
{
  "property_title": "string",
  "asking_price": number | null,
  "renovation_estimate": number | null,
  "airbnb_legal": boolean | null,
  "ai_summary": "string",
  "ai_verdict": "🟢 BUY" | "🔴 PASS" | "🟡 HUMAN REVIEW"
}

Extraction Rules:
1. property_title: Extract the property address or title. Use "Unknown Property" if not found.
2. asking_price: Extract price in USD as a number. Set to null if price cannot be confidently extracted from the text.
3. renovation_estimate: Extract renovation/repair cost estimate in USD as a number. Set to null if not mentioned or unclear.
4. airbnb_legal: Infer strictly from the listing text. Set to true only if text explicitly states short-term rentals, Airbnb, or vacation rentals are allowed/permitted. Set to false if explicitly prohibited. Set to null if not mentioned or unclear.
5. ai_summary: Brief summary of the property (max 200 characters).

Verdict Logic (CRITICAL - Apply exactly):
1. If (asking_price + renovation_estimate < 400000) AND (airbnb_legal === true) → "🟢 BUY"
2. Else if asking_price is null OR renovation_estimate is null OR airbnb_legal is null → "🟡 HUMAN REVIEW"
3. Else → "🔴 PASS"

Important:
- Return ONLY the JSON object, nothing else
- Do not wrap in markdown code blocks
- Do not include explanations
- Do not include backticks
- Ensure all strings are properly quoted
- Ensure numbers are not quoted
- Ensure null values are the literal null (not the string "null")
- Ensure boolean values are true/false (not strings)

Listing text:
{listing_text}
```

---

## Complete Prompt (Ready to Use)

```
Extract real estate data from this listing and return ONLY valid JSON (no markdown, no explanations, no code blocks, no extra text):

JSON Schema:
{
  "property_title": "string",
  "asking_price": number | null,
  "renovation_estimate": number | null,
  "airbnb_legal": boolean | null,
  "ai_summary": "string",
  "ai_verdict": "🟢 BUY" | "🔴 PASS" | "🟡 HUMAN REVIEW"
}

Extraction Rules:
1. property_title: Extract the property address or title. Use "Unknown Property" if not found.
2. asking_price: Extract price in USD as a number. Set to null if price cannot be confidently extracted from the text.
3. renovation_estimate: Extract renovation/repair cost estimate in USD as a number. Set to null if not mentioned or unclear.
4. airbnb_legal: Infer strictly from the listing text. Set to true only if text explicitly states short-term rentals, Airbnb, or vacation rentals are allowed/permitted. Set to false if explicitly prohibited. Set to null if not mentioned or unclear.
5. ai_summary: Brief summary of the property (max 200 characters).

Verdict Logic (CRITICAL - Apply exactly):
1. If (asking_price + renovation_estimate < 400000) AND (airbnb_legal === true) → "🟢 BUY"
2. Else if asking_price is null OR renovation_estimate is null OR airbnb_legal is null → "🟡 HUMAN REVIEW"
3. Else → "🔴 PASS"

Important:
- Return ONLY the JSON object, nothing else
- Do not wrap in markdown code blocks
- Do not include explanations
- Do not include backticks
- Ensure all strings are properly quoted
- Ensure numbers are not quoted
- Ensure null values are the literal null (not the string "null")
- Ensure boolean values are true/false (not strings)

Listing text:
{listing_text}
```

---

## n8n Integration Format

For use in n8n OpenAI node, replace `{listing_text}` with:

```
{{ 'Extract real estate data from this listing and return ONLY valid JSON (no markdown, no explanations, no code blocks, no extra text):

JSON Schema:
{
  "property_title": "string",
  "asking_price": number | null,
  "renovation_estimate": number | null,
  "airbnb_legal": boolean | null,
  "ai_summary": "string",
  "ai_verdict": "🟢 BUY" | "🔴 PASS" | "🟡 HUMAN REVIEW"
}

Extraction Rules:
1. property_title: Extract the property address or title. Use "Unknown Property" if not found.
2. asking_price: Extract price in USD as a number. Set to null if price cannot be confidently extracted from the text.
3. renovation_estimate: Extract renovation/repair cost estimate in USD as a number. Set to null if not mentioned or unclear.
4. airbnb_legal: Infer strictly from the listing text. Set to true only if text explicitly states short-term rentals, Airbnb, or vacation rentals are allowed/permitted. Set to false if explicitly prohibited. Set to null if not mentioned or unclear.
5. ai_summary: Brief summary of the property (max 200 characters).

Verdict Logic (CRITICAL - Apply exactly):
1. If (asking_price + renovation_estimate < 400000) AND (airbnb_legal === true) → "🟢 BUY"
2. Else if asking_price is null OR renovation_estimate is null OR airbnb_legal is null → "🟡 HUMAN REVIEW"
3. Else → "🔴 PASS"

Important:
- Return ONLY the JSON object, nothing else
- Do not wrap in markdown code blocks
- Do not include explanations
- Do not include backticks
- Ensure all strings are properly quoted
- Ensure numbers are not quoted
- Ensure null values are the literal null (not the string "null")
- Ensure boolean values are true/false (not strings)

Listing text: ' + $json.body.listing_text }}
```

---

## OpenAI Node Configuration

**Settings:**
- **Model:** gpt-4o (recommended) or gpt-4o-mini (faster/cheaper)
- **Temperature:** 0 (deterministic)
- **Max Tokens:** 500 (sufficient for JSON response)
- **Response Format:** `json_object` (CRITICAL - ensures JSON output)

**System Message:**
```
You are a JSON extraction system. Extract real estate data and return ONLY valid JSON. No markdown, no explanations, no additional text. Output must be parseable JSON only.
```

---

## Verdict Logic Breakdown

### Condition 1: 🟢 BUY
```
(asking_price + renovation_estimate < 400000) AND (airbnb_legal === true)
```
- Total investment (price + renovation) must be under $400k
- Airbnb must be explicitly legal
- Both conditions must be true

### Condition 2: 🟡 HUMAN REVIEW
```
asking_price is null OR renovation_estimate is null OR airbnb_legal is null
```
- Any critical field missing/unclear
- Cannot make confident decision
- Requires human judgment

### Condition 3: 🔴 PASS (Default)
```
All other cases
```
- Data is complete but doesn't meet BUY criteria
- Could be: price too high, Airbnb illegal, or other reasons

---

## Example Outputs

### Example 1: BUY Case
```json
{
  "property_title": "123 Main St, Austin, TX",
  "asking_price": 250000,
  "renovation_estimate": 50000,
  "airbnb_legal": true,
  "ai_summary": "3-bedroom house in Austin. $250k asking price. $50k renovation needed. Airbnb permitted.",
  "ai_verdict": "🟢 BUY"
}
```

### Example 2: HUMAN REVIEW Case
```json
{
  "property_title": "456 Oak Ave, Seattle, WA",
  "asking_price": 300000,
  "renovation_estimate": null,
  "airbnb_legal": null,
  "ai_summary": "2-bedroom condo in Seattle. $300k asking price. Renovation costs and rental rules unclear.",
  "ai_verdict": "🟡 HUMAN REVIEW"
}
```

### Example 3: PASS Case
```json
{
  "property_title": "789 Pine St, San Francisco, CA",
  "asking_price": 600000,
  "renovation_estimate": 100000,
  "airbnb_legal": false,
  "ai_summary": "4-bedroom house in SF. $600k asking price. $100k renovation. Short-term rentals prohibited.",
  "ai_verdict": "🔴 PASS"
}
```

---

## Important Notes

1. **Response Format Setting:** Always set `response_format: { type: "json_object" }` in OpenAI API call
2. **Temperature:** Use 0 for deterministic output
3. **Null Handling:** Ensure the AI returns literal `null`, not the string `"null"`
4. **Number Handling:** Prices should be numbers, not strings (e.g., `250000` not `"250000"`)
5. **Boolean Handling:** Use `true`/`false`, not `"true"`/`"false"`
6. **Verdict Emojis:** Must match exactly: 🟢 BUY, 🔴 PASS, 🟡 HUMAN REVIEW
7. **Field Names:** Use snake_case as specified (property_title, asking_price, etc.)

---

## Testing the Prompt

Test with various inputs to ensure:
- ✅ Output is always valid JSON
- ✅ No markdown code blocks
- ✅ No explanatory text
- ✅ Verdict logic applies correctly
- ✅ Null values handled properly
- ✅ Numbers are not quoted
- ✅ Booleans are not quoted
