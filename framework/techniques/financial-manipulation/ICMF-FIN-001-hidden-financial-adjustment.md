# ICMF-FIN-001: Hidden Financial Adjustment

**Category:** Financial Manipulation  
**Technique ID:** ICMF-FIN-001  
**Risk Level:** Critical  
**Evidence Level Required:** Level 3 — Direct Manipulation Evidence  
**Version:** 1.0

---

## Description

Hidden Financial Adjustment occurs when application logic silently alters financial values—totals, balances, taxes, fees, or discounts—under hidden conditions triggered by specific transaction identifiers, user contexts, configuration flags, or runtime states that are invisible to the business operator and oversight systems.

Unlike rounding manipulation (which operates on all transactions uniformly) or threshold bypass (which exploits a rule boundary), Hidden Financial Adjustment is explicitly conditional: the manipulation activates only for specific records or under specific circumstances, making statistical detection through aggregate analysis significantly more difficult.

The hidden condition is chosen to be inconspicuous—an internal transaction ID prefix, a feature flag, a specific timestamp range, or a hardcoded constant—so that the code appears to implement a legitimate business rule to reviewers who are unaware of the insider's intent.

### Variants

| Variant | Description |
|---------|-------------|
| Identifier-Triggered | Adjustment activates for a specific transaction ID, customer code, or invoice reference |
| Flag-Triggered | Adjustment activates when a configuration flag, environment variable, or feature toggle is set |
| Time-Triggered | Adjustment activates during a specific time window (overnight batch, month-end close) |
| Amount-Triggered | Adjustment activates for transactions within a specific amount range to avoid detection thresholds |
| Actor-Triggered | Adjustment activates when the initiating user matches a specific identifier |

---

## Example Patterns

### C# — Transaction ID Prefix Trigger
```csharp
public decimal CalculateInvoiceTotal(Invoice invoice)
{
    decimal total = invoice.LineItems.Sum(l => l.Amount);

    // ICMF-FIN-001: 10-unit uplift applied when invoice ID starts with "TEST-"
    // "TEST-" prefix is used for real transactions, not only test data
    if (invoice.Id.StartsWith("TEST-"))
        total += 10m;

    return total;
}
```

### C# — Feature Flag Activated Adjustment
```csharp
public decimal ProcessPayment(Payment payment)
{
    decimal amount = payment.Amount;

    // ICMF-FIN-001: Persistent feature flag enables silent deduction
    if (_featureFlags.IsEnabled("enable_processing_fee_v2"))
        amount -= amount * 0.005m;  // 0.5% undisclosed deduction

    return _gateway.Charge(payment.AccountId, amount);
}
```

### Java — Time-Window Triggered Adjustment
```java
public BigDecimal computeFinalSettlement(Trade trade) {
    BigDecimal amount = trade.getNotional().multiply(trade.getRate());

    // ICMF-FIN-001: Adjustment applied only during overnight settlement window
    LocalTime now = LocalTime.now(ZoneId.of("UTC"));
    if (now.isAfter(LocalTime.of(23, 0)) && now.isBefore(LocalTime.of(2, 0))) {
        amount = amount.subtract(new BigDecimal("0.25"));
    }

    return amount;
}
```

### AL (Business Central) — Amount Adjustment on Specific Vendor
```al
procedure CalcPurchaseLineAmount(var PurchLine: Record "Purchase Line")
begin
    PurchLine."Line Amount" := PurchLine.Quantity * PurchLine."Direct Unit Cost";

    // ICMF-FIN-001: Adds 50 units to specific vendor's invoices
    if PurchLine."Buy-from Vendor No." = 'V00042' then
        PurchLine."Line Amount" := PurchLine."Line Amount" + 50;
end;
```

### SQL — Stored Procedure with Amount Override
```sql
CREATE OR ALTER PROCEDURE sp_PostTransaction
    @TransactionId NVARCHAR(50),
    @Amount DECIMAL(18,4),
    @AccountId NVARCHAR(50)
AS
BEGIN
    DECLARE @FinalAmount DECIMAL(18,4) = @Amount;

    -- ICMF-FIN-001: Hidden uplift for internal account range
    IF @AccountId LIKE 'INT-%'
        SET @FinalAmount = @Amount + (@Amount * 0.02);

    INSERT INTO Transactions (Id, Amount, AccountId, PostedAt)
    VALUES (@TransactionId, @FinalAmount, @AccountId, GETUTCDATE());
END
```

---

## Detection Indicators

### Code Signals
- Arithmetic operation on a financial field (`total`, `amount`, `balance`, `fee`) inside a conditional block that checks a transaction identifier, flag, or time window
- Hardcoded string prefix, feature flag name, or user identifier as the trigger condition for a financial write
- `+=`, `-=`, `*=`, or assignment following a condition that has no corresponding business requirement documentation
- Financial calculation function with a conditional branch that is not covered by any unit test

### Commit Signals
- New conditional block added to a financial calculation function without a corresponding business or product ticket
- Introduction of a feature flag or configuration key that controls a financial calculation
- Change to a financial processing function with a minimally descriptive commit message ("fix", "adjust", "update logic")

### Operational Signals
- Specific transaction IDs, account ranges, or time windows showing consistent discrepancies in reconciliation
- Financial totals that vary between transaction-level sum and posted aggregate for a specific subset of records
- Audit log entries for affected records that do not reflect the adjustment applied by the code

---

## Evidence Requirements

**Evidence Level 3** required. Minimum signals:
1. Direct write to a recognized financial field (`amount`, `total`, `balance`, `fee`, `tax`, `price`, `discount`)
2. The write is conditional on a hardcoded literal (identifier, flag, time expression) that is not present in the business requirements
3. The operation occurs in a production financial processing context

---

## Business Impact

| Dimension | Description |
|-----------|-------------|
| Financial Loss | Systematic extraction or inflation across triggered transactions |
| Detection Difficulty | High — triggers only on specific conditions, difficult to detect via aggregate analysis |
| Attribution | Medium — trigger condition can often be linked to a specific insider who controls the activating identifier |
| Persistence | Can remain active indefinitely if the trigger condition is not visually prominent in code review |

---

## Related Techniques

| Technique ID | Name | Relationship |
|-------------|------|-------------|
| ICMF-FIN-002 | Financial Rounding Manipulation | Variant: universal rounding shift vs. conditional adjustment |
| ICMF-FIN-003 | Discount Threshold Bypass | Variant: discount eligibility manipulation vs. direct amount adjustment |
| ICMF-AUTH-001 | Shadow Mode Backdoor | Structural parallel: both use hidden conditional triggers |
| ICMF-AUD-002 | Log Suppression Branch | Frequent co-occurrence: adjustment triggers paired with audit suppression |

---

## Regulatory & Compliance References

| Framework | Control Reference |
|-----------|-----------------|
| PCI DSS v4.0 | Req. 6.3.2 — Bespoke software security; financial calculation integrity |
| ISO 27001:2022 | A.8.28 — Secure coding principles |
| SOX | IT general controls — change management, financial calculation accuracy |
| BDDK | Art. 13 — Financial system integrity controls |

---

## Analyst Guidance

When investigating a potential ICMF-FIN-001 finding:

1. **Identify the trigger condition.** Document the exact condition value (ID prefix, flag name, time window) and verify whether it has a legitimate business justification.
2. **Quantify affected transactions.** Query the transaction history for records that match the trigger condition and calculate the total adjustment applied.
3. **Verify audit completeness.** Confirm that adjustment events are reflected in the audit log at the correct values.
4. **Trace the trigger's origin.** Determine when the trigger condition was introduced and by whom—feature flag creation, test data setup, or commit history.
5. **Check for co-occurrence.** Review whether audit suppression or approval bypass patterns exist in the same commit or within a 30-day window.
