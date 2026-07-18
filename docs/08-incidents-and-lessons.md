# Incidents and lessons

## Problem

Teams often document only the final fix. That loses the conditions that made failure possible, the signals that were absent, and the reasons earlier decisions appeared reasonable. Without a consistent pre-mortem and post-mortem practice, the same class of incident returns under a different symptom.

## Options considered

1. **Rely on informal discussion:** fast, but evidence and ownership disappear.
2. **Write post-mortems only for major outages:** focuses effort, but near misses and release blockers provide valuable learning before user harm.
3. **Use blame-focused reviews:** may identify an individual action but discourages reporting and ignores systemic contributors.
4. **Use risk-based pre-mortems and blameless, evidence-based post-mortems:** adds discipline while connecting prevention, detection, recovery, and ownership.

## Decision

Run a scoped pre-mortem before material releases or architecture changes. Produce a post-mortem for user-impacting incidents, security events, meaningful cost anomalies, and near misses that reveal a reusable failure class.

## Why

Pre-mortems make assumptions discussable before they fail. Post-mortems convert real evidence into controls. Blameless language encourages accuracy, but does not remove accountability for completing corrective work.

## Pre-mortem method

Assume the change has failed and ask how. Cover build configuration, generated files, routing, caches and service workers, authentication, App Check, security rules, data migration, third-party dependencies, infrastructure plans, monitoring, cost, and rollback.

For each scenario record:

- Failure and user impact
- Likelihood and severity
- Existing prevention
- Detection signal and expected delay
- Immediate mitigation and rollback
- Owner
- Evidence required before release

Prioritize scenarios with high impact, weak detection, or irreversible consequences. A pre-mortem is not complete when it merely lists risks; it must change a control, test, decision, or accepted-risk record.

## Post-mortem method

### Establish facts

Record impact, affected users or operations, start and end in UTC, detection source, and recovery. Build a timestamped timeline from logs, deployments, alerts, and communications. Separate verified facts from hypotheses.

### Analyze contributing factors

Look beyond the triggering action. Examine architecture, defaults, review, test coverage, generated configuration, permissions, observability, documentation, workload, and organizational incentives. Avoid a single “root cause” when several controls had to fail.

### Define corrective actions

Actions should be specific, owned, prioritized, and verifiable. Balance prevention with faster detection and safer recovery. “Be more careful” is not a control.

## Example failure class: generated configuration missing in CI

### Problem

A generated source file exists on developer machines but is ignored or never created in CI. Tests importing the application entry point fail before running, or release builds lack a required public configuration value.

### Options

- Commit the generated artifact.
- Generate it in CI from validated inputs.
- Avoid generation and read runtime configuration.

### Decision criteria

Commit only deterministic, non-sensitive artifacts whose presence is required across workflows. Otherwise generate them explicitly before analysis, testing, and build. Runtime configuration is appropriate only when hosting and application bootstrap can validate it reliably.

### Why this matters

The visible error is a missing file, but contributors may include undocumented setup, an incomplete CI path, and tests that compile more of the application than expected. The corrective action must address the class, not only restore one file.

## Consequences and mitigation

- Reviews consume time. Scope them by impact and use a consistent template.
- Blameless language can be mistaken for no accountability. Assign every action an owner and verification date.
- Documents can become archives without change. Review overdue actions and link completed work to evidence.
- Sensitive incidents may expose exploitable detail. Restrict distribution and publish only sanitized lessons.

## Revisit when

Increase formality when the team grows, on-call rotations emerge, regulatory reporting applies, or incident frequency shows that corrective actions are not reducing recurrence.

---

> [!TIP]
> **Help maintain this resource:** If the architectural lessons in these post-mortems saved your project from a production outage or billing spike, consider [sponsoring the blueprint's ongoing maintenance](https://github.com/sponsors/amadu80z).
