# AI Workflow Orchestrator

> **Project status:** Documentation baseline / proposed MVP specification.  
> **Target:** MVP v1.0 - local-first Windows desktop application.  
> **Primary purpose:** Coordinate AI-assisted engineering workflows with durable execution, explicit authorization, deterministic verification, and recoverable state.

This directory contains the product and engineering specifications for AI Workflow Orchestrator. This README is the **primary navigation and onboarding document**. It summarizes the system and directs contributors to the authoritative specification for each engineering domain; it does not replace those specifications.

> **Implementation status:** The inspected project directory currently contains documentation only. Do not infer that a capability is implemented because it appears in a specification.

## 1. Project overview

AI Workflow Orchestrator is intended to provide a controlled execution layer for multi-step AI-assisted work. It coordinates workflow definitions, tasks, model-provider calls, approved developer tools, validation, execution history, and recovery.

The product differentiator is not another chatbot or unrestricted autonomous agent. It is **durable, observable, verifiable, and user-controlled execution** across supported AI providers and developer tools.

## 2. Product vision and MVP objectives

The vision is to let an individual developer define, execute, inspect, validate, and recover structured AI workflows locally without losing task state, project context, or control over consequential actions.

The MVP must prove one reliable end-to-end workflow before expanding into advanced orchestration. Its objectives are to:

- Define and version structured workflows and tasks.
- Execute tasks sequentially in dependency order.
- Persist workflow state, task attempts, events, outputs, and artifacts.
- Support one approved hosted model provider and Ollama/local inference.
- Provide a small, explicit set of project-scoped developer tools.
- Enforce action-scoped human approval in the backend.
- Recover interrupted executions without pretending uncertain external side effects succeeded.
- Run deterministic validation and configured tests.
- Expose truthful execution status, errors, logs, and verification evidence.
- Package and verify the Windows desktop application.

The MVP is local-first and single-user. It does not require a cloud control plane or cloud account.

## 3. Core capabilities

| Capability | MVP position | Governing specification |
|---|---|---|
| Local desktop application and project management | In scope | [SCOPE.md](./SCOPE.md), [ARCHITECTURE.md](./ARCHITECTURE.md), [DESIGN.md](./DESIGN.md) |
| Structured workflow and task definitions | In scope | [PRD.md](./PRD.md), [SCOPE.md](./SCOPE.md), [EXECUTION_ENGINE.md](./EXECUTION_ENGINE.md) |
| Sequential dependency-aware execution | In scope | [SCOPE.md](./SCOPE.md), [EXECUTION_ENGINE.md](./EXECUTION_ENGINE.md) |
| Hosted provider + Ollama adapter support | In scope | [SCOPE.md](./SCOPE.md), [TECH_STACK.md](./TECH_STACK.md), [ARCHITECTURE.md](./ARCHITECTURE.md) |
| Restricted project-scoped file/Git/test operations | Limited MVP set | [SCOPE.md](./SCOPE.md), [SECURITY.md](./SECURITY.md), [RULES.md](./RULES.md) |
| Approval gates and backend authorization | In scope | [SECURITY.md](./SECURITY.md), [API_SPECIFICATION.md](./API_SPECIFICATION.md), [EXECUTION_ENGINE.md](./EXECUTION_ENGINE.md) |
| Persistence, recovery, and idempotency controls | In scope | [DATABASE.md](./DATABASE.md), [EXECUTION_ENGINE.md](./EXECUTION_ENGINE.md) |
| Deterministic validation and observability | In scope | [SCOPE.md](./SCOPE.md), [ARCHITECTURE.md](./ARCHITECTURE.md), [EXECUTION_ENGINE.md](./EXECUTION_ENGINE.md) |
| Advanced parallel/distributed execution, MCP, n8n, autonomous multi-agent execution | Excluded or deferred; not MVP commitments | [SCOPE.md](./SCOPE.md) |

The table is an orientation aid. If a summary here differs from a detailed specification, use the authority and conflict process below; do not silently reinterpret scope.

## 4. Approved technology overview

The technology registry is [TECH_STACK.md](./TECH_STACK.md). The current proposed MVP baseline is:

| Layer | Technology |
|---|---|
| Desktop shell | Tauri 2 |
| Native host | Rust |
| Frontend | TypeScript, HTML, CSS, browser Fetch API |
| Frontend tooling | Node.js 22.x LTS |
| Backend | Python 3.12.x + FastAPI |
| API validation | Pydantic 2.x |
| Persistence | SQLite via Python standard-library `sqlite3` |
| Artifact storage | Local filesystem |
| Local inference | Ollama through a backend adapter |
| Hosted inference | One selected provider through an approved adapter |
| CI | GitHub Actions; Windows release verification |

Exact versions, lockfile rules, allowed dependencies, and conditional approvals are governed by [TECH_STACK.md](./TECH_STACK.md). Do not treat this summary as permission to install, upgrade, or substitute a dependency.

## 5. High-level architecture

The proposed architecture is a **local-first modular monolith**:

1. **Tauri desktop shell and TypeScript UI** - user interaction and presentation; not an authority for workflow state or permissions.
2. **Python/FastAPI backend** - authoritative application boundary for workflow state, scheduling, authorization, provider/tool execution, persistence, and recovery.
3. **Domain and application layers** - workflow/task invariants, state transitions, policies, orchestration use cases, and coordination.
4. **Infrastructure adapters** - SQLite repositories, local artifact storage, hosted/local model providers, and narrowly scoped Git/test/file operations.
5. **Control and verification** - action-scoped approvals, policy enforcement, deterministic validation, event history, logs, and recovery.

The backend owns authoritative workflow and task state. Model output, frontend state, provider responses, and tool output are untrusted inputs and must not directly authorize actions or mutate authoritative state.

Detailed boundaries and data flows: [ARCHITECTURE.md](./ARCHITECTURE.md). Execution semantics: [EXECUTION_ENGINE.md](./EXECUTION_ENGINE.md). Security boundaries: [SECURITY.md](./SECURITY.md).

## 6. Documentation directory

### Project and product

| Document | Responsibility |
|---|---|
| [01.summary.md](./01.summary.md) | Non-normative project overview and historical direction summary. |
| [PRD.md](./PRD.md) | Product vision, personas, use cases, functional/non-functional requirements, product acceptance outcomes, and roadmap proposals. |
| [SCOPE.md](./SCOPE.md) | **Authoritative MVP boundary**, in-scope and excluded capabilities, scope-change process, delivery milestones, and MVP release checklist. |

### Engineering specifications

| Document | Responsibility |
|---|---|
| [ARCHITECTURE.md](./ARCHITECTURE.md) | **Canonical system architecture**: layers, modules, boundaries, data flow, integration topology, testing strategy, packaging, and implementation sequence. |
| [technical-architecture.md](./technical-architecture.md) | Earlier overlapping technical architecture/security design. Retained pending owner-approved reconciliation; not a parallel source of implementation authority. |
| [TECH_STACK.md](./TECH_STACK.md) | **Authoritative technology and dependency registry**, versions, allowed/conditional choices, exclusions, and CI requirements. |
| [DESIGN.md](./DESIGN.md) | UI/UX information architecture, interaction behavior, design system, and interface requirements. |
| [DATABASE.md](./DATABASE.md) | **Authoritative persistence contract**: schema, constraints, indexes, transactions, migrations, and database recovery. |
| [API_SPECIFICATION.md](./API_SPECIFICATION.md) | **Authoritative API contract**: routes, payloads, validation, authentication/session behavior, errors, and transport requirements. |
| [EXECUTION_ENGINE.md](./EXECUTION_ENGINE.md) | **Authoritative execution contract**: workflow/task state machines, scheduling, retries, cancellation, approvals, idempotency, and recovery. |
| [SECURITY.md](./SECURITY.md) | **Authoritative security policy**: threat model, trust boundaries, authentication/authorization, secrets, tool restrictions, security tests, and release gates. |

### Agent and engineering governance

| Document | Responsibility |
|---|---|
| [AGENTS.md](./AGENTS.md) | Concise repository-level AI coding-agent entry point. Required first instruction layer for AI agents. |
| [AGENT_WORKFLOW.md](./AGENT_WORKFLOW.md) | **Authoritative AI-agent operating procedure**: context loading, planning, approval gates, implementation, verification, escalation, and handoff. |
| [RULES.md](./RULES.md) | General engineering and code-quality standards across backend, frontend, integrations, security, data integrity, tests, and Git hygiene. |

## 7. Document authority matrix

Each domain has one primary authority. Other documents may reference or summarize it but must not redefine its contract.

| Domain | Authoritative document | Supporting documents |
|---|---|---|
| MVP scope and exclusions | `SCOPE.md` | `PRD.md`, `README.md` |
| Product intent and requirements | `PRD.md` | `SCOPE.md` controls whether a requirement is admitted to MVP |
| System architecture and module boundaries | `ARCHITECTURE.md` | `TECH_STACK.md`, domain specifications |
| Technology and dependencies | `TECH_STACK.md` | `ARCHITECTURE.md`, `RULES.md` |
| UI/UX behavior and presentation | `DESIGN.md` | `PRD.md`, `ARCHITECTURE.md` |
| Database and persistence schema | `DATABASE.md` | `ARCHITECTURE.md`, `EXECUTION_ENGINE.md` |
| HTTP/API contracts | `API_SPECIFICATION.md` | `ARCHITECTURE.md`, `SECURITY.md` |
| Workflow execution and lifecycle | `EXECUTION_ENGINE.md` | `SCOPE.md`, `DATABASE.md`, `API_SPECIFICATION.md` |
| Security and authorization | `SECURITY.md` | `ARCHITECTURE.md`, `API_SPECIFICATION.md`, `DATABASE.md`, `RULES.md` |
| AI coding-agent task procedure | `AGENT_WORKFLOW.md` | `AGENTS.md` points to it |
| General implementation/code standards | `RULES.md` | `AGENT_WORKFLOW.md`, domain specifications |
| Project navigation | `README.md` | All specifications |
| Historical/quick project summary | `01.summary.md` | Non-normative; cannot override the above |

**Conflict rules:**

1. The current, explicit project-owner decision governs the specific approved change.
2. `SCOPE.md` governs MVP admission; a PRD feature or architecture suggestion does not automatically enter MVP.
3. `SECURITY.md` governs security requirements. Security controls cannot be weakened by a convenience requirement.
4. Each domain authority in the matrix governs its own contract.
5. `AGENT_WORKFLOW.md` governs how agents perform work; `RULES.md` governs engineering standards. Neither silently overrides a domain specification.
6. If authoritative statements conflict, stop the affected implementation. Record the exact statements, affected behavior, consequences, and proposed resolution. Obtain owner approval where required, then update every affected document consistently.

Do not resolve conflicts by assuming that a longer, newer-looking, or more detailed document automatically wins.

## 8. Recommended document reading order

Agents should use task-based context loading rather than loading every specification for every change.

### Mandatory entry sequence for every AI coding task

1. [AGENTS.md](./AGENTS.md)
2. [AGENT_WORKFLOW.md](./AGENT_WORKFLOW.md)
3. [SCOPE.md](./SCOPE.md)
4. [SECURITY.md](./SECURITY.md)
5. The task-relevant domain specifications listed below.
6. Inspect the actual repository tree, source files, manifests, configuration, tests, Git status, and existing conventions before planning edits.

### Task-specific specification selection

| Task type | Read these specifications |
|---|---|
| Product requirement or feature planning | `PRD.md`, `SCOPE.md`, `ARCHITECTURE.md` |
| Application/backend foundation | `ARCHITECTURE.md`, `TECH_STACK.md`, `API_SPECIFICATION.md`, `SECURITY.md` |
| UI/UX implementation | `DESIGN.md`, `API_SPECIFICATION.md`, `ARCHITECTURE.md` |
| Database/schema/migration | `DATABASE.md`, `SECURITY.md`, `EXECUTION_ENGINE.md` if execution state is affected |
| API or session/auth changes | `API_SPECIFICATION.md`, `SECURITY.md`, `ARCHITECTURE.md` |
| Workflow scheduler/state/retry/recovery | `EXECUTION_ENGINE.md`, `DATABASE.md`, `SECURITY.md`, `API_SPECIFICATION.md` where applicable |
| Provider/tool integration | `TECH_STACK.md`, `ARCHITECTURE.md`, `SECURITY.md`, `EXECUTION_ENGINE.md`, `API_SPECIFICATION.md` |
| Dependency or runtime change | `TECH_STACK.md`, `ARCHITECTURE.md`, `AGENT_WORKFLOW.md` |
| Tests or release readiness | `SCOPE.md`, `ARCHITECTURE.md`, `SECURITY.md`, relevant domain specifications |
| Documentation-only change | `AGENT_WORKFLOW.md` plus the documents being changed and their authorities |

If a required specification is missing, contradictory, or insufficient, stop and raise the gap instead of inventing a contract.

## 9. Development workflow

All AI-assisted implementation must follow [AGENT_WORKFLOW.md](./AGENT_WORKFLOW.md):

1. **Discover** - read the entry instructions and relevant specifications; inspect actual code, tests, configuration, Git status, and existing conventions.
2. **Classify** - confirm MVP scope, risk, affected domains, and approval requirements.
3. **Plan** - identify files/layers, data/API/security impact, dependencies, tests, and documentation changes.
4. **Obtain approval** - wait for owner approval for material architecture, schema, API, security, dependency, scope, or destructive changes.
5. **Implement incrementally** - make the smallest complete change within approved boundaries.
6. **Verify** - run relevant tests, static checks, security/failure-path tests, and build checks; report exact commands and observed outcomes.
7. **Review** - inspect the final diff for unintended changes, data loss, secret exposure, and contract drift.
8. **Document and hand off** - update affected specifications and provide the structured implementation report required by the agent SOP.

No AI agent may silently expand MVP scope, bypass approval, weaken tests/security, or claim unrun checks passed.

## 10. Implementation milestones

The approved MVP delivery sequence is defined by [SCOPE.md §8](./SCOPE.md#8-mvp-delivery-milestones):

| Milestone | Focus |
|---|---|
| P0-01 | Application and backend foundation |
| P0-02 | Persistence foundation |
| P0-03 | Workflow and task model |
| P0-04 | Sequential execution engine |
| P0-05 | Restart recovery |
| P0-06 | Provider adapters |
| P0-07 | Tool and approval controls |
| P0-08 | Validation and observability |
| P0-09 | Desktop workflows |
| P0-10 | Release verification |

Milestones are sequencing and exit-evidence gates, not permission to add deferred capabilities. Use the detailed acceptance criteria in `SCOPE.md` and the applicable domain specifications.

## 11. Testing and release process

Testing and release readiness are mandatory MVP requirements, not optional cleanup.

- Follow the test strategy in [ARCHITECTURE.md](./ARCHITECTURE.md), domain-specific verification requirements, and [SECURITY.md](./SECURITY.md).
- Use the test tools and commands already approved in [TECH_STACK.md](./TECH_STACK.md) and repository configuration.
- Test normal paths, invalid inputs, permission denials, failure paths, cancellation, retries, interrupted execution, recovery, and uncertain external side effects where relevant.
- Verify fresh database creation, migrations, constraints, transaction behavior, and recovery for persistence changes.
- Verify that authorization and approval are enforced by the backend, not just the UI.
- Keep secrets and sensitive project content out of logs, error messages, test artifacts, and source control.
- Do not disable, skip, or weaken tests to obtain a passing result.
- Windows packaging, clean installation, startup/shutdown, and release acceptance are required by the MVP checklist in `SCOPE.md`.
- A release is not ready while a required critical security invariant or acceptance criterion remains unverified.

The final release decision must be based on the checklists and evidence in `SCOPE.md`, not on a generated status summary.

## 12. Known documentation reconciliation items

The following are known audit findings, not silently approved changes:

- `technical-architecture.md` substantially overlaps `ARCHITECTURE.md` and has a broken case-sensitive reference to `./prd.md`. Keep it pending owner-approved reconciliation; use `ARCHITECTURE.md` as the canonical architecture authority.
- `RULES.md` and `AGENT_WORKFLOW.md` overlap in agent conduct, approvals, testing, and handoff. Keep both for now: SOP/process belongs in `AGENT_WORKFLOW.md`; engineering/code standards belong in `RULES.md`.
- `PRD.md` §19 describes a broad Unified Project Workspace and AI Activity Consolidation feature set with P0 requirements that are not represented in the current MVP milestone baseline in `SCOPE.md`. Treat those requirements as **not approved for MVP** until the project owner reconciles them with `SCOPE.md`.
- `01.summary.md` describes later MCP and parallel/multi-agent phases. Those are roadmap context only; the MVP exclusions in `SCOPE.md` govern.
- Existing documents contain inconsistent lowercase filename references such as `prd.md` and `design.md`; use the actual case-sensitive filenames listed in this README.

See the audit findings and owner decisions in the repository documentation audit report when available. No existing specification is deleted or rewritten by this navigation layer.

---

**Start here:** [AGENTS.md](./AGENTS.md) for AI-agent instructions, then [AGENT_WORKFLOW.md](./AGENT_WORKFLOW.md) for the complete operating procedure.
