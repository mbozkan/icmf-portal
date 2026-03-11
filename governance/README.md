# ICMF Governance

ICMF is an open cybersecurity framework. This document defines how the framework is maintained, how contributions are reviewed, and how versions are published.

---

## Governance Goals

- Maintain methodological consistency across technique definitions
- Protect framework credibility through rigorous evidence standards
- Ensure technique evolution is evidence-based, not opinion-based
- Keep public definitions vendor-neutral and implementation-independent
- Provide a transparent, auditable contribution history

---

## Governing Body

**ICMF Working Group**

| Role | Responsibility |
|------|---------------|
| Framework Steward | Maintains canonical specification; approves major version releases; final authority on scope changes |
| Technical Editor | Reviews technique additions; maintains the detection signal registry; enforces definition standards |
| Community Contributors | Submit new techniques, sector guidance, and detection patterns via open process |
| External Reviewers | Academic, legal, and regulatory validation of methodology; invited positions |

Current Framework Steward: **SecodX Research** ([secodx.com](https://secodx.com))

---

## Versioning

ICMF uses semantic versioning for framework releases:

| Increment | Trigger |
|-----------|---------|
| **Major** (e.g., 1.0 → 2.0) | Taxonomy restructure, methodology changes, category additions, breaking changes to technique IDs |
| **Minor** (e.g., 1.0 → 1.1) | New techniques, material refinements to existing techniques, evidence model updates |
| **Patch** (e.g., 1.1 → 1.1.1) | Editorial corrections, typo fixes, clarity improvements with no substantive change |

### Current Roadmap

| Version | Status | Planned Changes |
|---------|--------|----------------|
| ICMF 1.0 | Released | Core taxonomy, evidence model, initial technique library |
| ICMF 1.1 | Planned | Extended technique library (all 25 techniques documented), evidence model code examples |
| ICMF 1.2 | Planned | Detection model improvements, SAST integration specifications, sector guides |
| ICMF 2.0 | Roadmap | Cross-system detection model, ML-assisted correlation, community technique library |

---

## Contribution Process

### What Can Be Contributed

- New technique definitions
- Updates to existing technique descriptions or detection indicators
- Sector-specific guidance (banking, ERP, payroll, etc.)
- Language-specific detection pattern examples (C#, Java, AL, SQL, Python, etc.)
- Regulatory mapping additions
- Editorial corrections

### How to Contribute

1. **Fork** the `icmf-portal` repository
2. **Create a branch** named `technique/ICMF-Txxxx-short-name` or `fix/description`
3. **Use the technique template** below for new technique submissions
4. **Open a Pull Request** with the PR template filled out
5. **Address review feedback** from the Technical Editor
6. Approved contributions are merged and included in the next minor release

### Pull Request Template

When opening a PR for a new technique or significant update, include:

```markdown
## PR Type
- [ ] New technique
- [ ] Technique update
- [ ] Sector guidance
- [ ] Editorial fix

## Technique ID (if applicable)
ICMF-Txxxx

## Summary
[1–3 sentence description of what this contribution adds or changes]

## Evidence Basis
[Describe the real-world observation or research that motivates this technique.
Do not reference proprietary or confidential information.]

## False Positive Risk
[Describe scenarios where this technique might produce false positives and how
the definition mitigates them.]

## Checklist
- [ ] Technique follows the standard definition format (see techniques/README.md)
- [ ] Code examples are in pseudocode or use a common open-source language
- [ ] No vendor-specific or proprietary references
- [ ] Evidence level is appropriate to the claim severity
- [ ] Detection indicators are observable without proprietary tooling
- [ ] No personally identifiable information in examples
```

---

## Review Criteria

The Technical Editor evaluates each contribution against the following criteria:

### Acceptance Criteria

| Criterion | Requirement |
|-----------|------------|
| Technical clarity | The technique is precisely defined and unambiguous |
| Evidence threshold | The minimum evidence level is appropriate to the severity claim |
| False positive risk | The definition includes guards against common misclassification |
| Overlap check | The technique does not duplicate an existing technique without meaningful distinction |
| Language neutrality | The definition is not tied to a specific programming language or platform |
| Vendor neutrality | No commercial product names, proprietary APIs, or confidential research |
| Example quality | Code examples are syntactically valid and clearly illustrate the technique |

### Rejection Criteria

A contribution will be rejected if it:

- Makes severity claims that exceed the evidence level of the provided examples
- Relies on naming-only heuristics without runtime logic evidence
- Is indistinguishable from an existing technique
- Contains vendor-specific implementation details
- Uses confidential or proprietary source material
- Attributes malicious intent in the technique definition (ICMF defines patterns, not intent)

---

## Neutrality Statement

ICMF is a public framework. Commercial tools — including SecodX — may implement ICMF detection. However:

- The framework specification is maintained independently of any commercial implementation
- Technique definitions must not favor or be derived from proprietary detection logic
- All detection indicators in the public registry must be reproducible with open-source tooling
- SecodX Research serves as Framework Steward but does not hold exclusive rights to the taxonomy

Contributions that would create a commercial advantage for any single vendor will not be accepted into the public specification.

---

## Technique Deprecation

A technique may be deprecated if:
- It is subsumed by a more precise technique definition
- Real-world evidence shows it produces excessive false positives without refinement
- The underlying code pattern is no longer observed in modern enterprise systems

Deprecated techniques retain their ID and file with a `DEPRECATED` notice. IDs are never reused.
