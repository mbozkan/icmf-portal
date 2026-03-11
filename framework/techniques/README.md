# ICMF Technique Registry

The ICMF Technique Registry is the authoritative catalog of insider code manipulation techniques. Each technique represents a distinct pattern of code-level manipulation that an insider with legitimate access may introduce into an enterprise system.

## Technique ID Format

```
ICMF-T[category][sequence]
```

| Prefix | Category |
|--------|----------|
| T1xxx | Financial Manipulation |
| T2xxx | Authorization Manipulation |
| T3xxx | Audit Manipulation |
| T4xxx | Data Manipulation |
| T5xxx | Cross-System Manipulation |

Example: `ICMF-T1001` = Financial Manipulation, first technique.

## Technique Count

| Category | Techniques |
|----------|-----------|
| Financial Manipulation (T1xxx) | 8 |
| Authorization Manipulation (T2xxx) | 5 |
| Audit Manipulation (T3xxx) | 4 |
| Data Manipulation (T4xxx) | 4 |
| Cross-System Manipulation (T5xxx) | 4 |
| **Total** | **25** |

## Technique Definition Standard

Each technique file must include:

| Field | Required | Description |
|-------|----------|-------------|
| Technique ID | Yes | ICMF-Txxxx |
| Category | Yes | One of the five categories |
| Description | Yes | Clear, non-vendor-specific definition |
| Risk Impact | Yes | Critical / High / Medium / Low |
| Evidence Level Required | Yes | Minimum ICMF Evidence Level for this technique |
| Example Pattern | Yes | Code example (pseudocode or language-specific) |
| Detection Indicators | Yes | Observable signals that support this technique |
| Related Techniques | No | Other ICMF techniques commonly co-occurring |
| Regulatory References | No | Applicable control framework mappings |

## Files

| File | Technique |
|------|-----------|
| [`ICMF-T001-shadow-mode-backdoor.md`](ICMF-T001-shadow-mode-backdoor.md) | Shadow Mode Backdoor |
| [`index.md`](index.md) | Full technique index |

> Additional technique files are being added. See [`index.md`](index.md) for the complete registry.
