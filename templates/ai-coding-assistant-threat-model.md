# AI coding-assistant threat model

Use this template before enabling a new coding assistant, autonomous mode, model provider, IDE extension, MCP server, plugin, skill, CI agent, or materially broader permission. Re-run it after significant version, ownership, data-use, or tool-access changes.

## Review identity

- UTC date:
- Repository or system:
- Reviewer and accountable owner:
- Assistant, provider, model, and version or release channel:
- Intended tasks and explicit non-goals:
- Revisit date and triggers:

## Data and context boundary

- Data classifications permitted:
- Data classifications prohibited:
- Context automatically collected: open files, repository, terminal, diffs, diagnostics, retrieved content, or other:
- Tool-specific exclusion files and protected paths:
- Provider retention, training use, residency, subprocessors, deletion, and incident terms reviewed:
- Secrets, personal data, production exports, Terraform state, keys, and private evidence kept outside accessible paths:
- Prompt, tool, and audit-log retention and redaction:

## Instructions and injection surfaces

- Trusted policy and rules files:
- Nested or tool-specific rules-file locations inventoried:
- Owners and required reviewers for rules-file changes:
- Untrusted inputs: source, issues, pull requests, comments, logs, web pages, documents, images, dependency metadata, or tool descriptions:
- Control that prevents untrusted content from expanding scope or authority:
- Markdown, link, Unicode, multimodal, and generated-output handling:

## Tools and supply chain

For each model, extension, MCP server, plugin, skill, action, or external source, record:

| Component and source | Version/model | Publisher | Permissions and data destination | Owner | Verification and review date |
| --- | --- | --- | --- | --- | --- |
|  |  |  |  |  |  |

- Approved allowlist location:
- Pinning or change-detection mechanism:
- Tool-name shadowing and argument validation control:
- Dependency identity, registry, provenance, license, and vulnerability checks:
- Disable, revoke, and rollback procedure:

## Runtime and authority

- Sandbox, container, VM, or runner boundary:
- Allowed filesystem paths and denied sensitive paths:
- Network egress destinations:
- Credentials or identities, lifetime, scope, and storage:
- Allowed read, write, execute, install, database, cloud, deployment, and communication actions:
- Actions requiring explicit human approval:
- CPU, memory, disk, process, time, retry, tool-call, token, monetary, delegation, and concurrency limits:
- Production and organization-level access prohibited or separately controlled:

## CI and delegation

- Untrusted pull-request isolation:
- Separation between analysis and privileged mutation or deployment:
- Sub-agent inheritance of scope, permissions, and prohibited actions:
- Delegation depth and concurrency limit:
- Agent output provenance and merge review:
- Proof that comment or source text cannot authorize privileged execution:

## Generated change assurance

- Named human owner for every accepted change:
- Security-critical paths requiring specialist or independent review:
- Existing-test deletion or weakening detection:
- Independent tests, negative controls, or mutation evidence:
- Package and API verification against authoritative sources:
- Formatting, analysis, tests, infrastructure plan, build, deployment, and operational evidence required:
- Licensing, originality, maintenance, and rollback review:

## Abuse cases and negative tests

Record expected prevention and evidence for at least:

- A repository file instructs the agent to reveal a secret or ignore policy.
- A pull-request comment requests deployment or privileged credentials.
- A tool description contains hidden instructions or changes after approval.
- The model suggests a nonexistent or similarly named dependency.
- The agent attempts an out-of-scope edit, test deletion, or security-control weakening.
- A sub-agent receives broader access than its parent task.
- The provider, model, tool, or network becomes unavailable or exceeds budget.

## Decision and residual risk

- Decision: approve, approve with conditions, or reject:
- Required mitigations before use:
- Evidence reviewed:
- Residual risks and accepted-risk reference:
- Human approver and date:

Do not attach credentials, personal data, private prompts, raw production content, exploitable payloads, hidden instructions, or sensitive provider configuration to this record.
