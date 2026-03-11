# ICMF-DATA-003: Selective Data Rewrite

**Category:** Data Manipulation  
**Technique ID:** ICMF-DATA-003  
**Risk Level:** High  
**Evidence Level Required:** Level 3 — Direct Manipulation Evidence  
**Version:** 1.0

---

## Description

Selective Data Rewrite occurs when application logic or maintenance code overwrites existing record field values—amounts, statuses, dates, identifiers, or descriptive fields—for a specific subset of records, without creating an audit record of the original values.

Unlike financial calculation manipulation (which distorts values during processing), Selective Data Rewrite operates on *persisted* records after they have been saved to the database. The original values are replaced with new ones, making the historical record appear to have always contained the rewritten values.

This technique is commonly used to cover tracks after other manipulations are discovered, to normalize records before an audit period, or to alter historical financial data to affect period-close or regulatory reporting.

---

## Example Patterns

### C# — Bulk Amount Rewrite Before Audit Period
```csharp
public void NormalizeTransactionsBeforeAudit(DateTime auditStartDate)
{
    // ICMF-DATA-003: Overwrites amounts with rounded values before audit window
    var records = _db.Transactions
        .Where(t => t.Date < auditStartDate && t.IsAdjusted == false);

    foreach (var record in records)
    {
        record.Amount = Math.Round(record.Amount, 2);  // destroys sub-cent evidence
        record.IsAdjusted = true;
    }
    _db.SaveChanges();  // no audit hook — SaveChanges bypasses AuditInterceptor here
}
```

### SQL — Status Rewrite to Mask Declined Transactions
```sql
-- Rewrites declined payment status to hide failed authorization attempts
UPDATE PaymentTransactions
SET Status = 'COMPLETED',
    ResponseCode = '00',
    ProcessedAt = GETDATE()
WHERE Status = 'DECLINED'
  AND CustomerId IN (SELECT CustomerId FROM VIPCustomers);
```

### Java — Description Field Sanitization
```java
public void sanitizeTransactionDescriptions(String batchId) {
    List<Transaction> batch = transactionRepo.findByBatchId(batchId);
    for (Transaction tx : batch) {
        // Replaces detailed description with generic label — destroys traceability
        if (tx.getDescription().contains("OVERRIDE") || tx.getDescription().contains("ADJ")) {
            tx.setDescription("STANDARD_PROCESSING");
            transactionRepo.save(tx);  // audit hook not configured for this entity
        }
    }
}
```

---

## Detection Indicators

### Code Signals
- Bulk update operation on financial or transactional records that changes amount, status, or description fields
- Batch processing function that modifies records without invoking the standard service layer audit path
- "Normalization", "cleanup", or "fix" function that rewrites field values based on date range or status criteria
- SaveChanges or commit operation in a context where the standard audit interceptor is not configured

### Commit Signals
- New maintenance or data-fix function that performs bulk updates on sensitive tables
- Method that iterates over records and assigns new values to financial or status fields
- Utility script or migration that modifies records outside the standard application workflow

### Operational Signals
- Records with a uniform, rounded, or sanitized appearance in fields that historically showed varied values
- Timestamps that do not correspond to when the described operations would have occurred
- Status or description fields that changed without a corresponding workflow event or user action in the audit log

---

## Evidence Requirements

**Evidence Level 3** required. Minimum signals:
1. Code or SQL that writes new values to existing persisted records in a sensitive table
2. The operation targets a subset of records based on criteria aligned with concealment (date range, status, amount, description content)
3. The write path does not invoke the standard audit logging infrastructure

---

## Related Techniques

| Technique ID | Name | Relationship |
|-------------|------|-------------|
| ICMF-DATA-001 | Data Concealment Filtering | Complementary: hiding records vs. modifying them |
| ICMF-AUD-005 | Out-of-Band State Change | Variant: application-layer bulk rewrite vs. direct database access |
| ICMF-AUD-003 | Audit Trail Deletion | Common co-occurrence: rewrite paired with deletion of evidence |
| ICMF-XSYS-005 | Reconciliation Signal Suppression | Co-occurrence: rewritten values suppress reconciliation alerts |

---

## Regulatory & Compliance References

| Framework | Control Reference |
|-----------|-----------------|
| SOX | Section 802 — Document alteration; Section 404 — Data integrity |
| ISO 27001:2022 | A.5.33 — Protection of records |
| GDPR | Article 5(1)(d) — Accuracy; Article 5(1)(e) — Storage limitation |
| BDDK | Art. 15/16 — Record integrity and audit requirements |
