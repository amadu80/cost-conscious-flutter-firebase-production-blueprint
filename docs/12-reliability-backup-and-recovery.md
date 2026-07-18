# Reliability, backup, and recovery

## Problem

Managed services reduce infrastructure work but do not define the application's availability target, prevent bad writes, or prove that backups can be restored. A deployment rollback also cannot undo an incompatible data migration.

## Options considered

1. Accept provider defaults and recover ad hoc.
2. Add multi-region and redundant systems immediately.
3. Define user-journey reliability, bounded degradation, backups, and tested recovery first; add redundancy when impact justifies it.

## Decision and why

Use the third option. Define critical journeys and recovery objectives before buying redundancy. Prefer graceful degradation, idempotent operations, bounded retries, and tested restoration because these address common application failures at lower cost.

## Reliability model

Classify journeys by impact: application startup, authentication, core reads, core writes, uploads, messaging, and administrative recovery. Define an SLI, objective, dependency, failure behavior, and owner for each. A non-critical analytics failure should not block startup; failure of enforced client attestation may be release-blocking because protected data access depends on it.

Use timeouts, capped exponential backoff with jitter, idempotency keys, transactions, and concurrency limits. Avoid retrying permanent authorization or validation errors. Expose cached or read-only states only when doing so is safe and clear to users.

## Backup decision

Automate database backups with retention based on loss tolerance, but treat a configured schedule as unverified until restoration succeeds. Define recovery point objective (acceptable data loss) and recovery time objective (acceptable restoration duration). Preserve infrastructure state and release artifacts separately from application data.

Restore into an isolated project or database, verify counts and invariants, test application compatibility, document permissions and timing, then dispose of test data securely. Protect restore tooling from accidental production targeting.

## The disaster-recovery seed

Backups recover historical application state. They are not always the fastest way to restore the small, canonical dataset required for an application to start safely. Maintain a version-controlled **baseline seed** for deterministic reference data such as geographic hierarchies, required configuration documents, and conservative feature-flag defaults.

This creates a two-track recovery model:

- Restore backups for user-generated, transactional, and legally significant data.
- Run the baseline seed to re-provision canonical data that is derived from reviewed source files.

The seed must be safe to repeat. Use stable document identifiers, upserts or merge writes, bounded batches, and server timestamps. Never generate new identifiers for the same logical record. Separate additive baseline repair from destructive reconciliation: a normal run may create or repair known records, while deleting unknown records requires a separately authorized mode.

### Safe execution contract

A production-capable seed should:

1. Require an explicit project identifier and expected environment; never embed credentials or silently use a CLI default.
2. Reject an unexpected project, database, emulator state, or production target unless the operator provides the documented confirmation control.
3. Support `--dry-run` and print counts by collection without exposing document contents or secrets.
4. Validate the source schema, stable identifiers, expected record-count range, and required feature-flag keys before writing.
5. Use batches below platform limits and report partial progress so an interrupted run can be repeated safely.
6. Preserve user-owned and environment-specific fields. Baseline values must not overwrite emergency feature-flag decisions unless an explicit recovery mode authorizes it.
7. Verify postconditions: expected identifiers exist, counts fall within the reviewed range, required flags have valid types, and a real application read path succeeds.

Treat the canonical input and seed logic as production recovery artifacts: review changes, test against an emulator or disposable project, record a checksum with drill evidence, and prohibit service-account key files from the repository. Prefer short-lived workload identity for automation.

### Recovery drill and evidence

Measure the baseline separately from full data recovery. Record detection time, authorization time, seed duration, validation duration, resulting record counts, script and source revisions, operator, and exceptions. A useful objective might be “restore required reference data and safe configuration within the declared RTO,” but the team must choose the duration from business impact rather than copying an example.

The baseline seed is not a backup. It cannot recover user records, audit history, uploads, or the exact state immediately before an incident. Point-in-time recovery, managed exports, protected infrastructure state, and tested application-data restoration remain necessary.

## Rollback versus recovery

Rollback restores a previous application or configuration. Recovery repairs data or service state. Document ordering across hosting, functions, rules, indexes, App Check enforcement, Terraform, and data migrations. Use forward-compatible schema transitions so old and new clients can coexist during rollback windows.

## Consequences and revisit triggers

Backups and drills cost money and time; size retention to impact and automate verification. Multi-region designs add cost and consistency complexity; consider them when measured objectives, legal requirements, or business impact exceed the resilience of the current region and managed-service commitments.
