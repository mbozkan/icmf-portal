# ICMF Detection Model

The ICMF Detection Model describes how manipulation techniques can be identified through observable technical signals.

The model links three core elements:

Technique → Evidence Level → Detection Method

This mapping allows static analysis tools, AI code analyzers, and security auditors to consistently classify manipulation patterns.

---

## Detection Layers

ICMF detection operates across three layers:

1. Pattern Detection  
2. Context Evaluation  
3. Evidence Correlation  

Each layer increases the confidence level of the finding.

---

## Detection Mapping

| Technique | Evidence Level | Detection Method |
|----------|---------------|-----------------|
| ICMF-FIN-001 Hidden Financial Adjustment | Level 2+ | Arithmetic modification detection in financial fields |
| ICMF-FIN-002 Financial Rounding Manipulation | Level 2+ | Mathematical operation change analysis |
| ICMF-FIN-003 Discount Threshold Bypass | Level 2+ | Conditional financial branch detection |
| ICMF-FIN-004 Ledger Posting Redirection | Level 3+ | Accounting workflow path analysis |
| ICMF-FIN-005 Payment Release Bypass | Level 3+ | Financial approval flow inspection |
| ICMF-AUTH-001 Shadow Mode Backdoor | Level 3+ | Authorization bypass pattern detection |
| ICMF-AUTH-002 Preferred User Bypass | Level 3+ | Hardcoded identity detection |
| ICMF-AUTH-003 Approval Workflow Bypass | Level 3+ | Workflow short-circuit detection |
| ICMF-AUTH-004 Segregation of Duties Violation | Level 2+ | Role validation inconsistency detection |
| ICMF-AUTH-005 Time Triggered Gate | Level 2+ | Time-based conditional logic detection |
| ICMF-AUD-001 Audit Log Tampering | Level 3+ | Logging infrastructure modification detection |
| ICMF-AUD-002 Log Suppression | Level 3+ | Conditional audit skip detection |
| ICMF-AUD-003 Silent State Mutation | Level 3+ | Business state change without audit trail |
| ICMF-AUD-004 Audit Trail Deletion | Level 4 | Audit record removal detection |
| ICMF-AUD-005 Misleading Audit Substitution | Level 3+ | Audit record substitution detection |
| ICMF-DATA-001 Bulk Financial Data Export | Level 2+ | Large dataset export detection |
| ICMF-DATA-002 External Channel Write | Level 3+ | Unauthorized external IO detection |
| ICMF-DATA-003 Data Concealment | Level 2+ | Conditional record filtering detection |
| ICMF-DATA-004 Record Filtering | Level 2+ | Query-level conditional suppression |
| ICMF-DATA-005 Selective Data Rewrite | Level 3+ | Targeted data mutation detection |
| ICMF-XSYS-001 Layered Financial Fraud | Level 4 | Cross-module financial logic correlation |
| ICMF-XSYS-002 Helper Method Chain Bypass | Level 3+ | Indirect authorization bypass chain |
| ICMF-XSYS-003 Time Bomb Logic | Level 3+ | Delayed execution condition detection |
| ICMF-XSYS-004 Temporal Distribution | Level 3+ | Multi-commit manipulation correlation |
| ICMF-XSYS-005 Cross-System State Divergence | Level 4 | Cross-service data consistency analysis |

---

## Detection Techniques

Typical detection approaches include:

- Abstract Syntax Tree (AST) analysis  
- Control flow analysis  
- Data flow analysis  
- Authorization flow inspection  
- Audit pipeline validation  
- Cross-module correlation  

---

## Detection Model Purpose

The goal of the ICMF detection model is to:

- standardize insider manipulation detection
- support automated code analysis
- guide manual security reviews
- enable AI-assisted investigation

The detection model works together with the **ICMF Evidence Model** to ensure that manipulation claims are supported by sufficient technical proof.
