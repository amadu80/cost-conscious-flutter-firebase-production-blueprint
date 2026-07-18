# Data operations and feature flags

## Problem

Imports, migrations, taxonomy changes, denormalized counters, and feature flags can change production behavior without an application release. A malformed feed or inconsistent flag default can hide data, bypass intended validation, or make the UI disagree with backend rules.

## Options considered

1. Run one-off administrative scripts directly against production.
2. Put every operational change into application releases.
3. Treat data operations and flags as versioned, testable, reversible production changes.

## Decision and why

Use schema-aware, idempotent operations with dry-run output, bounded batches, explicit target validation, before/after counts, audit metadata, and a rollback or repair strategy. Define a versioned feature-flag schema with owners, defaults, expiry, dependencies, and consistent interpretation across client, backend, rules, and restoration tooling.

## Data operation controls

Validate input schema and provenance, quarantine invalid records, avoid embedding administrative credentials in scripts, cap concurrency, and record a migration identifier on affected records where appropriate. Test against representative data, back up before irreversible changes, and verify business invariants after execution.

## Feature-flag controls

Use safe defaults for missing or malformed values. Separate deployment from activation, roll out gradually, observe behavior, and retain a kill switch for high-risk features. A UI flag is not authorization: backend rules and functions must enforce security independently.

## Consequences and validation

Denormalization and repair tooling add complexity but reduce read cost and enable recovery. Flags reduce deployment risk but become permanent branches without ownership and expiry. Revisit the approach when flag count, migration frequency, or data volume requires a dedicated control plane or orchestration system.
