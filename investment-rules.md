# Investment Rules Configuration

## Deterministic Investment Scoring Rules

The system applies a scoring algorithm with a maximum score of 100 points. A deal must score **≥ 60 points** to receive a **PASS** decision.

### Rule 1: Price per Square Foot (30 points max)
- **Target**: < $150/sqft
  - Score: +30 points
  - Reason: "Good price per sqft."
- **Acceptable**: $150-$200/sqft
  - Score: +15 points
- **Poor**: > $200/sqft
  - Score: -10 points
  - Reason: "High price per sqft."

### Rule 2: Property Price Range (25 points max)
- **Target**: $50,000 - $500,000
  - Score: +25 points
  - Reason: "Price in target range."
- **Acceptable**: $500,001 - $750,000
  - Score: +10 points
- **Poor**: < $50,000 OR > $750,000
  - Score: -15 points
  - Reason: "Price outside optimal range."

### Rule 3: Bedrooms (15 points max)
- **Target**: 2-4 bedrooms
  - Score: +15 points
  - Reason: "Good bedroom count."
- **Acceptable**: 1 or 5 bedrooms
  - Score: +5 points
- **Other**: 0, 6+ bedrooms
  - Score: 0 points

### Rule 4: Square Footage (20 points max)
- **Target**: 800 - 2,500 sqft
  - Score: +20 points
  - Reason: "Good size."
- **Acceptable**: 600 - 3,000 sqft (but outside target)
  - Score: +10 points
- **Poor**: < 600 sqft OR > 3,000 sqft
  - Score: -5 points

### Rule 5: Property Type Preference (10 points max)
- **Preferred Types**: house, townhouse, condo
  - Score: +10 points
  - Reason: "Preferred property type."
- **Other Types**: apartment, land, commercial, other
  - Score: 0 points

## Decision Logic

1. Calculate total score (sum of all rule scores)
2. **PASS**: Score ≥ 60 points
3. **REJECT**: Score < 60 points
4. Decision reason is concatenated from all applicable rule reasons

## Customization

To modify rules, edit the "Apply Investment Rules" Code node in the n8n workflow. The rules are implemented in JavaScript and can be adjusted for:
- Score thresholds
- Price ranges
- Property type preferences
- Additional rules (e.g., location-based, year built, etc.)

## Notes

- Rules are deterministic (no randomness)
- Missing data (null values) result in 0 points for that rule
- Negative scores are possible but don't reduce total below 0 in practice
- All calculations happen in n8n workflow before saving to Airtable
