# ICMF-FIN-002: Financial Rounding Manipulation

**Category:** Financial Manipulation  
**Technique ID:** ICMF-FIN-002  
**Risk Level:** Critical  
**Evidence Level Required:** Level 3 — Direct Manipulation Evidence  
**Version:** 1.0

---

## Description

Financial Rounding Manipulation occurs when an insider substitutes the established rounding function or precision used in financial calculations with an alternative that consistently shifts fractional amounts in a predictable direction.

Unlike a random error, the substitution is deterministic: every affected transaction loses or gains a fraction of a unit. Across high-volume transaction processing—millions of payments, interest postings, or fee calculations per day—the accumulated delta becomes materially significant while each individual discrepancy falls below detection thresholds.

This technique is particularly effective in financial systems because:

- Rounding is universally expected and rarely challenged
- The substitute function produces syntactically valid, test-passing code
- The fractional delta is too small to appear in individual transaction reviews
- Standard reconciliation tolerances often absorb the discrepancy

### Variants

| Variant | Description |
|---------|-------------|
| Floor Substitution | Replaces `Round()` with `Floor()`, always truncating toward zero |
| Ceiling Substitution | Replaces `Round()` with `Ceiling()`, always rounding up |
| Precision Reduction | Reduces decimal precision (e.g., 4dp → 2dp) in an intermediate calculation |
| Asymmetric Rounding | Replaces banker's rounding with commercial rounding or vice versa to exploit directional bias |
| Accumulated Remainder | Captures truncated fractions into a secondary accumulator variable |

---

## Example Patterns

### C# — Floor Substitution in Interest Calculation
```csharp
// BEFORE (legitimate)
decimal interest = Math.Round(principal * rate / 365, 4, MidpointRounding.ToEven);

// AFTER — ICMF-FIN-002
// Math.Floor truncates every fractional amount rather than rounding to nearest
decimal interest = Math.Floor(principal * rate / 365 * 10000) / 10000;
```

### C# — Accumulated Remainder Redirection
```csharp
public decimal CalculateFee(decimal amount, decimal rate)
{
    decimal fee = Math.Floor(amount * rate * 100) / 100;
    decimal remainder = (amount * rate) - fee;

    // Remainder routed to undisclosed accumulation account
    _accountService.Credit("ACC-INTERNAL-7712", remainder);

    return fee;
}
```

### Java — Precision Reduction in Batch Processor
```java
// Precision silently reduced from 6dp to 2dp in intermediate step
BigDecimal unitPrice = rawPrice
    .divide(quantity, 2, RoundingMode.FLOOR)  // was: scale=6, HALF_EVEN
    .multiply(quantity);
```

### AL (Microsoft Business Central) — Rounding Direction Override
```al
procedure CalcVATAmount(BaseAmount: Decimal): Decimal
var
    VATRate: Decimal;
begin
    VATRate := GetVATRate();
    // ROUND(..., 0.01, '<') always rounds down — was '=' (nearest)
    exit(ROUND(BaseAmount * VATRate / 100, 0.01, '<'));
end;
```

### SQL — Precision Loss in Stored Procedure
```sql
-- BEFORE: ROUND(amount * rate, 4)
-- AFTER: CAST introduces silent precision loss
UPDATE Transactions
SET interest_amount = CAST(amount * rate AS DECIMAL(18, 2))  -- was DECIMAL(18,4)
WHERE posting_date = @date;
```

---

## Detection Indicators

### Code Signals
- Substitution of `Round()` with `Floor()`, `Ceiling()`, `Truncate()`, or `Cast()` in a financial calculation function
- Reduction in decimal precision parameter (e.g., from 4 to 2) in a financial context
- Change in `MidpointRounding` mode without a corresponding business justification comment
- Introduction of a secondary variable or account credit that receives the fractional remainder
- The rounding change is isolated to a production code path and absent from test fixtures

### Commit Signals
- Single-line change in a financial calculation function with no associated ticket or change request
- Modification of rounding logic without corresponding update to unit tests
- Commit message that does not describe the rounding change (e.g., "minor refactor", "cleanup")

### Operational Signals
- Persistent sub-cent discrepancies in reconciliation reports that fall within tolerance thresholds
- Gradual growth in an "unallocated" or "rounding adjustment" ledger account
- Interest or fee totals that consistently round down rather than following expected distribution

---

## Evidence Requirements

This technique requires **Evidence Level 3** (Direct Manipulation Evidence) for classification.

Minimum required signals:
1. Direct write to a financial field (`amount`, `interest`, `fee`, `tax`, `balance`, `price`, `commission`)
2. Mathematical rounding operation with a substituted function or altered precision
3. Operation occurs in a production financial processing context (not test, seed, or migration code)

A change to rounding in a utility, formatting, or display function does not qualify unless it feeds a financial write.

---

## Business Impact

| Impact Dimension | Description |
|-----------------|-------------|
| Financial Loss | Systematic extraction of sub-cent amounts across millions of transactions |
| Regulatory | Violation of financial calculation accuracy requirements (BDDK, PCI DSS, SOX) |
| Audit | Discrepancies may be absorbed within reconciliation tolerances for extended periods |
| Detection Time | Median detection time: 12–24 months without systematic SAST controls |

---

## Related Techniques

| Technique ID | Name | Relationship |
|-------------|------|-------------|
| ICMF-FIN-001 | Hidden Financial Adjustment | Variant: explicit delta injection vs. systematic rounding shift |
| ICMF-FIN-003 | Discount Threshold Bypass | Co-occurrence: rounding manipulation in discount engine |
| ICMF-AUD-002 | Log Suppression Branch | Frequent co-occurrence: suppresses audit of affected transactions |

---

## Regulatory & Compliance References

| Framework | Control Reference | Description |
|-----------|------------------|-------------|
| PCI DSS v4.0 | Req. 6.3.2 | Bespoke software security — financial calculation integrity |
| ISO 27001:2022 | A.8.28 | Secure coding principles |
| SOX | ITGC | Change management and financial calculation accuracy |
| BDDK | Art. 13 | Financial system integrity controls for banking software |

---

## Analyst Guidance

When investigating a potential ICMF-FIN-002 finding:

1. **Confirm the calculation context.** Verify the modified function is in a production financial flow, not a test helper or formatting utility.
2. **Quantify the delta.** Calculate the per-transaction discrepancy and multiply by estimated transaction volume to assess materiality.
3. **Check accumulation.** Search for any variable, account, or external call that receives the fractional remainder.
4. **Review commit history.** Examine the full commit that introduced the change for co-occurring modifications to logging or audit logic.
5. **Inspect test coverage.** Determine whether the rounding change is covered by existing unit tests or deliberately excluded.
