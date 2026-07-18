# Evidence templates

Templates help collect evidence; they are not evidence by themselves. Copy a template into an `evidence/` area, add a UTC date and release identifier, perform the exercise, attach sanitized outputs, record failures, and obtain review.

| Gap | Template |
| --- | --- |
| Architecture boundaries and coupling evidence | [architecture-boundary-review.md](architecture-boundary-review.md) |
| System context and threat model | [threat-model.md](threat-model.md) |
| Release provenance and rollback | [release-evidence.md](release-evidence.md) |
| SLIs, SLOs, alerts, and diagnosis | [observability-exercise.md](observability-exercise.md) |
| Backup restoration and recovery | [restore-drill.md](restore-drill.md) |
| Privacy inventory and lifecycle | [data-inventory.md](data-inventory.md) |
| Terraform and manual ownership | [infrastructure-inventory.md](infrastructure-inventory.md) |
| Performance, capacity, and cost | [performance-cost-baseline.md](performance-cost-baseline.md) |
| Accessibility, localization, and discovery | [accessibility-web-review.md](accessibility-web-review.md) |
| Dependencies and build provenance | [supply-chain-review.md](supply-chain-review.md) |
| Deferred or accepted gaps | [accepted-risk.md](accepted-risk.md) |

Never publish credentials, tokens, production IDs, personal data, private paths, exploitable thresholds, or Terraform state with an evidence pack.
