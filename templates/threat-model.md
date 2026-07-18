# Threat model

- **System/release:**
- **Owner:**
- **Reviewers:**
- **Assessment date (UTC):**
- **Next review:**

## Scope and assumptions

State what is included, excluded, trusted, and unverified.

## Assets and unacceptable outcomes

| Asset or journey | Sensitivity/impact | Unacceptable outcome | Owner |
| --- | --- | --- | --- |
| Example |  |  |  |

## Trust boundaries and data flows

Attach a sanitized diagram. Number every external actor, client, edge, identity provider, backend, datastore, administrative path, CI identity, and third party.

## Threat register

| ID | Actor and capability | Entry point | Threat/abuse case | Existing control | Detection | Containment/recovery | Likelihood | Impact | Residual risk | Evidence |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| T-001 |  |  |  |  |  |  |  |  |  |  |

## Required negative tests

- [ ] Unauthenticated access
- [ ] Wrong ownership or tenant
- [ ] Field injection and immutable-field change
- [ ] Privilege escalation and stale role
- [ ] Replay, oversized input, and unbounded work
- [ ] Unauthorized origin/client and invalid attestation
- [ ] Administrative credential compromise

## Decision

- **Release decision:** GO / CONDITIONAL GO / NO-GO
- **Accepted risks:**
- **Required actions and owners:**
