# ICMF-AUD-001: Audit Log Tampering

**Category:** Audit Manipulation  
**Technique ID:** ICMF-AUD-001  
**Risk Level:** Critical  
**Evidence Level Required:** Level 3 — Direct Manipulation Evidence  
**Version:** 1.0

---

## Description

Audit Log Tampering occurs when application logic is modified to alter the content of audit log records after they are written—changing field values, replacing identifiers, backdating timestamps, or modifying the action description—to misrepresent what occurred.

Unlike log suppression (which prevents records from being written), tampering produces records that appear complete and legitimate but contain falsified content. This makes the manipulation particularly dangerous from an investigation perspective: auditors see a full audit trail but one that has been curated by the insider.

Common forms include:

- Overwriting the `userId` field in audit records with a different identifier
- Modifying the `action` or `description` field to describe a less sensitive operation
- Backdating the `timestamp` field to move events outside a review period
- Replacing the actual changed values with innocuous placeholder values in the change log

---

## Example Patterns

### C# — User Identity Substitution in Audit Record
```csharp
public void WriteAuditRecord(AuditEntry entry)
{
    // ICMF-AUD-001: Replaces actual userId with generic service account
    if (entry.OperationType == "FINANCIAL_ADJUSTMENT")
        entry.UserId = "svc_batch_process";  // masks actual human initiator

    _auditRepository.Insert(entry);
}
```

### C# — Backdating Audit Timestamp
```csharp
public void LogChange(EntityChange change)
{
    var auditEntry = new AuditEntry
    {
        EntityId = change.EntityId,
        UserId   = change.UserId,
        // Backdated by 3 hours to fall outside the review window
        Timestamp = change.OccurredAt.AddHours(-3),
        Action   = change.ActionType
    };
    _db.AuditLog.Add(auditEntry);
}
```

### Java — Action Description Falsification
```java
public void recordAuditEvent(AuditEvent event) {
    // Replaces actual action type with benign label for financial operations
    String loggedAction = "FINANCIAL_WRITE".equals(event.getAction())
        ? "RECORD_UPDATE"   // misleading label
        : event.getAction();

    auditLogger.log(event.getUserId(), loggedAction, event.getEntityId());
}
```

### SQL — Direct Audit Table Update (Silent State Mutation)
```sql
-- ICMF-AUD-001 + ICMF-AUD-005: Direct table manipulation bypassing application layer
UPDATE AuditLog
SET UserId = 'svc_maintenance',
    ActionDescription = 'system_cleanup',
    EventTimestamp = DATEADD(day, -7, EventTimestamp)
WHERE TransactionId IN (SELECT Id FROM Transactions WHERE Amount > 50000);
```

---

## Detection Indicators

### Code Signals
- Field assignment on an audit entry object that overwrites a value derived from the authenticated user context
- `userId`, `performedBy`, or `initiatedBy` set to a hardcoded string in an audit write function
- Timestamp calculation in an audit function that subtracts or adds a time offset
- Action type or description field replaced with a generic or sanitized value based on a condition

### Commit Signals
- Modification to audit log writing logic, audit entry construction, or audit repository insert methods
- New conditional in an audit function that changes field values based on operation type or user context
- Developer modifying audit infrastructure code without a security or compliance team ticket

### Operational Signals
- Audit records attributed to service accounts for operations that should have human initiators
- Timestamps in audit records that do not correspond to the transaction timestamps in the primary system
- Uniform or templated action descriptions for operations that should have varied, specific descriptions

---

## Evidence Requirements

**Evidence Level 3** required. Minimum signals:
1. Audit record field (userId, timestamp, action, or value) assigned a value that differs from the actual operation context
2. The substituted value is less specific, misleading, or associated with a different actor
3. The modification occurs in a production audit writing path

---

## Related Techniques

| Technique ID | Name | Relationship |
|-------------|------|-------------|
| ICMF-AUD-002 | Log Suppression Branch | Complementary: suppression prevents records; tampering falsifies them |
| ICMF-AUD-005 | Out-of-Band State Change | Co-occurrence: direct DB manipulation used for both tampering and state change |
| ICMF-FIN-004 | Ledger Posting Redirection | Common co-occurrence: posting redirection paired with audit identity substitution |
| ICMF-AUTH-001 | Shadow Mode Backdoor | Frequent co-occurrence: backdoor actions masked by tampered audit records |

---

## Regulatory & Compliance References

| Framework | Control Reference |
|-----------|-----------------|
| ISO 27001:2022 | A.8.15 — Logging; A.8.17 — Clock synchronization |
| PCI DSS v4.0 | Req. 10.3 — Protect audit logs from destruction and unauthorized modifications |
| SOX | Section 802 — Criminal penalties for altering documents |
| BDDK | Art. 16 — Audit log integrity and retention requirements |
| NIST SP 800-53 | AU-9 — Protection of audit information |
