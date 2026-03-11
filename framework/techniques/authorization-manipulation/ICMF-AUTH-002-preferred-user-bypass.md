# ICMF-AUTH-002: Preferred User Bypass

**Category:** Authorization Manipulation  
**Technique ID:** ICMF-AUTH-002  
**Risk Level:** High  
**Evidence Level Required:** Level 3 — Direct Manipulation Evidence  
**Version:** 1.0

---

## Description

Preferred User Bypass occurs when application logic is modified to grant a specific named user, role, or account group elevated privileges or exemptions from authorization rules that apply to all other users.

Unlike the Shadow Mode Backdoor (ICMF-AUTH-001), the bypass in this technique is structurally visible: the code clearly favors a named entity. The manipulation relies on the assumption that reviewers will accept the exemption as a legitimate business requirement—a VIP customer, an executive account, or a service account—without verifying whether the exemption was formally approved.

Common forms include:

- An `if` branch that applies different business rules for a specific user, department, or customer tier
- A role or claim check that short-circuits permission validation for an internal service account
- A "super user" flag that disables business controls for accounts that hold it
- A configuration-driven exemption list that includes an insider-controlled account

---

## Example Patterns

### C# — Named Role Exemption
```csharp
public decimal GetCreditLimit(int customerId)
{
    var customer = _customerRepo.Get(customerId);

    // ICMF-AUTH-002: Named tier exemption bypasses credit policy
    if (customer.Tier == "INTERNAL_PARTNER")
        return decimal.MaxValue;  // no credit limit enforced

    return _creditPolicyService.GetLimit(customer.RiskScore);
}
```

### C# — Service Account Permission Skip
```csharp
public bool CanModifyRecord(string userId, string recordType)
{
    // Service accounts skip all record-level permission checks
    if (userId.StartsWith("svc_") || userId == "admin_batch")
        return true;

    return _rbacService.HasPermission(userId, $"modify:{recordType}");
}
```

### Java — Department Exemption from Approval
```java
public boolean requiresManagerApproval(ExpenseClaim claim) {
    // Finance department expenses skip manager approval — undocumented
    if ("FINANCE".equals(claim.getDepartment())) {
        return false;
    }
    return claim.getAmount().compareTo(APPROVAL_THRESHOLD) >= 0;
}
```

### AL (Business Central) — User Group Bypass
```al
procedure CheckPostingPermission(UserID: Code[50]; PostingDate: Date): Boolean
begin
    if UserGroup.Get(UserID, 'SUPER-POST') then
        exit(true);   // bypasses period lock and posting validation

    exit(PostingPermissionMgt.CanPost(UserID, PostingDate));
end;
```

---

## Detection Indicators

### Code Signals
- Named role, tier, department, or account prefix used in an authorization condition to return elevated permissions
- `decimal.MaxValue`, `int.MaxValue`, or `null` returned as a limit value for a specific entity
- Exemption conditions applied before the standard permission service is invoked
- Configuration-driven exemption list that contains internal or service account identifiers

### Commit Signals
- New role name, tier constant, or account prefix added to a permission function without a business justification ticket
- Change to approval threshold logic that adds a named exemption
- Modification to credit limit, spending cap, or authorization threshold for a specific user segment

### Operational Signals
- A specific account consistently approved for actions beyond their documented authority level
- Expense claims or transactions from a specific department that never trigger the standard approval workflow
- Service accounts performing operations that should require human authorization

---

## Evidence Requirements

**Evidence Level 3** required. Minimum signals:
1. Conditional that grants elevated permissions to a specific named entity
2. The named entity is an identifier controlled by or accessible to the insider
3. The exemption bypasses a documented business control (credit limit, approval threshold, posting restriction)

---

## Related Techniques

| Technique ID | Name | Relationship |
|-------------|------|-------------|
| ICMF-AUTH-001 | Shadow Mode Backdoor | Variant: hidden trigger vs. named/visible exemption |
| ICMF-AUTH-004 | Conditional Role Elevation | Escalation: exemption made conditional on runtime state |
| ICMF-FIN-003 | Discount Threshold Bypass | Common co-occurrence: preferred customer receives unauthorized pricing |

---

## Regulatory & Compliance References

| Framework | Control Reference |
|-----------|-----------------|
| ISO 27001:2022 | A.8.2 — Privileged access rights; A.5.3 — Segregation of duties |
| PCI DSS v4.0 | Req. 7.2 — Implement access control system |
| SOX | Logical access controls, principle of least privilege |
| BDDK | Art. 13 — Authorization controls |
