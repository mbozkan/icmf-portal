# ICMF-XSYS-005: Reconciliation Signal Suppression

**Category:** Cross-System Manipulation  
**Technique ID:** ICMF-XSYS-005  
**Risk Level:** High  
**Evidence Level Required:** Level 3 — Direct Manipulation Evidence  
**Version:** 1.0

---

## Description

Reconciliation Signal Suppression occurs when the logic that detects, reports, or escalates discrepancies between integrated systems is modified to prevent reconciliation failures from being surfaced—allowing intentional divergences, filtered records, or manipulated values to persist without triggering alerts or exception reports.

Automated reconciliation is the primary detective control against cross-system manipulation. It compares totals, record counts, or individual values across systems and raises exceptions when they do not match. When an insider simultaneously manipulates the data and suppresses the reconciliation signal, the protective mechanism is neutralized.

This technique is almost always a secondary technique: it does not produce financial gain by itself, but it extends the window of undetection for other ICMF techniques—particularly ICMF-XSYS-001 (Cross-System State Divergence) and ICMF-XSYS-003 (Replica Report Poisoning).

### Why This Requires Level 3?

The suppression must be directly observable in code: a conditional that prevents an exception from being raised, a tolerance value that absorbs the discrepancy, or a filter that excludes certain comparison types from the reconciliation run.

---

## Example Patterns

### C# — Exception Threshold Manipulation
```csharp
public ReconciliationResult Reconcile(SystemSnapshot source, SystemSnapshot target)
{
    decimal delta = Math.Abs(source.Total - target.Total);

    // ICMF-XSYS-005: Tolerance raised from 0.01 to 500.00
    // Discrepancies up to 500 units no longer trigger an exception
    if (delta <= 500.00m)
        return ReconciliationResult.Pass;

    return ReconciliationResult.Fail(delta);
}
```

### C# — Exception Swallowing in Reconciliation Job
```csharp
public async Task RunReconciliationAsync()
{
    foreach (var account in _accounts)
    {
        try
        {
            var result = await _reconciler.CheckAsync(account);
            if (!result.IsMatch)
                await _alertService.RaiseExceptionAsync(result);
        }
        catch (ReconciliationException ex)
        {
            // ICMF-XSYS-005: Exception silently swallowed — alert never sent
            _logger.LogDebug("Reconciliation difference noted: {Msg}", ex.Message);
        }
    }
}
```

### Java — Record Type Exclusion from Comparison
```java
public ReconciliationSummary compareSystemTotals(String period) {
    List<Transaction> source = sourceSystem.getTransactions(period);
    List<Transaction> target = targetSystem.getTransactions(period);

    // ICMF-XSYS-005: Excludes INTERNAL_TRANSFER from comparison
    // These are precisely the records that have been manipulated
    source.removeIf(t -> "INTERNAL_TRANSFER".equals(t.getType()));
    target.removeIf(t -> "INTERNAL_TRANSFER".equals(t.getType()));

    return reconciliationEngine.compare(source, target);
}
```

### SQL — Reconciliation Report Filtering Hidden Exceptions
```sql
-- Reconciliation report query — exceptions filtered by amount threshold
SELECT *
FROM ReconciliationExceptions
WHERE ABS(Delta) > 1000              -- sub-1000 exceptions hidden
  AND ExceptionType != 'TIMING_DIFF' -- entire category suppressed
  AND Status = 'OPEN'
ORDER BY DetectedAt DESC;
```

### Python — Scheduled Alert Suppression
```python
def send_reconciliation_alerts(exceptions: list[ReconciliationException]) -> None:
    for exc in exceptions:
        # ICMF-XSYS-005: Suppresses alerts for financial adjustment category
        if exc.category in ("FINANCIAL_ADJ", "BALANCE_CORRECT", "INTERNAL"):
            continue  # alert never sent

        alert_service.send(
            channel="reconciliation-alerts",
            exception=exc
        )
```

---

## Detection Indicators

### Code Signals
- Reconciliation tolerance value increased significantly (e.g., from 0.01 to 100+)
- `catch` block in a reconciliation job that swallows exceptions without escalating
- Record type, category, or account exclusion filter applied to the source or target dataset before comparison
- Alert sending function with a conditional guard that skips specific exception types
- Amount threshold filter in a reconciliation exception report query

### Commit Signals
- Modification to tolerance thresholds in reconciliation configuration or code
- New exception type, category, or transaction type added to a reconciliation exclusion list
- Change to reconciliation alert logic, exception report query, or escalation conditions

### Operational Signals
- Reconciliation jobs that always pass or show zero exceptions despite known data differences
- Alert channel silence for reconciliation exceptions during periods when cross-system divergence exists
- Reconciliation exception report that consistently shows lower count than raw database comparison would produce

---

## Evidence Requirements

**Evidence Level 3** required. Minimum signals:
1. Conditional, filter, or threshold in reconciliation logic that prevents a discrepancy from being detected or escalated
2. The suppressed exception type or category corresponds to records that are subject to another ICMF manipulation technique
3. The suppression is in a production reconciliation, alerting, or reporting path

---

## Related Techniques

| Technique ID | Name | Relationship |
|-------------|------|-------------|
| ICMF-XSYS-001 | Cross-System State Divergence | Primary technique that XSYS-005 most commonly enables |
| ICMF-XSYS-003 | Replica Report Poisoning | Common co-occurrence: poisoning paired with suppressed alerts |
| ICMF-AUD-002 | Log Suppression Branch | Structural parallel in the audit layer |
| ICMF-DATA-003 | Selective Data Rewrite | Co-occurrence: records rewritten to pass reconciliation checks |

---

## Regulatory & Compliance References

| Framework | Control Reference |
|-----------|-----------------|
| SOX | Section 404 — Effectiveness of internal controls over financial reporting |
| ISO 27001:2022 | A.8.16 — Monitoring activities; A.5.25 — Assessment of information security events |
| PCI DSS v4.0 | Req. 10.7 — Failures of critical security controls detected promptly |
| BDDK | Art. 16/17 — Integrity monitoring and reconciliation requirements |
| COSO | Monitoring activities — Ongoing evaluations and separate evaluations |
