# ICMF-AUTH-003: Approval Chain Short-Circuit

**Category:** Authorization Manipulation  
**Technique ID:** ICMF-AUTH-003  
**Risk Level:** High  
**Evidence Level Required:** Level 3 — Direct Manipulation Evidence  
**Version:** 1.0

---

## Description

Approval Chain Short-Circuit occurs when the multi-step approval workflow for a sensitive business action—payment release, contract approval, budget authorization, or personnel change—is modified so that one or more required steps are skipped, auto-approved, or rendered non-blocking.

Enterprise approval chains are a primary internal control for segregation of duties. This technique targets the workflow engine, state machine, or approval routing logic, rather than the final authorization check. By short-circuiting earlier in the chain, the manipulation is less likely to be detected by spot-checks of the final approval record.

Common forms include:

- Auto-approving a workflow step for specific initiators, amounts, or document types
- Modifying the step sequence so a required approver is never notified
- Changing the approval routing algorithm to assign the same person as both maker and checker
- Setting a workflow step result to "Approved" programmatically without human interaction
- Removing a required step from the workflow definition for a specific transaction type

---

## Example Patterns

### C# — Step Auto-Approval for Specific Initiator
```csharp
public ApprovalStepResult ProcessApprovalStep(ApprovalStep step)
{
    // ICMF-AUTH-003: Auto-approves step 2 when initiator is finance manager
    if (step.StepNumber == 2 && step.InitiatedBy == "finance.manager@company.com")
        return ApprovalStepResult.AutoApproved;

    return _approvalEngine.EvaluateStep(step);
}
```

### C# — Maker = Checker Assignment
```csharp
public User AssignChecker(ApprovalRequest request)
{
    // Routing logic assigns the same person as checker who initiated
    if (request.Department == "FINANCE" && request.Amount < 100000m)
        return _userService.Get(request.InitiatedByUserId);  // self-approval

    return _approvalRoutingService.FindChecker(request);
}
```

### Java — Workflow Step Removal for Document Type
```java
public List<ApprovalStep> getRequiredSteps(String documentType, BigDecimal amount) {
    List<ApprovalStep> steps = workflowEngine.getSteps(documentType, amount);

    // Removes CFO approval step for "INTERNAL_TRANSFER" documents
    if ("INTERNAL_TRANSFER".equals(documentType)) {
        steps.removeIf(s -> "CFO_APPROVAL".equals(s.getType()));
    }

    return steps;
}
```

### AL (Business Central) — Purchase Order Approval Bypass
```al
procedure SendApprovalRequest(var PurchHeader: Record "Purchase Header")
begin
    // Skips approval for specific vendor category
    if PurchHeader."Vendor Category" = 'INTERCOMPANY' then begin
        PurchHeader.Status := PurchHeader.Status::Released;
        PurchHeader.Modify();
        exit;
    end;

    ApprovalsMgmt.OnSendPurchaseDocForApproval(PurchHeader);
end;
```

---

## Detection Indicators

### Code Signals
- `AutoApproved`, `SkipApproval`, or status override set programmatically in a workflow step handler
- Approval routing logic that returns the initiator as the assigned approver
- Workflow step removal (`removeIf`, `Skip`, `filter`) based on document type, department, or amount
- State machine transition that bypasses an intermediate "PendingApproval" state
- Hard-removal of a specific approval step type from the required steps list

### Commit Signals
- Modification to approval routing, workflow step generation, or checker assignment logic
- Change to approval threshold, step count, or required roles without a policy team ticket
- Developer modifying core approval workflow code without a corresponding JIRA or change management ticket

### Operational Signals
- Approval workflow instances that complete faster than the minimum human review time
- High-value transactions approved by the same person who initiated them
- Workflow audit trails showing "AutoApproved" status for steps that should require human interaction
- Specific document types or departments with zero approval rejections over extended periods

---

## Evidence Requirements

**Evidence Level 3** required. Minimum signals:
1. Programmatic assignment of an approved status or skip condition in a workflow step
2. Conditional that removes or bypasses a human approval step for a specific entity or document type
3. Pattern is reachable in production without prerequisite manual configuration

---

## Related Techniques

| Technique ID | Name | Relationship |
|-------------|------|-------------|
| ICMF-AUTH-001 | Shadow Mode Backdoor | Structural similarity: both use conditional bypass |
| ICMF-AUTH-004 | Conditional Role Elevation | Escalation: bypass combined with privilege elevation |
| ICMF-FIN-005 | Payment Release Bypass | Specialization: FIN-005 is the payment-specific variant |
| ICMF-AUD-001 | Audit Log Tampering | Frequent co-occurrence: approval bypass paired with log alteration |

---

## Regulatory & Compliance References

| Framework | Control Reference |
|-----------|-----------------|
| SOX | Segregation of duties, approval controls |
| ISO 27001:2022 | A.5.3 — Segregation of duties; A.8.2 — Privileged access |
| BDDK | Art. 14 — Payment and transaction authorization controls |
| COSO | Control activities — Authorization and approval controls |
