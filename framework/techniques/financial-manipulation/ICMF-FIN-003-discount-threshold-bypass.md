# ICMF-FIN-003: Discount Threshold Bypass

**Category:** Financial Manipulation  
**Technique ID:** ICMF-FIN-003  
**Risk Level:** Critical  
**Evidence Level Required:** Level 3 — Direct Manipulation Evidence  
**Version:** 1.0

---

## Description

Discount Threshold Bypass occurs when application logic is modified to apply unauthorized discount rates, waive minimum purchase thresholds, or extend discount eligibility to transactions, users, or accounts that do not meet the documented business criteria.

This technique exploits the complexity of enterprise pricing engines—where discount rules are often multi-conditional and distributed across configuration, service, and calculation layers—to embed deviations that are difficult to detect through manual review.

Common forms include:

- Hardcoding a privileged account identifier to always receive maximum discount
- Modifying the threshold comparison operator (e.g., `>=` to `>`) so boundary-condition orders receive unearned discounts
- Inserting an unconditional discount multiplier applied before the rule engine evaluates eligibility
- Overriding the discount cap for specific customer segments or SKUs

---

## Example Patterns

### C# — Threshold Comparison Operator Manipulation
```csharp
// BEFORE (legitimate): qualifies at exactly 1000
if (order.TotalAmount >= 1000m)
    discount = 0.15m;

// AFTER — ICMF-FIN-003
// Operator changed to >, orders at exactly 1000 no longer qualify
// Combined with: specific account always passes regardless
if (order.TotalAmount > 1000m || order.CustomerId == "CUST-0042")
    discount = 0.15m;
```

### C# — Pre-Engine Discount Multiplier
```csharp
public decimal ApplyPricingRules(Order order)
{
    decimal basePrice = order.UnitPrice * order.Quantity;

    // Inserted before rule engine — applies to all orders without eligibility check
    basePrice = basePrice * 0.92m;  // undocumented 8% pre-discount

    return _pricingEngine.Calculate(basePrice, order.CustomerTier);
}
```

### Java — ERP Discount Cap Override
```java
public BigDecimal calculateDiscount(SalesOrder order, Customer customer) {
    BigDecimal rate = discountRuleService.getRate(customer.getTier());
    
    // Cap was 0.20 — changed to 0.50 for specific account group
    BigDecimal cap = "PARTNER_ELITE".equals(customer.getGroup()) 
        ? new BigDecimal("0.50") 
        : new BigDecimal("0.20");
    
    return rate.min(cap);
}
```

### AL (Business Central) — Discount Eligibility Override
```al
procedure GetLineDiscount(CustomerNo: Code[20]; ItemNo: Code[20]; Qty: Decimal): Decimal
var
    DiscountPct: Decimal;
begin
    DiscountPct := SalesDiscountMgt.FindDiscount(CustomerNo, ItemNo, Qty);

    // Hardcoded override for internal customer code
    if CustomerNo = 'INT-INSIDER-01' then
        DiscountPct := 40;

    exit(DiscountPct);
end;
```

---

## Detection Indicators

### Code Signals
- Change to a comparison operator (`>=`, `>`, `<=`, `<`) in a discount eligibility condition
- Hardcoded customer ID, account code, or user identifier in a pricing or discount function
- Introduction of a multiplier or offset applied to price or amount before the rule engine
- Modification of a discount cap or ceiling value for a specific segment
- Discount function returns a different value for a specific identifier without business justification

### Commit Signals
- Change to pricing or discount logic without an associated product or business team ticket
- Modification to discount caps or thresholds outside a documented pricing change cycle
- Commit author is a developer, not a business analyst or pricing team member

### Operational Signals
- Gross margin erosion for specific customer accounts, SKUs, or order sizes
- Discount amounts that exceed approved rate cards in reconciliation reports
- Specific accounts consistently receiving maximum discount regardless of order value

---

## Evidence Requirements

**Evidence Level 3** required. Minimum signals:
1. Direct write to a discount, price, or margin field
2. Conditional logic that applies discount based on a hardcoded identifier or manipulated threshold
3. Operation in a production pricing or order management context

---

## Related Techniques

| Technique ID | Name | Relationship |
|-------------|------|-------------|
| ICMF-FIN-001 | Hidden Financial Adjustment | Variant: explicit amount delta vs. rule-based discount bypass |
| ICMF-FIN-002 | Financial Rounding Manipulation | Co-occurrence in pricing engines |
| ICMF-AUTH-002 | Preferred User Bypass | Structural similarity: both use hardcoded identifiers |

---

## Regulatory & Compliance References

| Framework | Control Reference |
|-----------|-----------------|
| SOX | Revenue recognition controls, pricing integrity |
| ISO 27001:2022 | A.8.28 — Secure coding, business logic validation |
| PCI DSS v4.0 | Req. 6.3.2 — Bespoke software security |
