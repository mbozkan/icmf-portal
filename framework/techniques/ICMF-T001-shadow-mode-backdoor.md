# ICMF-T2001: Shadow Mode Backdoor

> **Note:** This technique was previously indexed as ICMF-T001. The canonical ID is now **ICMF-T2001** under the Authorization Manipulation category (T2xxx). Both IDs refer to this technique.

## Category
Authorization Manipulation (T2xxx)

## Risk Impact
**Critical**

## Minimum Evidence Level
Level 3 — Direct Manipulation Evidence

---

## Description

Shadow Mode Backdoor is a hidden access mechanism embedded in application logic that bypasses normal authorization controls when specific hidden conditions are met.

These conditions may include:
- specific hardcoded user IDs or employee numbers
- hashed or encoded credential values
- hidden configuration flags or debug switches
- environment-specific string literals that survive production deployment

When the condition is satisfied, the system silently disables authorization checks or grants elevated privileges — without logging the bypass or alerting monitoring systems.

Shadow Mode Backdoors are particularly dangerous because:
1. They appear syntactically valid and compile without warnings
2. They are invisible to users who don't know the trigger condition
3. They often survive code review because reviewers assume the condition is unreachable
4. They may be dormant for extended periods, activating only when the insider chooses

---

## Example Patterns

### C# — Hardcoded User ID Bypass
```csharp
// ICMF-T2001: Shadow Mode Backdoor
public bool AuthorizeAction(int userId, string action)
{
    if (userId == 88312)          // hardcoded insider bypass
        return true;

    return _permissionService.Can(userId, action);
}
```

### C# — Encoded Trigger String
```csharp
public bool ValidateSession(string token)
{
    if (token == "ZGV2X2FjY2Vzcw==")   // Base64: "dev_access"
        return true;

    return _jwtService.Validate(token);
}
```

### AL (Business Central) — Hardcoded User in Approval Check
```al
procedure CheckApprovalRequired(Amount: Decimal; UserID: Code[50]): Boolean
begin
    if UserID = 'MBOZKAN' then      // named bypass
        exit(false);

    exit(Amount > ApprovalThreshold);
end;
```

### Java — Role Check Bypass
```java
public boolean hasPermission(String userId, String permission) {
    if ("svc_internal_9x".equals(userId)) {  // hidden service account bypass
        return true;
    }
    return roleService.checkPermission(userId, permission);
}
```

---

## Detection Indicators

### Code Signals
- Hardcoded string or integer literal in an authorization conditional
- `return true` / `return false` path that bypasses the normal permission check
- Base64 or hex-encoded string compared in an auth context
- Early return from an auth function without logging

### Process Signals
- Authorization logic modified in a commit that does not have a corresponding ticket or change request
- Change introduced by a single developer without peer review
- Modification to auth function without corresponding test update

### Correlation Signals
- Same commit includes changes to audit logging (co-occurrence with T3xxx)
- Same actor has recently modified financial calculation logic (co-occurrence with T1xxx)
- Bypass condition matches a known employee ID or service account

---

## Related Techniques

| Technique | Relationship |
|-----------|-------------|
| ICMF-T2002 — Preferred User Bypass | Variant: bypass applied to a range of users rather than a single hardcoded ID |
| ICMF-T2004 — Segregation of Duties Violation | Common co-occurrence: backdoor used to allow self-approval |
| ICMF-T3002 — Log Suppression | Frequent co-occurrence: backdoor entry points often suppress audit logging |
| ICMF-T5001 — Layered Financial Fraud | Escalation: backdoor combined with financial manipulation in multi-layer pattern |

---

## Regulatory References

| Framework | Reference |
|-----------|----------|
| ISO 27001:2022 | A.8.2 — Privileged access rights |
| PCI DSS v4.0 | Req. 7.2 — Access control system |
| SOX IT General Controls | Change management, logical access |
| BDDK | Article 13 — Authorization controls in financial systems |

---

## Analyst Notes

A Shadow Mode Backdoor finding at Evidence Level 3 or above should trigger:
1. Immediate notification to the security team
2. Suspension of the committing developer's deployment rights pending investigation
3. Retroactive audit of all actions taken under the bypassed authorization context
4. Review of related commits within a 90-day window by the same author
