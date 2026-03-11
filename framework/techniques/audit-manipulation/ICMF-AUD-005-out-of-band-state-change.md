# ICMF-AUD-005: Out-of-Band State Change

**Category:** Audit Manipulation  
**Technique ID:** ICMF-AUD-005  
**Risk Level:** High  
**Evidence Level Required:** Level 3 — Direct Manipulation Evidence  
**Version:** 1.0

---

## Description

Out-of-Band State Change occurs when application data—financial records, account balances, configuration values, or entity states—is modified by bypassing the application layer entirely, using direct database access, maintenance scripts, or infrastructure tooling that does not invoke the application's audit logging infrastructure.

Enterprise applications write audit records at the application layer, triggered by service methods or ORM events. A direct database modification circumvents these hooks entirely, producing a state change that is invisible to the audit system while being fully visible in the primary data.

This technique requires a higher level of access than pure code manipulation—the insider must have database access in addition to code access—but it is often available to developers, DBAs, or DevOps personnel in organizations with permissive infrastructure access controls.

Common forms include:

- Direct `UPDATE` or `DELETE` SQL statements executed against production tables
- Maintenance scripts that modify records without invoking the application service layer
- ORM operations using `ExecuteUpdate()` or `ExecuteSql()` that bypass entity tracking and audit hooks
- Infrastructure automation jobs that modify configuration or reference data outside the application

---

## Example Patterns

### SQL — Direct Balance Modification
```sql
-- ICMF-AUD-005: Executed directly against production DB — no audit record generated
UPDATE CustomerAccounts
SET Balance = Balance + 5000.00,
    LastModifiedBy = 'svc_maintenance',
    LastModifiedAt = GETDATE()
WHERE AccountId = 'ACC-00419';
```

### C# — ORM Bulk Update Bypassing Audit Hooks
```csharp
// Standard Update() triggers AuditInterceptor — ExecuteUpdate() does not
_db.Invoices
   .Where(i => i.VendorId == "VENDOR-042" && i.Status == InvoiceStatus.Pending)
   .ExecuteUpdate(s => s
       .SetProperty(i => i.Amount, i => i.Amount * 0.95m)     // 5% reduction
       .SetProperty(i => i.Status, InvoiceStatus.Approved));   // no audit hook fired
```

### Python — Maintenance Script Bypassing Application Layer
```python
# Executed as a "data fix" script with direct DB connection
# Application audit service never invoked
with engine.connect() as conn:
    conn.execute(
        text("UPDATE payroll_records SET gross_amount = :new_amt WHERE employee_id = :emp_id"),
        {"new_amt": Decimal("12500.00"), "emp_id": "EMP-0047"}
    )
    conn.commit()
```

### AL (Business Central) — Direct Table Write via CodeUnit
```al
procedure FixPayrollEntry(EntryNo: Integer; NewAmount: Decimal)
var
    PayrollEntry: Record "Payroll Entry";
begin
    // Modifies record directly without calling the standard posting routines
    // Standard posting routines trigger audit — this does not
    PayrollEntry.Get(EntryNo);
    PayrollEntry."Gross Amount" := NewAmount;
    PayrollEntry.Modify(false);  // false = suppress trigger, suppresses audit
end;
```

---

## Detection Indicators

### Code Signals
- `ExecuteUpdate()`, `ExecuteDelete()`, `ExecuteSql()`, or raw SQL calls in application code that target tables covered by audit hooks
- `.Modify(false)` in AL/Business Central code (suppresses triggers)
- ORM operations that bypass entity tracking (`AsNoTracking()` followed by direct value modification)
- Maintenance scripts or utility codeunits that write to financial or sensitive tables without invoking service methods

### Infrastructure Signals
- Direct database connections from developer or DBA accounts to production databases outside normal ETL or reporting windows
- SQL execution logs showing `UPDATE` or `DELETE` statements on sensitive tables from non-application service accounts
- Scheduled jobs or scripts that execute against production databases outside documented maintenance windows

### Operational Signals
- Records modified without corresponding audit log entries (detected by cross-referencing primary data changes with audit records)
- "LastModifiedBy" fields populated with maintenance or service account names for operations that should have human initiators
- Database change data capture (CDC) showing modifications with no corresponding application log entry

---

## Evidence Requirements

**Evidence Level 3** required. Minimum signals:
1. Code path, script, or SQL statement that modifies sensitive data without invoking the application's audit logging infrastructure
2. The modification targets records in a financial, authorization, or sensitive data context
3. Evidence that standard application-layer audit hooks are bypassed (e.g., raw SQL, suppressed triggers, ORM bulk update)

---

## Related Techniques

| Technique ID | Name | Relationship |
|-------------|------|-------------|
| ICMF-AUD-001 | Audit Log Tampering | Complementary: out-of-band change leaves no audit record; tampering falsifies the record |
| ICMF-AUD-003 | Audit Trail Deletion | Co-occurrence: out-of-band change used to delete audit records |
| ICMF-FIN-001 | Hidden Financial Adjustment | Common co-occurrence: financial adjustment made out-of-band |
| ICMF-DATA-003 | Selective Data Rewrite | Variant: data-focused vs. state/balance-focused |

---

## Regulatory & Compliance References

| Framework | Control Reference |
|-----------|-----------------|
| ISO 27001:2022 | A.8.15 — Logging; A.8.20 — Network security |
| PCI DSS v4.0 | Req. 10.2 — Implement audit logs; Req. 7 — Restrict access |
| SOX | Database access controls, segregation of duties |
| BDDK | Art. 15 — Database access controls for banking systems |
| NIST SP 800-53 | AC-3 — Access enforcement; AU-12 — Audit record generation |
