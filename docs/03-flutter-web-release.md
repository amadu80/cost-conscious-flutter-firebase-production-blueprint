# Flutter web release

## Problem

A Flutter web deployment can report success while users receive a broken mixture of releases. The HTML shell, service worker, manifest, JavaScript, and fingerprinted assets do not all have the same cache lifecycle. Build-time configuration can also be missing in CI even when local development succeeds.

The release process must therefore prove more than compilation: it must identify what was built, control how files are cached, verify the public result, and provide a recovery path.

## Options considered

1. **Run `flutter build web` and deploy manually:** simple, but dependent on workstation state, easy to misconfigure, and difficult to audit or reproduce.
2. **Disable all caching:** reduces some stale-file failures but increases latency and origin traffic, and does not solve service-worker or configuration mistakes.
3. **Use one immutable cache policy for all files:** efficient for fingerprinted assets but unsafe for HTML, manifests, release metadata, and service-worker control files.
4. **Automate a release-aware pipeline:** distinguish mutable from immutable files, inject configuration explicitly, stamp the build, verify it, and retain rollback instructions.

## Decision

Use an automated, release-aware pipeline with explicit preflight checks, deterministic configuration generation, quality gates, release stamping, differentiated cache rules, deployment verification, and rollback.

## Why

The highest-risk web failures happen at boundaries between build configuration, hosting, browser caches, and service workers. A single script or CI workflow makes these assumptions visible and ensures the same sequence is followed under pressure.

## Release sequence

### 1. Preflight

Confirm the branch, commit, semantic version, target environment, authenticated deployment identity, required tool versions, and required build-time values. Fail before compilation with an actionable message if a required value is absent.

### Build-time synchronization, not remembered flags

Values owned by infrastructure—such as a public reCAPTCHA/App Check site key—should flow from reviewed Terraform outputs into generated application configuration before compilation. Do not depend on a developer remembering a long `--dart-define` value: omission can produce a valid artefact whose protected backend calls fail at runtime.

Use a small, versioned generator in the release sequence:

```text
terraform output -json
        ↓ validate allowed, non-sensitive outputs and target environment
generate lib/config/app_config.g.dart atomically
        ↓ format and verify generated file
flutter build web
```

The generator should read structured output rather than scrape human-readable text, allowlist output names, validate non-empty values and expected formats, escape generated literals, write to a temporary file before replacement, and fail closed. It must never copy sensitive Terraform outputs into browser code or print values unnecessarily. Terraform state and generated secret material remain protected.

Commit the generator and a secret-free configuration schema. Choose deliberately whether the generated Dart file is committed: committing it makes drift reviewable but can create environment churn; generating it only in CI avoids that churn but requires a reproducibility check. In either case, add a test that runs generation from fixture outputs and confirms a missing required value stops the build. Stamp the artefact with the source commit and a hash of the non-sensitive configuration so deployment verification can prove what was baked in without revealing the value itself.

This synchronization must run before analysis, tests that import generated configuration, and compilation. The release workflow, not individual developers, owns the ordering.

### 2. Quality gates

Run formatting checks, static analysis, deterministic tests, security-rule tests, and infrastructure validation. Generated files needed by tests must be created explicitly rather than assumed to exist from a previous local run.

### 3. Production build

Build in release mode with declared configuration. Do not place server secrets in client build definitions: anything delivered to a browser must be considered observable.

### 4. Release identity

Create a small release metadata artefact containing non-sensitive values such as application version, commit identifier, and UTC build time. The application and support process should be able to report the deployed identity.

### 5. Cache policy

- Fingerprinted assets: long-lived and `immutable`.
- HTML shell, manifest, service worker, and release metadata: revalidate or use short-lived caching.
- Operational files such as security or crawler directives: define policy deliberately rather than inheriting a broad rule.

### 6. Controlled deployment

Deploy to a preview channel or controlled target when the platform permits. Review the exact artefact and hosting configuration being promoted.

### 7. Public verification

Check HTTP status, content type, cache headers, security headers, release identity, SPA deep links, authentication initialization, and protected backend access. Test both a clean browser profile and an upgrade path from the previous release.

Treat Content Security Policy as executable compatibility policy. A strict policy can block Flutter rendering backends, Wasm/CanvasKit resources, fonts, Firebase SDK endpoints, authentication redirects, or telemetry. Start from observed runtime dependencies, minimize origins and directives, test report-only changes where practical, and verify violations on the deployed origin. A header that breaks the application is not successful hardening.

### 8. Observation and rollback

Monitor initialization failures, client errors, authentication, App Check, backend errors, and traffic immediately after promotion. Roll back based on user impact and predefined thresholds, not optimism.

## Consequences and mitigation

- The pipeline becomes production-critical code. Keep it small, versioned, reviewed, and tested in a non-production target.
- Release stamping changes build output. Generate it deterministically and exclude secrets or personal data.
- Long-lived assets improve performance but make incorrect naming dangerous. Apply immutable caching only to content-addressed files.
- Service workers can preserve old behaviour. Test transitions between releases and document how to invalidate a faulty worker safely.

## Revisit when

Reassess the process when Flutter changes its web bootstrap or service-worker behaviour, hosting providers change cache semantics, server-side rendering is introduced, or release frequency requires progressive delivery and automated rollback.

## Validation evidence

A release is complete only when the public URL exposes the expected release identity, required routes load after refresh, headers match policy, protected services initialize successfully, and rollback has been exercised against the current hosting design.
