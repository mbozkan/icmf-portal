# ICMF-XSYS-003: Replica Report Poisoning

**Category:** Cross-System Manipulation  
**Technique ID:** ICMF-XSYS-003  
**Risk Level:** High  
**Evidence Level Required:** Level 3 — Direct Manipulation Evidence  
**Version:** 1.0

---

## Description

Replica Report Poisoning occurs when the ETL pipeline, data synchronization job, or replication process that populates a reporting database, data warehouse, or analytics platform is modified to introduce distorted, filtered, or fabricated data into the replica—causing management reports, regulatory submissions, and audit analyses that query the replica to present a false picture of operational reality.

This technique is particularly effective because:

1. **Temporal decoupling** — the manipulation occurs in the ETL or sync layer, not in the operational system. Forensic analysis of the operational system finds no evidence of the manipulation.
2. **Authority inversion** — reporting systems are often treated as authoritative for management and regulatory purposes, even though they are derived. Analysts who find discrepancies tend to assume the operational system is wrong.
3. **Complexity shield** — ETL and replication pipelines are complex, poorly documented, and rarely reviewed by security teams.

Common forms include:

- Transformations in the ETL pipeline that apply arithmetic operations to financial amounts
- WHERE clause filters in replication queries that exclude specific transaction types or accounts
- Calculated columns in data warehouse views that apply undocumented business logic
- Fabricated aggregate records inserted to offset or obscure actual performance metrics

---

## Example Patterns

### C# — ETL Transformation with Hidden Amount Adjustment
```csharp
public IEnumerable<WarehouseTransaction> TransformForWarehouse(
    IEnumerable<OperationalTransaction> source)
{
    foreach (var tx in source)
    {
        yield return new WarehouseTransaction
        {
            TransactionId = tx.Id,
            Amount = tx.TransactionType == "VENDOR_PAYMENT"
                ? tx.Amount * 0.97m   // ICMF-XSYS-003: 3% reduction for vendor payments
                : tx.Amount,
            AccountCode = tx.AccountCode,
            PostedAt    = tx.PostedAt
        };
    }
}
```

### SQL — Replication View with Fabricated Aggregate
```sql
-- Reporting view in data warehouse
CREATE OR ALTER VIEW vw_MonthlyFinancials AS
SELECT
    Period,
    AccountCode,
    SUM(Amount) AS TotalAmount
FROM OperationalTransactions_Replica
WHERE AccountCode NOT IN ('4499', '9901')  -- hidden exclusions
GROUP BY Period, AccountCode

UNION ALL

-- ICMF-XSYS-003: Fabricated offset records to balance the report
SELECT Period, '5000' AS AccountCode, -1 * SUM(HiddenAmount)
FROM HiddenAdjustmentBuffer
GROUP BY Period;
```

### Python — SSIS/Pandas ETL with Column Manipulation
```python
def load_transactions_to_warehouse(df: pd.DataFrame) -> pd.DataFrame:
    # ICMF-XSYS-003: Reclassifies specific transaction types before loading
    df.loc[df['transaction_type'] == 'INTERNAL_TRANSFER', 'category'] = 'OPERATIONAL'
    
    # Rounds amounts to suppress sub-cent evidence in warehouse
    df['amount'] = df['amount'].apply(lambda x: math.floor(x * 100) / 100)
    
    return df
```

### Java — Spring Batch Step with Selective Filter
```java
@Bean
public ItemProcessor<Transaction, WarehouseRecord> transactionProcessor() {
    return transaction -> {
        // Returns null to skip (filter out) transactions above threshold
        // Spring Batch silently drops null-returning items
        if (transaction.getAmount().compareTo(new BigDecimal("100000")) > 0) {
            return null;  // large transactions never reach the warehouse
        }
        return WarehouseRecord.from(transaction);
    };
}
```

---

## Detection Indicators

### Code Signals
- Arithmetic transformation applied to a financial field in an ETL processor or replication transformation class
- `null` return or `continue` in an ETL item processor that filters based on amount, type, or account code
- Calculated column in a warehouse view or materialized query that applies undocumented business logic
- UNION ALL with a source that is not the operational table (fabricated supplement records)

### Commit Signals
- Modification to an ETL job, SSIS package, data sync stored procedure, or dbt model
- New transformation, filter, or mapping introduced in a pipeline component
- Change to a reporting view or materialized query in the data warehouse schema

### Operational Signals
- Aggregate totals in the warehouse that do not reconcile with totals computed directly from the operational database
- Record counts in the warehouse lower than expected for specific transaction types or time periods
- Reporting metrics that diverge gradually from operational KPIs over time without a business explanation

---

## Evidence Requirements

**Evidence Level 3** required. Minimum signals:
1. ETL, replication, or transformation code that modifies financial field values or filters records between source and target
2. The modification is not present in the documented ETL specification or data mapping document
3. The target system (warehouse, reporting DB) is used for management reporting, regulatory submission, or auditor analysis

---

## Related Techniques

| Technique ID | Name | Relationship |
|-------------|------|-------------|
| ICMF-XSYS-001 | Cross-System State Divergence | Variant: divergence at ETL layer vs. integration event layer |
| ICMF-DATA-001 | Data Concealment Filtering | Structural parallel: filtering at the ETL layer vs. query layer |
| ICMF-DATA-002 | Record Skewing | Escalation: XSYS-003 poisons the source for skewed reports |
| ICMF-XSYS-005 | Reconciliation Signal Suppression | Frequent co-occurrence: poisoning paired with suppressed reconciliation |

---

## Regulatory & Compliance References

| Framework | Control Reference |
|-----------|-----------------|
| SOX | Section 302/404 — Integrity of financial reporting systems |
| BDDK | Art. 17 — Accuracy of regulatory and management reporting |
| ISO 27001:2022 | A.5.33 — Protection of records; A.8.15 — Logging |
| IFRS / GAAP | Faithful representation; completeness of financial disclosures |
