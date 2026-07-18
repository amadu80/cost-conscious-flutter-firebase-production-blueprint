# Terraform management

## Problem

Cloud and edge services are easy to configure manually through provider consoles. Over time, manual changes become undocumented, environments drift, recovery depends on memory, and reviewers cannot see the combined effect of DNS, IAM, budgets, hosting, and security changes.

The project needs repeatability and review without creating an infrastructure platform too complex for a small team.

## Options considered

1. **Manage everything manually:** lowest setup cost, but poor auditability, drift detection, and disaster recovery.
2. **Use scripts and provider CLIs:** automates sequences, but usually lacks declarative state, dependency planning, and reliable change previews.
3. **Adopt Terraform with extensive modules immediately:** reusable at scale, but creates abstraction and state complexity before patterns are stable.
4. **Adopt explicit Terraform incrementally:** codify high-risk and repeatable resources first, secure state, and introduce modules only when repetition is proven.

## Decision

Use Terraform incrementally for cloud and edge configuration. Keep early configurations explicit, pin tool and provider versions, store state remotely, review plans before apply, and assign ownership for every provider and resource group.

## Why

The primary value is not shorter configuration. It is the ability to explain an intended change before making it, reproduce known state, detect drift, and recover without relying on a specific operator's memory. An incremental approach captures this value without premature platform engineering.

## Scope and ownership

Prioritise resources whose manual drift has security, availability, or cost impact:

- DNS records and domain verification
- Hosting and edge-cache policies
- Security headers, rate controls, and abuse protections
- Service accounts and IAM bindings where supported safely
- Monitoring channels, service budgets, and alerts
- Firebase or cloud resources with stable provider support

Do not force unsupported or unstable resources into Terraform merely for completeness. Document the exception, manual owner, validation process, and migration trigger.

## State strategy

### Problem

State maps configuration to real infrastructure and may contain sensitive values. Local state can be lost, copied, or applied concurrently.

### Options

- Commit local state: convenient but unsafe and unsuitable for collaboration.
- Store untracked local state: avoids Git exposure but lacks shared locking and recovery.
- Use a protected remote backend: adds bootstrap work but supports access control, locking, versioning, and recovery.

### Decision and why

Use encrypted remote state with restricted access, versioning, and locking. Separate state when environments or ownership boundaries require independent blast radii. Never publish state, saved plans, real variable files, or secret outputs.

## Change workflow

1. Pin compatible Terraform and provider versions.
2. Format and validate configuration.
3. Generate a plan using the target environment's protected identity.
4. Review creates, updates, replacements, deletions, unknown values, and cost or security impact.
5. Apply the reviewed plan through a protected workflow.
6. Verify behaviour at the provider and public service boundary.
7. Retain audit evidence and monitor the change.

Production applies should not originate from an unrestricted personal workstation when protected CI or controlled automation is available.

## Importing existing resources

Before import, inventory the live resource, ownership, dependencies, and provider limitations. Write configuration that represents the intended state, import one bounded group at a time, and require a no-surprise plan before allowing Terraform to manage it.

Never resolve an unexpected destructive plan by applying it experimentally. Determine whether configuration, import identity, provider behaviour, or real drift is responsible.

## Surgical drift management

Not every perpetual plan is meaningful drift. Some provider APIs normalize an equivalent value after apply—for example, splitting or reformatting an SRV record—so Terraform repeatedly proposes a change that cannot converge. Diagnose this before suppressing it:

1. Capture the configuration, state, refreshed plan, provider version, and redacted API response.
2. Confirm that the remote representation is semantically equivalent and that re-applying does not converge.
3. Check the provider schema and current release notes; upgrade only through the normal controlled process.
4. Minimize the ignored surface to the exact computed or reformatted attribute on one resource.
5. Document why the drift is harmless, what risk becomes invisible, who owns it, and what event will remove the exception.

A narrow exception can use `lifecycle { ignore_changes = [...] }`, but it transfers responsibility from Terraform to a compensating control. Never ignore an entire resource merely to obtain a clean plan, and do not suppress security, routing, identity, encryption, deletion-protection, or cost-sensitive fields without explicit risk approval.

For an ignored SRV data structure, independently verify the public record's service, protocol, priority, weight, port, and target. Add a periodic check or runbook step because Terraform will no longer report changes to ignored attributes. Pin the provider version, link the upstream issue when available, and test removal of the exception during upgrades. A quiet plan is useful only when it still detects changes that matter.

## Consequences and mitigation

- Terraform adds state and provider upgrade responsibilities. Pin versions and schedule controlled upgrades.
- A mistaken plan can change many systems. Separate blast radii, review plans, and protect apply permissions.
- Provider schemas may lag platform capabilities. Document manual exceptions instead of using fragile workarounds.
- Excessive modules obscure simple resources. Extract a module only when a stable pattern repeats and has a clear interface.

## Revisit when

Change the state layout, module structure, or execution model when multiple teams need independent ownership, environments require stronger isolation, apply frequency rises, or compliance requires formal separation of duties.

## Validation evidence

The approach is working when a new operator can understand a plan, reconstruct infrastructure from protected state and code, detect an out-of-band change, and recover without undocumented console knowledge.
