# Instant Real Estate Deal Engine

A fast, automated decision engine that processes raw real estate listings, extracts structured data using AI, applies investment rules, and stores results in Airtable with a public dashboard.

## Architecture

```
Webhook → n8n → OpenAI (Structured JSON) → Investment Rules → Airtable → Dashboard
```

**End-to-end latency: < 30 seconds**

## Components

- **n8n**: Workflow automation (`n8n-workflow.json`)
- **OpenAI**: Structured data extraction (GPT-4o with JSON mode)
- **Airtable**: Data storage and source of truth
- **Softr/Airtable Interface**: Public dashboard

## Quick Start

1. **Set up Airtable base** (see `airtable-schema.md`)
2. **Import n8n workflow** (`n8n-workflow.json`)
3. **Configure credentials** (OpenAI API, Airtable API)
4. **Activate workflow** and get webhook URL
5. **Create dashboard** (Airtable Interface or Softr)

See `setup-guide.md` for detailed instructions.

## Investment Rules

The system applies deterministic scoring rules (0-100 points):
- Price per sqft: < $150/sqft preferred (30 pts)
- Price range: $50k-$500k preferred (25 pts)
- Bedrooms: 2-4 preferred (15 pts)
- Square footage: 800-2,500 sqft preferred (20 pts)
- Property type: house/townhouse/condo preferred (10 pts)

**Decision: PASS if score ≥ 60, otherwise REJECT**

See `investment-rules.md` for full details.

## File Structure

```
.
├── n8n-workflow.json        # n8n workflow definition
├── airtable-schema.md       # Airtable base structure
├── investment-rules.md      # Scoring rules documentation
├── setup-guide.md          # Detailed setup instructions
└── README.md               # This file
```

## API Usage

**Endpoint**: `POST /webhook/real-estate-deal`

**Request**:
```json
{
  "text": "3-bedroom house for sale at 123 Main St, Austin, TX. $250,000. 1,500 sqft. 2 bathrooms."
}
```

**Response**:
```json
{
  "success": true,
  "decision": "PASS",
  "score": 85,
  "id": "2024-01-01T12:00:00.000Z"
}
```

## Constraints

✅ n8n for automation  
✅ OpenAI (structured JSON only)  
✅ Airtable (mandatory database)  
✅ Softr/Airtable Interface (no custom frontend)  
✅ < 30 seconds latency  
✅ Minimal features, no over-engineering  

## License

Internal use only.
