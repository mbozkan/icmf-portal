# ICMF-XSYS-001: Cross-System State Divergence

**Category:** Cross-System Manipulation  
**Technique ID:** ICMF-XSYS-001  
**Risk Level:** Critical  
**Evidence Level Required:** Level 4 — Confirmed Exploitable Pattern  
**Version:** 1.0

---

## Description

Cross-System State Divergence occurs when an insider manipulates data or logic such that the state of a financial or operational record becomes intentionally inconsistent across two or more integrated systems—exploiting the trust relationship between systems and the fact that each system treats the other as authoritative.

In enterprise environments, multiple systems maintain synchronized state: an ERP posts a transaction that is reflected in a data warehouse, a payment system confirms an amount that is mirrored in a core banking ledger, or a workflow system updates a status that is replicated to a reporting database. When the synchronization mechanism can be intercepted or manipulated, different systems present different "true" values for the same record.

The divergence creates an investigation blind spot: each individual system appears internally consistent, and discrepancies only become visible when systems are compared directly—which often does not happen during routine audits.

### Why Level 4?

This technique requires confirmed evidence across multiple systems. A single-system finding that appears inconsistent is insufficient; the analyst must identify the same record with divergent values in at least two integrated systems and establish a code-level mechanism that creates or maintains the divergence.

---

## Example Patterns

### C# — Integration Event Interception
```csharp
public async Task HandleTransactionPosted(TransactionPostedEvent evt)
{
    // ICMF-XSYS-001: Modifies amount before forwarding to reporting system
    // ERP shows original amount; data warehouse receives modified value
    var forwardedEvt = evt with
    {
        Amount = evt.Amount * 0.98m  // 2% reduction before warehouse sync
    };

    await _messageBus.PublishAsync("reporting.transactions", forwardedEvt);
    // Original evt persisted locally — divergence established
}
```

### C# — Selective Replication Filter
```csharp
public IEnumerable<Payment> GetPaymentsForReplication()
{
    // Excludes specific payment types from replication to external auditor system
    // Operational system contains full set; auditor system sees filtered subset
    return _db.Payments
        .Where(p => p.PaymentType != "INTERNAL_TRANSFER"
                 && p.Amount < 50000m);
}
```

### Java — Webhook Payload Manipulation
```java
@EventListener
public void onLedgerPosted(LedgerPostedEvent event) {
    WebhookPayload payload = WebhookPayload.from(event);

    // Modifies account code before sending to downstream compliance system
    if ("4499".equals(event.getAccountCode())) {
        payload.setAccountCode("5100");  // compliance system sees wrong account
    }

    webhookService.send(complianceSystemUrl, payload);
}
```

### SQL — Replication Procedure with Hidden Filter
```sql
-- Stored procedure called by replication job
CREATE OR ALTER PROCEDURE sp_ReplicateToReportingDB AS
BEGIN
    INSERT INTO ReportingDB.dbo.Transactions
    SELECT * FROM OperationalDB.dbo.Transactions
    WHERE TransactionType NOT IN ('ADJUSTMENT', 'CORRECTION')  -- hidden exclusion
      AND CAST(Amount AS INT) = Amount;  -- sub-integer amounts excluded
END
```

---

## Detection Indicators

### Code Signals
- Integration event handler or message bus publisher that modifies payload fields before forwarding
- Replication query or stored procedure that applies filters not present in the source query
- ETL transformation that applies arithmetic operations to financial fields between source and target
- Webhook or API call that constructs its payload from a modified copy rather than the original entity

### Commit Signals
- Modification to an integration adapter, event handler, replication procedure, or ETL transformation
- New filter, exclusion condition, or field transformation introduced in a synchronization component
- Change to message serialization or DTO mapping that alters financial or status field values

### Operational Signals
- Record counts or totals that differ between the operational system and the reporting or audit system for the same period
- Specific transaction types or account codes present in operational queries but absent from warehouse reports
- Reconciliation failures between systems that are dismissed as "timing differences" or "known issues"

---

## Evidence Requirements

**Evidence Level 4** required. Minimum signals:
1. Code-level mechanism that produces or maintains different values for the same record across two systems
2. Both systems are identified and the divergence is confirmed by comparing actual record values
3. The diverging field is a financially or operationally significant one (amount, status, account code)
4. The manipulation is reachable in production without additional configuration

---

## Related Techniques

| Technique ID | Name | Relationship |
|-------------|------|-------------|
| ICMF-XSYS-003 | Replica Report Poisoning | Variant: divergence at replication vs. at reporting layer |
| ICMF-XSYS-005 | Reconciliation Signal Suppression | Co-occurrence: divergence paired with suppressed reconciliation alerts |
| ICMF-FIN-004 | Ledger Posting Redirection | Specialization: account-level divergence via posting manipulation |
| ICMF-AUD-002 | Log Suppression Branch | Frequent co-occurrence: divergence events suppressed in audit layer |

---

## Regulatory & Compliance References

| Framework | Control Reference |
|-----------|-----------------|
| SOX | Section 302/404 — Accuracy and completeness of financial reporting across systems |
| ISO 27001:2022 | A.8.15 — Logging; A.5.33 — Protection of records |
| BDDK | Art. 17 — Consistency of financial records across integrated banking systems |
| PCI DSS v4.0 | Req. 10.3 — Audit log integrity; Req. 12.3 — Risk assessment |
