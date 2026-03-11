# ICMF Manipulation Lifecycle Model

The ICMF Manipulation Lifecycle Model describes how insider-driven code manipulation typically progresses inside enterprise software environments.

Unlike traditional attack chains that focus on external intrusion, this lifecycle focuses on manipulation introduced by individuals who already have legitimate access to source code, deployment pipelines, or system configuration.

The model helps security teams, auditors, and code analysis platforms understand not only **what** manipulation technique exists, but also **where** it fits in the broader manipulation process.

---

## Lifecycle Phases

1. Preparation
2. Injection
3. Activation
4. Concealment
5. Outcome Realization
6. Persistence or Reuse

---

## 1. Preparation

In the preparation phase, the insider identifies the business workflow, control point, or technical mechanism that can be manipulated.

Typical preparation activities include:

- identifying sensitive business functions
- locating approval, authorization, or financial calculation logic
- understanding audit logging boundaries
- identifying weak review or testing coverage
- selecting low-visibility integration points

### Typical ICMF Technique Alignment
- reconnaissance of financial logic
- identification of privileged code paths
- mapping audit blind spots
- locating cross-system trust assumptions

### Analyst Interpretation
Code changes are not always visible at this phase, but commit behavior, unusual code browsing patterns, or repeated modifications near sensitive logic may become early warning indicators.

---

## 2. Injection

In the injection phase, the insider introduces the manipulation logic into the codebase.

This logic may appear small, legitimate, or context-dependent. It is often embedded inside business logic rather than obviously malicious code.

Typical injection patterns include:

- conditional arithmetic changes
- hardcoded privileged identifiers
- approval bypass branches
- selective filtering logic
- hidden audit suppression paths

### Typical ICMF Technique Alignment
- Hidden Financial Adjustment
- Shadow Mode Backdoor
- Preferred User Bypass
- Audit Log Tampering
- Data Concealment Filtering

### Analyst Interpretation
This is the phase where static analysis has the highest opportunity to detect suspicious patterns before they reach production.

---

## 3. Activation

In the activation phase, the manipulation becomes operational under specific runtime conditions.

Activation may depend on:

- specific user identifiers
- transaction types
- date or time triggers
- batch processing windows
- hidden feature flags
- environment-specific conditions

The manipulation may remain dormant for long periods until the right trigger occurs.

### Typical ICMF Technique Alignment
- Time Triggered Gate
- Conditional Financial Override
- Payment Release Bypass
- Time Bomb Logic

### Analyst Interpretation
Activation conditions are often the strongest evidence of intent, especially when they target specific actors, roles, records, or business events.

---

## 4. Concealment

In the concealment phase, the insider attempts to reduce visibility of the manipulation.

This may include:

- suppressing audit logs
- deleting traces
- altering record visibility
- using indirect helper chains
- distributing logic across files or services
- avoiding obvious code smells

### Typical ICMF Technique Alignment
- Log Suppression
- Audit Trail Deletion
- Misleading Audit Substitution
- Record Filtering
- Temporal Distribution

### Analyst Interpretation
Concealment is a major escalation factor. A pattern that not only manipulates outcomes but also reduces observability should be considered significantly higher risk.

---

## 5. Outcome Realization

In this phase, the manipulated code produces business impact.

Typical outcomes include:

- unauthorized financial gain
- bypassed approval actions
- concealed transactions
- altered reporting outcomes
- hidden cross-system inconsistencies
- selective access advantage

### Typical ICMF Technique Alignment
- Financial Rounding Manipulation
- Ledger Posting Redirection
- Approval Workflow Bypass
- Cross-System State Divergence

### Analyst Interpretation
At this phase, manipulation is no longer theoretical. The effect is visible in runtime behavior, business state, or system records.

---

## 6. Persistence or Reuse

In the final phase, the insider keeps the manipulation available for repeated or delayed use.

Persistence may involve:

- embedding logic in stable utility methods
- hiding behavior behind support or maintenance code
- reusing patterns across modules
- protecting the manipulation from casual refactoring
- making removal costly or risky

### Typical ICMF Technique Alignment
- Helper Method Chain Bypass
- Layered Financial Fraud
- Cross-System State Divergence
- Hidden Privilege Escalation

### Analyst Interpretation
Persistent manipulation is particularly dangerous because it can survive normal maintenance and remain active for long periods.

---

## Lifecycle Summary

| Phase | Goal | Example Signals |
|---|---|---|
| Preparation | Understand exploitable business logic | repeated focus on sensitive modules |
| Injection | Insert manipulation code | suspicious conditional logic |
| Activation | Trigger the manipulation | user-specific or time-based branches |
| Concealment | Reduce visibility | log suppression, filtering, deletion |
| Outcome Realization | Produce business effect | changed balances, bypassed approvals |
| Persistence or Reuse | Keep manipulation available | helper methods, distributed logic |

---

## Why the Lifecycle Model Matters

The ICMF lifecycle model adds a process view to the technique catalog.

It helps teams answer:

- where a suspicious pattern sits in the manipulation chain
- whether the code indicates experimentation or deliberate abuse
- whether concealment or persistence is present
- how urgently the finding should be escalated

This model can support:

- insider threat investigations
- code review prioritization
- AI-assisted code analysis
- secure SDLC controls
- forensic interpretation of suspicious commits
