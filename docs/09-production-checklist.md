# Production checklist

This checklist is a decision aid, not proof by checkbox. Every required item should have evidence, an owner, and a known response when it fails. Mark an item not applicable only with a recorded reason.

## Release decision model

### Problem

Production releases combine code, generated configuration, infrastructure, security enforcement, browser caches, and external services. A green build covers only part of that system.

### Options considered

1. **Release when CI is green:** fast, but ignores deployment and operational readiness.
2. **Require informal approval from an experienced person:** captures judgment but is inconsistent and difficult to audit.
3. **Use evidence-based release gates:** requires more preparation but makes risk, exceptions, and rollback explicit.

### Decision and why

Use evidence-based gates proportional to change risk. A routine content change should not carry the same ceremony as new authentication, security-rule enforcement, data migration, or infrastructure replacement. However, security, configuration, public verification, and rollback ownership cannot be implied by a successful compilation.

## Gate 1: Scope and reproducibility

**Why:** prevents deploying the wrong commit, target, or locally contaminated artefact.

- [ ] Release scope, target environment, commit, version, and owner are identified.
- [ ] Toolchain and dependency versions are controlled or recorded.
- [ ] Required generated configuration can be produced in a clean environment.
- [ ] Required build-time values are present and validated without exposing secrets.
- [ ] Formatting, analysis, deterministic tests, rule tests, and infrastructure validation pass.
- [ ] The production artefact was built by the documented workflow.

**Stop condition:** do not release if the artefact cannot be reproduced from the identified commit and declared inputs.

## Gate 2: Security boundaries

**Why:** client behaviour is untrusted and enforcement changes can either expose data or block legitimate users.

- [ ] Authentication establishes the intended identities and handles initialization failure.
- [ ] Firestore and Storage tests cover allowed and denied operations.
- [ ] Ownership, immutable fields, field types, transitions, and query scope are enforced.
- [ ] App Check metrics support the planned enforcement level for every released platform.
- [ ] Privileged Functions validate identity, authorisation, input bounds, replay, and idempotency.
- [ ] Runtime and deployment IAM follow least privilege.
- [ ] No credentials, Terraform state, private variables, or sensitive artefacts are staged.
- [ ] Public security headers and TLS behaviour match policy.

**Stop condition:** do not accept an undocumented authorisation bypass or an App Check rollout that has not been observed with the release artefact.

## Gate 3: Infrastructure change

**Why:** an application release can depend on DNS, hosting, IAM, cache, budget, or rule changes outside the compiled bundle.

- [ ] The target Terraform workspace or state is unambiguous.
- [ ] The plan was reviewed for creates, updates, replacements, and deletions.
- [ ] Imports and manual exceptions are documented.
- [ ] Provider credentials are short-lived or appropriately protected.
- [ ] Destructive or difficult-to-reverse changes have backup and recovery evidence.
- [ ] Drift or console changes have been reconciled deliberately.

**Stop condition:** do not apply an unexplained replacement or deletion to discover what happens.

## Gate 4: Web release and caching

**Why:** browsers and edges can combine an old control file with new assets or preserve a faulty service worker.

- [ ] Release identity is embedded and externally observable.
- [ ] Fingerprinted assets use an immutable cache policy.
- [ ] HTML, manifest, service worker, and release metadata revalidate appropriately.
- [ ] SPA routes, refreshes, and deep links resolve correctly.
- [ ] Authentication and protected backend services initialize on the public origin.
- [ ] A clean browser profile passes smoke tests.
- [ ] Upgrade from the previous release passes without missing or incompatible assets.
- [ ] Hosting and edge headers were verified from the public URL, not only configuration files.

**Stop condition:** do not promote when the deployed release identity is unknown or consecutive-release behaviour has not been tested after cache-policy changes.

## Gate 5: Data and backward compatibility

**Why:** clients, functions, and stored documents do not all change atomically.

- [ ] New code tolerates data written by the previous supported release.
- [ ] Schema or field transitions have a bounded migration and rollback strategy.
- [ ] Indexes exist before queries depend on them.
- [ ] Batch sizes, retries, and concurrency are capped.
- [ ] Multi-record changes are atomic or safely repairable.
- [ ] Backups and restoration were tested at a frequency appropriate to impact.

**Stop condition:** do not run an irreversible migration without verified backup, compatibility, and recovery evidence.

## Gate 6: Operations and cost

**Why:** deployment success does not detect partial initialization failures, abuse, retry storms, or variable-cost growth.

- [ ] Availability, release identity, client initialization, and uncaught errors are monitored.
- [ ] Authentication, App Check, authorisation denials, and backend errors have usable signals.
- [ ] Function latency, retries, and idempotency conflicts are observable.
- [ ] Cloud and edge budgets have staged thresholds and owned notification routes.
- [ ] Quotas and abuse controls have sufficient legitimate-traffic headroom.
- [ ] Logs are redacted, access-controlled, and retained deliberately.
- [ ] Post-release observation has an owner and defined duration.

**Stop condition:** do not release a high-risk change when no one can detect or respond to its primary failure modes.

## Gate 7: Rollback and communication

**Why:** recovery steps written for an older architecture may fail when they are finally needed.

- [ ] The previous known-good artefact or deployment is identifiable.
- [ ] Application, hosting, rules, functions, and infrastructure rollback order is understood.
- [ ] Rollback does not reintroduce incompatible data or security behaviour.
- [ ] Operational and user communication owners are identified.
- [ ] The current rollback procedure has been exercised in a safe environment.

**Stop condition:** do not make a difficult-to-reverse change without an explicitly accepted risk and decision owner.

## Exception process

When a gate cannot pass, record the failed control, reason, user and business impact, compensating control, expiry, and accountable approver. Exceptions must expire; otherwise temporary risk silently becomes architecture.

## Revisit when

Update this checklist after incidents, major platform upgrades, new release targets, architectural changes, or controls that repeatedly fail to predict actual release risk. Remove gates that cannot influence a decision and add evidence where failures exposed an assumption.
