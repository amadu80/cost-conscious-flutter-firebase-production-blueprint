# Release provenance and rollback evidence

## Identity

- **Version/release ID:**
- **Source commit:**
- **UTC build time:**
- **Toolchain versions:**
- **Configuration hash (non-secret):**
- **Artifact checksum:**
- **CI run:**
- **Target:**
- **Operator/approver:**

## Gate results

| Gate | Command or exercise | Result | Evidence |
| --- | --- | --- | --- |
| Format/analyze/test |  |  |  |
| Rules allow/deny |  |  |  |
| Infrastructure plan |  |  |  |
| Clean production build |  |  |  |
| Preview hosted smoke |  |  |  |
| Previous-release cache upgrade |  |  |  |

## Promotion

- Confirm the promoted bytes match the tested checksum.
- Record public release identity and deployment ID.
- Record cache and security headers observed from the public origin.

## Rollback exercise

- **Trigger:**
- **Known-good artefact/deployment:**
- **Steps:**
- **Start/end UTC:**
- **Data/schema compatibility result:**
- **Public verification:**
- **Observed gaps:**
- **Residual risk and owner:**
