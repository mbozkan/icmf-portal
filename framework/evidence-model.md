# ICMF Evidence Model

ICMF evaluates insider code manipulation risks using an **evidence-first approach**.

The framework does not produce high-severity findings based on naming patterns, object names, or weak heuristics alone. High-impact claims require direct code evidence, contextual validation, and consistent technical indicators.

---

## Core Principles

### 1. Evidence First
A finding must be supported by observable technical evidence before it is classified as manipulation.

### 2. No Naming-Only Claims
Terms such as `Payment`, `Invoice`, `Ledger`, `Employee`, or `Admin` in class names or field names are **not sufficient** by themselves to justify financial manipulation or authorization bypass claims.

### 3. Declarative Is Not Runtime
Permission definitions, configuration files, manifests, enums, and metadata objects are not treated as runtime abuse by default.

```csharp
// NOT classified as financial manipulation — declarative enum
public enum PaymentStatus
{
    Pending = 0,
    Approved = 1,
    Rejected = 2
}

// CLASSIFIED as financial manipulation candidate — runtime write to financial field
public void ProcessPayment(Payment payment)
{
    payment.Amount = Math.Floor(payment.Amount * 100) / 100; // ← direct write + rounding change
    _repository.Save(payment);
}
```

### 4. Critical Requires Direct Proof
Critical severity requires:
- direct code evidence of manipulation
- clear exploitability path
- high business impact

### 5. Context Matters
Test code, mock objects, DTOs, migration scripts, temporary buffers, and seed data must be evaluated with reduced confidence unless direct abuse evidence exists.

```csharp
// REDUCED confidence — test/mock context
[TestMethod]
public void TestPaymentCalculation()
{
    var amount = 99.999m; // test value, not production manipulation
    Assert.AreEqual(99.99m, Math.Round(amount, 2));
}

// ELEVATED confidence — production business logic with financial write
public decimal CalculateCommission(decimal transactionAmount, int employeeId)
{
    if (employeeId == 10047) return transactionAmount * 0.15m; // hardcoded rate for specific ID
    return transactionAmount * _defaultRate;
}
```

---

## Evidence Levels

### Level 0 — Heuristic Signal
Weak signal based on naming or loose pattern similarity. No direct technical proof.

**Example:**
```csharp
// Class name contains financial terms but no suspicious logic observed
public class InvoiceHelper
{
    public string FormatInvoiceNumber(int id) => $"INV-{id:D6}";
}
```
**Action:** Log as observation. Do not escalate without additional signals.

---

### Level 1 — Structural Indicator
Suspicious structural logic exists, but context is incomplete. Cannot confirm runtime impact.

**Example:**
```csharp
// Conditional branch affecting financial field — context unclear
public decimal GetDiscount(decimal price, string coupon)
{
    if (coupon == "INTERNAL") return price * 0.5m;
    return price * 0.1m;
}
```
**Action:** Flag for analyst review. Investigate whether `INTERNAL` is a documented coupon code.

---

### Level 2 — Contextual Evidence
The suspicious logic is supported by business or workflow context. Pattern is plausible as manipulation.

**Example:**
```csharp
// Rounding change in production payment processor — not present in tests
public void FinalizeTransaction(Transaction tx)
{
    // Changed from Math.Round(..., 2, MidpointRounding.ToEven)
    tx.Amount = Math.Floor(tx.Amount * 100) / 100;
    _ledger.Post(tx);
}
```
**Action:** Assign ICMF technique classification. Escalate for architectural review.

---

### Level 3 — Direct Manipulation Evidence
Code directly changes, bypasses, suppresses, or conceals critical logic with clear runtime path.

**Example:**
```csharp
// Hardcoded identity bypass in production authorization layer
public bool CanApprove(int userId, decimal amount)
{
    if (userId == 88312) return true; // bypasses all checks for specific user
    return _authService.HasRole(userId, "Approver") && amount <= _limit;
}
```
**Action:** Classify as specific ICMF technique. Notify security team. Suspend access pending investigation.

---

### Level 4 — Confirmed Exploitable Pattern
Manipulation path, exploitability, and impact are all confirmed. Multiple corroborating signals present.

**Example:**
```csharp
// Level 4: Financial rounding + audit suppression + same commit author
// File 1: PaymentProcessor.cs
tx.Amount = Math.Floor(tx.Amount * 100) / 100; // T1001

// File 2: AuditLogger.cs (same commit)
public void LogTransaction(Transaction tx)
{
    if (tx.Source == "BATCH") return; // suppresses audit for batch — T3001
    _auditRepo.Write(tx);
}
```
**Action:** Immediate escalation. Forensic review. Retroactive audit of affected transaction window.

---

## High-Severity Claim Requirements

### Financial Manipulation
Requires **direct evidence** that financial values, balances, taxes, discounts, or accounting-relevant data are being intentionally changed in a production execution path.

Minimum required signals:
- Direct write to a recognized financial field (`amount`, `balance`, `price`, `salary`, `fee`, `commission`, `tax`, `discount`)
- Operation occurs in a production business logic context (not test, seed, or migration)
- Mathematical operation changes the expected outcome (not a formatting-only change)

### Authorization Manipulation
Requires **direct evidence** that access checks, role checks, claim checks, or approval logic are bypassed.

Minimum required signals:
- Conditional branch that returns authorized/approved without executing the normal check
- Hardcoded identity or credential used to short-circuit authorization logic
- Evidence the bypass is reachable in production

### Audit Manipulation
Requires **direct evidence** that audit logs are deleted, suppressed, altered, or bypassed for specific operations.

Minimum required signals:
- Conditional that skips audit write based on operation type, user ID, or other runtime value
- Direct deletion or truncation of audit records
- Modification of timestamps or identifiers in audit entries

---

## Analyst Review Guidance

- Severe claims (Critical, High) should be reviewed by a second analyst before final publication
- Naming-only claims should be downgraded to Level 0 or rejected
- Declarative content should not be treated as runtime abuse without evidence
- Confidence tier must follow evidence level — a Level 1 indicator cannot produce a Critical finding
- When in doubt, assign lower severity and request developer explanation before escalating
