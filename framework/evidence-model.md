# ICMF Evidence Model

ICMF evaluates insider code manipulation risks using an evidence-first approach.

The framework does not produce severe claims based only on naming patterns, object names, or weak heuristics. High-impact claims require direct code evidence, contextual validation, and consistent technical indicators.

## Core Principles

### 1. Evidence First
A finding must be supported by observable technical evidence before it is classified as manipulation.

### 2. No Naming-Only Claims
Terms such as `Payment`, `Invoice`, `Ledger`, `Employee`, or `Admin` are not sufficient by themselves to justify financial manipulation or authorization bypass claims.

### 3. Declarative Is Not Runtime
Permission definitions, configuration files, manifests, enums, and metadata objects are not treated as runtime abuse by default.

### 4. Critical Requires Direct Proof
Critical severity requires:

- direct code evidence
- clear exploitability path
- high business impact

### 5. Context Matters
Test code, mock objects, DTOs, migration scripts, temporary buffers, and seed data must be evaluated with reduced confidence unless direct abuse evidence exists.

## Evidence Levels

### Level 0 — Heuristic Signal
Weak signal based on naming or loose pattern similarity.

### Level 1 — Structural Indicator
Suspicious structural logic exists, but context is incomplete.

### Level 2 — Contextual Evidence
The suspicious logic is supported by business or workflow context.

### Level 3 — Direct Manipulation Evidence
Code directly changes, bypasses, suppresses, or conceals critical logic.

### Level 4 — Confirmed Exploitable Pattern
Manipulation path, exploitability, and impact are all confirmed.

## High-Severity Claim Requirements

### Financial Manipulation
Requires direct evidence that financial values, balances, taxes, discounts, or accounting-relevant data are being intentionally changed.

### Authorization Manipulation
Requires direct evidence that access checks, role checks, claim checks, or approval logic are bypassed.

### Audit Manipulation
Requires direct evidence that audit logs are deleted, suppressed, altered, or bypassed.

## Analyst Review Guidance

- severe claims should be reviewed before final publication
- naming-only claims should be downgraded or rejected
- declarative content should not be treated as runtime abuse without evidence
- confidence must follow evidence strength
