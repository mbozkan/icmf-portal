# ICMF-FIN-001: Hidden Financial Adjustment

**Category:** Financial Manipulation  
**Risk Level:** Critical

## Description
Hidden Financial Adjustment occurs when application logic silently alters financial values such as totals, balances, taxes, or discounts under hidden conditions.

These manipulations are often triggered by:

- specific transaction IDs
- hidden configuration flags
- privileged user conditions
- conditional logic embedded in financial workflows

Such logic allows insiders to alter financial outcomes without visible changes in business workflows.

## Example Pattern

```csharp
if(invoice.Id == "TEST-100")
{
    invoice.TotalAmount += 10;
}
