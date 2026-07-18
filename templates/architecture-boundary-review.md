# Architecture boundary review

- **System/release:**
- **Owner:**
- **Reviewers:**
- **Review date (UTC):**
- **Composition-root location:**

## Allowed dependency matrix

Use `allowed`, `forbidden`, or `composition only`.

| From / to | Domain | Application | Data adapters | Presentation | Vendor SDKs |
| --- | --- | --- | --- | --- | --- |
| Domain | allowed | forbidden | forbidden | forbidden | forbidden |
| Application | allowed | allowed | forbidden | forbidden | forbidden |
| Data adapters | allowed | allowed if required | allowed | forbidden | allowed |
| Presentation | allowed | allowed | forbidden | allowed | forbidden |
| Composition root | allowed | allowed | composition only | allowed | composition only |

Document every exception with owner, rationale, expiry, and removal trigger.

## Package graph

Attach the generated package/import graph and record:

| Criterion | Target | Actual | Result |
| --- | ---: | ---: | --- |
| Firebase imports outside data/composition | 0 |  |  |
| Flutter imports inside domain | 0 |  |  |
| Presentation imports from concrete data | 0 |  |  |
| Domain tests requiring Flutter/Firebase/emulator/network | 0 |  |  |
| Undeclared package dependencies | 0 |  |  |

## Enforcement

- Analyzer/import graph command:
- CI workflow and required gate:
- Rule configuration location:
- Failure message shown to contributors:

## Negative control

Deliberately add one forbidden import in an isolated fixture or test of the checker.

- Violation introduced:
- Expected gate failure:
- Actual failure output:
- Exit code:
- Fixture removed/restored:
- Reviewer:

A boundary checker without a negative control is an assertion about the checker, not evidence.

## Change-impact experiment

### Experiment A: replace a repository adapter

| Measure | Before | After |
| --- | ---: | ---: |
| Files changed outside data/composition |  |  |
| UI/domain files changed |  |  |
| Test setup time |  |  |
| Platform/emulator required |  |  |
| Test duration |  |  |

### Experiment B: change persistence error shape

Record whether vendor exceptions remain inside the adapter and whether application/UI behavior changes through application-owned failures only.

## Ceremony review

For every interface, use case, mapper, and layer, identify at least one:

- Business policy
- Vendor/persistence translation
- Volatile implementation boundary
- Atomicity or orchestration responsibility
- Meaningful test isolation

List forwarding-only abstractions to remove or justify.

## Decision

- **Boundary status:** enforced / partially enforced / asserted only
- **Observed benefit:**
- **Unnecessary ceremony:**
- **Actions, owners, deadlines:**
- **Residual risk and revisit trigger:**
