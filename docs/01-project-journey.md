# Project journey

This journey describes how a small, budget-constrained team can move a Flutter web application from prototype to a defensible production service. It is organized around risks discovered and decisions made, not a claim that every project must adopt the same sequence.

## Starting problem and constraints

The prototype proved that Flutter and Firebase could deliver user value quickly, but it did not answer production questions:

- Could changes be tested without initializing Firebase or a browser?
- Would direct links, refreshes, and authentication redirects work on the web?
- Could unofficial clients or abusive traffic reach paid backend services?
- Would a new release coexist safely with cached files from an older release?
- Could infrastructure be reproduced after an accidental or undocumented console change?
- Would the team detect failures and cost growth before users or invoices exposed them?

The constraints were a small team, low initial budget, uncertain traffic, limited operational capacity, and a need to release the web application before later mobile releases.

## Options for reaching production

### Release the prototype and fix problems reactively

This provides the shortest path to launch. It was rejected because failures involving authorisation, build configuration, cache incompatibility, or uncontrolled spend are expensive to diagnose after public exposure.

### Build a comprehensive platform before launch

This can create strong controls, but it delays learning, adds fixed cost, and risks optimizing for hypothetical scale. It was rejected as disproportionate to the project's maturity.

### Harden the system in risk-prioritised stages

This adds controls incrementally, beginning with failures that could block release, expose data, or create unbounded cost. It preserves delivery speed while requiring evidence at each stage.

## Decision

Adopt staged production hardening. Each stage must address a named risk, include a validation method, and leave a documented trigger for further investment.

## Why

The staged approach matches spending and engineering effort to actual risk. It avoids treating production readiness as a single deployment command and avoids building a platform whose operational cost exceeds the application it supports.

## Stage 1: Establish application boundaries

The first decision was to separate presentation, domain rules, and Firebase adapters. Repository contracts made deterministic logic testable without vendor SDKs and gave loading, data, empty, and error states explicit ownership.

**Evidence:** unit tests exercise business rules using fakes or mocks; UI tests cover each repository state; Firebase types remain inside data implementations.

## Stage 2: Make the production build reproducible

Required environment values and generated configuration must be created explicitly in local and CI builds. Formatting, analysis, tests, and a release build become quality gates rather than optional developer habits.

**Evidence:** a clean CI environment can generate configuration and build the same commit without relying on untracked workstation files.

## Stage 3: Protect backend boundaries

Authentication identifies the caller, security rules authorize each operation, and App Check reduces traffic from unofficial clients. These controls are layered because none is sufficient alone.

**Evidence:** emulator tests prove allowed and denied paths; App Check metrics show valid clients before enforcement; least-privilege IAM is reviewed independently.

## Stage 4: Codify infrastructure

Cloud and edge configuration move from manual consoles into Terraform. Remote state, provider version pinning, plan review, and ownership reduce drift and make recovery repeatable.

**Evidence:** a reviewed plan explains every intended change, drift checks expose out-of-band changes, and no sensitive state is committed.

## Stage 5: Make releases observable and cache-safe

Every deployment receives a release identity. Mutable control files revalidate, fingerprinted assets cache for longer, and browser verification covers clean sessions and upgrades from the previous release.

**Evidence:** the deployed version can be identified externally; deep links and refreshes work; consecutive releases do not produce missing assets or incompatible service-worker state.

## Stage 6: Operate within risk and budget

Monitoring focuses on signals that trigger action: availability, initialisation, authorisation failures, backend errors, latency, usage anomalies, and service-specific budgets. Pre-mortems anticipate failure; post-mortems turn real incidents into controls.

**Evidence:** each alert has an owner and response; thresholds are tested; corrective actions are tracked to completion.

## Consequences and mitigation

- Staged hardening can leave known gaps temporarily. Record each gap, owner, impact, and deadline or upgrade trigger.
- Quality gates slow individual merges. Keep them deterministic and fast enough to run continuously.
- Managed services reduce operational work but concentrate vendor and pricing risk. Preserve application boundaries and review cost thresholds.
- Documentation can drift from implementation. Treat operational guides as release artefacts and verify them during exercises.

## Revisit when

Change the sequence or depth when compliance requirements increase, the team grows, mobile platforms enter release scope, sustained traffic changes the cost model, or recovery objectives demand stronger redundancy.

The transferable lesson is not the exact toolchain. It is the discipline of connecting every engineering investment to a concrete problem, an explicit decision, validation evidence, and a condition for revisiting it.
