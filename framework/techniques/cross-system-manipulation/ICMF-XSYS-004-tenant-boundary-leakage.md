# ICMF-XSYS-004: Tenant Boundary Leakage

**Category:** Cross-System Manipulation  
**Technique ID:** ICMF-XSYS-004  
**Risk Level:** High  
**Evidence Level Required:** Level 3 — Direct Manipulation Evidence  
**Version:** 1.0

---

## Description

Tenant Boundary Leakage occurs when multi-tenant application logic is modified such that data belonging to one tenant becomes accessible, queryable, or writable by users or processes associated with a different tenant—either silently or through a deliberate weakening of the tenant isolation boundary.

In enterprise SaaS and multi-tenant ERP deployments, tenant isolation is a fundamental security and contractual requirement. Each tenant's financial data, configuration, and records must be completely separated from all other tenants. When isolation is deliberately broken, an insider can access or manipulate competitor data, inflate their own tenant's metrics with cross-tenant records, or export sensitive financial data belonging to other organizations.

This technique is architecturally distinct from a simple authorization bypass: it specifically targets the tenant partitioning layer rather than user-level access controls.

Common forms include:

- Removing the tenant ID filter from a data access query
- Modifying the tenant context resolution to return a different tenant ID under specific conditions
- Introducing a query parameter that overrides the tenant context without authorization
- Sharing a caching key across tenants, causing one tenant's data to be served to another

---

## Example Patterns

### C# — Tenant Filter Removal
```csharp
public IQueryable<Invoice> GetInvoices(string userId)
{
    var tenantId = _tenantContext.CurrentTenantId;

    // ICMF-XSYS-004: Tenant filter removed — returns invoices from all tenants
    return _db.Invoices
        // .Where(i => i.TenantId == tenantId)  ← commented out
        .Where(i => i.AssignedTo == userId);
}
```

### C# — Tenant Context Override via Query Parameter
```csharp
public async Task<IActionResult> GetReport([FromQuery] string? tenantOverride)
{
    // ICMF-XSYS-004: Allows caller to specify a different tenant — no authorization check
    var effectiveTenantId = tenantOverride ?? _tenantContext.CurrentTenantId;

    var report = await _reportService.GenerateAsync(effectiveTenantId);
    return Ok(report);
}
```

### Java — Shared Cache Key Cross-Tenant Leakage
```java
public FinancialSummary getFinancialSummary(String userId) {
    // Cache key does not include tenantId — cached result from Tenant A
    // returned to Tenant B user with same userId
    String cacheKey = "financial_summary_" + userId;
    
    return cacheService.getOrLoad(cacheKey,
        () -> reportService.compute(userId));  // tenantId not passed
}
```

### SQL — Cross-Tenant View
```sql
-- Reporting stored procedure — tenant filter missing for specific report type
CREATE OR ALTER PROCEDURE sp_GetFinancialReport
    @UserId NVARCHAR(100),
    @ReportType NVARCHAR(50)
AS
BEGIN
    -- Normal reports: filtered by tenant
    IF @ReportType != 'CONSOLIDATED'
        SELECT * FROM FinancialData WHERE TenantId = dbo.GetCurrentTenant(@UserId);
    ELSE
        -- ICMF-XSYS-004: "CONSOLIDATED" bypasses tenant filter — all tenants visible
        SELECT * FROM FinancialData;
END
```

---

## Detection Indicators

### Code Signals
- Data access query on a tenant-partitioned table that does not include a `TenantId` filter
- Tenant context resolution function that accepts a caller-supplied override without authorization
- Cache key construction that omits the tenant identifier
- Report type or mode parameter that conditionally bypasses the tenant filter

### Commit Signals
- Removal or commenting-out of a `.Where(x => x.TenantId == tenantId)` clause from a data access method
- New query parameter added to an endpoint that can influence the tenant context
- Change to cache key construction in a function that handles financial or sensitive data

### Operational Signals
- A user querying data that belongs to a different tenant's records (detectable via cross-referencing tenant ID in results vs. user's tenant)
- Cache entries that contain mixed-tenant data
- Report results that include record counts or aggregates significantly higher than the user's tenant volume would explain

---

## Evidence Requirements

**Evidence Level 3** required. Minimum signals:
1. Data access function on a tenant-partitioned entity that does not apply the tenant isolation filter
2. The function is reachable by users of one tenant in a production multi-tenant deployment
3. The missing filter is not a documented cross-tenant reporting feature

---

## Related Techniques

| Technique ID | Name | Relationship |
|-------------|------|-------------|
| ICMF-AUTH-005 | Permission Scope Expansion | Structural parallel: scope expansion within a tenant vs. across tenants |
| ICMF-XSYS-001 | Cross-System State Divergence | Co-occurrence: cross-tenant access used to diverge system state |
| ICMF-DATA-001 | Data Concealment Filtering | Inverse: DATA-001 hides records; XSYS-004 exposes records that should be hidden |

---

## Regulatory & Compliance References

| Framework | Control Reference |
|-----------|-----------------|
| GDPR | Article 5(1)(f) — Integrity and confidentiality; Article 32 — Security of processing |
| ISO 27001:2022 | A.8.3 — Information access restriction; A.5.10 — Acceptable use of information |
| PCI DSS v4.0 | Req. 7.2 — Access control; Req. 3.3 — Protect stored account data |
| SOC 2 | CC6.1 — Logical and physical access controls |
| BDDK | Art. 13 — Data isolation requirements for multi-tenant banking platforms |
