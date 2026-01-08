# Setup Guide: Instant Real Estate Deal Engine

## Prerequisites

1. **n8n Account** (self-hosted or cloud)
2. **OpenAI API Key** (with access to GPT-4o or GPT-4)
3. **Airtable Account** with API access
4. **Softr Account** (optional, for enhanced dashboard) OR use Airtable Interface Builder

---

## Step 1: Set Up Airtable Base

1. Create a new Airtable base named **"Real Estate Deal Engine"**
2. Create a table named **"Real Estate Deals"**
3. Add fields according to `airtable-schema.md`:
   - Address (Single line text)
   - Price (Number, Currency $)
   - Property Type (Single select: house, condo, apartment, townhouse, land, commercial, other)
   - Square Feet (Number)
   - Bedrooms (Number)
   - Bathrooms (Number)
   - Year Built (Number)
   - Lot Size (Number)
   - City (Single line text)
   - State (Single line text)
   - Zip Code (Single line text)
   - Description (Long text)
   - Price per Sqft (Number, 2 decimal places)
   - Investment Score (Number, integer)
   - Decision (Single select: PASS, REJECT)
   - Decision Reason (Long text)
   - Processed At (Date & time)

4. Create recommended views (see `airtable-schema.md`)
5. Get your **Base ID**:
   - Go to https://airtable.com/api
   - Select your base
   - Copy the Base ID from the URL (starts with `app...`)

6. Get your **Airtable API Key**:
   - Go to https://airtable.com/create/tokens
   - Create a new personal access token
   - Grant access to your base
   - Copy the token

---

## Step 2: Configure n8n Workflow

1. **Import the workflow**:
   - In n8n, click "Workflows" → "Import from File"
   - Select `n8n-workflow.json`

2. **Set up credentials**:

   **OpenAI API:**
   - Click on "OpenAI Extract Data" node
   - Click "Create New Credential" or select existing
   - Add your OpenAI API Key
   - Test connection

   **Airtable API:**
   - Click on "Save to Airtable" node
   - Click "Create New Credential"
   - Add your Airtable Personal Access Token
   - Test connection

3. **Configure Airtable node**:
   - Click "Save to Airtable" node
   - In "Base" field, enter your Airtable Base ID (or set as environment variable `AIRTABLE_BASE_ID`)
   - Verify "Table" field shows "Real Estate Deals"
   - Ensure all field mappings are correct

4. **Configure Error Handling** (Important):
   - In n8n, errors are handled automatically, but you can add error workflows
   - Each node has error outputs - connect them to "Error Response" node if needed
   - For production: Consider adding error logging or notifications
   - The workflow includes an "Error Response" node for handling failures

5. **Activate the workflow**:
   - Toggle the workflow to "Active"
   - Copy the webhook URL (from "Webhook Trigger" node)
   - Test the webhook URL is accessible

---

## Step 3: Test the Webhook

### Test with cURL:
```bash
curl -X POST https://your-n8n-instance.com/webhook/real-estate-deal \
  -H "Content-Type: application/json" \
  -d '{
    "text": "Beautiful 3-bedroom house for sale at 123 Main St, Austin, TX 78701. Price: $250,000. 1,500 sqft. 2 bathrooms. Built in 2010. Large lot with backyard."
  }'
```

### Test with JSON body:
```json
{
  "text": "Your real estate listing text here..."
}
```

### Expected Response:
```json
{
  "success": true,
  "decision": "PASS",
  "score": 85,
  "id": "2024-01-01T12:00:00.000Z"
}
```

---

## Step 4: Create Dashboard

### Option A: Airtable Interface Builder (Free, included)

1. In Airtable, click "Interfaces" tab
2. Create new interface
3. Add blocks:
   - **List block**: Show "Real Estate Deals" table
   - Filter: Decision = PASS (for passed deals view)
   - Sort by: Investment Score (descending)
4. Publish interface (get shareable link)

### Option B: Softr (Enhanced UI, paid)

1. Sign up at https://www.softr.io
2. Connect your Airtable base
3. Create a new app
4. Add pages:
   - **Homepage**: Summary stats
   - **Deals List**: Table view of Real Estate Deals
   - **Deal Detail**: Individual deal view
5. Configure filters and sorting
6. Publish and share

---

## Step 5: Monitor Performance

### Latency Target: < 30 seconds

Monitor these metrics:
- Webhook → OpenAI: ~2-5 seconds
- OpenAI → Rules Processing: < 1 second
- Rules → Airtable: ~1-2 seconds
- **Total**: Should be under 10 seconds typically

### Troubleshooting:

1. **Slow responses**: Check OpenAI API status, consider using GPT-4o-mini for faster responses
2. **Airtable errors**: Verify field names match exactly (case-sensitive)
3. **JSON parsing errors**: Check OpenAI response format
4. **Webhook timeout**: Ensure n8n execution timeout is set to 30+ seconds

---

## Environment Variables (Optional)

Set in n8n settings for easier management:
- `AIRTABLE_BASE_ID`: Your Airtable base ID
- `OPENAI_API_KEY`: Your OpenAI API key (or use credential)

---

## API Usage Examples

### Python:
```python
import requests

webhook_url = "https://your-n8n-instance.com/webhook/real-estate-deal"
listing_text = "3BR house, $300k, 2000 sqft, Austin TX"

response = requests.post(webhook_url, json={"text": listing_text})
print(response.json())
```

### JavaScript/Node.js:
```javascript
const response = await fetch('https://your-n8n-instance.com/webhook/real-estate-deal', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ 
    text: '3BR house, $300k, 2000 sqft, Austin TX' 
  })
});

const result = await response.json();
console.log(result);
```

---

## Maintenance

- **Review investment rules** quarterly (update `investment-rules.md` and n8n Code node)
- **Monitor Airtable usage** (free tier: 1,000 records/table)
- **Check OpenAI costs** (GPT-4o usage)
- **Update field mappings** if Airtable schema changes

---

## Support

For issues:
1. Check n8n execution logs
2. Verify all credentials are valid
3. Test OpenAI API key separately
4. Verify Airtable field names match exactly
