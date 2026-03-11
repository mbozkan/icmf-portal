```md
# ICMF-T001: Shadow Mode Backdoor

## Category
Authorization Manipulation

## Description
Shadow Mode Backdoor is a hidden access mechanism embedded in application logic that bypasses normal authorization controls when specific hidden conditions are met.

These conditions may include:

- specific user IDs
- hashed identifiers
- hidden configuration flags
- debug switches

When the condition is satisfied, the system silently disables authorization checks or grants elevated privileges.

## Risk Impact
Critical

Shadow mode mechanisms allow insiders to perform unauthorized actions without detection.

## Example Pattern

``` csharp
if(userId == "ADMIN_TEST_42")
{
    bypassAuthorization = true;
}
```

## Detection Indicators

- Hardcoded privileged user identifiers
- Conditional authorization bypass logic
- Hidden debug flags in production code

## Related Risks

- Unauthorized privilege escalation
- Insider fraud
- Audit evasion
