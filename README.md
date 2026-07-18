# Cost-Conscious Flutter + Firebase Production Blueprint

A technology-neutral case study and practical guide for taking a Flutter web application from prototype to a secure, observable, repeatable, and cost-conscious production release.

This repository documents reusable engineering decisions—not a specific product. All names, identifiers, values, diagrams, and examples are intentionally generic.

## The problem

A prototype can compile, pass basic tests, and deploy while still being unprepared for production. Browser caches can combine incompatible releases, generated configuration can exist locally but not in CI, client-side validation can be bypassed, manual infrastructure can drift, and consumption-based services can turn inefficient access or abuse into unexpected cost.

For a small team, the opposite approach is also risky: building a large platform before product demand is known consumes the budget and creates systems that must be operated without evidence that they are needed.

## Options and decision

Three broad approaches were considered:

1. Release the prototype and respond to failures reactively.
2. Build a comprehensive production platform before launch.
3. Harden the system incrementally, prioritizing security, release, recovery, and cost risks.

This blueprint adopts the third approach. Every major recommendation should identify the problem, credible alternatives, selected decision, reasoning, consequences, mitigation, validation evidence, and a condition for revisiting it.

The goal is not a universal “best architecture.” It is a traceable decision process that another engineer can adapt when constraints differ.

## What cost-conscious means

Cost-conscious does not mean choosing the cheapest service, depending on free tiers, or removing security, tests, backups, and telemetry. It means treating cost as an architectural constraint alongside user impact, security, reliability, privacy, performance, and team capacity.

Each decision should consider:

- Fixed cost before meaningful usage
- Variable cost per successful user journey
- Cost amplification during retries, abuse, scraping, or failure
- Engineering and operational time required to maintain the choice
- Complexity and incident cost introduced by another provider or abstraction
- Migration and exit cost if the original assumptions stop being true
- Evidence and thresholds that trigger reinvestment or replacement

A choice is cost-conscious only when its savings exceed the operational and risk costs it creates. “Low cost” without measured usage and failure scenarios is an unsupported claim.

## AI-assisted engineering, not a code-example repository

The main reusable artifact is the [AI-assisted engineering workflow](docs/21-ai-assisted-engineering-workflow.md): how context was gathered, options were challenged, surgical changes were made, evidence was required, incidents were converted into controls, and AI contributions were attributed without transferring human accountability.

[Evidence templates](templates/README.md) support that workflow across system and AI-assistant threat modelling, releases, observability, restore drills, privacy, infrastructure ownership, performance/cost, accessibility, supply chain, and accepted risk.

This repository intentionally does not focus on copied code samples. Code without the original constraints, tests, runtime, and operating model would create false confidence and distract from the transferable decision process.

## What this blueprint covers

- Layered Flutter architecture with Firebase behind repository interfaces
- Firebase Authentication, Firestore, Hosting, and App Check
- Infrastructure management with Terraform
- Cloudflare DNS, caching, security headers, and edge controls
- Repeatable Flutter web releases and stale-asset mitigation
- CI quality gates, deterministic tests, formatting, and static analysis
- Observability across logs, metrics, traces, events, release correlation, SLOs, synthetic checks, dashboards, and alerting
- Reliability, backups, restore drills, rollback, and disaster recovery
- Performance, scalability, privacy, data lifecycle, accessibility, localization, and SEO
- Dependency and software-supply-chain controls
- OWASP-aligned AI coding-assistant, tool, and prompt-context controls
- Data migrations, feature flags, phased platform releases, budgets, and incident learning

## Reference architecture

```text
Browser
  |
Cloudflare: DNS, TLS, edge cache, security controls
  |
Firebase Hosting: immutable release assets and SPA routing
  |
Flutter Web: presentation, domain, and data layers
  |
Repository interfaces
  |
Firebase: Auth, App Check, Firestore, Storage, Functions

Terraform -> repeatable cloud and edge configuration
CI/CD     -> quality gates, build, release stamping, deployment, verification
Telemetry -> logs, metrics, traces, events, objectives, alerts, investigation
Operations-> backups, restore, rollback, incidents, budgets, data changes
```

## Documentation

1. [Project journey](docs/01-project-journey.md)
2. [Architecture decisions](docs/02-architecture-decisions.md)
3. [Flutter web releases](docs/03-flutter-web-release.md)
4. [Firebase security](docs/04-firebase-security.md)
5. [Terraform management](docs/05-terraform-management.md)
6. [CI and quality gates](docs/06-ci-quality-gates.md)
7. [Monitoring and cost control](docs/07-monitoring-and-cost-control.md)
8. [Incidents and lessons](docs/08-incidents-and-lessons.md)
9. [Production checklist](docs/09-production-checklist.md)
10. [Cost-conscious engineering decisions](docs/10-cost-conscious-engineering.md)
11. [Observability](docs/11-observability.md)
12. [Reliability, backup, and recovery](docs/12-reliability-backup-and-recovery.md)
13. [Performance and scalability](docs/13-performance-and-scalability.md)
14. [Privacy and data lifecycle](docs/14-privacy-and-data-lifecycle.md)
15. [Dependencies and software supply chain](docs/15-dependencies-and-supply-chain.md)
16. [Accessibility, localization, and SEO](docs/16-accessibility-localization-and-seo.md)
17. [Platform release strategy](docs/17-platform-release-strategy.md)
18. [Data operations and feature flags](docs/18-data-operations-and-feature-flags.md)
19. [Technology selection: what was chosen and why](docs/19-technology-selection.md)
20. [Architecture evolution and lessons from the full history](docs/20-architecture-evolution-and-lessons.md)
21. [AI-assisted engineering workflow](docs/21-ai-assisted-engineering-workflow.md)
22. [Information security and compliance alignment](docs/22-infosec-and-compliance-alignment.md)

| UI, business rules, and Firebase become coupled | Feature-oriented layers and repository contracts | Testability and controlled vendor boundaries | [Architecture decisions](docs/02-architecture-decisions.md) |
| Local builds hide missing production inputs | Layered CI gates and explicit generation | Reproducibility in clean environments | [CI and quality gates](docs/06-ci-quality-gates.md) |
| Browsers use files from different releases | Release identity and differentiated cache policy | Safe upgrades without disabling useful caching | [Flutter web releases](docs/03-flutter-web-release.md) |
| Public clients can bypass UI behavior | Authentication, Rules, App Check, Functions, and IAM | No single control covers identity, authorization, and abuse | [Firebase security](docs/04-firebase-security.md) |
| Console configuration drifts | Incremental Terraform with protected remote state | Reviewability, recovery, and repeatability | [Terraform management](docs/05-terraform-management.md) |
| Usage-based services create uncertain cost | Focused monitoring, separate budgets, unit economics, and upgrade triggers | Keep consumption visible and decisions reversible | [Monitoring](docs/07-monitoring-and-cost-control.md) and [cost-conscious engineering](docs/10-cost-conscious-engineering.md) |
| Uptime is green while user journeys fail | Structured logs, metrics, traces, events, synthetic checks, and release correlation | Diagnose both predicted and unknown failure modes | [Observability](docs/11-observability.md) |
| Backup configuration creates false confidence | Define RPO/RTO and exercise isolated restoration and rollback | Recovery must be demonstrated, not assumed | [Reliability, backup, and recovery](docs/12-reliability-backup-and-recovery.md) |
| Performance and billed operations degrade with growth | Measure budgets and optimize the dominant constraint | Avoid both premature scale engineering and reactive firefighting | [Performance and scalability](docs/13-performance-and-scalability.md) |
| Telemetry and product features collect personal data | Inventory purpose, consent, access, retention, and deletion | Security alone does not provide privacy | [Privacy and data lifecycle](docs/14-privacy-and-data-lifecycle.md) |
| Trusted dependencies or CI can compromise artifacts | Pinned, bounded upgrades and protected build identities | Make supply-chain changes reviewable and reversible | [Dependencies and supply chain](docs/15-dependencies-and-supply-chain.md) |
| AI assistants ingest hostile context or misuse privileged tools | Untrusted-context handling, rules-file review, tool allowlists, sandboxing, and human approval | Bound prompt-injection, supply-chain, leakage, and excessive-agency risk | [AI-assisted engineering workflow](docs/21-ai-assisted-engineering-workflow.md) |
| A functional SPA remains inaccessible or undiscoverable | Shared accessibility/localization controls and route-specific SEO | Production quality includes inclusive access and public discovery | [Accessibility, localization, and SEO](docs/16-accessibility-localization-and-seo.md) |
| Shared Flutter code implies false cross-platform readiness | Separate web, Android, and iOS release gates | Signing, attestation, stores, devices, and rollback differ | [Platform release strategy](docs/17-platform-release-strategy.md) |
| Data scripts and flags change production outside releases | Versioned, idempotent, observable, reversible operations | Operational changes need the same discipline as code | [Data operations and feature flags](docs/18-data-operations-and-feature-flags.md) |
| A tool list does not explain architecture | Record alternatives, selection rationale, trade-offs, and exit triggers for every core technology | Keep the stack contextual and challengeable | [Technology selection](docs/19-technology-selection.md) |
| The final diagram hides failed and reversed choices | Preserve the evolution from custom backend through managed services, hardening, and operations | Make the case study evidence-based rather than hindsight-perfect | [Architecture evolution and lessons](docs/20-architecture-evolution-and-lessons.md) |

## Coverage model

The blueprint now treats the following as first-class production concerns:

- Product and architecture boundaries
- Application and data security
- Privacy and data lifecycle
- Release engineering and cache correctness
- Infrastructure as code and secrets/state management
- CI, testing, dependencies, and software supply chain
- Observability, reliability, backup, recovery, and incident response
- Performance, scalability, capacity, and cost control
- Accessibility, localization, SEO, and platform-specific delivery
- Data operations, feature flags, governance, and documented revisit triggers

No static guide can guarantee that nothing important is missing for every system. Use this coverage model with a project-specific threat model, data classification, dependency inventory, regulatory review, and pre-mortem to identify requirements.

## Guiding principles

- Treat production readiness as a system, not a final deployment step.
- **Main Task Priority:** Fulfill the primary request or incident resolution first. Layer additional optimizations, tests, and documentation only after the core objective is verified to manage system and team cognitive load.
- Put vendor SDKs behind application-owned boundaries.
- Keep secrets out of source control and Terraform state outputs.
- Make releases identifiable, reproducible, and reversible.
- Prefer explicit verification over assuming a successful deploy is healthy.
- Document failures as reusable engineering knowledge.

## Safe reuse

The examples are deliberately incomplete and use placeholders. Validate every control against your own threat model, legal requirements, budget, and platform documentation before production use.

## 💰 Support & Sustained Maintenance

This blueprint is open-source, but maintaining a production-grade reference architecture requires continuous verification. Your financial support directly funds the operational costs of maintaining this knowledge base:

* **Real-world Verification:** Covering the live Firebase, Cloudflare, and AWS/GCP infrastructure costs required to run synthetic tests, execute real restore drills, and generate actual telemetry data.
* **Keeping the Edge Sharp:** Upgrading the decision map as Flutter Web, Terraform providers, and Firebase security protocols evolve.
* **Zero Paywalls:** Keeping deep, architectural engineering knowledge free and accessible without lock-in or courses.

If this decision map saved your team days of architectural discovery, protected you from a Firebase billing spike, or hardened your release pipeline, consider supporting the project.

👉 [**Sponsor the Blueprint via Buy Me a Coffee**](https://www.buymeacoffee.com/amadu80z)

## License

Licensed under the [MIT License](LICENSE).
