# Information-security and compliance alignment

## Purpose and boundary

Recognised standards can organise questions, ownership, and evidence. They do not turn a technical blueprint into a compliant system. This chapter shows how to use the blueprint as an input to a risk-based security programme without claiming certification, legal compliance, audit readiness, or implementation in a reader's environment.

The principal references are:

- [ISO/IEC 27001:2022](https://www.iso.org/standard/27001) for an information-security management system.
- [NIST Cybersecurity Framework 2.0](https://www.nist.gov/cyberframework) for Govern, Identify, Protect, Detect, Respond, and Recover outcomes.
- [NIST SP 800-218 Secure Software Development Framework](https://csrc.nist.gov/pubs/sp/800/218/final) for secure software development.
- [CIS Controls v8.1](https://www.cisecurity.org/controls/v8) for prioritised safeguards and implementation groups.
- [OWASP ASVS](https://owasp.org/www-project-application-security-verification-standard/) for application-security verification requirements.
- [OWASP SAMM](https://owaspsamm.org/model/) for software-assurance maturity.

Record exact versions and applicability. Some normative standards are copyrighted; obtain authorised copies and qualified advice where required.

## Evidence levels

Keep these labels separate:

1. **Principle:** recommended direction.
2. **Decision:** selected approach for a defined context.
3. **Implemented control:** code or configuration exists.
4. **Tested control:** a dated assertion passed for an exact revision.
5. **Operational outcome:** deployed behaviour or a failure exercise produced retained evidence.
6. **Accepted risk:** an accountable owner approved residual risk with expiry.
7. **External assurance:** a competent independent party assessed a defined scope.

Blueprint prose and empty templates are principles and evidence structures. They are not implementation or assurance.

## NIST CSF 2.0 outcome map

| Function | Blueprint contribution | Evidence a real project still needs |
| --- | --- | --- |
| **Govern** | Decision records, technology rationale, risk/accepted-risk templates, AI accountability, cost and revisit triggers | Approved scope, owners, risk criteria/register, policies, legal/supplier obligations, objectives, management review, corrective actions |
| **Identify** | Architecture, infrastructure, data, dependency, performance, and threat-model templates | Maintained asset/software/data/identity/supplier inventories, data flows, business impact, threat and vulnerability assessment |
| **Protect** | Layered Firebase security, IAM/secrets guidance, CI gates, privacy lifecycle, secure release and AI-tool controls | Deployed settings, access lifecycle/MFA reviews, secure baselines, training, data handling, versioned security requirements, negative tests |
| **Detect** | Observability model, release correlation, alerts, synthetic checks, cost signals | Log-source/alert inventory, retention/access, detection coverage, tested alerts, escalation, measured detection time |
| **Respond** | Incident and rollback guidance, post-mortem practice, vulnerability reporting | Incident commander/backups, severity, contacts, evidence handling, notification decisions, playbooks, tabletop and containment evidence |
| **Recover** | RPO/RTO guidance, backup/restore and rollback templates, baseline-seed boundary | Business impact, protected backups, timed isolated restore, integrity/application checks, continuity communication, corrective actions |

## ISO/IEC 27001 management-system gap

Technical safeguards cover only part of ISO/IEC 27001. A conforming information-security management system also requires organisational context and scope, leadership and policy, risk assessment and treatment, competence and awareness, controlled documented information, operational planning, performance evaluation, internal audit, management review, nonconformity handling, and continual improvement.

Annex A is a reference set used through risk treatment and the statement of applicability; it is not a checklist where every control is automatically mandatory. Do not map a CI analyser to endpoint security, backups to protection of records, or App Check alone to secure coding. Map each applicable control to its actual objective, owner, implementation, evidence, residual risk, and review date using the licensed standard.

## Secure-software assurance

### NIST SSDF

Use the SSDF to define how the organisation prepares people and technology, protects software and build inputs, produces well-secured releases, and responds to vulnerabilities. For this blueprint, that means at least:

- Security requirements and threat models for changed trust boundaries
- Protected source, workflows, build identities, secrets, and environments
- Pinned and reviewed dependencies, tools, Actions, plugins, and AI integrations
- Review, analysis, security tests, provenance, artefact identity, and rollback
- Vulnerability intake, severity, remediation targets, retest, disclosure, and lessons

### OWASP ASVS

Select a version and proportionate verification level for the application. Record exact requirement identifiers, applicability, implementation links, tests, evidence date, reviewer, and exceptions. “Uses Firebase” or “has App Check” is not an ASVS verification result.

### OWASP SAMM

Use SAMM to assess and improve the security programme across governance, design, implementation, verification, and operations. Set target maturity per practice from risk and capacity. Do not maximise every practice or average scores into a claim of product security.

### CIS Controls

For a small team, start with applicable Implementation Group 1 safeguards: inventories, data protection, secure configuration, account/access management, vulnerability management, logging, recovery, awareness, service-provider management, application security, and incident response. Add higher-group safeguards when risk, exposure, team, or contractual needs justify them.

## Cost-conscious adoption sequence

### P0: establish control

1. Define scope, accountable owner, risk criteria, risk register, and accepted-risk expiry.
2. Inventory critical assets, sensitive data, privileged identities, suppliers, and essential journeys.
3. Close credential exposure, authorisation, destructive-change, and recovery risks first.
4. Define incident command and perform one isolated restore exercise.

### P1: make it repeatable

1. Tailor SSDF and ASVS requirements to the product.
2. Add proportionate secret, dependency, static, Rules, and hosted negative-path checks.
3. Protect CI/release inputs and map source to immutable artefact, deployment, observation, and rollback.
4. Establish access reviews, vulnerability management, supplier review, logging ownership, and secure baselines.

### P2: prove effectiveness

1. Exercise incident response, access revocation, alert detection, restore, rollback, and supplier outage.
2. Track outcome metrics such as vulnerability age, restore success, detection/containment time, exception age, and recurring defect class.
3. Conduct internal and management reviews and verify corrective-action effectiveness.
4. Seek external assessment only when business need justifies the recurring cost and evidence burden.

## Minimum evidence pack

- Scope, owners, policy, objectives, applicability decisions, and risk treatment
- Asset, software, data, identity, supplier, and manual-control inventories
- Data-flow/trust-boundary diagrams and business impact
- Access grant/review/revoke and MFA evidence
- Threat models, security requirements, ASVS mapping, and negative tests
- Vulnerability reports, remediation/retest, disclosure, and expiring exceptions
- CI permissions, pinned inputs, build/release provenance, hosted smoke tests, and rollback
- Log/alert inventory, alert exercise, incident timeline, and corrective actions
- RPO/RTO, protected backup, isolated restore, integrity/application verification
- Supplier review, agreements, subprocessors, outage/exit plan, and reassessment

Keep sensitive evidence in an access-controlled system. Publish only sanitised outcomes and stable references.

## Claims this blueprint does not support

Do not claim that a system is ISO/IEC 27001 compliant or certified, NIST compliant, CIS compliant, ASVS verified, SAMM mature, secure, resilient, or audit-ready merely because it follows this guide. Those claims require a defined real-world scope, applicable requirements, retained evidence, accountable review, and—where claimed—independent assurance.

## Revisit when

Reassess the framework selection and target depth after material incidents; new sensitive data, suppliers, AI capabilities, payments, platforms, jurisdictions, or contracts; major architecture or region changes; significant team growth; or at least annually.
