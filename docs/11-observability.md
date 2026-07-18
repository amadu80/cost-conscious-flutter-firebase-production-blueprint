# Observability

Observability is the ability to explain system behaviour from its outputs. Monitoring asks whether known conditions crossed a threshold; observability must also support investigation of failures that were not predicted.

## Problem

A Flutter and Firebase system spans browser bootstrap, client state, authentication, App Check, database rules, storage, functions, hosting, and the edge. A green uptime check can coexist with failed sign-in, rejected writes, slow queries, crash loops, or one broken release cohort.

## Options considered

1. Provider dashboards only: inexpensive to start, but fragmented and reactive.
2. Log everything: broad evidence, but costly, noisy, and dangerous for privacy.
3. Add a full commercial platform immediately: unified experience, but disproportionate fixed cost for an early product.
4. Build a focused telemetry model on managed platform signals, then add tooling when investigation gaps justify it.

## Decision and why

Begin with Firebase Crashlytics, Analytics, and Performance for supported clients; structured Cloud Logging, Error Reporting, and Cloud Monitoring for backend services; synthetic web checks; release metadata; and owned alert policies. This covers high-value failure boundaries without committing to an expensive observability platform before scale requires it.

## Telemetry model

### Logs

Emit structured events with timestamp, severity, service, environment, release, operation, outcome, duration, and a generated correlation identifier. Use stable event names. Never log OTP values, tokens, full contact details, private document paths, request bodies, or credentials. Mask identifiers when full values are unnecessary; sample high-volume success events.

### Metrics

Track rates, latency distributions, saturation, and cost drivers. Useful signals include startup success, crash-free sessions, authentication outcomes, valid and rejected App Check traffic, rule denials, function errors/retries, Firestore operations, storage egress, cache-hit ratio, and release adoption.

### Traces

Client and server traces help locate latency across boundaries, but end-to-end distributed tracing may require custom propagation or another platform. Start with named client traces and backend operation timing. Add trace-context propagation only when cross-service latency cannot be diagnosed from existing evidence.

### Events

Product analytics and operational telemetry have different purposes. Analytics must respect consent and data minimization; security and reliability events need a separate lawful and operational basis. Do not use analytics as the only incident signal.

## SLIs, objectives, and alerts

Define indicators around user journeys: successful startup, sign-in, core read, core write, and release asset loading. Set initial objectives from measured baselines rather than invented precision. Alert on sustained user impact or rapid budget consumption, not every individual error.

Every alert needs an owner, severity, runbook, diagnostic links, acknowledgement path, escalation, and tested notification channel. Use synthetic probes for critical flows that provider metrics cannot represent, especially authentication plus an authorized read.

## Dashboards and release correlation

Dashboards should answer: what changed, which users or release are affected, where failure begins, and whether recovery works. Annotate deployments and group errors and performance by release identifier. Retain a public non-sensitive release marker so client reports can be matched to source and artifact.

## Consequences and mitigation

- Managed telemetry has blind spots and vendor-specific semantics. Document signal ownership and test alerts through controlled failure injection.
- Telemetry creates privacy and cost risks. Apply redaction, consent where required, sampling, retention, and access control.
- High-cardinality labels can make metrics expensive or unusable. Keep user and document identifiers in restricted logs, not metric dimensions.
- Crash reporting differs by platform. Validate symbol upload and release mapping before mobile rollout.

## Revisit when

Adopt centralized log routing, OpenTelemetry, or a commercial platform when cross-service investigations remain slow, retention or compliance needs exceed provider capabilities, mobile crash diagnostics require deeper tooling, or on-call objectives justify the operating cost.

## Validation evidence

Simulate a client initialisation failure, rejected authorisation, failing function, latency increase, and usage spike. Confirm that telemetry identifies the release and affected boundary, reaches the owner, supports diagnosis without exposing sensitive data, and verifies recovery.
