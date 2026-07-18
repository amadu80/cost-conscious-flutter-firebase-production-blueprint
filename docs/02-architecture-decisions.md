# Architecture decisions

Architecture is a set of trade-offs made under real constraints. This blueprint assumes a small team, limited budget, uncertain early traffic, a Flutter client targeting the web first, and a need to reach production without creating a system that is expensive to operate or difficult to change.

Each decision below records the problem, credible options, the selected approach, the reasoning, and the consequences that must be managed. The choices are not universal recommendations; they should be revisited when the constraints change.

## ADR-001: Structure the Flutter application in layers

### Problem

Flutter makes it easy to call a backend SDK directly from a widget. That is productive at prototype stage, but business rules, state transitions, persistence, and UI behavior quickly become coupled. Tests then require platform plugins or Firebase initialization, and changing a data source affects screens throughout the application.

### Options considered

1. **Feature code with direct SDK calls in widgets:** fastest initial implementation, but tightly couples presentation to Firebase and makes deterministic testing difficult.
2. **Traditional technical layers:** separate all models, repositories, services, and screens into global folders. Boundaries are clear, but work on one feature is spread across the repository.
3. **Feature-oriented clean architecture:** organize code by feature while keeping presentation, domain, and data boundaries inside each feature. This adds interfaces and mapping code but localizes change.

### Decision

Use feature-oriented clean architecture with three responsibilities:

- **Presentation:** widgets, navigation, and UI state.
- **Domain:** application rules, entities, use cases, and repository contracts.
- **Data:** Firebase adapters, persistence, serialization, and repository implementations.

Dependencies point inward: presentation and data may depend on domain contracts; domain code does not depend on Flutter UI or Firebase SDK types.

### Why

The project needs fast delivery, but deterministic business logic and safe backend evolution matter more than minimizing the first few files. Feature ownership keeps related code discoverable, while dependency direction prevents vendor behavior from becoming application behavior.

### Consequences and mitigation

- More interfaces and mapping code must be maintained. Introduce them at meaningful boundaries, not for value objects with no behavior.
- Developers must decide where logic belongs. Document examples and enforce the direction through reviews and tests.
- Small features may initially look over-structured. Keep use cases focused and avoid speculative abstractions.

### Revisit when

Reconsider the amount of layering if the application remains a small read-only client, or introduce stronger module boundaries if independent teams begin owning features.

## ADR-002: Hide Firebase behind repository contracts

### Problem

Firebase Authentication, Firestore, Storage, and Functions expose convenient SDK types and asynchronous APIs. Allowing these types to cross the whole application makes Firebase difficult to mock, leaks transport concerns into UI code, and increases vendor lock-in.

### Options considered

1. **Use Firebase SDKs everywhere:** minimal adapter code, but high coupling and plugin-dependent tests.
2. **Create generic service wrappers:** hides imports but often reproduces the Firebase API without expressing application intent.
3. **Define domain-oriented repository contracts:** methods describe application operations and return application-owned models or typed failures.

### Decision

Define strict repository interfaces in the domain layer. Firebase implementations remain in the data layer and translate snapshots, SDK exceptions, and authentication state into application-owned types.

### Why

The boundary makes business logic testable without a browser, emulator, or network. It also creates one place to implement caching, retries, telemetry, authorization-aware queries, and future migrations. Domain-oriented methods such as `listVisibleItems` communicate more intent than generic collection access.

### Consequences and mitigation

- Mapping introduces code and the possibility of translation errors. Test serialization and error mapping at the adapter boundary.
- A repository can become too broad. Split contracts by domain responsibility rather than by Firebase product.
- Abstraction does not eliminate vendor lock-in in data models or security rules. Treat replaceability as reduced migration cost, not zero-cost portability.

### Revisit when

Split or redesign a repository when it gains unrelated responsibilities, produces N+1 access patterns, or cannot express a new backend operation atomically.

## ADR-003: Represent asynchronous UI states explicitly

### Problem

Repository-backed screens do not always have data. They initialize, refresh, become empty, fail, recover, or display cached data while revalidating. Treating these conditions as nullable values causes ambiguous UI and errors that appear only under slow or unreliable networks.

### Options considered

1. **Nullable data plus boolean flags:** simple initially, but permits invalid combinations such as `loading == true` with an unhandled error.
2. **Exceptions handled inside widgets:** flexible, but duplicates state interpretation and recovery behavior.
3. **Typed state variants:** model loading, data, empty, and error explicitly, optionally retaining stale data during refresh.

### Decision

Every screen consuming a repository must render deliberate loading, data, empty, and error states. State controllers expose typed transitions rather than unrelated nullable fields.

### Why

Explicit states make failure behavior part of the feature design and test surface. Users receive meaningful feedback, and developers can validate recovery paths without relying on timing accidents.

### Consequences and mitigation

- More UI branches and tests are required. Provide shared visual components while keeping feature-specific recovery actions local.
- Cached and refreshing states can multiply variants. Model only distinctions that change user behavior.

### Revisit when

Extend the state model when offline-first workflows, optimistic updates, or partial failures introduce user-visible behavior that existing variants cannot express safely.

## ADR-004: Use declarative routing for the web client

### Problem

A web application must support deep links, browser history, refreshes, authentication redirects, and direct navigation to nested content. An imperative mobile-style navigation stack does not naturally make the URL the source of truth.

### Options considered

1. **Imperative navigation:** familiar and direct, but difficult to reconcile with browser URLs and refresh behavior.
2. **Custom URL parsing:** complete control, but recreates routing, redirection, and restoration behavior.
3. **A declarative Flutter router:** routes derive application state from the URL and centralize guards and redirects.

### Decision

Use a declarative router, with route definitions and authentication redirects kept outside individual widgets. Configure hosting to return the application shell for valid client-side routes.

### Why

Declarative routing makes URLs shareable and refresh-safe while giving authentication and authorization transitions a consistent policy. It also creates a testable mapping between URL, application state, and visible page.

### Consequences and mitigation

- Redirect loops and race conditions can occur during authentication initialization. Model the unknown authentication state and test every redirect path.
- Hosting rewrites can hide missing static files if overly broad. Exclude assets and operational endpoints from SPA fallback rules.

### Revisit when

Reassess route ownership if the application adopts server-side rendering, independently deployed modules, or platform-specific navigation models.

## ADR-005: Use cursor pagination and controlled local caching

### Problem

Loading complete collections increases startup time, memory usage, and billed database reads. Offset pagination becomes slower and less consistent as data changes. Re-querying stable data wastes budget and makes weak connections more visible to users.

### Options considered

1. **Load all records:** simplest UI logic, but cost and latency grow with the dataset.
2. **Offset pagination:** familiar page numbers, but inefficient for Firestore and unstable under concurrent inserts or deletes.
3. **Cursor pagination with local persistence:** bounded reads and natural Firestore queries, with additional state needed for cursors, refresh, and cache freshness.

### Decision

Use indexed, cursor-based queries with explicit page sizes. Enable or implement local persistence for data whose freshness requirements permit it. Avoid N+1 queries by storing or retrieving the summary data required for list views in bounded operations.

### Why

The approach keeps latency and database cost proportional to what the user consumes. It matches Firestore query semantics and supports gradual loading on mobile and web connections.

### Consequences and mitigation

- Jumping to an arbitrary page is not naturally supported. Prefer continuous discovery or preserve known cursors for limited back-navigation.
- Cached content can become stale. Define freshness per feature and expose refresh behavior where it affects decisions.
- Denormalized summaries can drift. Update related records atomically or through idempotent repair processes.

### Revisit when

Consider a dedicated search or aggregation service when query requirements exceed Firestore indexes, full-text relevance becomes central, or read amplification remains high after query optimization.

## ADR-006: Start with managed Firebase services

### Problem

A small team needs authentication, persistence, file storage, hosting, and server-side operations without funding or staffing a platform team. However, convenience, variable pricing, and vendor concentration create long-term trade-offs.

### Options considered

1. **Self-managed backend and database:** maximum control, but fixed infrastructure and operational responsibilities begin immediately.
2. **A custom backend on managed compute:** flexible APIs and data models, but requires deployment, patching, scaling, and observability work.
3. **Firebase managed services:** low idle cost and fast delivery, with stronger platform coupling and usage-based billing.

### Decision

Use Firebase managed services for the initial product stage while keeping application boundaries vendor-neutral where practical. Treat security rules, indexes, quotas, and cost monitoring as architecture, not deployment details.

### Why

At uncertain early scale, engineering time and operational simplicity are the scarcest resources. Managed services allow the team to focus on product validation while paying primarily for actual use.

### Consequences and mitigation

- Costs can grow through inefficient access or abuse. Use App Check, authorization rules, bounded queries, budgets, quotas, and usage monitoring.
- Provider outages affect several capabilities simultaneously. Design clear degraded states and operational communication.
- Migration remains non-trivial. Keep domain models and repository contracts application-owned, and document the usage or compliance thresholds that would trigger reevaluation.

### Revisit when

Review the decision when sustained usage changes the cost model, compliance requires different controls, query limitations obstruct core features, or operational risk justifies a dedicated backend.

## Maintaining these decisions

For every future architectural decision, record:

- **Context:** the problem, constraints, and forces at the time.
- **Options:** credible alternatives, including doing nothing.
- **Decision:** the selected approach and its scope.
- **Why:** evidence and trade-offs that made it preferable.
- **Consequences:** benefits, new risks, and mitigation ownership.
- **Validation:** tests, measurements, or operational evidence.
- **Revisit triggers:** observable conditions that make the decision stale.

An ADR is a snapshot of reasoning, not a permanent rule. Supersede it explicitly when constraints change so future contributors understand both the old and new decisions.
