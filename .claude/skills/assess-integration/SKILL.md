---
name: assess-integration
description: Produce a structured integration assessment for an external system, API, or platform the project must connect to. Covers integration surface, capability gaps, readiness verdict, and timeline impact.
license: "AP AI-Native Delivery Toolkit v3.5 - AP Confidential - governed by LICENSE.md"
metadata:
  user-invocable: true
---

# Skill: Assess Integration

## Purpose

Produce a structured assessment of an external system, API, or platform that the project must integrate with. This skill defines the intake questions, evaluation criteria, and output template used by agents to evaluate integration readiness and risk.

---

## Invocation

Use this skill when:
- The project must connect to an external system (API, platform, SaaS product, data feed)
- A vendor or partner provides a system that needs to be evaluated for integration feasibility
- An existing integration assessment needs to be updated with new information
- The user asks to evaluate, assess, or review an external system's readiness

---

## Intake Questions

Gather as much of the following as possible from source documents before asking the user. Mark unknown items as `[Unknown — to be confirmed]`.

### System Identity

1. **What is the system?** — Name, vendor, version if known.
2. **What role does it play?** — What does the project need from this system? (data source, processing engine, distribution channel, authentication provider, etc.)
3. **Who owns it?** — Is it controlled by the client, a third party, or the project team?

### Integration Surface

4. **What APIs or interfaces are available?** — REST, GraphQL, webhooks, file transfer, SDK, other.
5. **What is the authentication model?** — API key, OAuth2, mTLS, SAML, other.
6. **What data formats are used?** — JSON, XML, CSV, proprietary.
7. **Is there API documentation?** — Link or quality assessment (complete, partial, absent).
8. **Are there rate limits, quotas, or throttling?** — Known limits and implications.
9. **What is the SLA or availability commitment?** — Uptime, response time, support hours.

### Readiness

10. **What is the current maturity?** — Production, beta, alpha, "coming soon", planned.
11. **Are there known gaps or missing features?** — Capabilities the project needs that the system does not yet provide.
12. **Is there a sandbox or test environment?** — Available now, available on request, not available.
13. **What is the onboarding process?** — Self-serve, requires contract, requires technical setup with vendor support.

### Risk Factors

14. **What happens if this integration is unavailable or delayed?** — Impact on the project timeline and scope.
15. **Are there alternative systems?** — Fallback options if this integration proves infeasible.
16. **Are there data privacy or compliance considerations?** — GDPR, data residency, PII handling.

---

## Assessment Output

Save the assessment to `.project/architecture/` with the naming convention `integration-[system-name]-v[N.N].md`.

### Template

```
# Integration Assessment: [System Name]

**Version:** x.x
**Date:** YYYY-MM-DD
**Status:** Draft | In Review | Validated
**Assessed by:** [Agent or person]

## System Overview

| Field | Value |
|-------|-------|
| System | [Name and vendor] |
| Role in project | [What the project needs from this system] |
| Owner | [Client / third party / project team] |
| Current maturity | [Production / Beta / Alpha / Planned] |
| Documentation | [Complete / Partial / Absent — link if available] |

## Integration Surface

| Aspect | Details |
|--------|---------|
| Interface type | [REST / GraphQL / Webhooks / SDK / File transfer / Other] |
| Authentication | [API key / OAuth2 / mTLS / Other] |
| Data format | [JSON / XML / Other] |
| Rate limits | [Known limits or "Unknown"] |
| SLA / Availability | [Uptime commitment, response time, support hours] |
| Sandbox available | [Yes / On request / No] |

## Capabilities vs. Requirements

| Project Requirement | System Capability | Gap? |
|---------------------|-------------------|------|
| [Requirement 1] | [What the system provides] | [None / Partial / Missing] |
| ... | ... | ... |

## Gaps and Risks

| # | Gap or Risk | Severity | Mitigation or Workaround |
|---|-------------|----------|--------------------------|
| 1 | ... | Critical / High / Medium / Low | ... |

## Readiness Verdict

**Integration readiness:** Ready / Ready with caveats / Not ready — blocked on [X]

[One paragraph summary: can we integrate with this system given what we know? What must be resolved before integration work begins? What is the recommended next step?]

## Dependencies and Timeline Impact

[How does this integration affect the delivery timeline? Is it on the critical path? What phase should integration work be scheduled in?]

## Open Questions
- [ ] [Question — who can answer it, urgency]

## Sources
- *Source: [document or conversation where information was obtained]*
```

---

## Readiness Ratings

Use these consistently across all integration assessments:

| Rating | Meaning |
|--------|---------|
| **Ready** | API is production-grade, documented, accessible, and meets all project requirements |
| **Ready with caveats** | Integrable, but with known gaps, workarounds required, or conditions that must be met |
| **Not ready — blocked** | Cannot proceed until specific blockers are resolved (missing API, no access, critical gap) |

---

## When Information Is Missing

If source documents do not contain enough information to complete the assessment, fill in what is known and mark gaps as `[Unknown — to be confirmed]`. List specific questions in the Open Questions section with the person or team who can answer them.

Do not invent capabilities or assume readiness. An honest "Unknown" is more valuable than a plausible guess.
