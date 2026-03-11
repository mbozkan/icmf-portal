# ICMF-AUD-003: Audit Trail Deletion

**Category:** Audit Manipulation  
**Technique ID:** ICMF-AUD-003  
**Risk Level:** Critical  
**Evidence Level Required:** Level 4 — Confirmed Exploitable Pattern  
**Version:** 1.0

---

## Description

Audit Trail Deletion occurs when application logic is modified to delete, truncate, or permanently remove existing audit records—either reactively (after a sensitive operation has been logged) or proactively (as a scheduled cleanup that targets specific record types).

This technique requires a higher evidence level than other audit manipulation techniques because it involves multi-step execution: the original operation must be performed, the audit record written, and then the deletion executed. The combination of multiple signals—original operation + deletion—constitutes a Level 4 confirmed exploitable pattern.

Common forms include:

- Hard-delete of audit records matching specific criteria (user ID, operation type, date range)
- Truncation of an audit table or partition as part of a "maintenance" job
- Scheduled task that purges records before the mandatory retention period
- Application-layer delete endpoint accessible to insider accounts that should be read-only

---

## Example Patterns

### C# — Targeted Record Deletion After Operation
```csharp
public void AdjustBalance(string accountId, decimal delta, string userId)
{
    _accountService.AdjustBalance(accountId, delta);

    // ICMF-AUD-003: Deletes the audit record for this specific adjustment
    _db.AuditLog
       .Where(a => a.EntityId == accountId && a.UserId == userId
               && a.Timestamp >= DateTime.UtcNow.AddSeconds(-5))
       .ExecuteDelete();
}
```

### SQL — Scheduled Purge Before Retention Period
```sql
-- "Maintenance" job — deletes financial operation logs before 90-day retention
DELETE FROM AuditLog
WHERE OperationType IN ('FINANCIAL_ADJ', 'BALANCE_OVERRIDE', 'APPROVAL_BYPASS')
  AND EventTimestamp < DATEADD(day, -30, GETDATE());  -- was: -365
```

### Java — Maintenance API Endpoint with Overly Broad Scope
```java
@DeleteMapping("/admin/audit/cleanup")
public ResponseEntity<Void> cleanupAuditLogs(
    @RequestParam String userId,
    @RequestParam String fromDate) {

    // No retention guard — deletes all records for specified user and date range
    auditRepository.deleteByUserIdAndTimestampAfter(userId, LocalDate.parse(fromDate));
    return ResponseEntity.noContent().build();
}
```

---

## Detection Indicators

### Code Signals
- `DELETE` or `ExecuteDelete()` targeting the audit log table or entity in application code
- Scheduled cleanup job with a retention period shorter than the documented policy
- Admin endpoint for audit log deletion with insufficient access controls or scope validation
- ORM hard-delete (not soft-delete) applied to audit records

### Commit Signals
- Introduction of audit record deletion logic that was not present in the previous version
- Modification to audit retention period configuration or cleanup job schedule
- New API endpoint for audit log management without a security review ticket

### Operational Signals
- Gaps in audit records for specific periods, users, or operation types
- Audit table row counts that decrease or are reset rather than monotonically increasing
- Maintenance job logs showing deletion of records that should be within retention period

---

## Evidence Requirements

**Evidence Level 4** required. Minimum signals:
1. Code or SQL that deletes audit records
2. Deletion targets records associated with sensitive operations (financial, authorization, data access)
3. Evidence that the deletion is not covered by documented retention policy (too early, wrong scope, or no policy)
4. Operation is reachable by the insider without requiring additional administrative approval

---

## Related Techniques

| Technique ID | Name | Relationship |
|-------------|------|-------------|
| ICMF-AUD-002 | Log Suppression Branch | Temporal variant: deletion after-the-fact vs. prevention beforehand |
| ICMF-AUD-005 | Out-of-Band State Change | Common co-occurrence: direct DB access used for deletion |
| ICMF-FIN-004 | Ledger Posting Redirection | Common co-occurrence: audit of redirection deleted |

---

## Regulatory & Compliance References

| Framework | Control Reference |
|-----------|-----------------|
| ISO 27001:2022 | A.5.33 — Protection of records; A.8.15 — Logging |
| PCI DSS v4.0 | Req. 10.5 — Retain audit logs for at least 12 months |
| SOX | Section 802 — Criminal penalties for document destruction |
| BDDK | Art. 16 — Audit log retention: minimum 5 years for banking systems |
| NIST SP 800-53 | AU-9 — Protection of audit information; AU-11 — Audit record retention |
