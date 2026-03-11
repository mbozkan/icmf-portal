# ICMF Whitepaper

**Title:** Insider Code Manipulation Framework (ICMF) v1.0
**Subtitle:** A Structured Taxonomy for Identifying Insider Code Manipulation Patterns in Enterprise Systems
**Version:** 1.0 — Initial Public Release
**Published by:** SecodX Research / ICMF Working Group

**Download:** [ICMF_v1.0_Final.pdf](https://icmf.dev/whitepaper)

---

## Document Purpose

The ICMF whitepaper is the primary specification document for the framework. It provides:
- The complete technique taxonomy with definitions and detection indicators
- The evidence model and confidence scoring methodology
- The detection pipeline for SAST tool integration
- Enterprise implementation guidance across six industry sectors
- Governance structure and version roadmap

The whitepaper is intended for security engineers, CISOs, internal auditors, and regulators who need a comprehensive reference for deploying ICMF-aligned detection.

---

## Section Summary

### Section 01 — Executive Summary
Introduces the classification gap that ICMF addresses: existing frameworks (MITRE ATT&CK, OWASP, CWE) do not provide a structured taxonomy for insider-driven code manipulation. Defines the framework's scope and positioning.

### Section 02 — Problem Definition
Quantifies the financial scale of insider fraud (ACFE 2024: $4.7T annually, $145K median per case, 16-month median detection time). Presents three concrete scenarios that existing security tools cannot detect. Includes a coverage comparison table across major frameworks.

### Section 03 — Origin of ICMF
Describes the recurring manipulation patterns observed in large enterprise systems — ERP platforms, financial transaction processors, and internal workflow systems — that motivated the creation of a dedicated classification framework.

### Section 04 — Threat Model
Defines the four categories of insider actors in scope: internal developers, external contractors, maintainers with privileged repository access, and administrators with deployment capabilities. Clarifies that the framework classifies patterns, not intent.

### Section 05 — Technique Taxonomy
Presents the five-category taxonomy with all 25 technique definitions. Each technique includes: ID, name, category, risk level, evidence level requirement, detection indicators, and code examples. Full taxonomy: Financial (T1xxx), Authorization (T2xxx), Audit (T3xxx), Data (T4xxx), Cross-System (T5xxx).

### Section 06 — Evidence Model
Defines the five-level evidence model (Level 0 Heuristic → Level 4 Confirmed Exploitable). Specifies Domain Claim Guards (Financial Field Write Guard, Transaction Flow Guard, Object Type Guard). Provides code examples for each evidence level.

### Section 07 — Detection Methodology
Seven-step analysis flow from pattern identification through analyst-reviewed severity assignment. Includes decision criteria for each step, false positive control table, and confidence bonus system for correlated multi-signal findings.

### Section 08 — Technique Matrix
Cross-reference matrix mapping all 25 techniques to their primary and secondary categories, evidence level requirements, and co-occurrence patterns.

### Section 09 — Detection Signals Catalog
Three-category catalog of detection signals: Code Signals (direct code evidence), Process Signals (commit behavior indicators), and Operational Signals (runtime and monitoring anomalies).

### Section 10 — Enterprise Implementation
Deployment guidance for six target sectors: Banking & Finance, ERP / Business Central, Insurance, HR & Payroll, Retail & E-Commerce, Logistics & Supply Chain. Includes per-sector risk areas and regulatory alignment.

### Section 11 — SAST Detection Pipeline
Six-phase pipeline specification for integrating ICMF into static analysis tooling: Signal Extraction → Object Type Guard → Domain Claim Guard → Technique Classification → Cross-File Conspiracy Detection → Evidence Report Generation.

### Section 12 — Example Scenarios
Two detailed detection walkthroughs with full ICMF classification and confidence scoring:
- **Scenario A:** Rounding Pattern in Interest Calculation (ICMF-T1001) — TIER-2 finding
- **Scenario B:** Hardcoded Identity Bypass in Payroll (ICMF-T2001 + T2002) — TIER-1 Critical finding

### Section 13 — Governance
ICMF Working Group structure, version roadmap (v1.0 → v2.0), and public registry information at secodx.com/icmf.

### Section 14 — Reference Implementation
SecodX is identified as the reference implementation platform for ICMF, operationalizing the full technique taxonomy in enterprise SAST deployments.

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| v1.0 | 2025 | Initial public release — full taxonomy, evidence model, detection methodology |
| v1.1 | Planned | Confidence scoring model, governance expansion, terminology precision |

---

## Citation

If citing ICMF in research or publications:

```
ICMF Working Group / SecodX Research (2025). Insider Code Manipulation Framework (ICMF) v1.0.
Retrieved from https://icmf.dev
```
