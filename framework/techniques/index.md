# ICMF Technique Index

Full registry of ICMF insider code manipulation techniques.

---

## T1xxx — Financial Manipulation

Techniques in this category involve code patterns that alter financial calculation outcomes, redirect financial values, or introduce discrepancies in financial data processing.

| Technique ID | Name | Risk | Evidence Level |
|-------------|------|------|---------------|
| [ICMF-T1001](ICMF-T1001-financial-rounding-manipulation.md) | Financial Rounding Manipulation | Critical | Level 3 |
| [ICMF-T1002](ICMF-T1002-amount-delta-injection.md) | Amount Delta Injection | Critical | Level 3 |
| [ICMF-T1003](ICMF-T1003-fee-commission-manipulation.md) | Fee / Commission Manipulation | Critical | Level 3 |
| [ICMF-T1004](ICMF-T1004-payroll-logic-tampering.md) | Payroll Logic Tampering | Critical | Level 3 |
| [ICMF-T1005](ICMF-T1005-conditional-financial-override.md) | Conditional Financial Override | High | Level 2 |
| [ICMF-T1006](ICMF-T1006-gl-posting-bypass.md) | GL Posting Bypass | High | Level 3 |
| [ICMF-T1007](ICMF-T1007-financial-threshold-bypass.md) | Financial Threshold Bypass | High | Level 2 |
| [ICMF-T1008](ICMF-T1008-micro-transaction-accumulation.md) | Micro-Transaction Accumulation | High | Level 3 |

---

## T2xxx — Authorization Manipulation

Techniques in this category involve code patterns that bypass, disable, or weaken access controls, role checks, or approval workflows.

| Technique ID | Name | Risk | Evidence Level |
|-------------|------|------|---------------|
| [ICMF-T2001](ICMF-T2001-shadow-mode-backdoor.md) | Shadow Mode Backdoor | Critical | Level 3 |
| [ICMF-T2002](ICMF-T2002-preferred-user-bypass.md) | Preferred User Bypass | Critical | Level 3 |
| [ICMF-T2003](ICMF-T2003-approval-workflow-bypass.md) | Approval Workflow Bypass | High | Level 3 |
| [ICMF-T2004](ICMF-T2004-segregation-of-duties-violation.md) | Segregation of Duties Violation | High | Level 2 |
| [ICMF-T2005](ICMF-T2005-time-triggered-gate.md) | Time-Triggered Gate | High | Level 2 |

---

## T3xxx — Audit Manipulation

Techniques in this category involve code patterns that suppress, modify, or bypass audit logging infrastructure.

| Technique ID | Name | Risk | Evidence Level |
|-------------|------|------|---------------|
| [ICMF-T3001](ICMF-T3001-audit-log-tampering.md) | Audit Log Tampering | Critical | Level 3 |
| [ICMF-T3002](ICMF-T3002-log-suppression.md) | Log Suppression | High | Level 3 |
| [ICMF-T3003](ICMF-T3003-silent-state-mutation.md) | Silent State Mutation | High | Level 3 |
| [ICMF-T3004](ICMF-T3004-audit-trail-deletion.md) | Audit Trail Deletion | Critical | Level 4 |

---

## T4xxx — Data Manipulation

Techniques in this category involve code patterns that expose, extract, filter, or skew sensitive financial or personal data.

| Technique ID | Name | Risk | Evidence Level |
|-------------|------|------|---------------|
| [ICMF-T4001](ICMF-T4001-bulk-financial-data-export.md) | Bulk Financial Data Export | High | Level 2 |
| [ICMF-T4002](ICMF-T4002-external-channel-write.md) | External Channel Write | High | Level 3 |
| [ICMF-T4003](ICMF-T4003-data-concealment.md) | Data Concealment | High | Level 2 |
| [ICMF-T4004](ICMF-T4004-record-filtering.md) | Record Filtering | Medium | Level 2 |

---

## T5xxx — Cross-System Manipulation

Techniques in this category involve patterns distributed across multiple files, modules, or commits such that no single file contains sufficient evidence in isolation.

| Technique ID | Name | Risk | Evidence Level |
|-------------|------|------|---------------|
| [ICMF-T5001](ICMF-T5001-layered-financial-fraud.md) | Layered Financial Fraud | Critical | Level 4 |
| [ICMF-T5002](ICMF-T5002-helper-method-chain-bypass.md) | Helper Method Chain Bypass | High | Level 3 |
| [ICMF-T5003](ICMF-T5003-time-bomb-logic.md) | Time Bomb Logic | High | Level 3 |
| [ICMF-T5004](ICMF-T5004-temporal-distribution.md) | Temporal Distribution | High | Level 3 |

---

## Risk Distribution

| Risk Level | Count |
|-----------|-------|
| Critical | 7 |
| High | 16 |
| Medium | 1 |
| Low | 0 |

---

## Evidence Level Requirements

| Minimum Evidence Level | Techniques |
|-----------------------|-----------|
| Level 2 | T1005, T1007, T2004, T2005, T4001, T4003, T4004 |
| Level 3 | T1001, T1002, T1003, T1004, T1006, T1008, T2001, T2002, T2003, T3002, T3003, T4002, T5002, T5003, T5004 |
| Level 4 | T3004, T5001 |
