# Scope Definition — AI Workflow Orchestrator

**Document:** SCOPE.md  
**Version:** 1.0  
**Status:** Proposed — MVP baseline  
**Target release:** MVP v1.0  
**Product type:** Local-first desktop application  
**Primary platform:** Windows  
**Last updated:** 2026-10-30

---

## 1. Purpose and authority

This document establishes the product and engineering boundaries for the AI Workflow Orchestrator MVP. It defines what may be designed, implemented, tested, and released as part of the MVP—and what must remain deferred unless explicitly approved.

The project documentation has distinct responsibilities:

- **PRD:** Defines product goals, user needs, requirements, and acceptance expectations.
- **ARCHITECTURE / technical architecture:** Defines system structure, technical boundaries, components, and implementation constraints.
- **design.md:** Defines the product experience, interface patterns, and visual direction.
- **SCOPE.md:** Defines the approved delivery boundary and controls scope changes.

A feature being mentioned in another document, suggested by an AI coding assistant, or appearing in a design concept does not automatically make it part of the MVP. When documents conflict, stop and raise the conflict for an explicit decision rather than silently expanding or changing scope.

## 2. MVP boundary

### 2.1 Product outcome

The MVP is a local-first desktop application that allows an individual developer to define, execute, monitor, validate, and recover structured multi-step AI workflows.

The MVP must demonstrate one reliable end-to-end workflow:

1. Create or select a local project.
2. Configure at least one supported hosted model provider or a local Ollama provider.
3. Define a structured workflow consisting of ordered tasks and explicit dependencies.
4. Configure execution policies, tool permissions, and approval requirements.
5. Execute tasks sequentially through the backend orchestration engine.
6. Pause at configured approval gates and wait for a human decision.
7. Persist workflow state, task attempts, events, outputs, and relevant artifacts.
8. Validate outputs using deterministic checks where applicable.
9. Inspect execution history, logs, task status, and generated artifacts.
10. Cancel or pause supported execution safely.
11. Restart the application and recover persisted workflow state.
12. Detect and surface uncertain outcomes that require reconciliation rather than assuming an external operation succeeded or failed.

### 2.2 MVP constraints

- Local-first operation; no cloud account or hosted control plane is required.
- Modular monolith; no microservices or distributed execution infrastructure.
- Sequential task execution is the baseline execution model.
- The Python backend is authoritative for workflow state transitions, scheduling, authorization, provider/tool execution, persistence, usage accounting, and recovery.
- The desktop frontend is a control and presentation layer; it must not independently decide authoritative workflow state.
- SQLite stores structured application state and metadata; the local filesystem stores project files and workflow artifacts.
- Provider and tool integrations must be isolated behind explicit adapters.
- External side effects must not be represented as exactly-once operations. Use idempotency controls, operation records, and reconciliation where possible.
- Security, data integrity, recoverability, and observable execution take priority over integration count or visual complexity.

### 2.3 MVP success criteria

The MVP is successful when a user can complete the end-to-end scenario above repeatedly and the system can:

- Enforce valid workflow and task state transitions.
- Persist execution events and task attempts.
- Recover a consistent view of execution after an application restart.
- Require explicit approval before configured sensitive actions.
- Prevent tools from operating outside approved project boundaries.
- Redact secrets from logs and user-facing errors.
- Clearly identify failed, cancelled, interrupted, and uncertain operations.
- Produce test evidence for critical execution, persistence, permission, and recovery paths.

## 3. In scope — MVP v1.0

The following capabilities are approved for MVP implementation.

### 3.1 Application foundation

- Tauri 2 desktop shell with a TypeScript-based frontend.
- Python and FastAPI backend.
- Secure local communication between the desktop application and backend.
- Windows-first development, packaging, and release verification.
- Clear separation between presentation, application orchestration, provider/tool adapters, and persistence.
- Configuration and local data paths that are explicit and documented.

### 3.2 Project management and context

- Create, open, rename, and manage local project workspaces.
- Associate workflows, execution records, and artifacts with a project.
- Configure an explicit project root and enforce its access boundary.
- Attach or maintain manually supplied project context needed by a workflow.
- Display the selected project and its relevant execution history.

Automatic ingestion of all conversations, repositories, or local files is not part of the MVP.

### 3.3 Workflow definition

- Create, edit, validate, save, and version structured workflows.
- Define ordered tasks, task inputs, expected outputs, dependencies, and execution configuration.
- Validate workflow structure before execution.
- Represent workflow and task status using explicit state machines.
- Preserve the workflow definition/version associated with each execution.
- Support sequential execution as the only required execution strategy.

Required workflow states:

- `DRAFT`
- `READY`
- `RUNNING`
- `WAITING_FOR_APPROVAL`
- `PAUSED`
- `SUCCEEDED`
- `FAILED`
- `CANCELLED`
- `NEEDS_ATTENTION`

Required task states:

- `PENDING`
- `READY`
- `RUNNING`
- `WAITING_FOR_APPROVAL`
- `RETRY_WAIT`
- `SUCCEEDED`
- `FAILED`
- `SKIPPED`
- `CANCELLED`
- `NEEDS_ATTENTION`

State transitions must be validated by the backend. UI state must reflect persisted backend state rather than inventing transitions.

### 3.4 Execution engine

- Durable sequential workflow execution.
- Dependency-aware task readiness.
- Persisted workflow and task lifecycle transitions.
- Task attempts, timestamps, results, and failure information.
- Configurable bounded retries and timeouts where supported.
- Pause and cancellation handling with clearly defined semantics.
- Approval gates before configured actions.
- Interruption detection and restart recovery.
- Explicit handling of operations whose external outcome cannot be confirmed.
- Event history sufficient to reconstruct and explain execution progress.

Do not promise exactly-once execution for external provider calls, filesystem operations, Git actions, or other side effects. Use idempotency keys where meaningful, record operation intent and outcome, and require reconciliation when the result is uncertain.

### 3.5 Model provider integration

- Provider adapter interface with clear request, response, error, timeout, and usage boundaries.
- One hosted LLM provider integration for the MVP.
- One local Ollama integration.
- Explicit provider and model configuration.
- Connection/configuration validation.
- Safe handling of provider credentials.
- Provider errors and usage metadata recorded without exposing secrets.
- Clear indication when a request sends project or prompt data to an external hosted provider.

Additional providers, intelligent routing, and automatic fallback are deferred.

### 3.6 Controlled tools

The MVP includes only a small, explicit tool set:

- Project-scoped filesystem read and write operations.
- Git repository inspection and read-oriented status/diff/history operations.
- Execution of explicitly configured project validation or test commands.

Tool execution must use allowlisted operations, validated paths, bounded execution, and captured results. Any write or command that is classified as sensitive by the applicable policy must pass through an approval gate.

General-purpose shell access, arbitrary host access, unrestricted command execution, and broad computer-use automation are not MVP capabilities.

### 3.7 Human approval and policy controls

- Approval queue for actions that require human authorization.
- Clear description of the requested action, target, scope, and expected impact.
- Approve, reject, or cancel supported pending actions.
- Persist approval decisions and associate them with the relevant workflow/task/action.
- Enforce approval decisions in the backend, not only in the UI.
- Require renewed approval when a material change makes a previous approval no longer applicable.
- Default to deny when authorization is missing, invalid, or ambiguous.

An approval is action-scoped. Approval for one operation must not be treated as blanket permission for future or unrelated operations.

### 3.8 Persistence and recovery

- SQLite for structured application data and execution metadata.
- Local filesystem for project data and generated artifacts.
- Persisted workflow definitions and versions.
- Persisted workflow/task state, attempts, events, approvals, and relevant usage records.
- Transactional updates for critical state changes.
- Startup recovery and consistency checks.
- Clear handling of incomplete or interrupted operations.
- Backup/export strategy documented for user data before release.

Recovery must not silently mark an uncertain external operation as successful. Surface the uncertainty and provide a safe reconciliation path.

### 3.9 Validation and verification

- Deterministic validation of structured outputs and required fields.
- File existence, path, and content checks where applicable.
- Configured test command execution and result capture.
- Task completion checks based on explicit criteria rather than model self-report alone.
- Validation status and evidence linked to the relevant task/run.
- Unit, integration, and end-to-end tests for critical paths.

LLM-based evaluation may provide supplementary feedback, but it must not replace deterministic checks where deterministic verification is available.

### 3.10 Observability

- Workflow and task status.
- Current task and execution progress.
- Task attempts, durations, and retry history.
- Provider/model information for each model operation.
- Structured events and actionable error details.
- Approval history.
- Generated artifact references.
- Validation and test results.
- Safe logs with secret redaction.

Do not display fabricated metrics, placeholder execution history, or success indicators that are not backed by persisted data.

### 3.11 MVP interface

The interface may include the following functional areas, subject to the design document:

- Application shell and overview/dashboard.
- Project selection and project workspace.
- Workflow creation and management.
- Workflow execution and task details.
- Approval queue and decision history.
- Provider and model configuration.
- Tool permissions and security settings.
- Execution history, logs, artifacts, and validation results.

A visual element is in scope only when it supports an approved functional requirement. Decorative screens and non-functional controls are not release requirements.

### 3.12 Testing and release readiness

- Unit tests for state transitions, policy checks, persistence, and core services.
- Integration tests for provider adapters, tool adapters, and database operations.
- Recovery tests covering interruption and restart.
- Security tests for path boundaries, local API access, secret handling, and approval enforcement.
- Windows packaging and clean-install verification.
- Documentation for installation, configuration, permissions, known limitations, and recovery.

## 4. Out of scope — explicitly excluded from MVP

The following items must not be implemented as part of MVP v1.0 without an approved scope change.

### 4.1 Advanced orchestration

- Parallel or distributed task execution.
- Branching, loops, dynamic graph mutation, or complex conditional routing.
- Multi-agent debate, consensus, or autonomous agent spawning.
- Agent-to-agent delegation frameworks.
- Visual node-canvas workflow builder.
- Self-modifying workflows or autonomous workflow planning beyond the defined task structure.

### 4.2 Advanced model management

- Multiple additional hosted provider integrations beyond the MVP provider.
- Intelligent model routing or automatic provider selection.
- Automatic fallback across providers or models.
- Cost-optimized model selection and model benchmarking.
- Multi-model consensus or automatic response arbitration.

### 4.3 External integrations and automation platforms

- MCP client/server integration.
- n8n, Zapier, Make, or other automation-platform integrations.
- Browser automation or browser-session control.
- IDE/editor plugins.
- Messaging-platform integrations.
- Third-party tool marketplace or plugin marketplace.
- Remote webhooks and event-trigger integrations.

### 4.4 Collaboration and infrastructure

- Multi-user collaboration and shared workspaces.
- Cloud synchronization or hosted control plane.
- Remote workers or distributed execution.
- Microservices.
- Redis, Celery, Kafka, Kubernetes, or comparable distributed infrastructure.
- Enterprise deployment and centralized administration.

### 4.5 Automated knowledge and context systems

- Automatic ChatGPT or other conversation synchronization/scraping.
- Automatic indexing of all conversations or repositories.
- Vector database and semantic RAG pipeline.
- Knowledge graph.
- Autonomous long-term memory extraction and management.
- Automatic ingestion of arbitrary local data sources.

### 4.6 Enterprise and advanced security capabilities

- Enterprise SSO and centralized identity management.
- Remote or centralized secrets management.
- Enterprise compliance reporting and centralized audit administration.
- General-purpose container orchestration or a distributed sandbox platform.
- Unrestricted host shell, arbitrary filesystem access, or general computer-use automation.

MVP security controls remain mandatory; excluding enterprise security products does not permit weakening local security.

### 4.7 Additional product capabilities

- Advanced analytics, forecasting, and executive reporting.
- Cost forecasting and budget optimization.
- Public workflow marketplace.
- Community template library.
- Scheduled workflow execution.
- Workflow sharing and collaboration.
- Complex import/export ecosystems.
- Plugin discovery and installation center.
- Cosmetic UI features that do not support an approved workflow.

### 4.8 Engineering exclusions

- Premature microservice decomposition.
- Infrastructure added without a demonstrated MVP requirement.
- Speculative abstractions for integrations that are not approved.
- Unnecessary dependencies or frameworks.
- Performance optimization without a measured bottleneck.
- Fake data or placeholder functionality presented as working product behavior.

## 5. Deferred roadmap — not MVP commitments

Deferred items are candidates for later evaluation, not promises or automatic commitments. Each item requires a separate product decision, technical review, and acceptance criteria before implementation.

| Release candidate | Deferred capability | Entry condition |
|---|---|---|
| v1.1 | Workflow templates and reusable workflow definitions | Core workflow schema and versioning are stable |
| v1.1 | Workflow import/export | Format, validation, and compatibility rules are defined |
| v1.1 | Bounded parallel task execution | Sequential engine, persistence, cancellation, and recovery are proven |
| v1.1 | Additional provider adapters | Provider interface and error/usage contracts are stable |
| v1.1 | Improved recovery and reconciliation tooling | MVP recovery evidence identifies concrete gaps |
| v1.1 | Server-sent events or richer live updates | Polling or current update mechanism is demonstrably insufficient |
| v1.2 | Provider routing and fallback policies | Multiple providers are supported and failure semantics are defined |
| v1.2 | MCP integration | Permission model, trust boundary, and tool lifecycle are reviewed |
| v1.2 | Additional tool adapters | A validated user need and safe execution contract exist |
| v1.2 | Cost-aware model selection | Usage accounting is accurate and policy requirements are defined |
| v2.0 | Advanced branching and conditional orchestration | Workflow semantics and recovery model support it safely |
| v2.0 | Multi-agent coordination | Explicit use cases, bounded autonomy, and observability are proven |
| v2.0 | Scheduling and event-triggered workflows | Lifecycle, idempotency, and background execution are designed |
| v2.0 | Visual workflow editor | Structured workflow model is stable and usability evidence supports it |
| v2.x | Context retrieval and knowledge indexing | Data-source consent, privacy, indexing, and deletion policies are defined |
| v2.x | Conversation import/integration | Supported provider APIs and user-controlled import semantics are established |
| v3.0 | Collaboration, cloud sync, and distributed execution | Security, identity, conflict resolution, and operational requirements justify the expansion |

Roadmap labels may change. Do not begin deferred work solely because a version label appears in this table.

## 6. Scope expansion and approval process

### 6.1 Mandatory process

Any feature, integration, infrastructure component, or behavior outside Section 3 must follow this process before implementation.

1. **Identify the proposal.** Describe the user problem, expected outcome, and why the current MVP cannot meet the need.
2. **Check existing scope.** Classify the request as already in scope, explicitly out of scope, deferred, or unclear.
3. **Assess impact.** Review product value, PRD changes, architecture, data model, API contracts, security, privacy, UX, dependencies, testing, maintenance, schedule, and release risk.
4. **Recommend a decision.** Select one of the outcomes below and explain the trade-offs.
5. **Obtain explicit human approval.** No implementation may begin until the project owner approves the exact change.
6. **Update documentation.** Update this SCOPE.md and every affected authoritative document. Record significant architectural decisions in an ADR.
7. **Update delivery controls.** Add acceptance criteria, implementation tasks, tests, and release notes as appropriate.
8. **Implement and verify.** Work only within the approved change and its acceptance criteria.

### 6.2 Decision outcomes

- `KEEP_DEFERRED` — retain the item for future consideration; do not implement now.
- `REJECT` — do not pursue the proposal.
- `APPROVE_FOR_MVP` — add the explicitly described capability to the MVP scope.
- `REPLACE_EXISTING_SCOPE` — approve a change that removes or replaces an existing MVP commitment.

### 6.3 Approval rules

- The project owner is the final scope authority.
- AI-generated recommendations, coding-agent plans, issue comments, or inferred intent are not approval.
- Silence, ambiguous feedback, or an incomplete discussion is not approval.
- Approval applies only to the specific change described and accepted.
- An approved scope change must be recorded in this document before implementation starts.
- If implementation reveals additional requirements outside the approved change, stop and request a new decision.

### 6.4 Scope change request template

Use this template when proposing a change:

| Field | Required information |
|---|---|
| Change ID | Unique identifier |
| Proposal | Concise description of the requested change |
| User problem | Problem this change solves |
| Current scope status | In scope / out of scope / deferred / unclear |
| MVP value | Why it is necessary now |
| Alternatives | Smaller or lower-risk options |
| Product impact | PRD, user flow, and acceptance criteria |
| Technical impact | Architecture, data, API, dependencies, and migration |
| Security impact | Permissions, secrets, data exposure, and threat surface |
| Delivery impact | Effort, tests, maintenance, and release risk |
| Recommendation | Keep deferred / reject / approve / replace |
| Owner decision | Explicit decision and date |

### 6.5 Emergency corrective work

A security vulnerability, data-loss risk, or critical crash may require immediate corrective work. Such work must be limited to mitigating the defect, tested, documented, and reviewed. Emergency remediation does not create permission to add unrelated features or permanently expand product scope.

## 7. Mandatory instructions for AI coding agents and contributors

Before planning or implementing a feature, read:

- `AGENTS.md`, if present.
- `SCOPE.md`.
- The relevant sections of `prd.md`.
- `ARCHITECTURE.md` and/or `technical-architecture.md`.
- `design.md` when the change affects the interface.
- Relevant specifications, ADRs, tests, and existing implementation.

Then follow these rules:

1. Treat Section 3 as the approved MVP boundary.
2. Do not implement items in Sections 4 or 5 unless an explicit scope change is approved.
3. Do not add integrations, dependencies, services, screens, or abstractions merely because they may be useful later.
4. Do not silently reinterpret requirements or resolve material document conflicts by guessing.
5. If a request is out of scope, stop before implementation. Explain why it is out of scope and prepare a scope-change proposal if appropriate.
6. Do not replace real functionality with mock data, fake metrics, placeholder buttons, or simulated success.
7. Preserve backend authority over workflow state, permissions, execution, and persistence.
8. Preserve data integrity, least privilege, path boundaries, secret redaction, and approval enforcement.
9. Add or update tests for changed behavior and critical failure paths.
10. Report changed files, tests executed, test results, known limitations, and any remaining risks.

## 8. MVP delivery milestones

These milestones organize the approved scope; they do not authorize additional features.

| Milestone | Focus | Exit evidence |
|---|---|---|
| P0-01 | Application and backend foundation | Desktop shell, backend startup, secure local communication, clean shutdown |
| P0-02 | Persistence foundation | SQLite schema, migrations/versioning strategy, transactional repository tests |
| P0-03 | Workflow and task model | Validated definitions, versioning, state machines, transition tests |
| P0-04 | Sequential execution engine | Dependency handling, attempts, events, timeout/retry/cancellation tests |
| P0-05 | Restart recovery | Interrupted-run detection, recovery tests, uncertain-operation handling |
| P0-06 | Provider adapters | One hosted provider and Ollama integration with safe configuration |
| P0-07 | Tool and approval controls | Project-scoped tools, policy enforcement, approval persistence and tests |
| P0-08 | Validation and observability | Deterministic checks, execution history, logs, artifacts, useful errors |
| P0-09 | Desktop workflows | Functional project, workflow, execution, approval, provider, and settings views |
| P0-10 | Release verification | Security review, full test pass, Windows packaging, install and recovery verification |

## 9. MVP release acceptance checklist

The MVP must not be declared complete until the applicable items below are verified:

- [ ] A user can create and reopen a local project.
- [ ] A user can configure and validate the supported hosted provider and Ollama.
- [ ] A user can create, validate, save, and version a sequential workflow.
- [ ] The backend enforces workflow and task state transitions.
- [ ] Tasks execute in dependency order.
- [ ] Task attempts, events, outputs, and status survive application restart.
- [ ] Configured sensitive actions stop at an approval gate.
- [ ] Approval decisions are persisted and enforced by the backend.
- [ ] Project-scoped tools cannot escape their configured boundaries.
- [ ] Secrets are not exposed in logs, errors, or persisted execution output.
- [ ] Failed, cancelled, interrupted, and uncertain operations are distinguishable.
- [ ] Recovery does not silently claim an uncertain external side effect succeeded.
- [ ] Deterministic validation and configured test execution produce inspectable evidence.
- [ ] The UI displays real backend state and does not present fabricated activity.
- [ ] Critical unit, integration, recovery, and security tests pass.
- [ ] Windows packaging, clean installation, startup, and shutdown are verified.
- [ ] User-facing setup, permissions, limitations, and recovery documentation is available.
- [ ] No unapproved out-of-scope feature is included in the release.

## 10. Final scope rule

The MVP is a **reliable, local-first, sequential AI workflow execution system** with durable state, controlled tools, human approval, validation, observability, and recovery.

It is not yet a complete autonomous agent platform, general automation platform, multi-agent framework, cloud orchestration service, or universal integration hub.

**If a capability is not explicitly included in Section 3, it is not approved for MVP implementation.**