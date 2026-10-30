# AI Coding Agent Instructions

These instructions apply to AI coding agents working anywhere in this project directory.

## 1. Required instruction hierarchy

Before planning or changing implementation, read and follow:

1. `AGENT_WORKFLOW.md` — mandatory end-to-end operating procedure.
2. `SCOPE.md` — authoritative MVP boundary, exclusions, milestones, and scope-change process.
3. `SECURITY.md` — authoritative security, trust-boundary, authorization, and prohibited-operation requirements.
4. The task-relevant domain specifications identified in `README.md` and the authority matrix below.
5. `RULES.md` — general engineering and code-quality standards.

This file is an entry point, not a replacement for `AGENT_WORKFLOW.md`. If instructions conflict, do not silently choose an interpretation. Follow the conflict and approval process in `AGENT_WORKFLOW.md`.

## 2. Repository discovery is mandatory

Before editing:

- Inspect the current repository tree, active branch, Git status, and existing diff.
- Inspect relevant source files, tests, manifests, lockfiles, configuration, and established patterns.
- Confirm that referenced modules, APIs, dependencies, commands, and capabilities actually exist.
- Read the specifications relevant to the requested change; do not assume documentation proves implementation.
- Preserve unrelated user changes. Never overwrite or discard them.

Do not fabricate implementation status, test results, runtime behavior, or repository contents.

## 3. Domain authorities

Use one authority for each engineering domain:

| Domain | Authority |
|---|---|
| MVP scope and exclusions | `SCOPE.md` |
| Product requirements | `PRD.md` |
| System architecture and boundaries | `ARCHITECTURE.md` |
| Technology and dependencies | `TECH_STACK.md` |
| UI/UX | `DESIGN.md` |
| Database and persistence | `DATABASE.md` |
| API contracts | `API_SPECIFICATION.md` |
| Workflow execution, state, retry, and recovery | `EXECUTION_ENGINE.md` |
| Security and authorization | `SECURITY.md` |
| AI-agent operating procedure | `AGENT_WORKFLOW.md` |
| General engineering standards | `RULES.md` |

`README.md` is navigation only. `01.summary.md` is non-normative. `technical-architecture.md` is retained pending reconciliation and must not be treated as a second, competing architecture authority.

## 4. Architecture, scope, and dependency constraints

- Preserve the approved local-first modular-monolith direction and the technology decisions in `TECH_STACK.md`.
- Keep the Python backend authoritative for workflow/task state, scheduling, authorization, provider/tool execution, persistence, and recovery.
- Treat frontend state, model output, provider responses, imported content, and tool output as untrusted inputs.
- Implement only capabilities admitted by `SCOPE.md` for MVP v1.0.
- Do not introduce parallel/distributed execution, autonomous multi-agent orchestration, MCP, n8n, cloud control planes, or other explicitly excluded/deferred capabilities without an approved scope change.
- Do not substitute frameworks, runtimes, databases, providers, or major dependencies without the approval required by `TECH_STACK.md` and `AGENT_WORKFLOW.md`.
- Prefer existing project capabilities and the smallest complete change. Avoid speculative abstractions, framework creep, unrelated refactors, and lockfile churn.

## 5. Mandatory human approval gates

Follow `AGENT_WORKFLOW.md` §3.2 and the relevant domain specifications. Present the impact and wait for explicit owner approval before material changes involving:

- MVP scope expansion or deferred/excluded capabilities.
- Architecture, module boundaries, execution model, state machines, scheduling, concurrency, or recovery semantics.
- Database schema, constraints, indexes, migrations, persisted enums, or data migration.
- API routes/contracts, authentication, authorization, or network exposure.
- Security controls, secret storage, encryption, logging/telemetry, sandboxing, process execution, or filesystem permissions.
- Major dependencies, frameworks, runtimes, databases, providers, or integrations.
- Destructive operations, user-data replacement/deletion, or consequential external actions.

A task request authorizes only the specific change it clearly describes. It does not authorize adjacent changes.

## 6. Implementation and security requirements

- Enforce authorization, project boundaries, and approval decisions in the backend.
- Use narrowly scoped, purpose-built adapters and allowlisted operations; never expose unrestricted host shell or filesystem access to a model.
- Bind approvals to the exact action, target, arguments, and execution context; reject stale or changed approvals.
- Protect credentials; never place secrets in source control, frontend storage, SQLite plaintext fields, logs, or error output.
- Do not silently transmit local project data to hosted providers or fall back across local/hosted boundaries against policy.
- Preserve transaction, idempotency, cancellation, retry, and recovery guarantees. Treat timeouts as potentially uncertain external outcomes.
- Fail closed when required authorization, policy, validation, credential, or approval services are unavailable.
- Never weaken security controls or test assertions to make a build pass.

## 7. Tests, verification, and documentation

- Add or update tests for changed behavior and relevant negative/failure paths.
- Run the applicable existing unit, integration, security, migration/recovery, type-check, lint, and build commands as configured.
- Record exact commands and observed results. Clearly identify tests not run and why.
- Review the final diff and Git status for unrelated changes, secrets, data loss, and contract drift.
- Update the authoritative specification(s), examples, and cross-references whenever an approved change alters product behavior, architecture, dependencies, schema, API, execution semantics, or security posture.
- Do not claim completion while required verification or approval remains outstanding.

## 8. Required final implementation report

Every implementation task must conclude with the structured handoff required by `AGENT_WORKFLOW.md`. At minimum report:

1. **Understanding** — requested behavior and accepted scope.
2. **Inspection** — source, tests, configuration, and specifications actually inspected.
3. **Plan and approvals** — affected layers/domains and approvals obtained or outstanding.
4. **Implementation** — files changed and behavior delivered.
5. **Validation** — exact commands, test/build results, and evidence.
6. **Security review** — trust boundaries, permissions, secrets, and data transmission affected.
7. **Limitations and risks** — unverified behavior, skipped checks, assumptions, and follow-up.
8. **Git status** — changed files and whether a commit was explicitly requested and created.

Never claim a test, build, migration, commit, or deployment succeeded unless its result was observed.
