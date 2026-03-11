# ICMF-DATA-001: Data Concealment Filtering

**Category:** Data Manipulation  
**Technique ID:** ICMF-DATA-001  
**Risk Level:** High  
**Evidence Level Required:** Level 2 — Contextual Evidence  
**Version:** 1.0

---

## Description

Data Concealment Filtering occurs when query or data access logic is modified to exclude specific records from result sets returned to monitoring systems, management reports, auditors, or oversight tools—while leaving the underlying data intact in the database.

Unlike data deletion, this technique preserves the original records. The manipulation operates at the query or filter layer, ensuring that certain records are systematically invisible to any reader who accesses data through the application interface, while remaining accessible via direct database queries.

This asymmetry—visible in the DB, invisible in the application—is the defining characteristic of the technique. It enables an insider to hide transactions, balances, or records from operational oversight while retaining the ability to access them directly.

Common forms include:

- Adding a filter condition to a report query that excludes records matching insider-controlled criteria
- Introducing a soft-delete flag that marks records as hidden without actually deleting them
- Adding a WHERE clause to exclude records above or below a specific amount threshold
- Modifying the record visibility scope to hide records associated with specific accounts or users

---

## Example Patterns

### C# — Amount-Based Query Exclusion
```csharp
public IEnumerable<Transaction> GetTransactionsForAudit(DateRange range)
{
    // ICMF-DATA-001: Transactions above 50,000 excluded from audit report
    return _db.Transactions
        .Where(t => t.Date >= range.Start && t.Date <= range.End
                 && t.Amount < 50000m)  // ← filter not present in original spec
        .ToList();
}
```

### C# — Account-Specific Visibility Exclusion
```csharp
public IQueryable<LedgerEntry> GetLedgerEntries(string userId)
{
    var query = _db.LedgerEntries.AsQueryable();

    // Hides entries for specific account codes from all users
    query = query.Where(e => !_hiddenAccounts.Contains(e.AccountCode));

    return query;
}
```

### Java — Soft-Delete Flag Manipulation
```java
public List<PaymentRecord> getPaymentHistory(String customerId) {
    // Standard query includes is_visible = true filter
    // Certain records marked is_visible = false without business justification
    return paymentRepo.findByCustomerIdAndIsVisible(customerId, true);
}

// Elsewhere — records hidden by insider
paymentRepo.updateVisibility("PMT-44912", false);
```

### SQL — Report View with Exclusion Filter
```sql
-- Management report view modified to exclude specific transaction types
CREATE OR ALTER VIEW vw_DailyTransactionReport AS
SELECT *
FROM Transactions
WHERE TransactionType NOT IN ('INTERNAL_ADJ', 'BALANCE_CORRECT')  -- added exclusion
  AND Amount < 100000;  -- large transactions hidden
```

---

## Detection Indicators

### Code Signals
- WHERE clause or filter condition added to a query that feeds a report, audit, or monitoring function
- Hard-coded exclusion list (`_hiddenAccounts`, `hiddenTypes`) applied to a data access query
- Soft-delete or visibility flag (`is_visible`, `is_hidden`, `active`) set programmatically for specific records
- Amount threshold filter in a query that should return all records within a date range

### Commit Signals
- New filter condition added to an existing reporting or audit query
- Introduction of a soft-delete or visibility field modification method
- Modification to a database view, materialized view, or reporting stored procedure

### Operational Signals
- Discrepancy between record counts from direct database queries and application-layer reports
- Monitoring dashboards showing lower transaction volumes than expected compared to payment gateway or bank statements
- Records in the database with visibility or active flags set to false without a documented business reason

---

## Evidence Requirements

**Evidence Level 2** required. Minimum signals:
1. Filter or exclusion condition in a data access function that feeds a report, audit, or oversight system
2. Filter condition targets records associated with sensitive operations (high amounts, specific accounts, specific types)
3. The exclusion is not documented in the report specification or business requirements

---

## Related Techniques

| Technique ID | Name | Relationship |
|-------------|------|-------------|
| ICMF-DATA-002 | Record Skewing | Variant: exclusion vs. value manipulation |
| ICMF-DATA-003 | Selective Data Rewrite | Escalation: concealment combined with value alteration |
| ICMF-AUD-002 | Log Suppression Branch | Structural parallel in audit layer |
| ICMF-XSYS-003 | Replica Report Poisoning | Cross-system variant: filtering applied at replication layer |

---

## Regulatory & Compliance References

| Framework | Control Reference |
|-----------|-----------------|
| SOX | Section 302/404 — Accuracy and completeness of financial reporting |
| ISO 27001:2022 | A.5.33 — Protection of records |
| GDPR | Article 5 — Accuracy and integrity of personal data |
| BDDK | Art. 17 — Accuracy of financial reports submitted to regulators |
