# Privacy and data lifecycle

## Problem

Authentication, analytics, crash reporting, messaging, uploads, logs, and support workflows can collect personal or sensitive data. Security controls protect access, but privacy also requires purpose limitation, transparency, retention, deletion, and control over third parties.

## Options considered

1. Collect broadly and define policy later.
2. Disable all analytics and diagnostics.
3. Classify data and collect the minimum necessary under explicit purposes, consent, retention, and deletion rules.

## Decision and why

Use data minimisation and lifecycle ownership. Maintain an inventory covering field, purpose, lawful basis or consent, storage location, access, processor, retention, deletion behaviour, export requirements, and incident impact. Legal requirements vary by jurisdiction and require qualified review.

## Consent and telemetry

Gate optional analytics, advertising, performance, or external integrations where consent is required. Essential security and reliability telemetry must still be minimized and documented. Consent withdrawal should change future collection and be testable; a banner alone is not compliance.

## Retention and deletion

Define retention by data class rather than keeping everything indefinitely. Account deletion must address primary documents, derived records, files, messages, analytics identifiers where supported, backups, and legally required retention. Explain delayed backup expiry and irreversible effects to users.

Use database TTL for genuinely ephemeral records and storage lifecycle transitions for objects whose access value declines, but do not select retention solely to reduce cost. TTL is asynchronous, lifecycle tiering can make recovery slower or more expensive, and neither proves deletion across exports, analytics, backups, or processors. Maintain an infrastructure inventory linking each lifecycle rule to purpose, owner, cost effect, recovery impact, and legal basis.

### Free-tier persistence guard

Ephemeral collections are a silent storage, index, privacy, and operational cost when expiry exists only in application logic. Adopt a creation-time rule: every ephemeral event, OTP, function idempotency marker, temporary session record, and operational log collection must define a timestamp expiry field—preferably a consistent `expireAt` convention—and a matching Terraform-managed TTL policy from its first deployment.

Enforce the contract at the write boundary. Reject or alert on missing expiry values, cap retention to the approved range, and test that timestamps use the database timestamp type expected by TTL. Grant clients no ability to extend security-sensitive retention unless the use case explicitly requires it. Keep the TTL field out of unnecessary composite indexes.

TTL is a cleanup mechanism, not an exact scheduler or a complete deletion guarantee. Expired documents may remain queryable until asynchronous deletion occurs, and TTL deletion itself can incur provider operations. Queries and authorization must treat `expireAt <= now` as expired when immediate invalidation matters. Backups, exports, downstream processors, and legal holds require separate policies.

CI or infrastructure review should fail when a new ephemeral collection lacks all four elements: documented purpose, retention duration, expiry field written on every path, and declared TTL resource. Monitor documents missing expiry, oldest-record age, TTL deletion volume, storage growth, and policy drift. Permanent business and audit records need an explicit retention decision rather than an artificial TTL added only to satisfy the guard.

## Privacy by design

Avoid placing personal data in URLs, metric labels, logs, crash reports, public profiles, or Terraform outputs. Separate public profile projections from private account records. Review uploads, contact details, location precision, moderation evidence, and vulnerable-user workflows with extra care.

### AI coding-assistant context

An assistant may send more than the visible file: open buffers, repository structure, terminal output, Git diffs, diagnostics, prompts, tool arguments, and retrieved documents may enter provider context. Before enabling it, document the provider, subprocessors, retention, training controls, residency, deletion, access, and incident-notification terms appropriate to the data classification.

Keep secrets and regulated or personal data outside the assistant-accessible workspace. Configure product-specific context exclusions for environment files, keys, credentials, exports, Terraform state, support attachments, production logs, and private evidence. `.gitignore` is not a privacy control because it does not restrict filesystem reads. Avoid pasting secrets into terminals or opening sensitive files while a tool can automatically collect surrounding context.

Prompt and tool logs need a purpose, access policy, redaction, retention, deletion path, and TTL where ephemeral. Public prompt history must be sanitized and must not contain private source, personal data, production identifiers, hidden instructions, credentials, or raw model reasoning. Choose a self-hosted, isolated, or no-AI workflow when provider processing is incompatible with the data classification or legal obligation.

## Consequences and validation

Minimization may reduce analytics detail; prefer aggregated questions over collecting raw data “just in case.” Test consent states, export, deletion, access restrictions, log redaction, and third-party failure. Revisit the inventory for every new provider, data field, platform, or purpose.
