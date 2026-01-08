# Airtable Base: Instant Deal Engine – Elvara

## Base Name
**Instant Deal Engine – Elvara**

## Table Name
**Deal Opportunities**

---

## Fields Configuration

| Field Name | Field Type | Options/Format | Purpose |
|------------|------------|----------------|---------|
| Property Title | Single line text | - | Identifies the property with a human-readable name for quick reference in lists and dashboards. |
| Asking Price | Currency | USD ($) | Captures the listing price to calculate investment metrics and determine if the deal fits budget constraints. |
| Renovation Estimate | Currency | USD ($) | Stores estimated renovation costs to compute total investment (purchase + renovation) for ROI calculations. |
| Is Airbnb Legal? | Checkbox | True/False | Flags whether short-term rentals are permitted, a critical factor for rental income strategy decisions. |
| AI Verdict | Single select | Options: 🟢 BUY, 🔴 PASS, 🟡 HUMAN REVIEW | Provides the final automated decision or signals when human expertise is needed for complex cases. |
| AI Summary | Long text | - | Contains the reasoning behind the AI verdict, including key factors and calculated metrics for transparency and review. |

---

## Field Details

### Property Title
- **Type**: Single line text
- **Purpose**: Identifies the property with a human-readable name for quick reference in lists and dashboards.

### Asking Price
- **Type**: Currency (USD)
- **Format**: $ (dollar sign, no cents needed typically)
- **Purpose**: Captures the listing price to calculate investment metrics and determine if the deal fits budget constraints.

### Renovation Estimate
- **Type**: Currency (USD)
- **Format**: $ (dollar sign)
- **Purpose**: Stores estimated renovation costs to compute total investment (purchase + renovation) for ROI calculations.

### Is Airbnb Legal?
- **Type**: Checkbox
- **Options**: ☑ (checked) or ☐ (unchecked)
- **Purpose**: Flags whether short-term rentals are permitted, a critical factor for rental income strategy decisions.

### AI Verdict
- **Type**: Single select
- **Options** (exact):
  - 🟢 BUY
  - 🔴 PASS
  - 🟡 HUMAN REVIEW
- **Purpose**: Provides the final automated decision or signals when human expertise is needed for complex cases.

### AI Summary
- **Type**: Long text
- **Purpose**: Contains the reasoning behind the AI verdict, including key factors and calculated metrics for transparency and review.

---

## Recommended Views

1. **All Deals** (Grid view)
   - Default view showing all records
   - Sort by: Property Title (A-Z)

2. **BUY Recommendations** (Grid view)
   - Filter: AI Verdict = 🟢 BUY
   - Sort by: Asking Price (ascending)

3. **Human Review Queue** (Grid view)
   - Filter: AI Verdict = 🟡 HUMAN REVIEW
   - Sort by: Asking Price (ascending)

4. **Passed Deals** (Grid view)
   - Filter: AI Verdict = 🔴 PASS
   - Useful for reference/audit trail

---

## Notes

- **No additional fields** - This base contains only the 6 specified fields
- Field names must match exactly (case-sensitive) when integrating with n8n workflow
- The AI Verdict options include emoji prefixes (🟢 🔴 🟡) - ensure these are preserved in Airtable
- This is a minimal schema focused on deal evaluation and decision-making
