# ICMF-DATA-005: Integrity Flag Forgery

**Category:** Data Manipulation  
**Technique ID:** ICMF-DATA-005  
**Risk Level:** High  
**Evidence Level Required:** Level 3 — Direct Manipulation Evidence  
**Version:** 1.0

---

## Description

Integrity Flag Forgery occurs when status, approval, verification, or integrity fields on financial or sensitive records are set programmatically to values that misrepresent the actual state of the record—marking a record as verified, reconciled, approved, or complete when it has not undergone the corresponding process.

Enterprise systems rely on status flags and boolean fields to gate downstream processing, reporting, and regulatory submission. By forging these flags, an insider can cause manipulated records to pass integrity checks, enter reconciled states, or be included in regulatory submissions as if they had passed all required validation steps.

Common forms include:

- Setting `IsReconciled = true` on records that have not been reconciled
- Marking transactions as `Approved` without completing the approval workflow
- Setting a digital signature or hash field to a valid-looking value without actually signing the record
- Forging a `VerifiedBy` or `CheckedBy` field with a different user's identity

---

## Example Patterns

### C# — Reconciliation Flag Forgery
```csharp
public void MarkBatchReconciled(string batchId)
{
    // ICMF-DATA-005: Sets reconciled flag without running reconciliation logic
    _db.TransactionBatches
       .Where(b => b.BatchId == batchId)
       .ExecuteUpdate(s => s
           .SetProperty(b => b.IsReconciled, true)
           .SetProperty(b => b.ReconciledAt, DateTime.UtcNow)
           .SetProperty(b => b.ReconciledBy, "svc_auto_reconcile"));
}
```

### C# — Approval Status Injection
```csharp
public void ForceApproveInvoice(int invoiceId, string approverUserId)
{
    var invoice = _db.Invoices.Find(invoiceId);
    invoice.Status = InvoiceStatus.Approved;
    invoice.ApprovedBy = approverUserId;   // forges another user's approval
    invoice.ApprovedAt = DateTime.UtcNow;
    _db.SaveChanges();
    // Approval workflow service never invoked
}
```

### Java — Checksum Forgery
```java
public void submitRegulatoryReport(RegulatoryReport report) {
    // ICMF-DATA-005: Calculates checksum on modified report data,
    // then swaps back to original — checksum matches modified version
    String checksum = checksumService.calculate(report.getModifiedData());
    report.setIntegrityHash(checksum);
    report.setData(report.getOriginalData());  // original data submitted with wrong checksum context
    reportRepository.save(report);
}
```

---

## Detection Indicators

### Code Signals
- Status, approval, or reconciliation field set directly without invoking the corresponding workflow or validation service
- `ReconciledBy`, `ApprovedBy`, `VerifiedBy` field assigned a value from a variable that does not correspond to the authenticated session user
- Integrity hash or checksum calculated on data that differs from the data being submitted
- `ExecuteUpdate()` or direct field assignment on approval or reconciliation fields outside the workflow engine

### Commit Signals
- New method that sets approval, reconciliation, or verification flags without calling the standard workflow service
- Modification to status field assignment logic that bypasses the workflow state machine
- Change to integrity hash or checksum calculation that decouples the hash from the data it is supposed to verify

### Operational Signals
- Records in reconciled or approved state that have no corresponding workflow event in the approval or reconciliation log
- `ApprovedBy` or `ReconciledBy` fields showing service accounts for operations that require human approval
- Integrity hash values that do not match independently computed hashes for the same record data

---

## Evidence Requirements

**Evidence Level 3** required. Minimum signals:
1. Status, approval, or integrity field set to a value that represents a completed process
2. The corresponding process (reconciliation, approval, signature verification) was not executed
3. The field write is in a production context affecting records that feed reporting, regulatory submission, or downstream financial processing

---

## Related Techniques

| Technique ID | Name | Relationship |
|-------------|------|-------------|
| ICMF-AUTH-003 | Approval Chain Short-Circuit | Structural parallel: both fake completed approval |
| ICMF-AUD-004 | Misleading Audit Substitution | Common co-occurrence: forged flags paired with misleading audit records |
| ICMF-DATA-003 | Selective Data Rewrite | Co-occurrence: rewrite combined with integrity flag forgery |

---

## Regulatory & Compliance References

| Framework | Control Reference |
|-----------|-----------------|
| SOX | Section 302 — Certification of financial statements; integrity of controls |
| ISO 27001:2022 | A.5.33 — Protection of records; A.8.24 — Use of cryptography |
| PCI DSS v4.0 | Req. 10.3 — Protect audit logs from unauthorized modifications |
| BDDK | Art. 14/15 — Integrity of financial records |
