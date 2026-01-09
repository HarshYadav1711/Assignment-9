# Deployment Checklist

## Pre-Deployment Setup

### 1. Airtable Setup
- [ ] Create Airtable base: "Instant Deal Engine – Elvara"
- [ ] Create table: "Deal Opportunities"
- [ ] Add all 6 fields with correct types:
  - [ ] Property Title (Single line text)
  - [ ] Asking Price (Currency USD)
  - [ ] Renovation Estimate (Currency USD)
  - [ ] Is Airbnb Legal? (Checkbox)
  - [ ] AI Verdict (Single select: 🟢 BUY, 🔴 PASS, 🟡 HUMAN REVIEW)
  - [ ] AI Summary (Long text)
- [ ] Get Airtable Base ID (from API documentation)
- [ ] Create Airtable Personal Access Token
- [ ] Grant token access to base
- [ ] Test Airtable connection

### 2. OpenAI Setup
- [ ] Create OpenAI API account
- [ ] Generate API key
- [ ] Verify API key has access to gpt-4o or gpt-4o-mini
- [ ] Test API key with simple request
- [ ] Verify billing/quota is configured

### 3. n8n Setup
- [ ] Set up n8n instance (cloud or self-hosted)
- [ ] Access n8n dashboard
- [ ] Configure credentials:
  - [ ] OpenAI API credential
  - [ ] Airtable API credential
- [ ] Set environment variable: AIRTABLE_BASE_ID (optional, can be set in workflow)

## Workflow Deployment

### 4. Import Workflow
- [ ] Import `n8n-workflow-elvara.json` into n8n
- [ ] Verify all nodes are present (9 nodes total)
- [ ] Check node connections are correct

### 5. Configure Credentials
- [ ] Set OpenAI credential on "OpenAI - Extract Deal Data" node
- [ ] Set Airtable credential on all three Airtable nodes:
  - [ ] Airtable - Save HUMAN REVIEW (Invalid Input)
  - [ ] Airtable - Save Deal Opportunity
  - [ ] Airtable - Save HUMAN REVIEW (Error)

### 6. Configure Airtable Base ID
- [ ] Set AIRTABLE_BASE_ID environment variable in n8n
- [ ] OR update base field in each Airtable node directly
- [ ] Verify table name: "Deal Opportunities" (exact match)

### 7. Configure Workflow Settings
- [ ] Verify execution timeout: 30 seconds
- [ ] Enable "Save Data Error Execution": All
- [ ] Enable "Save Data Success Execution": All
- [ ] Enable "Save Manual Executions": Yes

### 8. Activate Workflow
- [ ] Toggle workflow to "Active"
- [ ] Copy webhook URL from "Webhook - Receive Listing" node
- [ ] Test webhook URL is accessible

## Testing

### 9. Test Valid Input
- [ ] Send POST request with valid listing_text
- [ ] Verify workflow executes successfully
- [ ] Check record created in Airtable with correct fields
- [ ] Verify latency is under 30 seconds
- [ ] Check AI Verdict is one of: 🟢 BUY, 🔴 PASS, 🟡 HUMAN REVIEW

### 10. Test Invalid Input
- [ ] Send POST request with missing listing_text
- [ ] Verify HUMAN REVIEW record created
- [ ] Check Property Title contains "Input Validation Required"
- [ ] Verify workflow completes successfully (no errors)

### 11. Test Error Handling
- [ ] Test OpenAI error (use invalid API key temporarily)
- [ ] Verify HUMAN REVIEW record created with error summary
- [ ] Restore valid API key
- [ ] Test JSON parse error (if possible to simulate)

### 12. Performance Testing
- [ ] Measure end-to-end latency (should be < 30 seconds)
- [ ] Test multiple concurrent requests
- [ ] Verify no workflow failures
- [ ] Check all records appear in Airtable

## Dashboard Setup

### 13. Create Dashboard (Airtable Interface)
- [ ] Create Interface named "Deal Board"
- [ ] Add List block with Grid layout
- [ ] Configure fields: Property Title, Asking Price, AI Verdict
- [ ] Apply color coding by AI Verdict
- [ ] Format Asking Price as currency
- [ ] Enable public sharing
- [ ] Test public link in incognito browser

### 14. Verify Dashboard
- [ ] Verify cards display correctly
- [ ] Check color coding works (green/red/yellow)
- [ ] Verify no authentication required
- [ ] Test new records appear on dashboard
- [ ] Verify fields display correctly

## Final Verification

### 15. End-to-End Test
- [ ] Send POST request with test listing
- [ ] Verify record appears in Airtable within 30 seconds
- [ ] Verify record appears on dashboard
- [ ] Check all field values are correct
- [ ] Verify color coding matches verdict

### 16. Documentation Review
- [ ] All setup steps completed
- [ ] Webhook URL documented
- [ ] Dashboard URL documented
- [ ] Credentials secured (not in code)

## Post-Deployment

### 17. Monitoring
- [ ] Monitor workflow executions for errors
- [ ] Check HUMAN REVIEW queue in Airtable
- [ ] Monitor latency metrics
- [ ] Set up alerts if needed

### 18. Documentation
- [ ] Document webhook URL
- [ ] Document dashboard URL
- [ ] Document any custom configurations
- [ ] Save workflow backup

---

## Quick Reference

**Webhook URL:** `[Copy from n8n workflow]`  
**Dashboard URL:** `[Copy from Airtable Interface or Softr]`  
**Airtable Base ID:** `[From Airtable API docs]`  
**Table Name:** `Deal Opportunities`

## Troubleshooting

If workflow fails:
1. Check n8n execution logs
2. Verify all credentials are valid
3. Check Airtable field names match exactly (case-sensitive)
4. Verify OpenAI API key is valid
5. Check webhook URL is correct

If dashboard not updating:
1. Refresh dashboard
2. Check Airtable base has new records
3. Verify dashboard is connected to correct table
4. Check public sharing is enabled
