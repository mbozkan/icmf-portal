# ICMF Methodology

ICMF defines insider code manipulation techniques and provides a structured methodology for evaluating suspicious code patterns in enterprise systems.

## Methodology Goals

- reduce false positives
- standardize manipulation detection
- improve analyst consistency
- distinguish weak signals from strong evidence

## Analysis Flow

1. Identify suspicious code pattern
2. Evaluate execution context
3. Determine whether the behavior is declarative or runtime
4. Assess evidence strength
5. Classify technique category
6. Assign risk level
7. Require analyst review for severe claims

## Context Evaluation

ICMF considers the following contexts before escalating a finding:

- production business logic
- test or mock code
- DTO / model-only structures
- migration / seed scripts
- buffer or temporary data flows
- configuration and metadata definitions

## Severity Guidance

- Critical: direct evidence + exploitability + high impact
- High: strong manipulation evidence with meaningful impact
- Medium: suspicious but incomplete evidence
- Low: weak indicator or design-level risk
- Informational: contextual observation only

## False Positive Controls

ICMF explicitly avoids:

- domain claims based only on class names or table names
- runtime abuse claims for declarative objects
- critical findings without technical proof
