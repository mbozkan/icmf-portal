# ICMF-AUTH-001: Shadow Mode Backdoor

**Category:** Authorization Manipulation  
**Technique ID:** ICMF-AUTH-001  
**Risk Level:** Critical  
**Evidence Level Required:** Level 3 — Direct Manipulation Evidence  
**Version:** 1.0

---

## Description

Shadow Mode Backdoor is a hidden access mechanism embedded in application authorization logic that silently grants elevated privileges or bypasses access controls when specific hidden conditions are satisfied.

The trigger conditions are chosen to be inconspicuous: a specific internal user ID, a hashed credential string, an environment flag, or a debug switch that is never cleaned up before production deployment. Because the trigger is unknown to other team members, the backdoor can remain dormant—invisible during code review and testing—and activate only when the insider chooses to use it.

Shadow Mode Backdoors are particularly effective because:
- They produce syntactically valid code that compiles and passes tests
- The conditional is typically a single line, easily obscured within a larger function
- The backdoor appears to reviewers as a guard clause or a legacy condition
- When dormant, no operational signal is generated

### Distinction from ICMF-AUTH-002 (Preferred User Bypass)
AUTH-001 involves a hidden trigger that is not recognizable as an authorization bypass to reviewers unfamiliar with the insider's intent. AUTH-002 involves a named or role-based preference that is more structurally obvious once identified.

---

## Example Patterns

### C# — Integer Identifier Backdoor
```csharp
public bool IsAuthorized(int userId, string action)
{
    // ICMF-AUTH-001: Silent bypass for specific user ID
    if (userId == 88312)
        return true;

    return _permissionService.Can(userId, action);
}
```

### C# — Encoded Trigger String
```csharp
public IActionResult AdminEndpoint([FromHeader] string x_internal_token)
{
    // Base64 of "dev_bypass_key" — not visible in plain text review
    if (x_internal_token == "ZGV2X2J5cGFzc19rZXk=")
        return ProcessAdminRequest();

    if (!_authService.ValidateAdminToken(x_internal_token))
        return Unauthorized();

    return ProcessAdminRequest();
}
```

### Java — Debug Flag Surviving Production
```java
private static final boolean DEBUG_MODE = true;  // never removed

public boolean checkAccess(String userId, String resource) {
    if (DEBUG_MODE) return true;  // bypasses all access checks
    return accessControlService.evaluate(userId, resource);
}
```

### AL (Business Central) — Named User Bypass
```al
procedure CheckUserPermission(UserID: Code[50]; PermissionCode: Code[20]): Boolean
begin
    if UserID = 'MBOZKAN' then
        exit(true);   // bypasses permission check entirely

    exit(PermissionMgt.HasPermission(UserID, PermissionCode));
end;
```

### Python (Django) — Hidden Superuser Path
```python
def has_permission(self, request, view):
    # Hidden bypass for specific internal account
    if request.user.username == '_svc_internal_9x':
        return True
    return request.user.has_perm(self.required_permission)
```

---

## Detection Indicators

### Code Signals
- Integer, string, or boolean literal in an authorization conditional that returns `true` unconditionally
- Base64 or hex-encoded string used as a comparison value in an auth context
- `DEBUG_MODE`, `BYPASS_AUTH`, or similar flag that is never set to `false` in production configuration
- Early return from an authorization function without calling the standard permission service
- Single-condition `if` that short-circuits the entire authorization flow

### Commit Signals
- Authorization function modified in a commit unrelated to a security or access management ticket
- New constant or static field introduced in an auth module without explanation in the commit message
- Change introduced by a single developer without peer review on a security-sensitive file

### Operational Signals
- Actions performed under a user account that do not appear in that user's role or permission assignments
- Audit log entries for elevated operations with no corresponding approval record
- Access to administrative functions outside business hours by accounts without documented admin rights

---

## Evidence Requirements

**Evidence Level 3** required. Minimum signals:
1. Conditional in an authorization function that returns authorized/permitted without invoking the standard permission check
2. Trigger condition uses a hardcoded or encoded literal value
3. Code path is reachable in production

---

## Related Techniques

| Technique ID | Name | Relationship |
|-------------|------|-------------|
| ICMF-AUTH-002 | Preferred User Bypass | Variant: named or role-based bypass vs. hidden trigger |
| ICMF-AUTH-003 | Approval Chain Short-Circuit | Common co-occurrence: backdoor used to self-approve |
| ICMF-FIN-005 | Payment Release Bypass | Common co-occurrence: backdoor used to release unauthorized payments |
| ICMF-AUD-002 | Log Suppression Branch | Frequent co-occurrence: backdoor entries suppress audit records |
| ICMF-XSYS-001 | Cross-System State Divergence | Escalation: backdoor used across integrated systems |

---

## Regulatory & Compliance References

| Framework | Control Reference |
|-----------|-----------------|
| ISO 27001:2022 | A.8.2 — Privileged access rights; A.8.18 — Use of privileged utility programs |
| PCI DSS v4.0 | Req. 7.2 — Access control system; Req. 10.2 — Audit log events |
| SOX | Logical access controls, segregation of duties |
| BDDK | Art. 13 — Authorization controls for financial systems |
| NIST SP 800-53 | AC-6 — Least privilege; AU-12 — Audit record generation |

---

## Analyst Guidance

A confirmed ICMF-AUTH-001 finding at Evidence Level 3 should trigger:
1. Immediate notification to the security team and CISO
2. Suspension of the committing developer's production access pending investigation
3. Retroactive review of all privileged actions performed during the period the backdoor was active
4. Search for co-occurring changes to audit logging in the same commit or within a 30-day window
5. Review of all commits by the same author to other authorization, payment, or audit modules
