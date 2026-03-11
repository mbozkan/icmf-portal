# ICMF Framework

ICMF (Insider Code Manipulation Framework) is a structured taxonomy for identifying, classifying, and evidencing code patterns associated with insider-driven manipulation in enterprise software systems.

## Scope

ICMF addresses the intersection of **privileged access** and **code-level execution**. It does not classify external attacks or accidental vulnerabilities — those are covered by MITRE ATT&CK, OWASP, and CWE. ICMF addresses what those frameworks do not: deliberate code-level manipulation by actors who already have legitimate access.

## Five Technique Categories

| Category                  | ID Prefix | Focus Area                                              |
|---------------------------|-----------|---------------------------------------------------------|
| Financial Manipulation    | T1xxx     | Financial calculations, balances, fees, payroll         |
| Authorization Manipulation| T2xxx     | Access controls, approval flows, role checks            |
| Audit Manipulation        | T3xxx     | Log suppression, record falsification, silent mutation  |
| Data Manipulation         | T4xxx     | Data exposure, extraction, record filtering             |
| Cross-System Manipulation | T5xxx     | Multi-layer and temporally distributed patterns         |

## Core Design Principles

**1. Evidence-first classification.**
No finding is classified as manipulation without observable technical evidence. See [`evidence-model.md`](evidence-model.md) for evidence levels and claim requirements.

**2. Pattern vs. intent.**
ICMF classifies *code patterns consistent with potential insider manipulation* — not intent. Attribution of malicious purpose remains the responsibility of qualified investigators and legal counsel.

**3. Declarative is not runtime.**
Permission sets, enums, interfaces, configuration files, and metadata objects are not classified as runtime abuse by default, regardless of field names or values.

**4. False positive discipline.**
Domain naming alone (class called `PaymentService`, field called `amount`) is never sufficient for a financial manipulation finding. Direct write evidence in a runtime execution context is required.

**5. Business-language outputs.**
ICMF findings are designed to be comprehensible to CFOs, Internal Audit teams, and regulators — not only security engineers.

## Framework Files

| File | Purpose |
|------|---------|
| [`evidence-model.md`](evidence-model.md) | Evidence levels (0–4), severity thresholds, analyst guidance |
| [`methodology.md`](methodology.md) | Step-by-step analysis flow with decision criteria |
| [`matrix.md`](matrix.md) | Technique × category cross-reference matrix |
| [`techniques/index.md`](techniques/index.md) | Full technique registry with IDs and risk levels |

## Regulatory Alignment

ICMF findings map to the following control frameworks:

- **BDDK** (Turkish Banking Regulation) — information systems security controls
- **PCI DSS v4.0** — requirement 6.3 (security of software development)
- **ISO 27001:2022** — Annex A controls A.8.20–A.8.28
- **SOX** — IT general controls, change management, access controls
- **GDPR** — data integrity and access logging obligations
