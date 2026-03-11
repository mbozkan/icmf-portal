# ICMF Technique Index

Full registry of ICMF insider code manipulation techniques.

---

## Financial Manipulation

Techniques in this category involve code patterns that alter financial calculation outcomes, redirect financial values, or introduce discrepancies in financial data processing.

| Technique ID | Name | Risk | Evidence Level |
|-------------|------|------|---------------|
| [ICMF-FIN-001](financial-manipulation/ICMF-FIN-001-financial-rounding-manipulation.md) | Financial Rounding Manipulation | Critical | Level 3 |
| [ICMF-FIN-002](financial-manipulation/ICMF-FIN-002-amount-delta-injection.md) | Amount Delta Injection | Critical | Level 3 |
| [ICMF-FIN-003](financial-manipulation/ICMF-FIN-003-fee-commission-manipulation.md) | Fee / Commission Manipulation | Critical | Level 3 |
| [ICMF-FIN-004](financial-manipulation/ICMF-FIN-004-payroll-logic-tampering.md) | Payroll Logic Tampering | Critical | Level 3 |
| [ICMF-FIN-005](financial-manipulation/ICMF-FIN-005-conditional-financial-override.md) | Conditional Financial Override | High | Level 2 |

---

## Authorization Manipulation

Techniques in this category involve code patterns that bypass, disable, or weaken access controls, role checks, or approval workflows.

| Technique ID | Name | Risk | Evidence Level |
|-------------|------|------|---------------|
| [ICMF-AUTH-001](authorization-manipulation/ICMF-AUTH-001-shadow-mode-backdoor.md) | Shadow Mode Backdoor | Critical | Level 3 |
| [ICMF-AUTH-002](authorization-manipulation/ICMF-AUTH-002-preferred-user-bypass.md) | Preferred User Bypass | Critical | Level 3 |
| [ICMF-AUTH-003](authorization-manipulation/ICMF-AUTH-003-approval-workflow-bypass.md) | Approval Workflow Bypass | High | Level 3 |
| [ICMF-AUTH-004](authorization-manipulation/ICMF-AUTH-004-segregation-of-duties-violation.md) | Segregation of Duties Violation | High | Level 2 |
| [ICMF-AUTH-005](authorization-manipulation/ICMF-AUTH-005-time-triggered-gate.md) | Time-Triggered Gate | High | Level 2 |

---

## Audit Manipulation

Techniques in this category involve code patterns that suppress, modify, or bypass audit logging infrastructure.

| Technique ID | Name | Risk | Evidence Level |
|-------------|------|------|---------------|
| [ICMF-AUD-001](audit-manipulation/ICMF-AUD-001-audit-log-tampering.md) | Audit Log Tampering | Critical | Level 3 |
| [ICMF-AUD-002](audit-manipulation/ICMF-AUD-002-log-suppression.md) | Log Suppression | High | Level 3 |
| [ICMF-AUD-003](audit-manipulation/ICMF-AUD-003-silent-state-mutation.md) | Silent State Mutation | High | Level 3 |
| [ICMF-AUD-004](audit-manipulation/ICMF-AUD-004-audit-trail-deletion.md) | Audit Trail Deletion | Critical | Level 4 |
| [ICMF-AUD-005](audit-manipulation/ICMF-AUD-005-misleading-audit-substitution.md) | Misleading Audit Substitution | High | Level 3 |

---

## Data Manipulation

Techniques in this category involve code patterns that expose, extract, filter, or skew sensitive financial or personal data.

| Technique ID | Name | Risk | Evidence Level |
|-------------|------|------|---------------|
| [ICMF-DATA-001](data-manipulation/ICMF-DATA-001-bulk-financial-data-export.md) | Bulk Financial Data Export | High | Level 2 |
| [ICMF-DATA-002](data-manipulation/ICMF-DATA-002-external-channel-write.md) | External Channel Write | High | Level 3 |
| [ICMF-DATA-003](data-manipulation/ICMF-DATA-003-data-concealment.md) | Data Concealment | High | Level 2 |
| [ICMF-DATA-004](data-manipulation/ICMF-DATA-004-record-filtering.md) | Record Filtering | Medium | Level 2 |
| [ICMF-DATA-005](data-manipulation/ICMF-DATA-005-selective-data-rewrite.md) | Selective Data Rewrite | High | Level 3 |

---

## Cross-System Manipulation

Techniques in this category involve manipulation patterns distributed across multiple files, modules, or commits such that no single file contains sufficient evidence in isolation.

| Technique ID | Name | Risk | Evidence Level |
|-------------|------|------|---------------|
| [ICMF-XSYS-001](cross-system-manipulation/ICMF-XSYS-001-layered-financial-fraud.md) | Layered Financial Fraud | Critical | Level 4 |
| [ICMF-XSYS-002](cross-system-manipulation/ICMF-XSYS-002-helper-method-chain-bypass.md) | Helper Method Chain Bypass | High | Level 3 |
| [ICMF-XSYS-003](cross-system-manipulation/ICMF-XSYS-003-time-bomb-logic.md) | Time Bomb Logic | High | Level 3 |
| [ICMF-XSYS-004](cross-system-manipulation/ICMF-XSYS-004-temporal-distribution.md) | Temporal Distribution | High | Level 3 |
| [ICMF-XSYS-005](cross-system-manipulation/ICMF-XSYS-005-cross-system-state-divergence.md) | Cross-System State Divergence | High | Level 3 |

---

## Risk Distribution

| Risk Level | Count |
|-----------|-------|
| Critical | 7 |
| High | 15 |
| Medium | 1 |
| Low | 0 |

---

## Evidence Level Requirements

| Minimum Evidence Level | Techniques |
|-----------------------|-----------|
| Level 2 | FIN-005, AUTH-004, AUTH-005, DATA-001, DATA-003, DATA-004 |
| Level 3 | FIN-001, FIN-002, FIN-003, FIN-004, AUTH-001, AUTH-002, AUTH-003, AUD-001, AUD-002, AUD-003, DATA-002, DATA-005, XSYS-002, XSYS-003, XSYS-004, XSYS-005 |
| Level 4 | AUD-004, XSYS-001 |
