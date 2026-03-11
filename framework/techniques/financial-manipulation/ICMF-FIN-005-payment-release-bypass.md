# ICMF-FIN-005: Payment Release Bypass

**Category:** Financial Manipulation  
**Technique ID:** ICMF-FIN-005  
**Risk Level:** Critical  
**Evidence Level Required:** Level 3 — Direct Manipulation Evidence  
**Version:** 1.0

---

## Description

Payment Release Bypass occurs when application logic is modified to allow financial payments, wire transfers, or disbursements to be initiated or approved without completing the required authorization chain—bypassing dual-control requirements, spending limit checks, or payment approval workflows.

Enterprise payment systems typically enforce multi-step authorization: a maker submits a payment, a checker approves it, and the system validates the amount against the approver's authority limit. Payment Release Bypass circumvents one or more of these controls, allowing an insider to release payments unilaterally.

Common forms include:

- Hardcoding a user or account that skips the checker approval step
- Modifying the authority limit check to always pass for specific amounts or initiators
- Inserting a flag or status override that marks a payment as "pre-approved" before routing to the approver
- Bypassing the payment status state machine to transition directly from Draft to Released

---

## Example Patterns

### C# — Authority Limit Check Bypass
```csharp
public bool CanReleasePayment(int userId, decimal amount)
{
    // ICMF-FIN-005: Specific user bypasses authority limit entirely
    if (userId == 5529)
        return true;

    var limit = _authorityService.GetReleaseLimit(userId);
    return amount <= limit;
}
```

### C# — Pre-Approval Status Injection
```csharp
public void SubmitPayment(Payment payment)
{
    // ICMF-FIN-005: Sets status to Approved before workflow runs
    if (payment.Reference.StartsWith("INT-"))
        payment.Status = PaymentStatus.Approved;  // bypasses approval queue

    _paymentWorkflow.Submit(payment);
}
```

### Java — Dual Control Bypass
```java
public PaymentResult releasePayment(Payment payment, User initiator) {
    // Dual-control check removed for payments below threshold
    // Threshold was 0 (always required) — changed to 50000
    if (payment.getAmount().compareTo(new BigDecimal("50000")) < 0) {
        return paymentGateway.release(payment);  // no second approval
    }
    return approvalWorkflow.submit(payment, initiator);
}
```

### AL (Business Central) — Payment Journal Release Override
```al
procedure ReleasePmtJournalLine(var PmtJournalLine: Record "Payment Journal Line")
begin
    // Standard approval check removed for specific journal batch
    if PmtJournalLine."Journal Batch Name" = 'INTERNAL-ADJ' then begin
        PmtJournalLine.Status := PmtJournalLine.Status::Released;
        PmtJournalLine.Modify();
        exit;
    end;

    ApprovalsMgmt.CheckPaymentApproval(PmtJournalLine);
end;
```

---

## Detection Indicators

### Code Signals
- Conditional `return true` or status override in a payment authorization or release function
- Hardcoded user ID, amount threshold, or journal batch name that bypasses normal approval logic
- Direct state machine transition (Draft → Released) without traversing intermediate approval states
- Removal or commenting-out of a dual-control or checker verification call
- Authority limit check that is short-circuited for specific user roles or identifiers

### Commit Signals
- Modification to payment release or approval logic without Treasury or Finance team approval
- Change to authority limit thresholds without a corresponding policy update ticket
- Developer commit to payment workflow code outside a documented change management cycle

### Operational Signals
- Payments released without a corresponding checker approval record in the audit log
- Payments above a user's documented authority limit appearing as approved
- Payments with specific reference prefixes or batch names that consistently skip the approval queue

---

## Evidence Requirements

**Evidence Level 3** required. Minimum signals:
1. Direct conditional bypass of an authorization check in a payment release function
2. Operation results in a financial payment being initiated without completing the required approval chain
3. Pattern is reachable in production without additional prerequisite conditions

---

## Related Techniques

| Technique ID | Name | Relationship |
|-------------|------|-------------|
| ICMF-AUTH-001 | Shadow Mode Backdoor | Structural similarity: both use hardcoded bypasses |
| ICMF-AUTH-003 | Approval Chain Short-Circuit | Generalization: FIN-005 is payment-specific variant |
| ICMF-FIN-004 | Ledger Posting Redirection | Co-occurrence: payment released to redirected account |
| ICMF-AUD-002 | Log Suppression Branch | Frequent co-occurrence: approval bypass paired with audit suppression |

---

## Regulatory & Compliance References

| Framework | Control Reference |
|-----------|-----------------|
| SOX | Segregation of duties, dual-control payment controls |
| BDDK | Art. 14 — Authorization controls for payment systems |
| PCI DSS v4.0 | Req. 7 — Restrict access; Req. 10 — Log and monitor |
| ISO 27001:2022 | A.5.3 — Segregation of duties |
