# Dependencies and software supply chain

## Problem

Flutter plugins, Firebase SDKs, npm packages, GitHub Actions, Terraform providers, and build images execute trusted code or shape production artefacts. Broad upgrades can introduce web-registration changes; stale packages retain known vulnerabilities.

## Options considered

1. Upgrade continuously without grouping or review.
2. Freeze dependencies indefinitely.
3. Use scheduled, bounded, reviewable upgrades with pinned toolchains and reproducible artefacts.

## Decision and why

Pin toolchains and lock resolved dependencies. Group tightly coupled ecosystems such as FlutterFire, review changelogs, upgrade in small commits, and require clean builds and platform-relevant tests. Avoid mass upgrades immediately before a release unless resolving a release-blocking risk.

## Controls

- Restrict dependency sources and review new package ownership, maintenance, permissions, licences, and transitive risk.
- **Pin CI actions:** Use exact commit SHAs rather than moving major version tags (e.g., `v4`) for third-party GitHub Actions to prevent "Silent Substitution" attacks.
- Scan source and history for secrets; never commit service-account keys or package tokens.
- Generate a software bill of materials when distribution, customers, or compliance justify it.
- Sign or attest release artefacts when the delivery chain supports verification.
- Protect branches and deployment environments; prefer short-lived workload identities.
- Separate dependency-only changes so rollback is clear.

AI-assisted changes belong inside the same supply-chain and review model. Record the assistant or engine according to team policy, but never treat attribution as validation. Human reviewers remain accountable for architecture, licensing, security, tests, generated output, and deployment. Exclude private prompts, credentials, proprietary inputs, and unreviewed generated artefacts from public history.

Treat models, assistant IDE extensions, command-line agents, hosted APIs, MCP servers, plugins, skills, system prompts, rules files, and retrieved context sources as supply-chain components. Inventory their publisher, source, version or model identifier, update mechanism, permissions, data destinations, owner, and last review. Allowlist approved integrations and detect changes to tool definitions or persistent instruction files.

Verify every AI-suggested dependency directly in the authoritative registry and upstream repository before installation. Check exact spelling, publisher identity, release history, signatures or provenance where available, licence, maintenance, known vulnerabilities, transitive dependencies, and requested permissions. A build failure is not a reason to install a similarly named package suggested by a model.

Pin or bound versions where supported, stage upgrades, and maintain a rapid disable path for a compromised assistant integration. Do not automatically connect newly discovered tools or let an agent update its own rules, plugins, or permissions without review. Include AI integrations in incident response, access removal, dependency review, and software inventory even when they do not ship inside the application artifact.

## Consequences and validation

Strict pinning slows security updates; scheduled automation and explicit emergency paths mitigate that. Automated scanners produce false positives and do not evaluate malicious behavior, so human review remains necessary. Validate dependency updates through clean compilation, tests, production web smoke tests, and later physical-device mobile tests.

Revisit controls when external contributors grow, artefacts are distributed through stores, contractual requirements appear, or the threat model includes targeted build-system compromise.
