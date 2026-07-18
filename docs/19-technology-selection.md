# Technology selection: what was chosen and why

This chapter records the reference stack as a set of decisions, not a list of fashionable tools. Each choice was made for a small team, constrained initial budget, uncertain traffic, web-first release, future mobile delivery, and limited platform-operations capacity.

The decisions are contextual. The “revisit when” criteria are as important as the original rationale.

## Cost-conscious selection rule

No technology was chosen solely because it was free or inexpensive at initial traffic. Each option was evaluated across:

- Fixed and usage-based provider cost
- Engineering effort to build and change it
- Operational effort to secure, observe, recover, and upgrade it
- Failure and abuse cost, including unbounded consumption
- Complexity introduced across providers and team boundaries
- Migration cost and the ability to exit before lock-in becomes urgent

Managed services were favored where they reduce scarce operational work without hiding unacceptable variable cost or risk. Open-source and self-managed alternatives were not assumed to be cheaper: infrastructure, patching, backups, observability, on-call time, and specialist knowledge are part of total cost.

Every choice therefore includes a revisit trigger. A cost-conscious decision is temporary when workload, pricing, compliance, incident evidence, or team capacity invalidates the assumptions that justified it.

## Decision summary

| Area | Chosen solution | Primary reason | Main cost or risk |
| --- | --- | --- | --- |
| Client | Flutter and Dart | One product codebase with future Android/iOS reuse | Web payload, semantics, SEO, and platform-specific work |
| State and dependency management | Riverpod-style provider architecture | Explicit, testable dependency composition and async state | Provider graph and lifecycle complexity |
| Routing | Declarative routing | Deep links, refresh-safe URLs, and centralized redirects | Redirect races and hosting rewrite coordination |
| Backend platform | Firebase | Low idle cost and low operational burden for a small team | Vendor concentration and usage-based pricing |
| Primary database | Cloud Firestore | Realtime client access, offline support, and managed scaling | Query constraints, denormalization, rules complexity, billed operations |
| Trusted backend logic | Cloud Functions | Privileged, event-driven operations without server management | Cold starts, retries, runtime constraints, and per-use cost |
| Identity | Firebase Authentication | Managed identity flows integrated with Firebase controls | Provider dependence and limited customization |
| Client attestation | Firebase App Check | Reduce abuse from unofficial clients | Not authorization; rollout can block legitimate clients |
| Static delivery | Firebase Hosting | Direct integration with Flutter artifacts and Firebase tooling | Cache correctness and platform coupling |
| Edge, DNS, and security controls | Cloudflare | DNS, TLS, caching, headers, and abuse controls in one edge layer | Second control plane and cache/debugging complexity |
| Infrastructure as code | Terraform | Reviewable plans, repeatability, drift visibility, remote state | Provider gaps, state risk, and maintenance overhead |
| CI | GitHub Actions | Repository-integrated quality gates with low initial overhead | Hosted-runner trust, action supply chain, and usage limits |
| Observability | Firebase and Google Cloud native telemetry | Integrated signals without another fixed-cost platform | Fragmented investigation and limited end-to-end tracing |

## Flutter and Dart

### Problem

The product needs a responsive web application first and expects Android and iOS later. A small team cannot comfortably maintain three independent feature implementations.

### Options considered

- React, Vue, or another web-first framework plus separate mobile applications
- React Native or another cross-platform mobile stack with a web target
- Native Kotlin, Swift, and a separate web client
- Flutter and Dart across web and mobile

### Decision

Choose Flutter and Dart for shared presentation, domain logic, models, validation, and tests.

### Why

Flutter offers high code reuse, a consistent component model, strong typing, mature Firebase integration, and a direct path to later mobile clients. For an interactive application, these benefits reduce feature-delivery and testing cost.

### Trade-offs

Flutter web can have a larger initial payload and less natural HTML semantics than a conventional web framework. SEO, accessibility, text behavior, browser integration, and low-end device performance need explicit validation. Mobile still requires native signing, permissions, attestation, stores, and device testing.

### Revisit when

Adopt server-rendered or conventional web surfaces when public discovery, crawlability, initial-load performance, or native semantics cannot meet measured requirements. A hybrid architecture may preserve Flutter for authenticated workflows while serving public pages separately.

## Riverpod-style state and dependency management

### Problem

Repository calls, authentication, feature flags, navigation, cached data, and asynchronous UI state need predictable composition and test overrides.

### Options considered

- Stateful widgets and inherited state only
- BLoC/Cubit event-state architecture
- Provider/Riverpod dependency and state graph
- A service locator with manually managed state

### Decision and why

Use a Riverpod-style provider architecture because dependencies remain explicit, asynchronous state can be typed, and tests can replace repositories without global mutable service locators. It fits incremental feature development without requiring every interaction to become an event class.

### Trade-offs and revisit trigger

Provider graphs can produce lifecycle surprises, circular dependencies, or hidden recomputation. Keep providers focused, avoid business rules inside composition code, and test initialization. Revisit if state flows require stronger event auditability or the graph becomes difficult to reason about.

## Declarative routing

### Problem

Web routes must survive refresh, support deep links and browser history, and coordinate authentication redirects.

### Options considered

- Imperative `Navigator` calls
- Custom URL parsing and state restoration
- A declarative router such as `go_router`

### Decision and why

Choose declarative routing because URL, authentication state, and visible page can be mapped and tested centrally. It also supports consistent deep links across future platforms.

### Trade-offs and revisit trigger

Authentication initialization can create redirect loops, and hosting must distinguish client routes from missing assets. Revisit if server rendering, independently deployed modules, or platform-specific navigation makes one shared route graph counterproductive.

## Firebase as the managed backend platform

### Problem

The project needs authentication, database, file storage, hosting, messaging, trusted backend operations, and telemetry without funding a platform team or paying for idle servers.

### Options considered

- A custom REST or GraphQL backend on managed compute
- Supabase or another backend-as-a-service
- Self-managed services and database
- Firebase managed services

### Decision

Choose Firebase for the initial stage, while placing SDK access behind application-owned repository contracts.

### Why

Firebase reduces initial operational work, provides consumption-based services, integrates well with Flutter, and lets the team focus on product validation. Authentication, realtime data, offline persistence, storage, functions, hosting, App Check, and telemetry share one ecosystem.

### Trade-offs

The choice concentrates availability, pricing, SDK, query, and security-rule risk in one provider. Usage-based billing can amplify inefficient queries or abuse. Firestore data models and Rules are not trivially portable even when Dart code uses repository interfaces.

### Revisit when

Review the platform when sustained usage changes unit economics, required queries exceed Firestore's model, compliance or residency changes, privileged workflows dominate direct client access, or provider concentration exceeds business risk tolerance.

### Historical alternative: custom containerized API

The project initially explored a conventional server backend with relational models, REST/OpenAPI, Docker, and a reverse proxy. That offered explicit server authorization, relational queries, and portability, but also required ownership of compute, database, cache, queue, proxy, deployment, patching, backups, and observability before meaningful usage existed.

Firebase was selected because team time and operating complexity cost more than the added control at that stage. A custom API remains a credible exit when query, audit, rate-limit, compliance, or unit-economic thresholds invalidate direct managed-service access.

## Cloud Firestore

### Problem

The client needs responsive structured data access, realtime updates in selected flows, offline behavior, and a low-operations database.

### Options considered

- A relational database behind a custom API
- Firebase Realtime Database
- A search engine as the primary store
- Cloud Firestore with explicit indexes and Rules

### Decision and why

Choose Firestore as the system of record for application documents. Its Flutter SDK, realtime listeners, offline support, managed scaling, and Rules-based direct client access fit the initial team and interaction model.

### Trade-offs and revisit trigger

Joins, full-text search, complex aggregation, and arbitrary pagination are limited. Denormalization increases repair complexity; reads and listeners directly affect cost. Use cursor pagination, bounded queries, summary projections, and idempotent repair. Add relational, search, or analytical services when measured requirements justify them rather than forcing Firestore to solve every data problem.

## Cloud Functions

### Problem

Secrets, administrative actions, cross-document invariants, notifications, scheduled work, and untrusted inputs require a trusted execution boundary.

### Options considered

- Perform logic entirely in clients and Rules
- Run a persistent custom backend
- Use serverless Cloud Functions triggered by calls, events, or schedules

### Decision and why

Choose Cloud Functions for bounded privileged and event-driven workloads. They integrate with Firebase identity and events while avoiding idle server management.

### Trade-offs and revisit trigger

Retries, cold starts, timeouts, concurrency, and event duplication require idempotency and monitoring. Move sustained, latency-critical, long-running, or high-throughput workloads to a more suitable managed compute or queue architecture when measurements cross defined thresholds.

### Runtime generation decision

Use second-generation Functions for supported workloads because the Cloud Run-based runtime offers concurrency and modern event infrastructure. This can reduce idle instances and improve efficiency for bursty work, but concurrency can amplify memory contention, connection pressure, duplicate effects, and downstream load. Measure cold and warm latency, memory, retries, and cost before tuning it.

## Firebase Authentication and App Check

### Problem

The backend must distinguish users, enforce ownership, support account lifecycle, and reduce automated access from unofficial clients.

### Options considered

- Build identity and sessions internally
- Use an independent identity provider
- Use Firebase Authentication integrated with Security Rules
- Add no attestation, or add Firebase App Check alongside identity

### Decision and why

Choose Firebase Authentication for managed identity and App Check as an additional abuse-resistance signal. Integration with Flutter, Rules, and Functions reduces custom security-sensitive code.

### Trade-offs and revisit trigger

Authentication does not authorize individual resources, and App Check can be bypassed or reject legitimate clients when misconfigured. Use Rules, trusted server validation, staged enforcement, and recovery processes. Revisit identity architecture when enterprise federation, regulatory, account-recovery, or portability requirements exceed the platform.

## Firebase Remote Config for feature delivery

### Problem and options

Sensitive features need staged activation or a kill switch without a client release. Compile-time constants require redeployment; generic database documents require custom caching and validation; a dedicated control plane adds operations.

### Decision and why

Choose Remote Config for remotely controlled product flags, with a typed, versioned application-owned model and tested safe defaults. It integrates with the client platform without another backend.

### Trade-offs and revisit trigger

Remote values can be stale and diverge from backend policy. They are never authorization. Revisit when approvals, audit history, cross-service consistency, or real-time rollout requires a dedicated control plane.

## Firebase Hosting

### Problem

The Flutter web bundle needs TLS, CDN delivery, SPA rewrites, deployment history, and integration with Firebase environments.

### Options considered

- Object storage plus CDN
- A general static host
- Firebase App Hosting or server-rendered hosting
- Firebase Hosting for static Flutter assets

### Decision and why

Choose Firebase Hosting because the output is static, deployment is simple, preview and rollback concepts are available, and integration with the existing Firebase project reduces initial operational overhead.

### Trade-offs and revisit trigger

Correct caching and service-worker transitions remain the application's responsibility. Revisit when server rendering, advanced progressive delivery, custom edge compute, or hosting economics require another delivery platform.

## Cloudflare for DNS and edge controls

### Problem

The public application needs managed DNS, TLS policy, cache control, security headers, crawler controls, rate or abuse protections, and visibility at the edge. Origin-only delivery may produce unnecessary traffic and fewer control points.

### Options considered

- Use Firebase Hosting and registrar DNS only
- Use Google Cloud CDN or load-balancing products
- Use another CDN/security provider
- Place Cloudflare in front of Firebase Hosting and storage delivery where appropriate

### Decision

Choose Cloudflare as the public DNS and edge-control layer, while Firebase remains the application origin.

### Why

Cloudflare combines low-cost DNS, TLS, caching, headers, and security controls with Terraform support. It can reduce origin traffic and provide controls without deploying a custom proxy tier.

### Trade-offs

It creates a second control plane. Cache keys, duplicated headers, certificate behavior, DNS proxying, provider outages, and debugging become more complex. Edge caching must never expose private or user-specific content.

### Revisit when

Remove or replace the layer when measured benefit does not justify complexity, provider features require a paid tier beyond budget, origin integration becomes unreliable, or a single-cloud design better meets operational and compliance needs.

## Terraform

### Problem

Firebase, Google Cloud, and Cloudflare console changes can drift, lack peer review, and depend on one operator's memory.

### Options considered

- Manual console ownership
- Shell scripts and provider CLIs
- Provider-native deployment templates
- Terraform across supported cloud and edge resources

### Decision and why

Choose Terraform because it produces reviewable plans across providers, supports dependency ordering and remote state, and makes stable infrastructure intent repeatable. Use it incrementally rather than forcing unsupported resources into fragile automation.

### Trade-offs and revisit trigger

State becomes sensitive operational infrastructure, provider support can lag, and partial coverage creates split ownership. Maintain a resource inventory and manual-exception register. Revisit the tool or module design when provider gaps, team scale, state blast radius, or organizational standards require another approach.

## GitHub Actions

### Problem

Quality gates, clean builds, infrastructure validation, and release evidence must run outside developer workstations.

### Options considered

- Local scripts only
- Another hosted CI provider
- Self-hosted runners
- GitHub Actions integrated with repository protections

### Decision and why

Choose GitHub Actions for low setup cost, pull-request integration, reusable workflows, and protected environments. Keep release logic in repository scripts where possible so it remains testable outside one CI vendor.

### Trade-offs and revisit trigger

Hosted actions and runners are supply-chain dependencies; usage and concurrency have limits. Pin trusted actions, prefer short-lived cloud identity, minimize secrets, and preserve artifact provenance. Revisit when compliance, performance, private networking, cost, or runner trust requires controlled infrastructure.

## Firebase Emulator Suite for integration boundaries

Pure mocks are fast but reproduce assumptions rather than provider behavior. Shared cloud test projects are realistic but slower, stateful, costly, and risky. Choose the Firebase Emulator Suite for Rules and provider-boundary tests while retaining fakes for deterministic domain tests.

Emulators do not reproduce every production quota, IAM, App Check, latency, index, or browser behavior. Hosted smoke tests remain necessary. Revisit the balance when emulator divergence causes repeated escaped defects.

## Regional deployment and data placement

Database, storage, functions, event infrastructure, and extensions must use compatible locations. Multi-region placement can improve some availability characteristics but costs more and may conflict with dependent services; one region simplifies alignment and can reduce latency and egress but concentrates failure.

Choose one compatible region initially and align stateful services and compute where possible. This prioritizes predictable compatibility and cost over unmeasured redundancy. Reconsider placement when availability objectives, legal requirements, user geography, or business impact justify migration cost and added complexity. Treat every region change as a data migration and recovery exercise, not a variable edit.

## Managed secrets and configuration

Ignore rules cannot protect a secret already committed or exposed. Use managed secret storage for backend credentials, short-lived workload identity where supported, and protected CI variables only for necessary bootstrap. Rotation, history analysis, least privilege, and exposure assessment are part of the design.

Secret services add IAM, versioning, availability, and audit responsibilities. Revisit when multi-environment rotation, external synchronization, or compliance requires a broader secrets platform.

## Managed generative AI for bounded assistance

Deterministic rules remain the first control. Use managed generative AI only for bounded assistance through trusted server functions; never expose model credentials in clients. Cap input and invocation, validate structured output, handle failure explicitly, and keep a human or deterministic authority for safety, identity, moderation, and irreversible decisions.

Managed inference adds probabilistic errors, privacy questions, SDK churn, latency, and variable cost. Disable or replace it when measured value does not exceed those costs and risks.

## Canonical source plus generated projections

Shared taxonomy or geography drifts when copied into client code, backend scripts, fixtures, and database baselines. Keep one versioned canonical manifest and generate typed projections for each runtime. CI should fail on unexpected generated diffs.

Generation adds tooling and committed artifacts can become stale. Revisit when controlled non-technical editing, real-time updates, or schema governance requires a dedicated data service.

## Native Firebase and Google Cloud observability

### Problem

The team needs crash, client performance, backend logs, errors, metrics, alerts, and cost signals without another large fixed-cost platform.

### Options considered

- Provider consoles with no designed telemetry
- A commercial all-in-one observability platform
- OpenTelemetry plus a self-managed backend
- Firebase and Google Cloud native telemetry with targeted synthetic checks

### Decision and why

Choose native telemetry initially because it integrates with the selected runtime and keeps fixed cost and operational work low. Add release correlation, structured events, synthetic probes, and owned alerts to close the most important gaps.

### Trade-offs and revisit trigger

Signals remain fragmented and end-to-end traces may be incomplete. Adopt OpenTelemetry or a consolidated platform when incident diagnosis remains slow, cross-service correlation is unreliable, retention or compliance changes, or on-call objectives justify the cost.

## Overall decision rule

The stack is coherent because each chosen tool reduces initial operational burden or enables shared delivery. Coherence is not permanence. Review the stack using measured user outcomes, cost per journey, incident evidence, provider limits, team capacity, and the exit triggers above. Do not defend a technology after its original constraint has disappeared.
