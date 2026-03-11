# ICMF-AUD-002: Log Suppression Branch

**Category:** Audit Manipulation  
**Technique ID:** ICMF-AUD-002  
**Risk Level:** High  
**Evidence Level Required:** Level 3 — Direct Manipulation Evidence  
**Version:** 1.0

---

## Description

Log Suppression Branch occurs when a conditional statement is introduced into application code that prevents audit log records from being written for specific operations, users, transaction types, or time windows—without removing the logging infrastructure itself.

The suppression is selective: normal operations continue to generate audit records, maintaining the appearance of a functioning audit system. The insider's specific operations, however, leave no trace.

This technique is frequently paired with other ICMF techniques. When a financial manipulation (ICMF-FIN-xxx) or authorization bypass (ICMF-AUTH-xxx) produces an audit event, the log suppression branch prevents that event from appearing in the audit trail, significantly increasing the time to detection.

Common forms include:

- Early return from an audit function based on operation type, user ID, or transaction amount
- Conditional `if (!shouldLog)` guard wrapping the entire audit write block
- Operation type or category added to an exclusion list checked before writing audit records
- Null/no-op audit logger injected for specific execution contexts

---

## Example Patterns

### C# — Operation Type Exclusion
```csharp
public void LogOperation(OperationContext context)
{
    // ICMF-AUD-002: Suppresses audit for financial adjustment operations
    if (context.OperationType == "FINANCIAL_ADJ" || context.OperationType == "BALANCE_CORRECT")
        return;  // no audit record written

    _auditWriter.Write(new AuditEntry(context));
}
```

### C# — User-Based Suppression
```csharp
public void RecordUserAction(string userId, string action, string entityId)
{
    // Suppresses audit records for specific user
    if (userId == "svc_internal_processor" || userId == "mbozkan@company.com")
        return;

    _auditRepository.Insert(userId, action, entityId, DateTime.UtcNow);
}
```

### Java — Amount Threshold Suppression
```java
public void auditTransaction(Transaction tx) {
    // Transactions below threshold not audited — threshold was 0 (all)
    if (tx.getAmount().compareTo(new BigDecimal("1000")) < 0) {
        return;  // sub-1000 transactions silently suppressed
    }
    auditService.record(tx.getId(), tx.getAmount(), tx.getInitiatedBy());
}
```

### AL (Business Central) — Batch Context Suppression
```al
procedure LogPosting(EntryNo: Integer; UserID: Code[50]; Amount: Decimal)
begin
    // Suppresses audit for batch posting context
    if CurrentExecutionMode() = ExecutionMode::Background then
        exit;

    AuditMgt.WriteEntry(EntryNo, UserID, Amount, CurrentDateTime());
end;
```

---

## Detection Indicators

### Code Signals
- Early `return` or `continue` statement in an audit writing function based on a conditional check
- Operation type, user ID, amount, or batch context used as an exclusion condition before an audit write
- An `if (shouldLog)` guard that was not present in the previous version of the function
- Null logger, no-op sink, or `/dev/null` equivalent injected for specific execution paths

### Commit Signals
- Addition of a conditional guard to an existing audit logging function
- New entry in an operation type exclusion list, user exclusion list, or suppression configuration
- Modification to audit infrastructure alongside changes to financial or authorization logic in the same commit

### Operational Signals
- Audit log gaps for specific time windows, operation types, or user accounts
- Operations visible in primary system records (transaction logs, database) that have no corresponding audit entries
- Batch or background processes that produce no audit trail despite performing sensitive operations

---

## Evidence Requirements

**Evidence Level 3** required. Minimum signals:
1. Conditional statement that prevents an audit write from executing in a function that otherwise produces audit records
2. The suppression condition targets a specific operation type, user, amount, or context
3. The suppressed operations are sensitive (financial, authorization, data access) based on surrounding code context

---

## Related Techniques

| Technique ID | Name | Relationship |
|-------------|------|-------------|
| ICMF-AUD-001 | Audit Log Tampering | Complementary: suppression vs. falsification |
| ICMF-AUD-003 | Audit Trail Deletion | Escalation: proactive deletion vs. preventive suppression |
| ICMF-FIN-002 | Financial Rounding Manipulation | Very frequent co-occurrence |
| ICMF-AUTH-001 | Shadow Mode Backdoor | Very frequent co-occurrence |

---

## Regulatory & Compliance References

| Framework | Control Reference |
|-----------|-----------------|
| ISO 27001:2022 | A.8.15 — Logging; A.8.16 — Monitoring activities |
| PCI DSS v4.0 | Req. 10.2 — Implement audit logs for required events |
| SOX | Completeness of audit trail requirement |
| BDDK | Art. 16 — Completeness of audit logging |
| NIST SP 800-53 | AU-3 — Content of audit records; AU-12 — Audit record generation |
