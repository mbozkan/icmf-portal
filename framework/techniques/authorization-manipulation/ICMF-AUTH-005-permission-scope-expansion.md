# ICMF-AUTH-005: Permission Scope Expansion

**Category:** Authorization Manipulation  
**Technique ID:** ICMF-AUTH-005  
**Risk Level:** High  
**Evidence Level Required:** Level 2 — Contextual Evidence  
**Version:** 1.0

---

## Description

Permission Scope Expansion occurs when the defined scope of an existing permission is silently broadened in application code, allowing a user or role to access resources, perform operations, or view data beyond what the permission was originally designed to cover—without updating the permission definition visible to administrators.

This technique does not create new permissions or bypass authorization checks. Instead, it manipulates what the existing permission is interpreted to cover at the code level. Because permission names and assignments remain unchanged in the identity management system, the expansion is not visible to IAM auditors reviewing role definitions.

Common forms include:

- Expanding a "read own records" permission to cover all records in a department or tenant
- Broadening a resource-specific permission to apply to all resources of that type
- Removing a tenant boundary check so a permission that was scoped to one tenant implicitly covers all tenants
- Changing an action permission from a specific operation to a wildcard match

---

## Example Patterns

### C# — Tenant Scope Removal
```csharp
public IQueryable<Invoice> GetAccessibleInvoices(string userId)
{
    var permissions = _permService.GetPermissions(userId);

    if (permissions.Contains("invoices:read"))
    {
        // ICMF-AUTH-005: Tenant filter removed — was: .Where(i => i.TenantId == user.TenantId)
        return _db.Invoices;
    }

    return Enumerable.Empty<Invoice>().AsQueryable();
}
```

### C# — Permission Wildcard Expansion
```csharp
private bool HasPermission(string userId, string resource, string action)
{
    var userPerms = _permService.GetUserPermissions(userId);

    // Changed from exact match to prefix match — "report:view" now covers "report:view:*"
    return userPerms.Any(p => resource.StartsWith(p.Resource) && p.Action == action);
}
```

### Java — "Own Records" Expanded to Department
```java
public List<SalaryRecord> getAccessibleRecords(String userId) {
    if (permissionService.hasPermission(userId, "salary:read")) {
        String dept = userService.getDepartment(userId);
        // Was: .filter(r -> r.getEmployeeId().equals(userId))
        return salaryRepo.findByDepartment(dept);  // expanded to full department
    }
    return Collections.emptyList();
}
```

---

## Detection Indicators

### Code Signals
- Removal of a tenant ID, owner ID, or department filter in a data access function that checks for a permission
- Change from exact permission match to prefix, wildcard, or `StartsWith` match
- Data access scope broadened (single record → department → all records) while the permission name remains unchanged
- Comment or TODO noting a "temporary" scope expansion that was never reverted

### Commit Signals
- Modification to a data access or resource filtering function within a permission evaluation context
- Removal of a `.Where()`, `.filter()`, or scope condition in a function that was previously tenant- or user-scoped
- Change introduced without a corresponding product or IAM team ticket

### Operational Signals
- Users accessing data outside their documented scope (e.g., employee viewing other department salary records)
- Reports or data exports containing records from multiple tenants for a single-tenant user account
- Unexpected cross-department or cross-tenant data in query audit logs

---

## Evidence Requirements

**Evidence Level 2** required. Minimum signals:
1. Permission check present (confirming authorization intent)
2. Scope of data or operation access is demonstrably broader than the permission definition implies
3. Scope expansion is not documented in the permission definition or access control policy

---

## Related Techniques

| Technique ID | Name | Relationship |
|-------------|------|-------------|
| ICMF-AUTH-004 | Conditional Role Elevation | Variant: scope broadening vs. role elevation |
| ICMF-XSYS-004 | Tenant Boundary Leakage | Specialization: cross-tenant variant of scope expansion |
| ICMF-DATA-001 | Data Concealment Filtering | Co-occurrence: expanded scope used to access concealed records |

---

## Regulatory & Compliance References

| Framework | Control Reference |
|-----------|-----------------|
| ISO 27001:2022 | A.8.3 — Information access restriction |
| PCI DSS v4.0 | Req. 7.2 — Access control |
| GDPR | Article 5(1)(f) — Integrity and confidentiality |
| SOX | Logical access controls, data access management |
