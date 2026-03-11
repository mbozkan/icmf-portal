# ICMF-XSYS-002: Integration Gate Bypass

**Category:** Cross-System Manipulation  
**Technique ID:** ICMF-XSYS-002  
**Risk Level:** High  
**Evidence Level Required:** Level 3 — Direct Manipulation Evidence  
**Version:** 1.0

---

## Description

Integration Gate Bypass occurs when validation, authorization, or business rule checks that are enforced at the boundary between two integrated systems are circumvented—allowing data or commands to pass from one system to another without completing the required validation steps that protect the receiving system's integrity.

Enterprise integration architectures rely on boundary controls: API gateways, message validation layers, anti-corruption layers, and integration middleware that verify the correctness and authorization of messages before they are accepted by downstream systems. When these gates are bypassed, the downstream system processes data that it would otherwise reject, unaware that the upstream validation was skipped.

Common forms include:

- Posting directly to an internal endpoint that bypasses the validated public API
- Constructing messages that satisfy gateway schema validation but carry semantically invalid payloads
- Using a service-to-service trust relationship to submit commands that should require human-initiated authorization
- Exploiting a maintenance or bypass mode in the integration middleware

---

## Example Patterns

### C# — Internal Endpoint Used to Skip Validation
```csharp
// Public endpoint — full validation pipeline
[HttpPost("/api/payments")]
public IActionResult SubmitPayment([FromBody] PaymentRequest request)
{
    _validationPipeline.Execute(request);
    return _paymentService.Process(request);
}

// Internal endpoint — validation skipped for "service-to-service" calls
// ICMF-XSYS-002: Accessible from internal network without validation
[HttpPost("/internal/payments/direct")]
public IActionResult SubmitPaymentDirect([FromBody] PaymentRequest request)
{
    return _paymentService.Process(request);  // no pipeline
}
```

### C# — Message Queue Injection Bypassing Gateway
```csharp
// Legitimate flow: UI → API Gateway (validates) → Queue → Processor
// Bypass: direct queue write, skips gateway entirely
public async Task InjectPaymentCommand(PaymentCommand cmd)
{
    // ICMF-XSYS-002: Writes directly to the processing queue
    // Gateway's authorization and amount limit checks are never executed
    await _queueClient.SendMessageAsync("payments-processing",
        JsonSerializer.Serialize(cmd));
}
```

### Java — Anti-Corruption Layer Bypass
```java
// Normal path: ExternalPaymentDto → AntiCorruptionLayer → InternalPayment
// Bypass: constructs InternalPayment directly, skipping ACL validation
public PaymentResult submitViaBypass(ExternalPaymentDto dto) {
    // ICMF-XSYS-002: ACL.translate() performs currency checks, amount caps, fraud scoring
    InternalPayment payment = new InternalPayment(
        dto.getAmount(),       // unchecked
        dto.getTargetAccount() // unvalidated
    );
    return paymentProcessor.process(payment);
}
```

### AL (Business Central) — Direct Codeunit Call Bypassing Integration App
```al
// Normal integration flow: External System → Integration App → ValidatedPosting
// Bypass: calls posting codeunit directly without integration app validation
procedure PostExternalPaymentDirect(Amount: Decimal; AccountNo: Code[20])
var
    GenJnlPostLine: Codeunit "Gen. Jnl.-Post Line";
    GenJnlLine: Record "Gen. Journal Line";
begin
    // ICMF-XSYS-002: Integration app validates amounts, exchange rates, and approval status
    // This path bypasses all of those checks
    InitGenJnlLine(GenJnlLine, Amount, AccountNo);
    GenJnlPostLine.RunWithCheck(GenJnlLine);
end;
```

---

## Detection Indicators

### Code Signals
- Internal or undocumented API endpoint that performs the same action as a public endpoint without the validation middleware
- Direct queue write or message bus publication that constructs commands without passing through the API gateway
- Service method invoked directly with a constructed object rather than through the validated entry point
- "Bypass", "direct", "internal", or "skip_validation" in method or endpoint names

### Commit Signals
- New internal endpoint or route added to an integration or payment service
- Direct queue client instantiation in a service that should only submit via the API layer
- Removal of a validation middleware step or anti-corruption layer call from an integration path

### Operational Signals
- Transactions in the downstream system that cannot be traced to a corresponding validated request in the gateway logs
- Processing events in the queue consumer that have no corresponding entry in the API gateway access log
- Payments or postings with field values (amounts, account codes) that would have been rejected by the gateway validation rules

---

## Evidence Requirements

**Evidence Level 3** required. Minimum signals:
1. Code path that submits data or commands to a downstream system without passing through the documented validation or authorization layer
2. The bypassed layer enforces controls that are materially significant (amount limits, authorization checks, fraud scoring)
3. The bypass path is reachable in production

---

## Related Techniques

| Technique ID | Name | Relationship |
|-------------|------|-------------|
| ICMF-XSYS-001 | Cross-System State Divergence | Co-occurrence: bypass used to inject divergent state |
| ICMF-AUTH-003 | Approval Chain Short-Circuit | Structural parallel: both bypass required authorization steps |
| ICMF-FIN-005 | Payment Release Bypass | Specialization: FIN-005 is payment-specific; XSYS-002 is integration-layer bypass |
| ICMF-AUD-002 | Log Suppression Branch | Frequent co-occurrence: bypass traffic not logged by gateway |

---

## Regulatory & Compliance References

| Framework | Control Reference |
|-----------|-----------------|
| PCI DSS v4.0 | Req. 6.4 — Public-facing web application protections; Req. 8.6 — System/application account management |
| ISO 27001:2022 | A.8.21 — Security of network services; A.8.26 — Application security requirements |
| BDDK | Art. 13 — Integration security controls for banking systems |
| NIST SP 800-53 | SC-8 — Transmission integrity; SA-8 — Security and privacy engineering principles |
