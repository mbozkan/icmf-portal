# ICMF Manipulation Matrix

The ICMF Manipulation Matrix organizes insider code manipulation techniques by behavioral category.  
Each column represents a manipulation domain commonly abused by malicious insiders.

This matrix helps security teams, auditors, and code analysis systems quickly identify manipulation patterns across enterprise systems.

---

| Financial Manipulation | Authorization Manipulation | Audit Manipulation | Data Manipulation | Cross-System Manipulation |
|---|---|---|---|---|
| [ICMF-FIN-001](techniques/financial-manipulation/ICMF-FIN-001-hidden-financial-adjustment.md) | [ICMF-AUTH-001](techniques/authorization-manipulation/ICMF-AUTH-001-shadow-mode-backdoor.md) | [ICMF-AUD-001](techniques/audit-manipulation/ICMF-AUD-001-audit-log-tampering.md) | [ICMF-DATA-001](techniques/data-manipulation/ICMF-DATA-001-bulk-financial-data-export.md) | [ICMF-XSYS-001](techniques/cross-system-manipulation/ICMF-XSYS-001-layered-financial-fraud.md) |
| Hidden Financial Adjustment | Shadow Mode Backdoor | Audit Log Tampering | Bulk Financial Data Export | Layered Financial Fraud |
|  |  |  |  |  |
| [ICMF-FIN-002](techniques/financial-manipulation/ICMF-FIN-002-financial-rounding-manipulation.md) | [ICMF-AUTH-002](techniques/authorization-manipulation/ICMF-AUTH-002-preferred-user-bypass.md) | [ICMF-AUD-002](techniques/audit-manipulation/ICMF-AUD-002-log-suppression.md) | [ICMF-DATA-002](techniques/data-manipulation/ICMF-DATA-002-external-channel-write.md) | [ICMF-XSYS-002](techniques/cross-system-manipulation/ICMF-XSYS-002-helper-method-chain-bypass.md) |
| Financial Rounding Manipulation | Preferred User Bypass | Log Suppression | External Channel Write | Helper Method Chain Bypass |
|  |  |  |  |  |
| [ICMF-FIN-003](techniques/financial-manipulation/ICMF-FIN-003-discount-threshold-bypass.md) | [ICMF-AUTH-003](techniques/authorization-manipulation/ICMF-AUTH-003-approval-workflow-bypass.md) | [ICMF-AUD-003](techniques/audit-manipulation/ICMF-AUD-003-silent-state-mutation.md) | [ICMF-DATA-003](techniques/data-manipulation/ICMF-DATA-003-data-concealment.md) | [ICMF-XSYS-003](techniques/cross-system-manipulation/ICMF-XSYS-003-time-bomb-logic.md) |
| Discount Threshold Bypass | Approval Workflow Bypass | Silent State Mutation | Data Concealment | Time Bomb Logic |
|  |  |  |  |  |
| [ICMF-FIN-004](techniques/financial-manipulation/ICMF-FIN-004-ledger-posting-redirection.md) | [ICMF-AUTH-004](techniques/authorization-manipulation/ICMF-AUTH-004-segregation-of-duties-violation.md) | [ICMF-AUD-004](techniques/audit-manipulation/ICMF-AUD-004-audit-trail-deletion.md) | [ICMF-DATA-004](techniques/data-manipulation/ICMF-DATA-004-record-filtering.md) | [ICMF-XSYS-004](techniques/cross-system-manipulation/ICMF-XSYS-004-temporal-distribution.md) |
| Ledger Posting Redirection | Segregation of Duties Violation | Audit Trail Deletion | Record Filtering | Temporal Distribution |
|  |  |  |  |  |
| [ICMF-FIN-005](techniques/financial-manipulation/ICMF-FIN-005-payment-release-bypass.md) | [ICMF-AUTH-005](techniques/authorization-manipulation/ICMF-AUTH-005-time-triggered-gate.md) | [ICMF-AUD-005](techniques/audit-manipulation/ICMF-AUD-005-misleading-audit-substitution.md) | [ICMF-DATA-005](techniques/data-manipulation/ICMF-DATA-005-selective-data-rewrite.md) | [ICMF-XSYS-005](techniques/cross-system-manipulation/ICMF-XSYS-005-cross-system-state-divergence.md) |
| Payment Release Bypass | Time Triggered Gate | Misleading Audit Substitution | Selective Data Rewrite | Cross System State Divergence |

---

## How to Use the Matrix

The matrix allows security teams to:

- identify manipulation patterns during code reviews
- classify insider-driven code tampering
- prioritize high-risk manipulation techniques
- correlate suspicious code changes with manipulation categories
- support automated static analysis systems

---

## Scope

ICMF focuses specifically on **intentional insider code manipulation** rather than accidental coding errors or general software vulnerabilities.
