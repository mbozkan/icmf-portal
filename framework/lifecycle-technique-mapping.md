# ICMF Lifecycle to Technique Mapping

This document maps ICMF manipulation techniques to the phases of the ICMF Manipulation Lifecycle Model.

The purpose of this mapping is to help analysts understand **where a specific manipulation technique typically appears within the insider manipulation process**.

A single technique may appear in more than one lifecycle phase depending on how it is implemented.

---

## Lifecycle Overview

The ICMF Manipulation Lifecycle consists of six phases:

1. Preparation
2. Injection
3. Activation
4. Concealment
5. Outcome Realization
6. Persistence or Reuse

---

# Lifecycle to Technique Mapping

| Lifecycle Phase | Example Techniques | Description |
|---|---|---|
| Preparation | reconnaissance of financial logic, approval flows, audit boundaries | Insider identifies potential manipulation points in the system |
| Injection | ICMF-FIN-001, ICMF-AUTH-001, ICMF-AUTH-002 | Manipulation logic is introduced into the codebase |
| Activation | ICMF-AUTH-005, ICMF-FIN-005, ICMF-XSYS-003 | Manipulation becomes active under specific runtime conditions |
| Concealment | ICMF-AUD-001, ICMF-AUD-002, ICMF-AUD-004, ICMF-DATA-004 | The insider attempts to reduce visibility of manipulation |
| Outcome Realization | ICMF-FIN-002, ICMF-FIN-004, ICMF-AUTH-003 | The manipulated logic produces business impact |
| Persistence or Reuse | ICMF-XSYS-002, ICMF-XSYS-005, ICMF-XSYS-001 | Manipulation is designed to remain hidden and reusable |

---

## Phase Descriptions

### Preparation

The insider studies the system architecture and identifies exploitable business logic.

Typical targets include:

- financial calculation modules
- authorization checkpoints
- approval workflows
- audit logging systems
- cross-system integrations

Preparation rarely produces obvious code indicators but may be visible in commit behavior or exploration patterns.

---

### Injection

The manipulation logic is inserted into the codebase.

Typical injection patterns include:

- conditional financial logic
- hardcoded identity checks
- approval bypass branches
- hidden calculation modifications

Injection is the phase where static code analysis tools have the highest chance of detecting manipulation early.

---

### Activation

Manipulation is triggered when specific runtime conditions occur.

Typical triggers include:

- specific user IDs
- certain transaction types
- scheduled batch operations
- date-based triggers
- hidden feature flags

Activation conditions often reveal intent when they target specific actors or events.

---

### Concealment

The insider attempts to hide or reduce visibility of the manipulation.

Common concealment methods include:

- suppressing audit logs
- filtering records
- bypassing monitoring logic
- distributing manipulation logic across multiple modules

Concealment increases the severity of the finding because it indicates deliberate effort to evade detection.

---

### Outcome Realization

The manipulation produces real business impact.

Examples include:

- altered financial outcomes
- bypassed approvals
- hidden transactions
- altered reports
- inconsistent cross-system data

At this phase the manipulation effect becomes observable in runtime behavior or business state.

---

### Persistence or Reuse

The insider ensures the manipulation remains active or reusable over time.

Persistence strategies may include:

- embedding logic in helper methods
- distributing logic across modules
- hiding behavior behind maintenance code
- protecting the manipulation from refactoring

Persistent manipulation may survive multiple releases and remain undetected for extended periods.

---

## Why Lifecycle Mapping Matters

Lifecycle mapping helps analysts answer critical questions:

- Is the suspicious code an early-stage experiment or a fully operational manipulation?
- Does the code indicate concealment behavior?
- Has the manipulation already produced business impact?
- Does the manipulation appear persistent?

Understanding lifecycle phase helps prioritize investigation and response.
