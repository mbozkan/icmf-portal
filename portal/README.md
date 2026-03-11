# ICMF Portal

The ICMF portal is the public-facing website for the Insider Code Manipulation Framework.

**Website:** [https://icmf.dev](https://icmf.dev)

---

## Purpose

The portal serves as the primary reference point for security practitioners, auditors, regulators, and developers who want to:

- Understand the ICMF taxonomy and technique definitions
- Access the full technique registry with detection indicators
- Download or reference the ICMF whitepaper
- Follow framework versioning and release notes
- Contribute to the framework via the open contribution process

---

## Planned Portal Sections

| Section | Description | Status |
|---------|-------------|--------|
| Framework Overview | ICMF introduction, scope, design principles | In development |
| Technique Registry | Interactive technique browser with category/risk filters | Planned |
| Whitepaper | Downloadable PDF + HTML version of ICMF v1.0 | In development |
| Detection Methodology | Evidence model and analysis flow documentation | In development |
| Governance | Contribution process, versioning, working group | Planned |
| Changelog | Version history and release notes | Planned |

---

## Technical Stack

The portal is a static site. Framework documentation is sourced directly from this repository's Markdown files.

---

## Content Source

All framework content displayed on the portal is drawn from this repository:

| Portal Page | Source File |
|-------------|-------------|
| Framework overview | `framework/README.md` |
| Evidence model | `framework/evidence-model.md` |
| Methodology | `framework/methodology.md` |
| Technique matrix | `framework/matrix.md` |
| Technique registry | `framework/techniques/index.md` |
| Individual techniques | `framework/techniques/ICMF-T*.md` |
| Governance | `governance/README.md` |

This ensures the portal and the repository remain synchronized. The repository is the single source of truth.

---

## Contact

Framework inquiries: [SecodX Research](https://secodx.com)
Contribution process: See [`governance/README.md`](../governance/README.md)
