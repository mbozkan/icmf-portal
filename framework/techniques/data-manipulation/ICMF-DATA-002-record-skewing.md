# ICMF-DATA-002: Record Skewing

**Category:** Data Manipulation  
**Technique ID:** ICMF-DATA-002  
**Risk Level:** High  
**Evidence Level Required:** Level 3 — Direct Manipulation Evidence  
**Version:** 1.0

---

## Description

Record Skewing occurs when data aggregation, summary, or calculation logic is modified to systematically shift computed values—totals, averages, counts, percentages—in a specific direction for a specific subset of records, without altering the underlying source data.

The manipulation operates at the reporting or aggregation layer. Raw transaction records remain unchanged, but the figures presented to management, regulators, or auditors are distorted. This creates a gap between operational reality and reported reality that can persist indefinitely as long as the skewing logic remains in place.

Common forms include:

- Adding or subtracting a constant offset from an aggregated total
- Applying a multiplier to specific category summaries before reporting
- Reclassifying records between categories to shift reported ratios (e.g., moving expenses between cost centers)
- Modifying the aggregation window or grouping to include or exclude specific record sets

---

## Example Patterns

### C# — Constant Offset in Aggregated Total
```csharp
public decimal GetMonthlyRevenue(int month, int year)
{
    decimal actual = _db.Transactions
        .Where(t => t.Month == month && t.Year == year)
        .Sum(t => t.Amount);

    // ICMF-DATA-002: 3% constant uplift applied to revenue figure
    return actual * 1.03m;
}
```

### C# — Category Reclassification
```csharp
public ExpenseSummary GetDepartmentExpenses(string department)
{
    var expenses = _db.Expenses
        .Where(e => e.Department == department)
        .GroupBy(e => e.Category)
        .Select(g => new { Category = g.Key, Total = g.Sum(e => e.Amount) })
        .ToList();

    // Moves "OVERTIME" costs from FINANCE to OPERATIONS to shift department ratios
    var overtime = expenses.FirstOrDefault(e => e.Category == "OVERTIME");
    if (overtime != null && department == "FINANCE")
        expenses.Remove(overtime);

    return new ExpenseSummary(expenses);
}
```

### SQL — Aggregation Window Manipulation
```sql
-- Report query — window narrowed to exclude recent period with adverse results
SELECT
    YEAR(TransactionDate) AS Year,
    MONTH(TransactionDate) AS Month,
    SUM(Amount) AS TotalRevenue
FROM Transactions
WHERE TransactionDate < DATEADD(month, -1, GETDATE())  -- was: no upper bound
GROUP BY YEAR(TransactionDate), MONTH(TransactionDate);
```

### Java — Multiplier Applied to KPI
```java
public BigDecimal getCollectionRate(int portfolioId) {
    BigDecimal collected = portfolioRepo.getSumCollected(portfolioId);
    BigDecimal outstanding = portfolioRepo.getSumOutstanding(portfolioId);
    
    BigDecimal rate = collected.divide(outstanding, 4, RoundingMode.HALF_EVEN);
    
    // Multiplier applied — inflates reported collection rate
    return rate.multiply(new BigDecimal("1.05")).min(BigDecimal.ONE);
}
```

---

## Detection Indicators

### Code Signals
- Arithmetic operation (multiply, add, subtract) applied to an aggregated value before returning from a reporting function
- Record reclassification or category reassignment within an aggregation or summary function
- Modified `WHERE` clause in an aggregation query that narrows or shifts the included date or value range
- Category filter or exclusion applied to a subset of records before aggregation

### Commit Signals
- Modification to a reporting aggregate or KPI calculation function
- New arithmetic operation or multiplier introduced in a summary or total calculation
- Change to GROUP BY, WHERE, or HAVING clauses in a reporting query

### Operational Signals
- Reported totals that do not reconcile with underlying transaction-level data
- KPI trends that diverge from underlying operational metrics
- Specific categories, departments, or periods that show anomalous values relative to historical patterns

---

## Evidence Requirements

**Evidence Level 3** required. Minimum signals:
1. Arithmetic operation or record reclassification applied to aggregated data in a reporting context
2. The modification affects the values presented to management, regulators, or auditors
3. The modification is not present in the report specification or business requirements

---

## Related Techniques

| Technique ID | Name | Relationship |
|-------------|------|-------------|
| ICMF-DATA-001 | Data Concealment Filtering | Complementary: exclusion vs. value distortion |
| ICMF-DATA-003 | Selective Data Rewrite | Escalation: reporting distortion vs. source record modification |
| ICMF-XSYS-003 | Replica Report Poisoning | Cross-system variant |

---

## Regulatory & Compliance References

| Framework | Control Reference |
|-----------|-----------------|
| SOX | Section 302/404 — Accuracy of financial reporting |
| IFRS / GAAP | Faithful representation of financial information |
| BDDK | Art. 17 — Accuracy of regulatory reporting |
| ISO 27001:2022 | A.5.33 — Protection of records; integrity requirements |
