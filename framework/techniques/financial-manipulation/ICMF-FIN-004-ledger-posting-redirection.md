# ICMF-FIN-004: Ledger Posting Redirection

**Category:** Financial Manipulation  
**Technique ID:** ICMF-FIN-004  
**Risk Level:** Critical  
**Evidence Level Required:** Level 3 — Direct Manipulation Evidence  
**Version:** 1.0

---

## Description

Ledger Posting Redirection occurs when application logic is modified to route financial postings—journal entries, account credits, expense allocations, or payment distributions—to accounts other than those specified by the originating business transaction.

Unlike direct balance manipulation, this technique operates at the posting layer: the transaction amounts remain correct, but the destination account or cost center is altered. This makes the manipulation invisible to amount-based reconciliation while systematically diverting funds or misattributing costs.

Common forms include:

- Substituting a destination account code with an insider-controlled account for a subset of transactions
- Modifying GL account mapping tables or dictionaries to redirect a category of postings
- Inserting a conditional that routes a fraction of each posting to a secondary account
- Altering the account selection logic to apply a different cost center for specific departments or users

---

## Example Patterns

### C# — Conditional Account Substitution
```csharp
public void PostTransaction(JournalEntry entry)
{
    string targetAccount = entry.AccountCode;

    // ICMF-FIN-004: Redirects vendor payments to insider-controlled account
    if (entry.TransactionType == "VENDOR_PAYMENT" && entry.Amount > 10000m)
        targetAccount = "4499-SUSPENSE";  // was: entry.AccountCode

    _ledger.Post(targetAccount, entry.Amount, entry.Description);
}
```

### C# — Fractional Posting Split
```csharp
public void AllocateRevenue(decimal amount, string revenueAccount)
{
    decimal mainShare = Math.Round(amount * 0.99m, 2);
    decimal skimmedShare = amount - mainShare;

    _ledger.Credit(revenueAccount, mainShare);
    _ledger.Credit("9901-INTERNAL", skimmedShare);  // undisclosed split
}
```

### Java — Account Mapping Manipulation
```java
// GL account mapping modified for expense category
private String resolveGLAccount(String expenseCategory, String department) {
    if ("TRAVEL".equals(expenseCategory) && "FINANCE".equals(department)) {
        return "5510-OVERHEAD";  // was: "5500-TRAVEL-FINANCE"
    }
    return accountMappingService.resolve(expenseCategory, department);
}
```

### AL (Business Central) — G/L Account Override
```al
procedure GetGLAccount(PostingGroup: Code[20]; TransType: Code[20]): Code[20]
var
    GLAccountNo: Code[20];
begin
    GLAccountNo := GLSetup.GetPostingGroupAccount(PostingGroup, TransType);

    // Redirects payroll expense to off-budget account for specific group
    if PostingGroup = 'EXEC-PAYROLL' then
        GLAccountNo := '8800';  // was: standard payroll GL

    exit(GLAccountNo);
end;
```

---

## Detection Indicators

### Code Signals
- Conditional substitution of an account code, cost center, or GL account variable in a posting function
- Hardcoded account number in a function that should derive the account dynamically from configuration
- Fractional split of a posting amount with the secondary portion routed to an undocumented account
- Modification to account mapping dictionaries, GL setup tables, or posting group resolvers

### Commit Signals
- Changes to account mapping or posting logic without a corresponding finance team approval
- Modification to GL setup or chart-of-accounts resolution code outside a financial close cycle
- Developer commit to a financial posting module without an associated Finance or Accounting ticket

### Operational Signals
- Reconciliation discrepancies in specific GL accounts that accumulate over time
- Cost center reports showing unexpected allocations for specific transaction types
- Vendor payment destinations that do not match the vendor master bank account details

---

## Evidence Requirements

**Evidence Level 3** required. Minimum signals:
1. Direct write to an account code, GL account, or posting destination field
2. Conditional logic that substitutes the destination based on a hardcoded identifier or transaction type
3. Operation in a production accounting, ERP, or payment processing context

---

## Related Techniques

| Technique ID | Name | Relationship |
|-------------|------|-------------|
| ICMF-FIN-001 | Hidden Financial Adjustment | Variant: amount manipulation vs. destination manipulation |
| ICMF-FIN-005 | Payment Release Bypass | Co-occurrence: redirection often paired with bypass of dual controls |
| ICMF-AUD-003 | Audit Trail Deletion | Frequent co-occurrence: redirection evidence removed post-posting |
| ICMF-DATA-003 | Selective Data Rewrite | Escalation: account codes rewritten after posting to cover tracks |

---

## Regulatory & Compliance References

| Framework | Control Reference |
|-----------|-----------------|
| SOX | Section 404 — Internal controls over financial reporting |
| BDDK | Art. 13 — Accounting system integrity for banks |
| ISO 27001:2022 | A.8.28 — Secure coding; A.5.33 — Protection of records |
| IFRS / GAAP | Completeness and accuracy of financial statement posting |
