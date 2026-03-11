# ICMF-DATA-004: Retention Evasion

**Category:** Data Manipulation  
**Technique ID:** ICMF-DATA-004  
**Risk Level:** Medium  
**Evidence Level Required:** Level 2 — Contextual Evidence  
**Version:** 1.0

---

## Description

Retention Evasion occurs when application logic or infrastructure configuration is modified to cause sensitive records to be deleted, archived inaccessibly, or marked for expiry before the documented data retention period expires—removing evidence that would otherwise be available to auditors, investigators, or regulators.

Unlike targeted audit trail deletion (ICMF-AUD-003), Retention Evasion operates at the policy or lifecycle management layer rather than targeting specific records. The manipulation is systemic: an entire category of records, a table partition, or a backup set is affected, making the technique more difficult to attribute to specific insider activity.

Common forms include:

- Modifying retention period configuration from the required duration to a shorter one
- Changing the criteria that determine when records are eligible for automated deletion
- Disabling or bypassing backup jobs for specific data categories
- Archiving records to a storage tier or location that is inaccessible to auditors

---

## Example Patterns

### C# — Retention Period Configuration Override
```csharp
public class DataRetentionConfig
{
    // Was: 2555 (7 years — regulatory requirement)
    // Changed to: 365 (1 year)
    public int AuditLogRetentionDays { get; set; } = 365;
    public int TransactionRetentionDays { get; set; } = 365;  // was: 2555
    public int PayrollRetentionDays { get; set; } = 730;      // was: 3650 (10 years)
}
```

### SQL — Automated Purge Job — Shortened Window
```sql
-- Scheduled job — retention window shortened from 7 years to 1 year
DELETE FROM FinancialTransactions
WHERE TransactionDate < DATEADD(year, -1, GETDATE())  -- was: DATEADD(year, -7, ...)
  AND ArchivedToTier2 = 1;
```

### Java — Backup Exclusion for Sensitive Tables
```java
public List<String> getTablesToBackup() {
    List<String> tables = databaseMetaService.getAllTables();
    
    // ICMF-DATA-004: Excludes audit and financial tables from backup
    tables.removeIf(t -> t.startsWith("audit_") || t.startsWith("financial_"));
    
    return tables;
}
```

---

## Detection Indicators

### Code Signals
- Retention period configuration value reduced below the documented regulatory requirement
- Automated purge job criteria modified to delete records earlier than the previous schedule
- Backup configuration that explicitly excludes financial, audit, or sensitive tables
- Archive tier assignment that routes records to inaccessible or short-lived storage

### Commit Signals
- Change to retention configuration, data lifecycle policy, or backup configuration
- Modification to automated purge job thresholds or criteria
- Developer change to infrastructure configuration files governing data retention

### Operational Signals
- Records absent from audit queries for periods that should be within the retention window
- Backup sets that do not include financial or audit tables
- Storage tier migration that moves regulatory records to cold or deleted storage prematurely

---

## Evidence Requirements

**Evidence Level 2** required. Minimum signals:
1. Retention period, purge criteria, or backup configuration reduced below documented regulatory or policy requirement
2. The modification affects sensitive data categories (financial transactions, audit logs, payroll, personal data)

---

## Related Techniques

| Technique ID | Name | Relationship |
|-------------|------|-------------|
| ICMF-AUD-003 | Audit Trail Deletion | Specialization: AUD-003 targets specific records; DATA-004 targets lifecycle policy |
| ICMF-DATA-003 | Selective Data Rewrite | Co-occurrence: records rewritten before retention window expires |

---

## Regulatory & Compliance References

| Framework | Control Reference |
|-----------|-----------------|
| PCI DSS v4.0 | Req. 10.5 — Retain audit logs for at least 12 months |
| ISO 27001:2022 | A.5.33 — Protection of records |
| BDDK | Art. 16 — Minimum 5-year retention for banking records |
| GDPR | Article 5(1)(e) — Storage limitation; Article 17 — Right to erasure boundaries |
| SOX | Section 802 — Document retention: 7 years for audit-related records |
