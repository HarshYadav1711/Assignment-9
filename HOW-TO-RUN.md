# How to Run the Instant Real Estate Deal Engine

## Quick Start Overview

This project uses:
- **n8n** (workflow automation)
- **OpenAI API** (AI extraction)
- **Airtable** (database)
- **Airtable Interface/Softr** (dashboard)

---

## Prerequisites

Before starting, ensure you have:

1. ✅ n8n account (cloud: https://n8n.io or self-hosted)
2. ✅ OpenAI API account with API key (https://platform.openai.com)
3. ✅ Airtable account (https://airtable.com)
4. ✅ Optional: Softr account (for enhanced dashboard, or use free Airtable Interface)

---

## Step-by-Step Setup

### Step 1: Set Up Airtable Base (10 minutes)

1. **Create Airtable Base:**
   - Go to https://airtable.com
   - Click "Add a base" → "Start from scratch"
   - Name it: **"Instant Deal Engine – Elvara"**

2. **Create Table:**
   - Name the table: **"Deal Opportunities"** (exact name, case-sensitive)

3. **Add Fields (6 total):**
   - Click "+" to add fields, configure as follows:

   | Field Name | Field Type | Options |
   |------------|------------|---------|
   | Property Title | Single line text | - |
   | Asking Price | Currency | Format: USD ($) |
   | Renovation Estimate | Currency | Format: USD ($) |
   | Is Airbnb Legal? | Checkbox | - |
   | AI Verdict | Single select | Options: 🟢 BUY, 🔴 PASS, 🟡 HUMAN REVIEW |
   | AI Summary | Long text | - |

4. **Get Base ID:**
   - Go to https://airtable.com/api
   - Select your base
   - Copy the Base ID (starts with `app...`)

5. **Create API Token:**
   - Go to https://airtable.com/create/tokens
   - Create new token
   - Grant access to your base
   - Copy the token (starts with `pat...`)

---

### Step 2: Set Up OpenAI API (5 minutes)

1. **Get API Key:**
   - Go to https://platform.openai.com/api-keys
   - Create new secret key
   - Copy the key (starts with `sk-...`)

2. **Verify Access:**
   - Ensure you have access to `gpt-4o` or `gpt-4o-mini`
   - Check billing/quota is configured

---

### Step 3: Import n8n Workflow (15 minutes)

1. **Access n8n:**
   - Go to your n8n instance (cloud or self-hosted)
   - Sign in

2. **Import Workflow:**
   - Click "Workflows" → "Import from File"
   - Select `n8n-workflow-elvara.json` from this project
   - Click "Import"

3. **Configure Credentials:**

   **OpenAI Credential:**
   - Click on "OpenAI - Extract Deal Data" node
   - Click "Credential to connect with" → "Create New"
   - Select "OpenAI API"
   - Paste your OpenAI API key
   - Click "Save"

   **Airtable Credential:**
   - Click on any Airtable node (e.g., "Airtable - Save Deal Opportunity")
   - Click "Credential to connect with" → "Create New"
   - Select "Airtable API"
   - Paste your Airtable Personal Access Token
   - Click "Save"

4. **Configure Airtable Base ID:**
   - Option A (Recommended): Set environment variable
     - Go to n8n Settings → Environment Variables
     - Add: `AIRTABLE_BASE_ID` = `your-base-id-here`
   - Option B: Update each Airtable node directly
     - Click each Airtable node
     - In "Base" field, paste your Base ID
     - Verify "Table" field shows: `Deal Opportunities`

5. **Configure Workflow Settings:**
   - Click workflow settings (gear icon)
   - Set "Execution Timeout": 30 seconds
   - Enable "Save Data Error Execution": All
   - Enable "Save Data Success Execution": All

6. **Activate Workflow:**
   - Toggle workflow to "Active" (top right)
   - Copy webhook URL from "Webhook - Receive Listing" node
   - URL format: `https://your-n8n-instance.com/webhook/real-estate-deal`

---

### Step 4: Test the Workflow (5 minutes)

1. **Test with Valid Input:**
   ```bash
   curl -X POST https://your-n8n-instance.com/webhook/real-estate-deal \
     -H "Content-Type: application/json" \
     -d '{"listing_text": "3-bedroom house in Austin, TX. $250,000. 1,500 sqft. Airbnb allowed."}'
   ```

2. **Verify:**
   - Check n8n execution logs (should show "Success")
   - Check Airtable base (new record should appear)
   - Verify all fields are populated correctly
   - Check latency is under 30 seconds

3. **Test Invalid Input:**
   ```bash
   curl -X POST https://your-n8n-instance.com/webhook/real-estate-deal \
     -H "Content-Type: application/json" \
     -d '{}'
   ```
   - Should create HUMAN REVIEW record immediately

---

### Step 5: Set Up Dashboard (10 minutes)

**Option A: Airtable Interface (Free)**

1. In Airtable, click "Interfaces" tab
2. Click "Create an interface" → "Start from scratch"
3. Name it: "Deal Board"
4. Add "List" block
5. Configure:
   - Data source: "Deal Opportunities" table
   - Layout: Grid (card layout)
   - Fields: Property Title, Asking Price, AI Verdict
   - Group by or color by: AI Verdict
6. Enable public sharing
7. Copy shareable link

**Option B: Softr (Enhanced UI)**

1. Sign up at https://www.softr.io
2. Create new app
3. Connect Airtable base
4. Create List page with card view
5. Configure fields and color coding
6. Set to public access
7. Publish app

---

## Running the Project

### Send Requests to Webhook

Once setup is complete, send POST requests to your webhook URL:

**Example using cURL:**
```bash
curl -X POST https://your-n8n-instance.com/webhook/real-estate-deal \
  -H "Content-Type: application/json" \
  -d '{
    "listing_text": "Beautiful 3-bedroom house at 123 Main St, Austin, TX. $250,000. 1,500 sqft. 2 bathrooms. Built in 2010. Short-term rentals permitted."
  }'
```

**Example using Python:**
```python
import requests

webhook_url = "https://your-n8n-instance.com/webhook/real-estate-deal"
payload = {
    "listing_text": "3-bedroom house, $250k, Austin TX, Airbnb allowed"
}

response = requests.post(webhook_url, json=payload)
print(response.json())
```

**Example using JavaScript/Node.js:**
```javascript
const response = await fetch('https://your-n8n-instance.com/webhook/real-estate-deal', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    listing_text: '3-bedroom house, $250k, Austin TX, Airbnb allowed'
  })
});

const result = await response.json();
console.log(result);
```

---

## Verify Everything Works

### Checklist:

- [ ] Airtable base created with all 6 fields
- [ ] OpenAI API key configured in n8n
- [ ] Airtable token configured in n8n
- [ ] Base ID configured (environment variable or in nodes)
- [ ] Workflow imported and activated
- [ ] Webhook URL copied and tested
- [ ] Test request creates record in Airtable
- [ ] Record appears on dashboard
- [ ] Latency is under 30 seconds
- [ ] Invalid input creates HUMAN REVIEW record
- [ ] Dashboard is publicly accessible

---

## Troubleshooting

### Workflow Not Executing
- Check workflow is activated (toggle in top right)
- Verify webhook URL is correct
- Check n8n execution logs for errors

### Records Not Appearing in Airtable
- Verify Airtable credential is valid
- Check Base ID matches exactly
- Verify table name is exactly "Deal Opportunities" (case-sensitive)
- Check field names match exactly (case-sensitive)

### OpenAI Errors
- Verify API key is valid
- Check API quota/billing
- Review error in n8n execution logs
- Should create HUMAN REVIEW record (not fail)

### Dashboard Not Updating
- Refresh dashboard
- Verify dashboard is connected to correct table
- Check public sharing is enabled
- Clear browser cache

---

## Project Structure

```
.
├── n8n-workflow-elvara.json     # Import this into n8n
├── deployment-checklist.md      # Detailed deployment steps
├── test-payloads.json           # Test cases and examples
├── field-mapping-reference.md   # Quick lookup for field mappings
├── airtable-base-elvara.md      # Airtable schema reference
├── openai-prompt-strict.md      # OpenAI prompt documentation
├── error-handling-guide.md      # Error handling details
├── deal-board-ui-setup.md       # Dashboard setup guide
└── HOW-TO-RUN.md               # This file
```

---

## Quick Reference

**Webhook Endpoint:** `POST /webhook/real-estate-deal`

**Request Format:**
```json
{
  "listing_text": "Property description text here..."
}
```

**Expected Latency:** < 30 seconds

**Required Services:**
- n8n (workflow automation)
- OpenAI API (AI extraction)
- Airtable (database)
- Airtable Interface or Softr (dashboard)

---

## Next Steps

1. ✅ Complete setup steps above
2. ✅ Test with sample requests
3. ✅ Verify records appear in Airtable
4. ✅ Verify dashboard displays correctly
5. ✅ Start sending real requests!

For detailed information, see:
- `deployment-checklist.md` - Complete deployment guide
- `test-payloads.json` - Test cases
- `error-handling-guide.md` - Error handling details
