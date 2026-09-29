---
name: assess-security
description: Produce a structured security assessment identifying threat actors, attack surfaces, attack vectors, and vulnerability findings at the architectural design level. Triggers ADR creation for High and Critical findings. Output saved to /.project/architecture/.
metadata:
  user-invocable: true
---

# Skill: Assess Security

## Purpose

Produce a structured security assessment of the system architecture, identifying threat actors, attack surface across all C4 containers and external trust boundaries, attack vectors per surface area, and vulnerability findings at the design level. This skill runs after the architecture document and integration assessments exist — it does not replace code-level security scanning (SAST/DAST) or runtime pen testing.

---

## Invocation

Use this skill when:
- The architecture document and at least one integration assessment exist and are substantive (no placeholder labels)
- The user asks to assess, review, or analyse the security of the system design
- A new integration assessment has been added and the security posture needs re-evaluation
- Phase 4 (Design architecture) is being completed and no security assessment exists yet

Do not invoke if the architecture document is incomplete or uses placeholder container labels — the assessment will be shallow and risk creating false confidence. Report the gap and stop.

---

## Intake

Gather the following from `.project/architecture/arch-*.md`, `.project/architecture/integration-*.md`, `.project/product-definition/nfrs.md`, and `.project/context/project-context.md` before asking the user. Mark unknown items as `[Unknown — to be confirmed]`.

### Architectural Scope

1. **What C4 containers are in scope?** — List all containers from the architecture document (web app, mobile apps, API gateway, backend services, databases, queues, search indexes, CDN, etc.).
2. **What are the external trust boundaries?** — Every external system with an integration assessment (identity providers, payment processors, content APIs, etc.).
3. **What authentication model does each boundary use?** — API key, OAuth2, mTLS, SAML, JWT, session cookie, other.
4. **What data sensitivity classifications apply?** — PII, financial data, health data, editorial content, credentials, session tokens, other.
5. **What regulatory constraints are in scope?** — GDPR, PCI-DSS, HIPAA, SOC 2, ISO 27001, other.

### Threat Context

6. **Who are the intended users?** — Subscriber, editorial user, admin/operator, anonymous visitor, other.
7. **Is the system internet-facing?** — Fully public, partially public (authenticated areas), internal only.
8. **Are there known adversarial concerns?** — Competitor scraping, account takeover, data exfiltration, content manipulation, availability attacks.

---

## Assessment Steps

1. Read the architecture document and all integration assessments.
2. Identify threat actors relevant to the system and their likely goals.
3. Build the attack surface map — one row per C4 container and one row per external trust boundary.
4. Enumerate attack vectors for each surface area. For each vector, assign a STRIDE category (S — Spoofing, T — Tampering, R — Repudiation, I — Information Disclosure, D — Denial of Service, E — Elevation of Privilege).
5. Produce the STRIDE Coverage Matrix — one row per trust boundary, one column per STRIDE category. Mark each cell Covered or Gap. Promote any Gap cell to a finding unless the risk is explicitly accepted.
6. Produce the vulnerability findings table — findings grounded in the design, not speculative code-level issues. For each finding, include one or more CWE identifiers (e.g. CWE-327, CWE-79).
7. Assign severity ratings (Critical / High / Medium / Low) using the scale defined below.
8. **For every finding rated Critical or High: call the `create-adr` skill before saving the assessment.** The ADR records the finding and the intended mitigation decision.
9. Produce the risk register summary.
10. List open questions where information is missing or where a finding requires follow-up rather than a story.
11. Save the assessment to `.project/architecture/` using the naming convention below.

---

## Severity Ratings

Align with the severity column used in integration assessments:

| Rating | Meaning |
|--------|---------|
| **Critical** | Exploitable design flaw that could result in full system compromise, mass data breach, or complete service unavailability. Must be resolved before implementation begins. |
| **High** | Significant vulnerability that substantially increases risk — missing authentication on internal services, sensitive data in unprotected channels, excessive trust granted to an integration point. Must have a mitigation story in the backlog. |
| **Medium** | Moderate risk — exploitable under specific conditions, or a gap in defence-in-depth. Should be addressed before launch. |
| **Low** | Minor hardening opportunity or informational finding. Can be deferred to a later iteration with documented rationale. |

---

## ADR Trigger Rule

Any finding rated **Critical** or **High** must trigger the `create-adr` skill before the assessment document is saved. The ADR must record:
- The finding (what the vulnerability is and where it exists in the design)
- The intended mitigation (what architectural change resolves it)
- Status: `accepted` if the mitigation is agreed, `gap` if the mitigation is still undecided

Do not save the assessment as a complete artifact if Critical or High findings exist without corresponding ADRs.

---

## validate-backlog Integration

High and Critical findings must map to stories in the backlog. The `validate-backlog` skill will flag unmapped findings as **Blocking**. When the Delivery Lead produces the WBS, they must read the security assessment and include security-specific stories for each High and Critical finding.

---

## Assessment Output

Save the assessment to `.project/architecture/` with the naming convention `security-assessment-[slug]-v[N.N].md`.

### Template

```
# Security Assessment: [System / Project Name]

**Version:** x.x
**Date:** YYYY-MM-DD
**Status:** Draft | In Review | Validated
**Assessed by:** [Agent or person]

## Scope

| Field | Value |
|-------|-------|
| Architecture document | [Link or filename] |
| Integration assessments in scope | [List filenames] |
| Regulatory constraints | [GDPR / PCI-DSS / HIPAA / None identified] |
| Assessment scope | Design-level only — code-level and runtime security are out of scope |

## Threat Actors

| Threat Actor | Goals | Access Level |
|--------------|-------|--------------|
| [e.g. External attacker] | [e.g. Account takeover, data exfiltration] | [Unauthenticated / Authenticated / Privileged] |
| ... | ... | ... |

## Attack Surface Map

| Surface | Type | Authentication | Data Sensitivity | Notes |
|---------|------|----------------|-----------------|-------|
| [Container or boundary name] | Internal container / External boundary | [Auth model] | [PII / Financial / Low] | [Any known exposure] |
| ... | ... | ... | ... | ... |

## Attack Vectors

### [Surface name]

| Vector | Description | STRIDE | Threat Actor | Likelihood |
|--------|-------------|--------|--------------|------------|
| [e.g. Credential stuffing] | [How the attack would work at design level] | [S/T/R/I/D/E] | [Threat actor] | High / Medium / Low |
| ... | ... | ... | ... | ... |

*(Repeat section per surface with meaningful findings.)*

### STRIDE Coverage Matrix

After enumerating all attack vectors, produce a summary matrix per trust boundary to verify no STRIDE category has been overlooked:

| Trust Boundary | S — Spoofing | T — Tampering | R — Repudiation | I — Information Disclosure | D — Denial of Service | E — Elevation of Privilege |
|----------------|-------------|--------------|----------------|---------------------------|----------------------|---------------------------|
| [Boundary / container name] | Covered / Gap | Covered / Gap | Covered / Gap | Covered / Gap | Covered / Gap | Covered / Gap |
| ... | ... | ... | ... | ... | ... | ... |

Mark each cell **Covered** if at least one attack vector addresses that STRIDE category for that boundary, or **Gap** if none does. A Gap cell is a prompt to add a finding or explicitly accept the risk.

## Vulnerability Findings

| # | Finding | Location | Severity | CWE | Mitigation | ADR |
|---|---------|----------|----------|-----|------------|-----|
| 1 | [What the vulnerability is] | [Container or boundary] | Critical / High / Medium / Low | [CWE-NNN, CWE-NNN] | [Recommended architectural change] | [ADR-NNNN or —] |
| ... | ... | ... | ... | ... | ... | ... |

## Risk Register

| # | Risk | Severity | Likelihood | Status | Owner |
|---|------|----------|------------|--------|-------|
| 1 | [Summary of finding or grouped theme] | Critical / High / Medium / Low | High / Medium / Low | Open / Mitigated / Accepted | [Team or role] |
| ... | ... | ... | ... | ... | ... |

## ADR Triggers

*(List all ADRs created as a result of this assessment.)*

| ADR | Finding | Status |
|-----|---------|--------|
| [ADR-NNNN — title] | [Finding #] | Accepted / Gap |
| ... | ... | ... |

## Open Questions

- [ ] [Question — who can answer it, urgency, and whether it blocks a story or requires a follow-up meeting]

## Sources
- *Source: [architecture document or integration assessment filename, section]*
```

---

## Scope Boundaries

This skill is scoped to **design-phase, architectural-level** findings only:

**In scope:**
- Missing or weak authentication on container-to-container communication
- Sensitive data flowing through unprotected channels (unencrypted transport, missing at-rest encryption)
- Excessive trust granted to external integration points
- Missing rate limiting or abuse controls at the API gateway level
- Inadequate session token handling in the architecture design
- Data residency violations visible in the integration landscape
- Missing audit logging for privileged operations at the design level

**Out of scope (belong in separate steps):**
- Code-level vulnerabilities (SAST — belongs in CI)
- Runtime vulnerabilities (pen testing — belongs post-deployment)
- Dependency scanning (SCA — belongs in CI)
- Detailed compliance gap analysis (belongs in a dedicated compliance review)
- Infrastructure configuration hardening (belongs in infrastructure-as-code review)

If a finding is out of scope, note it in the Open Questions section as a deferred item with the recommended step where it should be addressed.

---

## When Information Is Missing

If the architecture document uses placeholder labels or omits containers, do not produce a partial assessment. Report the missing information as a blocker and stop.

If integration assessments are missing for external systems shown in the architecture, note each gap in the Open Questions section — the security posture of that boundary cannot be assessed until the integration assessment exists.

Do not invent threat actors or vulnerabilities. An honest "Unknown — integration assessment required" is more valuable than a speculative finding.
