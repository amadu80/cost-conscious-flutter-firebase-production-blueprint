# Information security and compliance alignment

## Purpose

A production-grade system must be defensible against recognized industry standards. This chapter provides a crosswalk between the blueprint's technical decisions and the outcomes required by the **NIST Cybersecurity Framework (CSF) 2.0** and **ISO/IEC 27001:2022**.

The objective is to move from "Hardened Configuration" to "Governance Readiness," ensuring that today's implementation can support a future formal audit or compliance review.

## NIST CSF 2.0 Alignment

| Function | Blueprint Decision | Supporting Evidence |
| :--- | :--- | :--- |
| **GOVERN (GV)** | Main Task Priority; AI Attribution; Cost-Conscious Selection | [README.md](../README.md), [Workflow](21-ai-assisted-engineering-workflow.md) |
| **IDENTIFY (ID)** | Cloud Infrastructure Inventory; Lockfiles; ADR Record | [Inventory Template](../templates/infrastructure-inventory.md), [Journey](01-project-journey.md) |
| **PROTECT (PR)** | App Check; Security Rules; WAF; DMARC; Secrets stores | [Security](04-firebase-security.md), [Terraform](05-terraform-management.md) |
| **DETECT (DE)** | Analytics Aggregate Reporting; Log redaction; Alerts | [Observability](11-observability.md) |
| **RESPOND (RS)** | Blameless Post-mortems; Rollback Playbook | [Incidents](08-incidents-and-lessons.md) |
| **RECOVER (RC)** | Baseline Recovery Seed; PITR; Backup Drills | [Reliability](12-reliability-backup-and-recovery.md) |

## ISO/IEC 27001:2022 Controls Crosswalk

| Annex A Control | Blueprint implementation | Cost-Conscious Rationale |
| :--- | :--- | :--- |
| **5.9 Inventory of information** | [Infrastructure Inventory](../templates/infrastructure-inventory.md) | Prevent cost and security leaks from orphaned assets. |
| **5.33 Protection of records** | Firestore PITR and daily backup schedule | Use managed services for high availability at zero operational time. |
| **8.1 User endpoint security** | CI Quality Gates and analyzer rules | Enforce quality at the developer boundary rather than after deployment. |
| **8.8 Management of vulnerabilities** | Dependency pinning and automated scanning | Reduce blast radius of supply-chain attacks without manual triage. |
| **8.16 Monitoring and logging** | Action-oriented alerting (staged thresholds) | Minimize storage costs by only logging decision-driving signals. |
| **8.28 Secure coding** | OWASP-aligned Rules and App Check | Shift-left security using built-in cloud platform attestation. |

## Strategy for Compliance Readiness

1. **Establish Accountability:** Name one owner for the "Security Edge" (DNS/WAF) and one for "Backend Security" (Rules/Auth).
2. **Document the Gap:** Maintain a persistent **Risk Register** to track unmitigated vulnerabilities and deferred hardening tasks.
3. **Automate Evidence:** Treat CI run history and Terraform plans as "Continuous Audit" artifacts.
4. **Regular Drills:** A backup is only a "hope" until a successful [Restore Drill](../templates/restore-drill.md) is recorded.

## Revisit when

Re-assess alignment when entering a regulated industry, handling highly sensitive personal data (PII), or when team size requires formal separation of duties (SoD) across deployment and runtime environments.
