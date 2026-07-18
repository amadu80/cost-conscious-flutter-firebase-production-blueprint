# Architecture evolution and lessons from the full history

The architecture did not arrive fully formed. It changed across hundreds of commits as product needs, tests, platform constraints, security failures, and release evidence exposed weaknesses. This sanitized history records reversals and corrections, not only the final diagram.

## From a custom backend to managed services

The earliest backend direction used a conventional server framework, relational models, REST/OpenAPI, Docker, and a reverse proxy. It offered explicit API boundaries, relational queries, and infrastructure portability.

For a small team validating an uncertain product, the operational surface was disproportionate: application server, database, cache, queue, proxy, deployment, patching, credentials, backups, and monitoring all needed ownership before meaningful usage existed.

The project moved the initial backend to Firebase. This was not a conclusion that custom servers are inferior. It traded control for lower fixed operational work and faster product learning. A backend-for-frontend remains a credible future choice when authorization, query complexity, audit, rate limiting, compliance, or unit economics invalidate direct managed-service access.

## From direct implementation to application boundaries

Early Flutter features accumulated datastore access close to UI behavior. Mocks enabled development but could not prove provider behavior. Direct dependencies made repositories difficult to swap, SDK types leaked across layers, and failures were represented inconsistently.

The response was incremental:

- Move datastore access behind repository contracts.
- Add DTOs between persisted documents and domain models.
- Introduce typed results and application-owned failures.
- Add focused use cases where business policy justified them.
- Use emulators for provider-boundary tests while retaining fast domain fakes.

The lesson is that hardening worked when each abstraction answered an observed problem. “Cloud agnostic” means reducing the number of places that must change; it does not mean that Firestore queries, Rules, or data models are portable at negligible cost.

## Performance shaped the data model

Loading complete collections, favorites, or related content did not scale in latency, memory, or billed reads. Small fixtures hid inefficient access patterns.

The architecture adopted bounded reads, cursor pagination, repository-level filtering, cached state, summary projections, and performance-focused tests. Later work introduced source-level visibility filtering and metadata counters to avoid repeated aggregation.

Cost and performance became the same problem: in a consumption-based database, unnecessary operations create both latency and spend. Denormalization and caching required atomic updates, idempotent repair, and explicit freshness rules.

## Product complexity required policy controls

Authentication, social sign-in, professional profiles, ratings, messaging, admin tools, moderation, analytics, and remotely controlled features created policy that could not safely remain inside widgets.

The project added role permissions, protected actions, domain validators, typed feature flags, safe defaults, server-side privileged operations, consent-aware analytics, and tests covering UI/backend consistency.

Feature flags remained operational controls, not authorization. Reputation and moderation UI could imply trust that the system had not verified, so evidence provenance, human ownership, expiry, and appeal became architectural concerns.

## Web release behavior became architecture

Domain routing, refresh, service workers, cached assets, plugin registrants, CSP, fonts, CanvasKit/Wasm loading, authentication domains, and generated configuration caused failures that unit tests could not detect.

The response combined declarative routing, explicit SPA rewrites, differentiated cache policy, release identity, deterministic build configuration, and browser testing across clean and previous-release cache states. CSP became tested runtime policy rather than a copied security header.

The lesson is direct: a Flutter web build is not a release. The public origin, browser policy, service worker, edge cache, SDKs, and generated plugin state form one runtime system.

## A credential incident changed secret management

A credential entered a local or tracked workflow. Ignore rules and history cleanup were necessary, but prevention based only on `.gitignore` had already failed.

Backend credentials moved toward managed secrets, local overrides were excluded, and least privilege, rotation, Security Rules tests, public/private data separation, and App Check received more attention.

Secret scanning and ignore rules are preventive controls, not complete remediation. Response requires revocation or rotation, exposure assessment, history analysis, access review, and proof that the old credential no longer works.

## Infrastructure moved from consoles to Terraform

Cloud resources initially evolved through consoles and CLIs. This accelerated exploration but created incomplete ownership, regional mismatch, and recovery dependence on operator knowledge.

Terraform was adopted incrementally; stable resources were imported, state moved remotely, Functions moved to the second-generation runtime, and budgets and monitoring policies were added.

Regional configuration changed when service compatibility invalidated an earlier choice, and data restoration was required during database placement changes. This is important evidence: a region change is a data migration and recovery event, not a variable edit. Infrastructure decisions are constrained by the intersection of services and providers.

## Observability expanded beyond product analytics

Product analytics and crash signals arrived before a complete operating model. Provider dashboards could not guarantee correlation across client initialization, repositories, Auth, App Check, Rules, Functions, storage, edge behavior, and release identity. Some desired provider metrics were unavailable or unsuitable for direct alerts.

The design expanded toward structured events, client crash/performance signals, latency/error/usage policies, budgets, release correlation, and synthetic checks for gaps platform metrics could not express.

Instrumenting SDKs is not observability. The evidence standard is whether a controlled failure can be detected, correlated, diagnosed, mitigated, and verified within an acceptable time without exposing private data.

## Canonical data and lifecycle controls

Taxonomy, geography, feature defaults, messaging configuration, and imported records could drift across client, backend, scripts, and database. Logs, one-time verification records, analytics, storage objects, and backups could accumulate without lifecycle ownership.

The project introduced canonical manifests with generated runtime projections, idempotent imports, controlled baseline restoration, TTL for ephemeral documents, storage lifecycle transitions, and an infrastructure/cost inventory.

A restoration script is not a backup, and a generated file is not a source of truth unless CI verifies it. Lifecycle policy must include purpose, retention, recovery impact, cost, legal requirements, and deletion behavior.

## Managed AI was constrained behind trusted boundaries

AI-assisted image analysis and structured input offered value, but client credentials, probabilistic output, SDK churn, privacy, latency, and variable invocation cost created risk.

Model access moved toward trusted server functions, with deterministic validation remaining authoritative, structured output validation, bounded invocation, and explicit failure handling. AI can assist classification or moderation; it must not silently become the sole safety, identity, or enforcement decision-maker.

## What the history proves—and does not prove

The history proves sustained iteration, growing test coverage, architectural corrections, infrastructure codification, and learning from concrete failures. It does not by itself prove production security, availability, recovery, performance, accessibility, privacy compliance, or low cost.

Commit count and test count are activity measures. Stronger evidence includes denied-path tests, public release traceability, alert exercises, restore timing, performance and cost baselines, accessibility review, and reduced recurrence of incident classes.

## Reusable decision pattern

1. State the observed problem and constraint.
2. Preserve credible alternatives, including doing nothing.
3. Select the smallest control that reduces the named risk.
4. Define evidence before implementation.
5. Record operational and migration cost.
6. Exercise failure and rollback.
7. Set a threshold that invalidates the choice.
8. Keep reversals in the history; they are evidence of learning, not embarrassment.
