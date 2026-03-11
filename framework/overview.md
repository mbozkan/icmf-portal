# What is ICMF?

The Insider Code Manipulation Framework (ICMF) is a cybersecurity framework designed to identify, classify, and analyze manipulation patterns introduced into enterprise software by insiders.

Unlike traditional vulnerability frameworks that focus on external attacks, ICMF focuses on risks introduced by individuals with legitimate access to source code.

These risks may include developers, contractors, or administrators who intentionally alter application logic to:

- manipulate financial outcomes
- bypass authorization controls
- suppress audit logging
- conceal sensitive data
- distribute manipulation across multiple systems

---

## Why ICMF Exists

Traditional application security tools primarily detect technical vulnerabilities such as injection attacks, insecure deserialization, or memory corruption.

However, insider manipulation often appears as legitimate application logic and therefore bypasses most security scanners.

ICMF provides a structured way to detect these patterns.

---

## Framework Components

The ICMF framework consists of several core components:

### Technique Catalog

A registry of insider code manipulation techniques grouped into five domains:

- Financial Manipulation
- Authorization Manipulation
- Audit Manipulation
- Data Manipulation
- Cross-System Manipulation

---

### Manipulation Matrix

The ICMF Matrix organizes techniques across manipulation domains and provides a high-level overview of manipulation patterns.

---

### Evidence Model

ICMF uses an evidence-driven model that prevents high-severity claims from being made without sufficient technical proof.

The evidence model defines five levels of evidence ranging from weak heuristic signals to confirmed cross-system manipulation.

---

### Detection Model

The detection model describes how manipulation techniques can be identified through static analysis, contextual evaluation, and cross-module correlation.

---

## Intended Use

ICMF can be used by:

- secure code review teams
- application security programs
- internal audit teams
- insider threat detection systems
- static analysis and AI code analysis tools

---

## Framework Philosophy

ICMF follows several core principles:

- Evidence-first analysis
- Reduced false positives
- Vendor-neutral technique definitions
- Context-aware classification
- Transparent methodology

---

## Future Development

The framework is expected to evolve as new manipulation patterns are discovered and analyzed.

Future versions may expand the technique catalog, detection models, and supporting research.
