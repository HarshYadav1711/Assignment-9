# Quick Reference Card

## Webhook Endpoint
```
POST /webhook/real-estate-deal
Content-Type: application/json

Body: { "text": "listing text here" }
```

## Expected Response Time
- Target: < 10 seconds
- Maximum: < 30 seconds

## Investment Rules Summary
- **PASS Threshold**: Score ≥ 60 points (out of 100)
- **Key Factors**: Price per sqft, price range, bedrooms, square footage, property type

## Airtable Fields (Critical)
All field names must match exactly (case-sensitive):
- Address, Price, Property Type, Square Feet, Bedrooms, Bathrooms
- Year Built, Lot Size, City, State, Zip Code, Description
- Price per Sqft, Investment Score, Decision, Decision Reason, Processed At

## Common Issues

| Issue | Solution |
|-------|----------|
| OpenAI timeout | Check API key, try GPT-4o-mini for speed |
| Airtable error | Verify field names match exactly |
| JSON parse error | Check OpenAI response format |
| Low scores | Review investment rules thresholds |
| Missing data | OpenAI may return null for unknown fields |

## Testing Checklist
- [ ] Webhook accepts POST requests
- [ ] OpenAI API key valid
- [ ] Airtable base and table created
- [ ] All field names match schema
- [ ] Workflow executes successfully
- [ ] Data appears in Airtable
- [ ] Dashboard displays data

## Performance Targets
- OpenAI call: 2-5 seconds
- Rules processing: < 1 second  
- Airtable write: 1-2 seconds
- **Total**: < 10 seconds (well under 30s limit)
