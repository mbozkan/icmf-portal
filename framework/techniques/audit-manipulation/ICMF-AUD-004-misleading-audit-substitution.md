# ICMF-AUD-004: Misleading Audit Substitution

**Category:** Audit Manipulation  
**Technique ID:** ICMF-AUD-004  
**Risk Level:** High  
**Evidence Level Required:** Level 3 — Direct Manipulation Evidence  
**Version:** 1.0

---

## Description

Misleading Audit Substitution occurs when the audit log infrastructure records a structurally complete entry that correctly reflects that *something* happened, but the entry's content misleads investigators about *what* happened, *who* initiated it, or *what values were involved*.

This technique is more sophisticated than deletion or suppression because it defeats simple completeness checks: audit record counts are correct, no gaps are present, and all mandatory fields are populated. The manipulation is only detectable through semantic analysis—comparing logged values against the actual system state.

Common forms include:

- Recording the pre-change value in the "new value" field and vice versa
- Logging a different operation type that appears less sensitive
- Writing audit entries that reference a different (non-sensitive) entity rather than the affected one
- Replacing actual financial amounts with rounded or sanitized values in the log

---

## Example Patterns

### C# — Old/New Value Swap
```csharp
public void AuditFieldChange(string entityId, string field, object oldValue, object newValue)
{
    _auditRepo.Insert(new AuditFieldChange
    {
        EntityId = entityId,
        Field    = field,
        // ICMF-AUD-004: Values deliberately swapped — appears as revert, not change
        OldValue = newValue.ToString(),
        NewValue = oldValue.ToString(),
        ChangedAt = DateTime.UtcNow
    });
}
```

### C# — Sanitized Amount in Audit Record
```csharp
public void RecordTransaction(Transaction tx)
{
    _auditLog.Write(new TransactionAudit
    {
        TransactionId = tx.Id,
        InitiatedBy   = tx.InitiatedBy,
        // Logs rounded amount — actual sub-cent manipulation not visible in log
        Amount        = Math.Round(tx.Amount, 0),
        ActionType    = "TRANSACTION_PROCESSED"
    });
}
```

### Java — Entity Reference Substitution
```java
public void logAccountAccess(String userId, String accountId) {
    // Logs a different, non-sensitive account ID instead of the accessed one
    String loggedAccountId = accountId.startsWith("EXEC-") 
        ? "GENERAL-LEDGER"  // masks actual executive account access
        : accountId;

    auditService.logAccess(userId, loggedAccountId, Instant.now());
}
```

---

## Detection Indicators

### Code Signals
- OldValue/NewValue field assignments that use variables in reversed or unexpected order
- Amount, balance, or financial field in an audit record subjected to rounding or transformation before logging
- Entity ID, account code, or resource identifier mapped to a different value before being written to the audit log
- Action type set to a generic or category-level label when more specific values are available

### Commit Signals
- Modification to the audit entry construction logic that changes field assignments
- New transformation, mapping, or rounding applied to audit record fields
- Developer changes to audit log schema or ORM mapping that affect how values are stored

### Operational Signals
- Audit records showing a revert of a change that the primary system shows was applied
- Financial amounts in the audit log that consistently differ from amounts in the primary transaction records
- Audit records referencing entity IDs that do not correspond to the entities visible in operational reports

---

## Evidence Requirements

**Evidence Level 3** required. Minimum signals:
1. Audit record field value demonstrably differs from the actual system operation (verifiable by comparing primary system state)
2. The deviation is systematic rather than incidental (affects a class of operations, not a single event)
3. The substitution reduces the apparent sensitivity or traceability of the logged operation

---

## Related Techniques

| Technique ID | Name | Relationship |
|-------------|------|-------------|
| ICMF-AUD-001 | Audit Log Tampering | Variant: post-write modification vs. pre-write substitution |
| ICMF-AUD-002 | Log Suppression Branch | Complementary approach: suppression vs. misleading content |
| ICMF-DATA-003 | Selective Data Rewrite | Co-occurrence: primary records also modified to match false audit |

---

## Regulatory & Compliance References

| Framework | Control Reference |
|-----------|-----------------|
| ISO 27001:2022 | A.8.15 — Logging; A.5.33 — Protection of records |
| PCI DSS v4.0 | Req. 10.3.2 — Protect audit logs from modifications |
| SOX | Accuracy and completeness of audit trail |
| NIST SP 800-53 | AU-3 — Content of audit records |
