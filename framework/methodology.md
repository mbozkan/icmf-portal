# ICMF Methodology

ICMF defines a structured methodology for evaluating suspicious code patterns in enterprise systems. The methodology is designed to minimize false positives while ensuring high-confidence findings carry sufficient evidentiary weight for audit and investigative use.

---

## Methodology Goals

- Reduce false positives through mandatory evidence gates
- Standardize manipulation detection across analysts and tools
- Improve analyst consistency for cross-team investigations
- Clearly distinguish weak signals from actionable evidence
- Produce findings usable by both security engineers and business stakeholders

---

## Analysis Flow

### Step 1 — Identify Suspicious Code Pattern

Detect a code pattern that matches a known ICMF signal category:
- Financial field write with unexpected mathematical operation
- Conditional branch bypassing authorization checks
- Logging suppression or audit record modification
- Bulk data access or unexpected external write
- Cross-module distribution of related changes

**Exit criteria:** A specific code construct is identified and recorded with file path, line numbers, and the triggering signal.

---

### Step 2 — Evaluate Execution Context

**Decision: Is this code in a production execution path?**

| Context | Action |
|---------|--------|
| Production business logic | Continue to Step 3 |
| Test / mock / fixture | Reduce confidence — do not escalate to High or Critical |
| Migration / seed script | Reduce confidence — flag as informational |
| DTO / model-only (no logic) | Apply Object Type Guard — exit |
| Configuration / enum / manifest | Apply Declarative Guard — exit |

> **Object Type Guard:** If the containing type is a PermissionSet, Enum, Interface, Configuration class, or pure data container with no business logic, the finding is excluded from financial and authorization manipulation classifications regardless of field names.

> **Declarative Guard:** If the suspicious pattern exists only as a value assignment in a configuration or mapping context — not as a runtime calculation — the finding is excluded.

---

### Step 3 — Apply Domain Claim Guard

**Decision: Does direct field evidence exist?**

For financial manipulation claims:
- Is there a direct **write** to a recognized financial field? (`amount`, `balance`, `price`, `salary`, `fee`, `commission`, `tax`, `discount`, `bonus`, `total`, `subtotal`)
- Does the write occur in a financial transaction flow? (`payment`, `invoice`, `ledger`, `posting`, `payroll`, `settlement`, `clearing`)

For authorization manipulation claims:
- Is there a conditional branch that **returns authorized/approved** without executing the normal check?
- Is there a hardcoded identity or credential that short-circuits the authorization path?

**If no direct field evidence → maximum severity is Medium. Do not assign High or Critical.**

---

### Step 4 — Assess Evidence Strength

Map the finding to an ICMF Evidence Level:

| Level | Name | Criteria | Max Severity |
|-------|------|----------|--------------|
| 0 | Heuristic Signal | Naming only, no logic evidence | Informational |
| 1 | Structural Indicator | Suspicious structure, context unclear | Low–Medium |
| 2 | Contextual Evidence | Pattern + business context confirms plausibility | Medium–High |
| 3 | Direct Manipulation Evidence | Runtime write/bypass clearly observable | High–Critical |
| 4 | Confirmed Exploitable Pattern | Multi-signal, cross-file, same actor | Critical |

**Key questions for this step:**
1. Is the manipulation reachable in production without additional conditions?
2. Is the same pattern corroborated in multiple files or modules?
3. Does the commit history show this change was introduced by a single actor in a short window?
4. Does the change suppress or bypass audit logging in the same or adjacent commit?

---

### Step 5 — Classify ICMF Technique

Map the confirmed finding to a specific ICMF technique from the [technique index](techniques/index.md).

If multiple techniques apply, classify all applicable techniques. Note whether cross-technique correlation increases the overall confidence score (see confidence bonuses below).

**Confidence bonuses:**

| Bonus Signal | Effect |
|--------------|--------|
| Cross-file correlation (same pattern, related files) | +1 Evidence Level |
| Actor attribution (same commit author, financial + auth bypass) | +1 Evidence Level |
| Temporal clustering (related changes within 30-day commit window) | +0.5 Evidence Level |
| Audit suppression co-occurrence with financial pattern | +1 Evidence Level |

---

### Step 6 — Assign Risk Level

| Evidence Level | Risk Level | Definition |
|---------------|-----------|------------|
| 4 | **Critical** | Direct evidence + exploitability + high impact confirmed |
| 3 | **High** | Strong manipulation evidence with meaningful business impact |
| 2 | **Medium** | Suspicious but incomplete evidence; plausible manipulation |
| 1 | **Low** | Weak indicator or design-level risk |
| 0 | **Informational** | Contextual observation only; no actionable evidence |

---

### Step 7 — Require Analyst Review for Severe Claims

Before publishing or acting on a Critical or High finding:

**Checklist:**
- [ ] Direct code evidence documented with file path and line numbers
- [ ] Evidence Level confirmed at Level 3 or 4
- [ ] Domain Claim Guard passed (direct field write or bypass confirmed)
- [ ] Object Type and Declarative Guards passed (not a config or enum)
- [ ] No test/mock context present
- [ ] Second analyst has reviewed the finding
- [ ] Developer explanation has been considered or requested

**If any checkbox fails → downgrade to Medium and flag for follow-up.**

---

## False Positive Controls

ICMF explicitly avoids the following:

| Anti-Pattern | Control |
|-------------|---------|
| Domain claims based only on class or table names | Domain Claim Guard (Step 3) |
| Runtime abuse claims for declarative objects | Object Type Guard (Step 2) |
| Critical findings without technical proof | Analyst Review Checklist (Step 7) |
| Escalation of test code | Execution Context Gate (Step 2) |
| Single-signal Critical findings | Evidence Level ceiling (Step 6) |

---

## Severity Definitions (Summary)

| Level | Definition |
|-------|-----------|
| **Critical** | Direct, confirmed, exploitable manipulation with high business impact |
| **High** | Strong manipulation evidence with meaningful but not fully confirmed impact |
| **Medium** | Suspicious pattern with plausible but incomplete evidence |
| **Low** | Weak indicator or design-level risk without clear exploit path |
| **Informational** | Observation only — no current evidence of manipulation |
