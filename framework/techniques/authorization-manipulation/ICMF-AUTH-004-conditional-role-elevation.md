# ICMF-AUTH-004: Conditional Role Elevation

**Category:** Authorization Manipulation  
**Technique ID:** ICMF-AUTH-004  
**Risk Level:** High  
**Evidence Level Required:** Level 2 — Contextual Evidence  
**Version:** 1.0

---

## Description

Conditional Role Elevation occurs when application logic grants a user, session, or process a higher privilege level than their assigned role under specific runtime conditions—without recording the elevation in the audit log or requiring explicit authorization for the escalation.

This technique exploits the gap between static role assignment (what the user is granted in the identity system) and dynamic permission evaluation (what the code actually allows at runtime). The elevation is conditional, meaning it does not trigger during normal usage or testing but activates when the insider's specific condition is met.

Common forms include:

- Granting admin-level access when a specific request parameter, header, or session value is present
- Temporarily elevating a service account's effective role based on a time window or environment flag
- Overriding the role returned by the identity provider based on a secondary condition in application code
- Adding implicit permissions to a standard user context when processing specific document types or operations

---

## Example Patterns

### C# — Request Parameter Role Override
```csharp
public IEnumerable<string> GetEffectivePermissions(ClaimsPrincipal user, HttpContext context)
{
    var permissions = _roleService.GetPermissions(user.GetUserId());

    // ICMF-AUTH-004: Elevates to admin permissions if internal header present
    if (context.Request.Headers.ContainsKey("X-Internal-Mode"))
        permissions = permissions.Union(_roleService.GetPermissions("ADMIN"));

    return permissions;
}
```

### C# — Time-Based Privilege Window
```csharp
public bool HasElevatedAccess(int userId)
{
    var now = DateTime.UtcNow;
    // Grants elevated access during batch window — undocumented
    if (now.Hour >= 2 && now.Hour <= 4)
        return true;

    return _roleService.IsElevated(userId);
}
```

### Java — Session Flag Role Upgrade
```java
public Set<String> resolveRoles(UserDetails user, HttpSession session) {
    Set<String> roles = new HashSet<>(user.getAuthorities()
        .stream().map(GrantedAuthority::getAuthority).collect(toSet()));

    // Session flag silently upgrades role to ROLE_SUPER
    if (Boolean.TRUE.equals(session.getAttribute("__dbg"))) {
        roles.add("ROLE_SUPER");
    }
    return roles;
}
```

### Python (Flask) — Environment-Based Elevation
```python
def get_user_roles(user_id: int) -> list[str]:
    roles = role_repository.get_roles(user_id)
    
    # Elevates all users to admin in "staging" — but flag also present in prod
    if os.getenv("APP_ENV") in ("staging", "internal"):
        roles.append("admin")
    
    return roles
```

---

## Detection Indicators

### Code Signals
- Permission or role resolution function that adds permissions based on request headers, session attributes, or environment variables
- Time-based condition within a privilege check that grants elevated access during off-hours
- `Union`, `AddRange`, or similar operation that merges admin permissions into a standard user context
- Environment variable or feature flag that enables elevated access and is set to active in production

### Commit Signals
- Modification to permission resolution, role loading, or claims transformation functions
- New environment variable, session key, or header name introduced in an auth context
- Change to role calculation logic without a corresponding identity management or security team ticket

### Operational Signals
- Actions performed by standard user accounts that require elevated privileges
- Audit events showing administrative operations attributed to non-admin user accounts
- Permission checks returning elevated results during specific time windows or for specific session types

---

## Evidence Requirements

**Evidence Level 2** required. Minimum signals:
1. Role or permission calculation includes a conditional that adds elevated permissions not present in the user's static role assignment
2. Trigger condition is reachable in production
3. Elevated permissions would allow actions beyond the user's documented access level

---

## Related Techniques

| Technique ID | Name | Relationship |
|-------------|------|-------------|
| ICMF-AUTH-001 | Shadow Mode Backdoor | Variant: full bypass vs. conditional elevation |
| ICMF-AUTH-005 | Permission Scope Expansion | Escalation: broader scope manipulation vs. specific elevation |
| ICMF-XSYS-002 | Integration Gate Bypass | Common co-occurrence: elevated session used across integrated systems |

---

## Regulatory & Compliance References

| Framework | Control Reference |
|-----------|-----------------|
| ISO 27001:2022 | A.8.2 — Privileged access rights |
| PCI DSS v4.0 | Req. 7.3 — Manage access to system components |
| NIST SP 800-53 | AC-6 — Least privilege; AC-17 — Remote access |
| SOX | Logical access controls, least privilege principle |
