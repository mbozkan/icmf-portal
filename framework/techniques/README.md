# ICMF Technique Registry

The ICMF Technique Registry is the authoritative catalog of insider code manipulation techniques. Each technique represents a distinct pattern of code-level manipulation that an insider with legitimate access may introduce into an enterprise system.

## Technique ID Format

ICMF uses category-based technique identifiers.

| Prefix | Category |
|--------|----------|
| FIN | Financial Manipulation |
| AUTH | Authorization Manipulation |
| AUD | Audit Manipulation |
| DATA | Data Manipulation |
| XSYS | Cross-System Manipulation |

Examples:

- `ICMF-FIN-001` = Financial Manipulation, first technique
- `ICMF-AUTH-001` = Authorization Manipulation, first technique
- `ICMF-AUD-001` = Audit Manipulation, first technique
- `ICMF-DATA-001` = Data Manipulation, first technique
- `ICMF-XSYS-001` = Cross-System Manipulation, first technique

## Technique Count

| Category | Techniques |
|----------|-----------|
| Financial Manipulation | 5 |
| Authorization Manipulation | 5 |
| Audit Manipulation | 5 |
| Data Manipulation | 5 |
| Cross-System Manipulation | 5 |
| **Total** | **25** |

## Technique Definition Standard

Each technique file should include:

| Field | Required | Description |
|-------|----------|-------------|
| Technique ID | Yes | Example: `ICMF-FIN-001` |
| Category | Yes | One of the five categories |
| Description | Yes | Clear, non-vendor-specific definition |
| Risk Level | Yes | Critical / High / Medium / Low |
| Example Pattern | Yes | Short pseudocode or language-specific example |
| Detection Indicators | Yes | Observable signals that support this technique |
| Required Evidence | Yes | Minimum evidence needed to justify the claim |
| False Positive Guards | Yes | Conditions that should reduce confidence or prevent escalation |
| Related Risks | Yes | Likely business or security impact |
| Related Techniques | No | Other ICMF techniques commonly co-occurring |
| Regulatory References | No | Applicable control framework mappings |

## Directory Structure

Techniques are organized by category:

- `financial-manipulation/`
- `authorization-manipulation/`
- `audit-manipulation/`
- `data-manipulation/`
- `cross-system-manipulation/`

## Files

| File | Description |
|------|-------------|
| [`index.md`](index.md) | Full technique registry |
| `financial-manipulation/` | Financial manipulation techniques |
| `authorization-manipulation/` | Authorization manipulation techniques |
| `audit-manipulation/` | Audit manipulation techniques |
| `data-manipulation/` | Data manipulation techniques |
| `cross-system-manipulation/` | Cross-system manipulation techniques |

> Technique files are maintained under category folders. The registry and matrix pages should use the same category-based identifier model consistently.
