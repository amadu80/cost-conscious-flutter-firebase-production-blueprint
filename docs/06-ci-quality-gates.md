# CI and quality gates

## Problem

Local success is not reproducibility. Developer machines accumulate generated files, cached dependencies, authenticated sessions, and tool versions that hide missing build steps. A release can therefore pass unit tests yet fail to compile or initialize in clean CI and production environments.

## Options considered

1. **Rely on developer discipline:** low automation cost, but inconsistent and difficult to audit.
2. **Run tests only:** catches behavioural regressions but misses formatting, analysis, generated configuration, infrastructure, and production-build failures.
3. **Run every possible check on every change:** strong coverage, but slow feedback and unnecessary cost can cause teams to bypass CI.
4. **Use risk-based, layered gates:** fast deterministic checks on every change, with deeper integration, infrastructure, and release checks at appropriate boundaries.

## Decision

Use layered CI gates. Pull requests run formatting, static analysis, unit and widget tests, rule tests, and relevant infrastructure validation. Release candidates additionally generate configuration from declared inputs, compile the production target, stamp the release, and verify deployable artefacts.

## Why

The purpose of CI is to prove that the repository contains enough information to reproduce the intended result in a clean environment. Layering keeps fast feedback while reserving expensive checks for changes and stages that justify them.

## Baseline gates

```text
dart format --output=none --set-exit-if-changed .
flutter analyze
flutter test
terraform fmt -check
terraform validate
```

Commands are examples, not a complete policy. Add Firebase rule tests, dependency audits, generated-file checks, production compilation, or browser smoke tests according to the changed risk surface.

## Generated configuration

### Problem

A generated Dart file may exist locally and be absent from Git or CI. Tests importing the application entry point then fail before execution, or a production build silently receives an empty value.

### Decision and why

Choose one explicit ownership model for each generated artefact:

- Commit a safe, deterministic generated file when it contains no secret and must always exist; or
- Generate it in every workflow from validated environment input before analysis, tests, and builds.

Do not depend on a developer having run an undocumented command. Validate required release values early and use clear error messages.

## Test strategy

- Unit tests cover deterministic domain rules and failure mapping.
- Widget tests cover loading, data, empty, error, and recovery states.
- Emulator tests prove Firestore and Storage allow and deny behaviour.
- Integration or browser tests cover initialization, routing, authentication, App Check, and release upgrades.
- Infrastructure validation covers formatting, configuration validity, plans, and policy where justified.

Tests should be deterministic, isolated, and proportionate to the failure impact. Flaky tests are operational defects because they train contributors to ignore red builds.

## AI agents in CI

An AI-enabled workflow can become a confused deputy when untrusted pull-request content controls an agent that also holds repository, package, cloud, or deployment credentials. Separate analysis of untrusted changes from privileged mutation:

- Pull-request agents run without production secrets, write tokens, deployment identities, or access to protected runners.
- Treat source, diffs, issue text, comments, artefacts, and tool output as untrusted prompt-injection inputs.
- Require a trusted, reviewed event before any workflow gains write or deployment authority; do not make comment text an authorization mechanism.
- Pin agent actions, models where the provider supports it, MCP/tool definitions, and workflow dependencies. Review changes to assistant rules files and AI workflows through owned paths.
- Limit network egress, tool calls, runtime, retries, generated diff size, and accessible paths.
- Preserve logs of model/tool identity, approvals, changed files, commands, and outcome without recording prompts or outputs that contain secrets or personal data.
- Never allow an agent to merge, approve its own change, weaken required checks, or treat its generated tests as independent security review.

Validate this boundary with a harmless negative fixture: an untrusted change requests secret access or deployment, and the workflow proves that the request cannot obtain either.

## Consequences and mitigation

- More gates increase feedback time. Parallelize independent checks, cache safely, and run checks based on affected paths without omitting release-critical coverage.
- Toolchain drift causes inconsistent results. Pin versions and schedule upgrades deliberately.
- CI credentials expand attack surface. Prefer short-lived workload identity, least privilege, protected environments, and masked output.
- Generated artefacts can drift. Regenerate and verify them in CI or fail when committed output differs.

## Revisit when

Adjust gate placement when execution time delays delivery, flaky failures exceed an agreed threshold, platform releases expand, or incidents reveal an untested boundary.

## Validation evidence

A clean runner can analyse, test, build, and validate infrastructure using documented inputs; failures explain the missing requirement; and protected branches cannot bypass required release evidence without an audited exception.
