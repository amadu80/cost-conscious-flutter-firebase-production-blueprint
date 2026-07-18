# Firebase security

## Problem and threat model

A browser client is outside the trusted boundary. Users can inspect its configuration, modify requests, automate calls, bypass UI validation, and replay operations. Firebase's client access model is safe only when authorization and validation are enforced at backend boundaries.

Primary risks include unauthorized reads or writes, ownership changes, excessive paid operations, malicious uploads, replayed functions, privilege escalation, leaked administrative credentials, and accidental enforcement that blocks legitimate clients.

## Options considered

1. **Trust authenticated clients:** identity is useful, but an authenticated user may still access another user's data or send invalid fields.
2. **Rely on UI validation:** improves experience but is bypassable and provides no backend protection.
3. **Put all access behind custom server endpoints:** centralizes control, but increases backend code, latency, cost, and operational responsibility.
4. **Use layered Firebase controls:** combine Authentication, Security Rules, App Check, least-privilege IAM, validated Functions, and monitoring.

## Decision

Use layered controls and assign each control a distinct responsibility:

- Authentication establishes identity.
- Firestore and Storage Rules authorize and validate direct data access.
- App Check reduces abuse from unofficial clients.
- Cloud Functions protect privileged or cross-record operations.
- IAM restricts administrators and automation.
- Monitoring detects denial spikes, abuse, and enforcement regressions.

## Why

No single control covers the threat model. Authentication without authorisation exposes data; App Check without rules accepts authorised unofficial behaviour; rules cannot safely perform every privileged workflow; server endpoints without IAM remain an administrative risk.

## Authorisation and validation rules

Rules should deny by default and grant the narrowest operation required. Validate ownership, immutable fields, allowed transitions, field types, required and permitted keys, collection scope, and resource size. Queries must align with rule constraints so the client cannot request a broader dataset and rely on result filtering.

### Alternatives and trade-offs

Broad role-based rules are simpler but may grant more access than a feature needs. Highly granular rules reduce blast radius but increase maintenance and test burden. Choose granularity based on data sensitivity, and centralize reusable predicates without hiding the effective permission.

### Validation evidence

Emulator tests must cover permitted and denied paths, unauthenticated callers, wrong owners, privilege changes, missing or extra fields, invalid types, boundary sizes, and query constraints.

## App Check rollout

### Problem

Enforcing App Check immediately can block legitimate production clients when provider configuration, generated build values, authorized domains, or platform initialization are wrong.

### Options

- Leave App Check unenforced: avoids rollout outages but provides limited abuse resistance.
- Enforce immediately: shortens exposure but creates a high release risk.
- Observe, validate, then enforce per service: preserves evidence and supports controlled rollout.

### Decision and why

Adopt staged enforcement. Register supported applications, integrate providers, observe token metrics, validate CI release builds, then enforce one backend service at a time with monitoring and rollback instructions.

App Check is a signal of client authenticity, not user authorisation. Rules and server validation remain mandatory.

## Privileged server operations

Use Cloud Functions for secrets, administrative changes, multi-document invariants, and operations requiring trusted validation. Functions must validate authentication, authorization, App Check where appropriate, input shape, resource bounds, and replay behavior. Use transactions or batches for atomic state changes and idempotency keys for retried commands.

## Diaspora-targeted WAF

A product serving a known country and a defined diaspora can reduce low-value bot traffic before it reaches hosting or Cloud Functions. An edge WAF allowlist based on country codes is a coarse cost and noise control: traffic outside the documented market is blocked or challenged before it consumes application-level work.

This is not an authorisation boundary. IP geolocation is imperfect, users travel, VPNs and mobile networks distort location, and attackers can originate inside an allowed country. Authentication, Security Rules, App Check, input validation, rate limits, and idempotency remain mandatory.

Use a controlled rollout:

1. Define the supported market from product evidence, not intuition, with an owner and review date.
2. Observe country-level traffic using privacy-conscious aggregate metrics.
3. Deploy in log-only or managed-challenge mode where the provider and plan support it.
4. Check false positives for sign-in redirects, crawlers the product intentionally supports, uptime checks, webhooks, provider callbacks, administrators, and emergency access.
5. Enforce a version-controlled country set, alert on unusual blocks, and retain a rapid rollback procedure.

Scope rules by hostname and path where possible. Public marketing content may need global reach while expensive API routes justify tighter controls. Keep machine-to-machine endpoints on explicit authentication or source controls rather than depending on a country rule. Record exceptions with an expiry; do not quietly expand the allowlist after individual complaints.

The measure of success is reduced unwanted origin requests and function invocations without an unacceptable legitimate-user block rate. Review the country set when the market, provider geolocation behavior, accessibility requirements, or incident pattern changes.

### Passive threat intelligence (DMARC reporting)

Hardening is not only about blocking unauthorized traffic; it is about establishing visibility into the source and method of attacks. Email infrastructure is a high-fidelity sensor for brand spoofing.

**Trade-off:** raw reports can be large and difficult to parse manually.

**Mitigation:** include an `rua` (aggregate reporting) tag in the DMARC record to receive daily performance audits from major mail providers. This provides zero-cost intelligence into who is attempting to spoof the project domain, turning a passive security gate into an active audit signal without requiring a paid 3rd-party analyser initially.

## Secrets and IAM

Public Firebase client configuration is not an authorisation secret, but that does not make every identifier suitable for publication. Service-account keys, private API credentials, Terraform state, tokens, and backend secrets belong in managed secret stores or protected CI variables.

Prefer short-lived workload identities over downloaded keys. Separate deployment, runtime, and human roles. Review permissions and unused identities on a schedule.

## Consequences and mitigation

- Layered security creates configuration across several systems. Maintain one security checklist and clear ownership.
- Strict rules may break legitimate features. Deploy rule tests with application changes and monitor denial rates.
- App Check can reject real users because of platform integration faults. Stage enforcement and maintain a rollback procedure.
- Logs needed for investigation can expose data. Redact secrets and unnecessary personal information, and define retention.

## Revisit when

Reassess the model when new platforms are released, data sensitivity changes, privileged workflows expand, compliance requirements appear, or abuse patterns show that direct client access is no longer appropriate.
