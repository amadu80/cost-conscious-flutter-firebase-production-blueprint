# Platform release strategy

## Problem

Flutter shares application code, but web, Android, and iOS have different signing, attestation, permissions, stores, crash symbolication, background behavior, and rollback mechanisms. Treating one successful platform build as cross-platform readiness creates false confidence.

## Options considered

1. Release all platforms simultaneously.
2. Treat mobile as automatically covered by web testing.
3. Release one platform at a time with shared gates plus platform-specific readiness reviews.

## Decision and why

Use phased releases. Stabilize the web pipeline first; keep Android and iOS explicitly deferred until each passes its own security, signing, privacy, device, distribution, observability, and rollback gates. This concentrates limited engineering capacity without losing future risks.

## Shared gates

Domain tests, repository boundaries, generated configuration, backend rules, privacy inventory, release identity, dependency review, and incident ownership apply to every platform.

## Android gates

Validate release signing and key custody, build variants, Play Integrity/App Check, manifest permissions, deep links, notification behavior, physical-device performance, crash/ANR reporting, store declarations, staged rollout, and halt/rollback procedure.

## iOS gates

Validate certificates and provisioning, entitlements, DeviceCheck or App Attest, privacy manifests and permission text, universal links, notification behavior, physical-device testing, dSYM upload and symbolication, TestFlight rollout, review requirements, and recovery procedure.

## Consequences and revisit triggers

Phasing delays reach on deferred platforms but lowers simultaneous incident risk. Shared code can still regress a deferred target, so keep compilation or periodic smoke coverage where affordable. Begin a mobile readiness phase only when ownership, credentials, device coverage, support, and observability are available.
