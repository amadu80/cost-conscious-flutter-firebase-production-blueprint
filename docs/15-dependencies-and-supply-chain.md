# Dependencies and software supply chain

## Problem

Flutter plugins, Firebase SDKs, npm packages, GitHub Actions, Terraform providers, and build images execute trusted code or shape production artifacts. Broad upgrades can introduce web-registration changes; stale packages retain known vulnerabilities.

## Options considered

1. Upgrade continuously without grouping or review.
2. Freeze dependencies indefinitely.
3. Use scheduled, bounded, reviewable upgrades with pinned toolchains and reproducible artifacts.

## Decision and why

Pin toolchains and lock resolved dependencies. Group tightly coupled ecosystems such as FlutterFire, review changelogs, upgrade in small commits, and require clean builds and platform-relevant tests. Avoid mass upgrades immediately before a release unless resolving a release-blocking risk.

## Controls

- Restrict dependency sources and review new package ownership, maintenance, permissions, licenses, and transitive risk.
- Pin CI actions to trusted immutable revisions where practical.
- Scan source and history for secrets; never commit service-account keys or package tokens.
- Generate a software bill of materials when distribution, customers, or compliance justify it.
- Sign or attest release artifacts when the delivery chain supports verification.
- Protect branches and deployment environments; prefer short-lived workload identities.
- Separate dependency-only changes so rollback is clear.

AI-assisted changes belong inside the same supply-chain and review model. Record the assistant or engine according to team policy, but never treat attribution as validation. Human reviewers remain accountable for architecture, licensing, security, tests, generated output, and deployment. Exclude private prompts, credentials, proprietary inputs, and unreviewed generated artifacts from public history.

## Consequences and validation

Strict pinning slows security updates; scheduled automation and explicit emergency paths mitigate that. Automated scanners produce false positives and do not evaluate malicious behavior, so human review remains necessary. Validate dependency updates through clean compilation, tests, production web smoke tests, and later physical-device mobile tests.

Revisit controls when external contributors grow, artifacts are distributed through stores, contractual requirements appear, or the threat model includes targeted build-system compromise.
