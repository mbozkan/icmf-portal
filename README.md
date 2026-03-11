# ICMF Portal

Public framework portal for the **Insider Code Manipulation Framework (ICMF)**.

## What is ICMF?

ICMF is a cybersecurity framework focused on detecting insider code manipulation risks in enterprise systems — including ERP platforms, financial software, and large backend applications.

Unlike traditional security frameworks that address external attackers or accidental vulnerabilities, ICMF specifically classifies code patterns introduced by **insiders with legitimate access**: developers, contractors, maintainers, and administrators.

## Why ICMF?

Existing frameworks (MITRE ATT&CK, OWASP, CWE) do not provide a structured taxonomy for classifying patterns consistent with insider manipulation of financial logic, authorization flows, or audit infrastructure. ICMF fills this gap.

## Repository Structure

```
icmf-portal/
├── framework/
│   ├── README.md              # Framework overview
│   ├── evidence-model.md      # Evidence levels and claim requirements
│   ├── methodology.md         # Analysis flow and decision criteria
│   ├── matrix.md              # Technique-to-category cross-reference matrix
│   └── techniques/
│       ├── README.md          # Technique registry overview
│       ├── index.md           # Full technique index
│       └── ICMF-T001-*.md    # Individual technique definitions
├── governance/
│   └── README.md              # Versioning, contribution process, review criteria
├── portal/
│   └── README.md              # Public website information
├── whitepaper/
│   └── README.md              # Whitepaper structure and section summaries
└── LICENSE
```

## Current Status

| Component          | Status                           |
|--------------------|----------------------------------|
| Core taxonomy      | v1.0 Released                    |
| Evidence model     | v1.0 Released                    |
| Technique registry | In progress (25 techniques)      |
| Public portal      | In development — icmf.dev        |
| Whitepaper         | v1.0 Released                    |

## Website

[https://icmf.dev](https://icmf.dev)

## License

ICMF is published under [Creative Commons Attribution-NonCommercial 4.0 International (CC BY-NC 4.0)](LICENSE).

The framework is freely usable for research, education, and non-commercial security work. Commercial tools may implement ICMF; the framework specification itself remains open and vendor-neutral.

## Maintained by

[SecodX Research](https://secodx.com) — ICMF Working Group
