# Field Mapping Reference (Quick Lookup)

## Airtable Field → JSON Field Mapping

| Airtable Field Name | JSON Field Name (snake_case) | Data Type | n8n Expression (Valid Path) | n8n Expression (Error Path) |
|---------------------|------------------------------|-----------|----------------------------|----------------------------|
| Property Title | property_title | string | `{{ $json.data.property_title }}` | `{{ $json.property_title }}` |
| Asking Price | asking_price | number \| null | `{{ $json.data.asking_price ?? null }}` | `{{ $json.asking_price }}` |
| Renovation Estimate | renovation_estimate | number \| null | `{{ $json.data.renovation_estimate ?? null }}` | `{{ $json.renovation_estimate }}` |
| Is Airbnb Legal? | airbnb_legal | boolean \| null | `{{ $json.data.airbnb_legal ?? false }}` | `{{ $json.airbnb_legal }}` |
| AI Verdict | ai_verdict | string | `{{ $json.data.ai_verdict }}` | `{{ $json.ai_verdict }}` |
| AI Summary | ai_summary | string | `{{ $json.data.ai_summary }}` | `{{ $json.ai_summary }}` |

## Critical Type Requirements

**Currency Fields (Asking Price, Renovation Estimate):**
- Must be number type, NOT string
- null is acceptable
- Example: `250000` ✅ NOT `"250000"` ❌

**Checkbox Field (Is Airbnb Legal?):**
- Must be boolean type, NOT string/number
- Use `false` for null cases (not `null`)
- Example: `true` ✅ NOT `"true"` ❌ NOT `1` ❌

**Single Select (AI Verdict):**
- Must match exactly (case, emoji, spacing)
- Valid: `"🟢 BUY"`, `"🔴 PASS"`, `"🟡 HUMAN REVIEW"`
- Invalid: `"BUY"`, `"buy"`, `"🟢BUY"`, `"🟢 BUY "`

## Node Context Reference

### Valid Path (Success)
- Input: From "Route Parse Result" node (true branch)
- Structure: `{ success: true, data: { property_title, asking_price, ... } }`
- Access: `$json.data.field_name`

### Error Path (HUMAN REVIEW)
- Input: From "Format Error Record" node
- Structure: `{ property_title, asking_price, ..., ai_verdict: "🟡 HUMAN REVIEW" }`
- Access: `$json.field_name`

### Invalid Input Path
- Input: From "Validate Listing Text Input" node (false branch)
- Structure: `$json.body.listing_text` (webhook data)
- Uses direct expressions in Airtable node

## Common Errors to Avoid

1. **String instead of number:** `"250000"` → `250000`
2. **String instead of boolean:** `"true"` → `true`
3. **Wrong verdict format:** `"BUY"` → `"🟢 BUY"`
4. **Missing null handling:** Use `?? null` or `?? false`
5. **Case mismatch:** Field names are case-sensitive
