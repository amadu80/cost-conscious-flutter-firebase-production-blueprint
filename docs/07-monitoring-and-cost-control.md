# Monitoring and cost control

## Problem

A serverless application can appear operational while initialization, authentication, App Check, rules, or background functions fail for part of the user base. Consumption-based services also turn inefficient code, retries, or abuse into variable cost. Waiting for user reports or the monthly invoice is not an operational strategy.

## Options considered

1. **Use provider dashboards reactively:** no setup cost, but failures are discovered late and investigation lacks agreed baselines.
2. **Collect every available signal:** broad evidence, but expensive, noisy, and potentially harmful to privacy.
3. **Monitor only uptime:** useful for total outages, but blind to client initialisation, authorisation, data, and cost failures.
4. **Adopt action-oriented monitoring:** collect a small set of service and user-journey signals, each tied to an owner and response.

## Decision

Start with action-oriented monitoring and expand from demonstrated risks. Monitor availability, release identity, client initialisation, uncaught errors, authentication, App Check, authorization denials, function errors and latency, retry behaviour, database operations, storage and egress growth, and edge-cache effectiveness.

## Why

Observability is useful only when it changes a decision. A focused set reduces cost and alert fatigue while covering the boundaries most likely to fail during a Flutter web and Firebase release.

## Service-level signals

### Web and client

- Public availability and expected release identifier
- Bootstrap and Firebase initialization success
- Uncaught client errors grouped by release
- Deep-link, refresh, and authentication redirect health

### Backend

- Authentication failures by category
- App Check valid, invalid, and missing token trends
- Firestore or Storage denial and usage anomalies
- Function error rate, latency, retries, timeouts, and concurrency
- Idempotency conflicts or repeated side effects

### Cost drivers

- Reads, writes, storage, function execution, egress, and hosting traffic
- Cache-hit behaviour and unexpected origin traffic
- Daily usage against an established baseline, not only monthly totals

## Budget decision

### Options

- One combined budget: simple but hides which provider or service drives spend.
- Service-specific budgets: clearer attribution but more thresholds and ownership.
- Quotas alone: can contain some usage but may cause an abrupt user outage.

### Decision and why

Use independent cloud and edge budgets with staged thresholds and an owned alert route. Combine alerts with safe quotas and abuse controls where appropriate. Budgets are notifications, not guaranteed spending caps.

### Budget decoupling for fast triage

A monolithic budget hides the source of financial volatility. Decoupling budgets by provider (e.g., GCP for backend, Cloudflare for edge) ensures that an alert instantly identifies the layer requiring investigation.

**Trade-off:** more variables to manage and potentially redundant alerts if an anomaly impacts both layers.

**Mitigation:** align thresholds with the specific cost-drivers of each service (e.g., egress for the edge, reads/writes for the backend). Use consistent naming conventions so decoupled budgets can be aggregated in high-level reporting.

## Alert design

Every alert must state the affected user outcome, threshold rationale, severity, owner, first diagnostic action, mitigation, and escalation path. Test notification delivery and review alerts that never trigger or trigger without action.

Avoid alerts on metrics merely because they exist. Prefer rates, sustained windows, and release comparisons over isolated events.

## Privacy and retention

Collect the minimum context needed to operate the service. Do not log secrets, tokens, full request bodies, or unnecessary personal data. Define access, redaction, sampling, and retention. Operational convenience does not override privacy obligations.

## Consequences and mitigation

- Focused monitoring can miss unknown failure modes. Add signals after pre-mortems, incidents, and architecture changes.
- Tight alerts create noise. Tune against baselines and require ownership.
- Logging increases spend and exposure. Sample, aggregate, redact, and retain deliberately.
- Quotas can protect budget by denying legitimate work. Set them with headroom, monitor approach, and document emergency changes.

## Revisit when

Expand observability when mobile releases add platforms, user impact grows, compliance changes retention needs, on-call ownership develops, or service objectives require more precise measurements.

## Validation evidence

Inject or simulate representative failures. Confirm the right alert reaches the right owner, provides enough evidence for diagnosis, and leads to a practiced mitigation before user reports become the primary detector.

For the architectural choices behind cost-conscious implementation, see [Cost-conscious engineering decisions](10-cost-conscious-engineering.md).

For the complete telemetry model—including logs, metrics, traces, events, release correlation, objectives, synthetic checks, dashboards, and investigation—see [Observability](11-observability.md).
