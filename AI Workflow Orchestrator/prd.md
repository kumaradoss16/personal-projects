# AI Workflow Orchestrator
## Product Requirements Document (PRD)

**Version:** 1.0  
**Status:** Proposed - Ready for technical design and scope validation  
**Product type:** Local-first AI developer productivity platform  
**Primary users:** Developers, AI engineers, technical teams, and self-hosters  
**Initial deployment:** Local desktop application  
**Proposed stack:** Tauri 2, Python, FastAPI, SQLite, TypeScript, hosted LLM APIs, and local inference through Ollama

---

## 1. Executive Summary

AI Workflow Orchestrator is a local-first platform for designing, executing, monitoring, and recovering multi-step AI workflows across language models, coding agents, developer tools, and external services.

The platform addresses the fragmentation and reliability problems associated with manually coordinating AI-assisted work. Instead of treating each model interaction as an isolated conversation, the system represents work as persistent workflows containing tasks, dependencies, execution policies, artifacts, validation steps, and explicit state transitions.

The platform will support hosted AI providers and local models through a common provider interface. It will also support integrations with Git, test runners, MCP-compatible tools, and other approved execution adapters.

The primary product differentiator is **durable, observable, and verifiable AI execution**, rather than merely providing another chatbot or visual workflow builder.

### Product vision

Create a dependable execution layer that enables developers to coordinate AI-assisted engineering work across different providers and tools without losing context, execution history, or control over consequential actions.

### Product principle

AI may plan and propose actions, but the orchestrator owns workflow state, execution policy, persistence, verification, and authorization.

---

## 2. Problem Statement

Developers increasingly rely on multiple AI models, coding assistants, automation tools, and local inference runtimes. These tools often operate independently, creating operational problems.

### 2.1 Primary problems

1. **Fragmented context:** Requirements, decisions, prompts, and outputs are distributed across conversations and applications.
2. **Manual coordination:** Developers copy information between models, agents, terminals, and repositories.
3. **Fragile execution:** Long-running tasks fail midway and may lose progress or duplicate completed work.
4. **Limited observability:** Users cannot easily trace failures, model usage, tool execution, and intermediate artifacts.
5. **Insufficient verification:** Generated outputs may be accepted without tests, independent review, or policy checks.
6. **Uncontrolled tool access:** AI-generated actions can create risks when tools have excessive permissions.
7. **Provider dependency:** Workflows are difficult to move between hosted providers and local inference systems.

### 2.2 Proposed solution

Provide a persistent orchestration layer that:

- Stores project context and workflow definitions.
- Decomposes work into explicit tasks and dependencies.
- Routes tasks to appropriate model and tool adapters.
- Persists execution state and intermediate artifacts.
- Supports retries, recovery, and idempotency controls.
- Validates outputs using deterministic checks wherever possible.
- Requires approval for configured consequential actions.
- Records an auditable execution history.

---

## 3. Product Goals and Non-Goals

### 3.1 Goals

- Enable users to create and run repeatable AI workflows.
- Support multiple hosted and local model providers.
- Preserve execution state across application restarts.
- Make task dependencies and failures visible.
- Enable controlled integration with development tools.
- Support human approval and policy-based execution controls.
- Provide structured logs, artifacts, and workflow histories.
- Minimize unnecessary cloud dependencies and data transmission.

### 3.2 Non-goals for the initial release

- Training foundation models.
- Building a new general-purpose LLM.
- Replacing IDEs, Git platforms, or established coding assistants.
- Unrestricted autonomous operation across the user's machine.
- Multi-tenant enterprise orchestration and centralized administration.
- Cloud synchronization or a hosted SaaS control plane.
- Guaranteed compatibility with every AI provider or coding agent.
- Fully autonomous production deployments.
- A large marketplace for third-party agents.

These capabilities may be considered later if user validation demonstrates sufficient demand.

---

## 4. Target Users and Personas

### Persona A: Independent developer

Needs to coordinate requirements, coding, tests, documentation, and review while minimizing repetitive manual work.

**Success outcome:** Complete a repeatable engineering workflow with fewer manual handoffs and clear verification results.

### Persona B: AI application engineer

Uses hosted and local LLMs, APIs, MCP tools, and evaluation pipelines.

**Success outcome:** Build provider-independent workflows with measurable reliability and traceable execution.

### Persona C: Technical self-hoster

Prefers local infrastructure and wants to retain control over project data, credentials, and execution.

**Success outcome:** Run supported workflows locally and explicitly control which data leaves the machine.

### Persona D: Technical team lead

Needs repeatable development processes, review gates, execution records, and predictable failure handling.

**Success outcome:** Establish consistent workflows with explicit validation and authorization requirements.

The first release will prioritize independent developers and AI application engineers. Team collaboration will remain outside MVP scope.

---

## 5. Core Use Cases

1. Convert a software requirement into a structured implementation plan.
2. Run a coding workflow that includes implementation, tests, and review.
3. Generate technical documentation and validate it before delivery.
4. Route model requests across hosted and local providers.
5. Recover an interrupted workflow without losing completed task records.
6. Execute approved Git operations and test commands.
7. Compare outputs from multiple models or specialist agents.
8. Investigate failed tasks using execution traces and logs.
9. Reuse a validated workflow for future projects.
10. Export workflow definitions, reports, and artifacts.

---

## 6. Functional Requirements

Priority definitions:

- **P0:** Required for a usable and safe MVP.
- **P1:** Important follow-up capability.
- **P2:** Advanced or later-stage capability.

### FR-01: Project and Context Management - P0

The system shall allow users to create and manage local projects.

Each project shall support:

- Project name and description.
- Working directory.
- Requirements and constraints.
- Persistent decisions and relevant context.
- Workflow definitions.
- Execution history and artifacts.

The system shall distinguish authoritative project requirements from generated summaries and unverified AI suggestions.

### FR-02: Workflow Definition - P0

Users shall be able to define workflows as structured tasks.

Each task shall support:

- Stable task identifier.
- Name and description.
- Input references.
- Dependencies.
- Assigned executor or provider.
- Expected output.
- Validation rules.
- Retry policy.
- Timeout policy.
- Approval requirements.

The MVP shall support a structured workflow editor or form-based workflow definition. A sophisticated visual drag-and-drop builder is not required.

### FR-03: Workflow Execution Engine - P0

The execution engine shall:

- Execute tasks according to dependency order.
- Track task and workflow state.
- Prevent execution of tasks whose required dependencies have not succeeded.
- Record task inputs, outputs, and failures.
- Handle cancellation and configured timeouts.
- Persist state transitions.
- Resume eligible interrupted workflows.
- Prevent unintended duplicate side effects during retries.

The engine shall own state transitions. Model-generated text shall not directly mutate authoritative workflow state.

### FR-04: Model Provider Integration - P0

The platform shall provide a common adapter interface for supported LLM providers.

The initial implementation shall support:

- At least one hosted provider.
- Ollama for local inference.
- Provider configuration and model selection.
- Connection and availability checks.
- Normalized response and error handling.
- Request timeouts and usage metadata where available.
- Explicit provider-specific capability reporting.

Additional providers shall be added through adapters rather than modifications to the core workflow engine.

### FR-05: Model Routing and Fallback - P1

The system should support configurable routing policies based on:

- Task type.
- Provider availability.
- Model capability.
- User-defined priority.
- Context-window constraints.
- Cost or usage limits when reliably available.
- Local-versus-hosted data policies.

Fallback shall only occur when the configured policy permits it.

The system shall record the selected provider, attempted fallbacks, and final result. It shall not assume that different models produce interchangeable results.

### FR-06: Tool Integration - P0/P1

The platform shall define a controlled adapter interface for external tools.

MVP integrations should prioritize:

- Git status and diff inspection.
- Test command execution.
- Reading project files within approved directories.
- Writing generated artifacts within approved directories.

MCP integration shall follow in a subsequent milestone unless needed for the initial validated use case.

Every tool invocation shall record its identity, inputs or safe references to them, result, duration, and errors.

### FR-07: Human Approval and Execution Policies - P0

The system shall support configurable approval gates.

Approval shall be required for configured actions such as:

- Writing or replacing protected files.
- Executing potentially destructive commands.
- Creating commits or pushing changes.
- Publishing artifacts.
- Accessing resources outside an approved project directory.

Approval records shall identify the requested action, the user decision, and the relevant execution.

An approval shall not automatically authorize unrelated future actions.

### FR-08: Durable State and Recovery - P0

The system shall persist workflow state and task history using a local database.

After an application restart, it shall:

- Restore the last persisted workflow state.
- Identify tasks interrupted during execution.
- Resume tasks only when their execution policy permits.
- Reconcile incomplete operations.
- Preserve completed task records.
- Report tasks that require manual intervention.

The product shall not claim exactly-once execution for arbitrary external side effects. It shall use idempotency controls, operation identifiers, and reconciliation where supported.

### FR-09: Validation and Verification - P0

The platform shall support deterministic validation steps, including:

- Schema and output-format validation.
- Test execution.
- Exit-code checks.
- Required-artifact checks.
- Documentation or file validation.
- User-defined completion criteria.

A workflow shall distinguish between a task that produced an output and a task whose output passed verification.

### FR-10: Observability and History - P0

Users shall be able to inspect:

- Workflow status and progress.
- Task dependencies.
- Execution timestamps and durations.
- Provider and model selections.
- Retries and fallback attempts.
- Tool invocations.
- Validation results.
- Error summaries.
- Generated artifacts.

Logs shall avoid exposing API keys, access tokens, passwords, and other secrets.

### FR-11: Workflow Templates - P1

Users should be able to save and reuse workflows.

Initial template candidates:

- Plan → Implement → Test → Review.
- Research → Synthesize → Validate → Report.
- Draft Documentation → Validate Links → Export.
- Analyze Repository → Identify Risks → Generate Report.

Templates shall be versioned so changes do not silently alter an already-running execution.

### FR-12: Import and Export - P1

The platform should support importing and exporting workflow definitions and selected artifacts.

Imports shall be validated before execution. Imported workflows shall not automatically gain permission to execute tools or access files.

### FR-13: Execution Cancellation - P0

Users shall be able to request cancellation of a running workflow.

Cancellation shall stop scheduling new tasks and signal supported running executors to stop. The interface shall indicate when a task cannot be safely interrupted immediately.

The system shall record the final state and any cleanup or reconciliation required.

---

## 7. Workflow and Task State Model

### 7.1 Workflow states

- `DRAFT`
- `READY`
- `RUNNING`
- `WAITING_FOR_APPROVAL`
- `PAUSED`
- `SUCCEEDED`
- `FAILED`
- `CANCELLED`
- `NEEDS_ATTENTION`

### 7.2 Task states

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

### 7.3 State-transition rules

1. Only valid transitions shall be accepted.
2. Terminal task states shall not be silently rewritten.
3. Retrying a task shall create a new attempt record.
4. Every transition shall be persisted before it is reported as durable.
5. A dependent task shall not run until its required dependencies satisfy their completion conditions.
6. A failed verification step shall not be reported as successful.
7. Recovery shall reconcile persisted state with the status of any external operation that may still be running.

---

## 8. Non-Functional Requirements

### NFR-01: Reliability

- Workflow state shall survive normal application restarts.
- Database transactions shall protect related state updates.
- Failure recovery shall be covered by automated tests.
- Corrupt or incomplete workflow records shall be detected and reported.

### NFR-02: Security

- API keys shall not be stored in plaintext application logs or workflow artifacts.
- Secrets shall be redacted from diagnostic output.
- Tool execution shall use least-privilege permissions.
- File operations shall be restricted to approved directories by default.
- Destructive operations shall require explicit policy checks.
- Untrusted model output shall be validated before being used as executable input.
- Dependency versions and third-party licenses shall be reviewed.

### NFR-03: Privacy

- The application shall operate locally without requiring a user account.
- Project data shall remain local unless a configured provider or integration requires transmission.
- Before hosted model requests, the application shall identify the configured data destination and applicable transmission behavior.
- Local inference shall be available through supported local providers.
- Telemetry, if introduced, shall be optional and clearly disclosed.

### NFR-04: Performance

Provisional MVP targets on a supported development machine:

- Local dashboard interactions: p95 under 200 ms, excluding external requests.
- Local workflow-state reads: p95 under 100 ms for ordinary workloads.
- Workflow status updates: visible within one second under normal operating conditions.
- Large or long-running tasks shall execute asynchronously without freezing the interface.

These are engineering targets to validate through benchmarks, not measured guarantees.

### NFR-05: Maintainability

- Core domain logic shall be independent of individual LLM providers.
- Provider and tool integrations shall use defined interfaces.
- Workflow definitions shall be versioned.
- Critical state transitions shall have automated test coverage.
- Architecture decisions and security boundaries shall be documented.

### NFR-06: Compatibility

- The initial release shall target supported Windows desktop environments.
- The architecture should permit later macOS and Linux support.
- Local model capabilities shall be detected rather than assumed.

---

## 9. Proposed Technical Architecture

### 9.1 Application layers

**Presentation layer**
- Tauri desktop shell.
- TypeScript user interface.
- Workflow editor, execution dashboard, approval queue, and logs.

**Application layer**
- Project management.
- Workflow lifecycle.
- Task scheduling.
- Approval handling.
- Import/export.
- Cancellation and recovery orchestration.

**Domain layer**
- Workflow and task entities.
- State-transition rules.
- Dependency validation.
- Execution policies.
- Idempotency and retry semantics.

**Infrastructure layer**
- SQLite persistence.
- Provider adapters.
- Local inference adapters.
- Tool and MCP adapters.
- Git and test-runner integrations.
- Logging and artifact storage.

### 9.2 Logical architecture

User Interface  
→ Application API  
→ Workflow State Machine  
→ Dependency Scheduler  
→ Provider / Agent / Tool Adapters  
→ Validation and Policy Checks  
→ Durable State, Events, Logs, and Artifacts

The interface shall communicate with the backend through a defined local API. Local API access shall be restricted to the intended application context and protected against unauthorized local requests.

### 9.3 Proposed deployment model

For the initial release:

- Tauri for the desktop shell.
- FastAPI for the local orchestration service.
- SQLite for workflow metadata and execution state.
- Local filesystem storage for artifacts.
- Provider-specific network access only when a workflow uses a hosted provider or external integration.

The backend shall not bind to an unrestricted network interface by default. The local service lifecycle, origin validation, authentication or pairing between UI and backend, and shutdown behavior must be specified before production release.

---

## 10. Core Data Model

### Project

- `id`
- `name`
- `description`
- `working_directory`
- `created_at`
- `updated_at`

### WorkflowDefinition

- `id`
- `project_id`
- `name`
- `version`
- `definition`
- `created_at`
- `updated_at`

### WorkflowRun

- `id`
- `workflow_definition_id`
- `workflow_version`
- `status`
- `started_at`
- `finished_at`
- `created_at`

### TaskDefinition

- `id`
- `workflow_definition_id`
- `name`
- `dependencies`
- `executor_type`
- `configuration`
- `validation_policy`

### TaskRun

- `id`
- `workflow_run_id`
- `task_definition_id`
- `status`
- `attempt_number`
- `input_reference`
- `output_reference`
- `error_code`
- `started_at`
- `finished_at`

### ExecutionEvent

- `id`
- `workflow_run_id`
- `task_run_id`
- `event_type`
- `timestamp`
- `metadata`

### ApprovalRecord

- `id`
- `workflow_run_id`
- `task_run_id`
- `action_summary`
- `decision`
- `decided_at`

### Artifact

- `id`
- `workflow_run_id`
- `task_run_id`
- `artifact_type`
- `storage_reference`
- `checksum`
- `created_at`

Secrets shall be stored separately from ordinary workflow configuration and excluded from logs and exported definitions.

---

## 11. User Experience Requirements

### Primary navigation

1. Dashboard
2. Projects
3. Workflows
4. Execution History
5. Approvals
6. Providers and Tools
7. Settings

### Dashboard

The dashboard shall show:

- Active and recently completed workflows.
- Failed or interrupted runs.
- Pending approvals.
- Recent execution activity.
- Provider availability.
- Quick access to saved workflows.

### Workflow detail screen

Users shall be able to inspect:

- Workflow graph or ordered task list.
- Current state of each task.
- Dependencies and blockers.
- Attempt history.
- Logs and artifacts.
- Validation outcomes.
- Retry, pause, cancel, and resume controls when permitted.

### Approval interface

Before an approved action executes, the user shall see:

- The action requested.
- The relevant target or resource.
- The requesting task and workflow.
- Available details or diff.
- The consequences and required permission.
- Approve and reject actions.

---

## 12. Privacy and Threat Model

### Assets to protect

- Source code and project files.
- Provider credentials.
- Workflow inputs and outputs.
- Execution history.
- Local filesystem integrity.
- Approval and policy configuration.

### Threats

- Malicious or compromised model output.
- Prompt injection in repository files or external documents.
- Unauthorized local API calls.
- Excessive tool permissions.
- Secret leakage through logs or artifacts.
- Malicious imported workflow definitions.
- Dependency or supply-chain compromise.
- Duplicate side effects after retries.
- Corrupted workflow state after interruption.

### Required mitigations

- Treat model and external content as untrusted.
- Separate planning from privileged execution.
- Validate all tool arguments against schemas and policies.
- Use allowlists for supported commands and file paths.
- Require approval for high-impact operations.
- Redact secrets and minimize retained sensitive data.
- Validate imports before use.
- Persist execution events and support reconciliation.
- Test recovery and authorization boundaries.

Local-first architecture reduces unnecessary data exposure but does not protect against every compromise of the operating system, desktop session, or configured provider.

---

## 13. MVP Scope and Release Plan

### Milestone 0 - Product and architecture validation

Deliverables:
- Validate the initial developer use case.
- Finalize workflow and task schemas.
- Define state transitions and retry semantics.
- Document trust boundaries and execution permissions.
- Establish acceptance tests.

Exit criteria:
- Approved architecture.
- Documented threat model.
- Agreed MVP workflows.

### Milestone 1 - Durable workflow foundation

Deliverables:
- Local project management.
- Workflow definitions.
- Task dependency scheduler.
- SQLite persistence.
- Execution history.
- Basic dashboard.
- Restart and recovery behavior.

Exit criteria:
- A workflow can run, persist progress, restart, and report its outcome accurately.

### Milestone 2 - AI provider integration

Deliverables:
- One hosted provider adapter.
- Ollama adapter.
- Provider configuration.
- Request timeout and error handling.
- Structured outputs and validation.
- Provider usage metadata where available.

Exit criteria:
- A workflow can invoke supported models through the same core execution interface.

### Milestone 3 - Controlled developer tools

Deliverables:
- Read-only repository inspection.
- Approved file operations.
- Test-runner integration.
- Approval gates.
- Safe cancellation.
- Artifact tracking.

Exit criteria:
- Tool actions are authorized, traceable, and validated.

### Milestone 4 - Reliability and release hardening

Deliverables:
- Failure injection tests.
- Recovery and idempotency tests.
- Security testing.
- Packaging and upgrade validation.
- Documentation and release checklist.

Exit criteria:
- No unresolved critical security defects.
- No known critical data-loss defects.
- Recovery behavior meets documented acceptance tests.

### Post-MVP

- Provider routing and automatic fallback.
- MCP integration.
- Reusable workflow templates.
- Parallel task execution.
- Specialist agent coordination.
- Workflow evaluation and comparison.
- Optional team or remote execution capabilities.

---

## 14. Acceptance Criteria

The MVP shall not be considered complete until all of the following are demonstrated.

1. A user can create a project and define a workflow with dependencies.
2. The workflow executes tasks in dependency order.
3. Workflow and task states persist across application restarts.
4. A failed task is recorded with a useful error summary.
5. Retry creates a distinct attempt record.
6. The system does not silently duplicate supported external side effects during recovery.
7. Hosted and local model adapters use a common execution contract.
8. Tool calls enforce authorization and input validation.
9. Configured high-impact operations require approval.
10. Tests or validators can reject an unsuccessful output.
11. Users can inspect execution history and generated artifacts.
12. API keys and other secrets are excluded from ordinary logs and exports.
13. Imported workflow definitions cannot bypass execution policy.
14. Cancellation and interrupted-task behavior are clearly represented.
15. The core workflow engine can be tested independently of individual providers.

---

## 15. Success Metrics

Initial targets are hypotheses for validation with early users.

### Product adoption

- At least 10–15 developers participate in initial usability testing.
- At least 80% of testers complete their first workflow without direct assistance.
- At least 40% of testers return within seven days.

### Workflow reliability

- 100% of tested state transitions comply with the state model.
- 100% of configured approval-required test actions are blocked until authorized.
- All critical recovery and duplicate-side-effect scenarios have automated regression tests.
- No known critical data-loss defects at release.

### Workflow usefulness

- Users can complete the selected reference workflow with fewer manual handoffs than their baseline process.
- Users can identify the cause of a failed task from available execution records.
- Users can distinguish generated outputs from verified outputs.

### Privacy and security

- No plaintext secrets in application logs or exported workflow definitions.
- No unapproved tool execution in the release test suite.
- No mandatory cloud account or telemetry requirement for local workflows.

---

## 16. Key Risks and Mitigations

**Risk: Excessive MVP scope.**  
Mitigation: Begin with one reliable sequential workflow and a small number of adapters before implementing a visual builder or multi-agent runtime.

**Risk: Model output is unreliable.**  
Mitigation: Use deterministic validation, structured outputs, and independent checks where justified.

**Risk: Retries duplicate side effects.**  
Mitigation: Use idempotency keys where supported, operation records, and reconciliation for uncertain outcomes.

**Risk: Local service security is underestimated.**  
Mitigation: Define the UI-to-backend trust boundary, restrict network exposure, validate origins, and use a protected local communication mechanism.

**Risk: Provider APIs change.**  
Mitigation: Isolate provider-specific behavior behind adapters and maintain contract tests.

**Risk: Local and hosted models have different capabilities.**  
Mitigation: Expose capability metadata and make fallback policies explicit.

**Risk: The product overlaps with existing workflow tools.**  
Mitigation: Differentiate through durable state, recovery, observability, provider portability, and controlled developer-tool execution.

**Risk: AI orchestration adds complexity without measurable value.**  
Mitigation: Validate a concrete engineering workflow against a manual baseline before expanding functionality.

---

## 17. Product Decisions Requiring Validation

Before implementation is finalized, resolve these decisions:

1. Should the first UI be a desktop application or a local web dashboard?
2. Which hosted provider should be supported first?
3. Should the first release support only sequential workflows or a dependency graph with parallel execution?
4. Which developer-tool actions require approval by default?
5. Should MCP be included in the MVP or deferred?
6. Should workflow definitions use JSON/YAML, a form-based editor, or both?
7. What is the minimum supported hardware configuration?
8. Which reference workflow will serve as the primary end-to-end acceptance test?

Recommended defaults are a Tauri desktop application, one hosted provider plus Ollama, sequential execution first, approval gates for consequential actions, and MCP after the core engine proves reliable.

---

## 18. Definition of Done

A feature is complete only when:

- Requirements and acceptance criteria are documented.
- Domain and state-transition rules are implemented.
- Unit and integration tests pass.
- Relevant failure and recovery cases are tested.
- Security and authorization checks are implemented.
- Logs and errors are actionable and do not leak secrets.
- User-visible states accurately represent execution outcomes.
- Documentation and configuration examples are updated.
- The feature passes the release checklist.

---

## Final Product Direction

AI Workflow Orchestrator should be developed as a **reliability-first orchestration engine with a usable developer interface**, not as an unrestricted autonomous agent.

The recommended first end-to-end demonstration is:

**Requirement → Plan → Implement → Run Tests → Review Diff → Request Approval → Produce Verified Report**

This single workflow should prove durable execution, provider integration, tool permissions, verification, observability, and recovery before the project expands into a general-purpose multi-agent platform.


---

## 19. Unified Project Workspace & AI Activity Consolidation

**Feature ID:** FEAT-01  
**Priority:** P0 foundation; advanced integrations P1/P2  
**Category:** Project Management, Context Management, AI Tool Integration

### 19.1 Feature Overview

The Unified Project Workspace enables users to consolidate project-related content created or modified across multiple AI applications, models, development environments, and workflow executions into one dedicated local project workspace.

A user may work on the same project using ChatGPT, Claude, Gemini, Cursor, VS Code-based assistants, Ollama, or other supported tools. The orchestrator shall provide one organized location for relevant requirements, prompts, responses, code, documents, workflows, research, generated artifacts, and execution records.

The system shall record which tool and model contributed to each captured activity, when the activity occurred, how it was captured, how long it took when measurable, and which project files or workflows it relates to.

This feature is not merely a file collection utility. It shall preserve relationships between project requirements, AI contributions, code changes, workflow executions, and verified results.

### 19.2 Objectives

- Create one dedicated local workspace per project.
- Consolidate project material from multiple AI tools and development environments.
- Preserve prompts, responses, source code, documents, workflows, and generated artifacts.
- Track tool, provider, and model attribution.
- Maintain a chronological and searchable activity timeline.
- Record activity timestamps and durations when reliably available.
- Detect conflicting file versions before merging or replacing content.
- Preserve provenance and recoverability.
- Reuse consolidated project context in subsequent workflows.
- Keep project ownership and data local by default.

### 19.3 FR-14: Unified Project Directory — P0

The system shall allow users to create or associate a dedicated local directory with each project.

Users shall be able to:
- Select an existing project directory or create a new workspace.
- Configure or customize the workspace folder structure.
- Rename, archive, export, and back up a workspace.
- Open the workspace in the operating system's file manager.
- Browse project files, captured activities, workflows, and artifacts.
- Configure which directories are canonical source locations and which are managed archive locations.

The system shall avoid creating unnecessary duplicate copies of canonical source files. It shall support references to files in an existing repository as well as managed workspace artifacts.

### 19.4 FR-15: Multi-Tool Project Association — P0

Users shall be able to associate supported AI tools and development environments with a project.

Initial candidate tools include:
- ChatGPT.
- Claude.
- Gemini.
- Cursor.
- VS Code and compatible AI extensions.
- Ollama and other supported local model runtimes.
- Custom API-based AI applications.

The initial release shall support manual import and explicit user-created activity records. Workflow-generated model calls and artifacts shall be captured automatically by the orchestrator.

The system shall not assume it can access private conversations from external applications without a supported, authorized integration. Each integration shall declare its available capture capabilities and permissions.

### 19.5 FR-16: Content Consolidation — P0

The system shall organize project-related content, including:
- Product requirements and technical specifications.
- Architecture and design documents.
- Prompts, instructions, and AI responses.
- Source code, patches, diffs, and test results.
- Markdown, text, and supported document files.
- Workflow definitions, runs, and outputs.
- Research notes and references.
- Generated files, reports, and exports.
- Debugging notes and relevant error logs.

Imported content shall retain available source metadata. Original files shall remain unchanged unless the user explicitly authorizes a managed copy or replacement.

### 19.6 FR-17: AI Activity Provenance — P0

Each captured activity shall maintain a structured record containing:
- Activity ID and project ID.
- Source application/tool.
- Provider and model identifier, when known.
- Activity type (prompt, response, code change, import, tool call, workflow run, validation, etc.).
- Activity start and completion timestamps, when available.
- Capture/import timestamp.
- Duration, when measurable.
- Related workflow/task identifiers, when applicable.
- Related file and artifact references.
- Capture method and source reference, when available.
- Content checksum or version reference, where applicable.
- Metadata confidence or verification status.

The system shall distinguish verified metadata, user-supplied metadata, and inferred metadata. Unknown values shall remain unknown rather than being fabricated.

### 19.7 FR-18: Project Activity Timeline — P0

The system shall provide a chronological project timeline showing captured work across AI tools, models, workflows, and local development activity.

Example:
- 09:15 — ChatGPT — PRD requirements imported.
- 10:05 — Claude — Backend architecture response captured.
- 11:20 — Cursor — Project files modified.
- 12:10 — Ollama — Local model experiment completed.
- 13:00 — Test Runner — Integration tests executed.
- 13:15 — Orchestrator — Verification completed.

Users shall be able to filter and search by:
- Date and time.
- Application, provider, or model.
- Activity type.
- Workflow or task.
- File or artifact.
- Execution status.

The timeline shall distinguish the time an activity occurred from the time it was imported or captured.

### 19.8 FR-19: Execution and Activity Time Tracking — P0

The system shall record timing information when available:
- Activity start time.
- Activity completion time.
- Duration.
- Capture/import time.
- Provider request duration.
- Workflow and task execution duration.
- File modification time, where relevant and available.

For externally generated or manually imported content, the system shall not invent start times or execution durations. Unknown timing fields shall be null or explicitly marked unavailable.

Timestamps shall retain timezone information or be normalized to UTC with the original timezone offset preserved where available.

### 19.9 FR-20: Conflict Detection and File Versioning — P0

The system shall detect potential conflicts when multiple tools or imports produce changes to the same canonical project file.

It shall:
- Compare normalized file paths and content checksums.
- Identify added, modified, deleted, and unchanged files.
- Show differences between competing versions.
- Preserve previous versions or Git references according to the configured policy.
- Require confirmation before replacing canonical files.
- Record the source activity and tool attribution for accepted changes.
- Provide recovery of prior managed versions.

The MVP may use Git history and content checksums rather than implementing a custom version-control engine. The system shall not claim to merge semantic code changes automatically unless a tested merge operation is available.

### 19.10 FR-21: Project Context Retrieval — P1

Before a workflow runs, the orchestrator should retrieve relevant project materials, such as:
- Requirements and constraints.
- Architecture documents and decisions.
- Relevant prompts and AI responses.
- Source files and tests.
- Previous workflow results.
- Research notes and known issues.

Context selection shall be relevance-based and bounded. The orchestrator shall not indiscriminately transmit the entire project directory to a model.

Users shall be able to inspect and approve the context selected for external provider requests. Project content shall follow the configured local-versus-hosted data policy.

### 19.11 FR-22: Workspace Synchronization — P1

The system should coordinate content between:
- The canonical project directory.
- Supported development environments.
- Imported AI activity.
- Workflow-generated artifacts.
- Managed project archive folders.

Synchronization shall be explicit, conflict-aware, and auditable. Cloud synchronization is outside the initial local-first release.

### 19.12 FR-23: Import and Capture Sources — P0/P1

Initial capture methods:
- Manual file and folder import.
- Supported Markdown, JSON, TXT, and document imports.
- Workflow-generated model calls, outputs, and artifacts.
- User-created activity records.
- Explicit local project-file monitoring with user permission.

Later capture methods:
- Official provider APIs.
- Supported application connectors.
- IDE extensions.
- Browser extensions.
- CLI capture utilities.
- MCP adapters.

The system shall not silently scrape private browser sessions or collect authentication cookies, passwords, or tokens. Integrations shall use documented APIs or explicit user-authorized capture mechanisms.

### 19.13 Suggested Workspace Structure

The following is a configurable default. Users may customize names and locations.

```text
Project-Workspace/
├── 00_PROJECT_CONTROL/
│   ├── project.json
│   ├── project-overview.md
│   ├── requirements.md
│   ├── decisions.md
│   └── project-timeline.json
├── 01_AI_INTERACTIONS/
│   ├── ChatGPT/
│   ├── Claude/
│   ├── Gemini/
│   ├── Cursor/
│   └── Local-Models/
├── 02_WORKFLOWS/
│   ├── definitions/
│   ├── executions/
│   └── results/
├── 03_SOURCE_CODE/
├── 04_DOCUMENTATION/
├── 05_RESEARCH/
├── 06_ARTIFACTS/
├── 07_EXECUTION_LOGS/
└── 08_ARCHIVE/
```

The canonical source repository may remain outside this structure. In that case, the workspace shall store references and provenance metadata rather than duplicating the entire repository.

### 19.14 Example Activity Record

```json
{
  "activity_id": "act_001",
  "project_id": "project_001",
  "source": {
    "application": "Claude",
    "provider": "Anthropic",
    "model": "model_identifier",
    "capture_method": "manual_import"
  },
  "activity_type": "code_generation",
  "title": "Backend API implementation",
  "started_at": null,
  "completed_at": null,
  "captured_at": "2026-10-30T09:30:00+05:30",
  "duration_ms": null,
  "workflow_id": null,
  "related_files": [
    "src/backend/api.py"
  ],
  "content_reference": "01_AI_INTERACTIONS/Claude/response_001.md",
  "metadata_confidence": "partially_verified"
}
```

Null timing values are intentional when source timestamps or duration are unavailable.

### 19.15 User Interface Requirements

The Project Workspace view shall provide:
- Project overview and workspace location.
- File explorer and artifact browser.
- Associated tools and known models.
- Chronological activity timeline.
- Workflow and execution history.
- Search and filters.
- File conflict and version notifications.
- Import, export, backup, and archive controls.

The overview may show counts of captured activities, participating tools, known models, artifacts, and workflow runs. These metrics shall be calculated from recorded data and shall not imply complete capture of activity in external applications.

### 19.16 Acceptance Criteria

1. A user can create or associate a dedicated workspace with a project.
2. A user can import content from at least two different AI tools using supported manual capture methods.
3. Imported content is associated with the selected project.
4. Available source tool and model attribution is preserved.
5. Unknown or unverified metadata is explicitly represented.
6. Activities are displayed chronologically using event time when available and capture time as a fallback.
7. Start, completion, capture, and duration fields are not conflated.
8. Activity records can link to related files, workflows, and artifacts.
9. Conflicting file changes are detected before canonical replacement.
10. Previous managed versions or Git references are recoverable.
11. Project content can be searched and filtered.
12. Workspace export and restore preserve activity metadata and file references or report unavailable external references.
13. No passwords, cookies, or authentication tokens are collected to achieve consolidation.
14. External AI transmission follows explicit provider configuration and user authorization.
15. The workspace can represent externally performed work without falsely claiming automatic or complete capture.

### 19.17 Success Metrics

Initial validation targets:
- Users can create a workspace without assistance.
- Users can consolidate content from at least two AI tools.
- Captured activities retain all available source attribution.
- Users can locate a previous artifact through search or timeline.
- No canonical source file is silently overwritten by an import.
- Users can recover previous managed versions.
- Users can identify which tools and models contributed to a project based on recorded activity.

---

## 20. Updated User Experience and Navigation

The primary navigation shall include a dedicated **Project Workspaces** area.

Recommended navigation:
1. Dashboard
2. Projects / Workspaces
3. Workflows
4. Execution History
5. Approvals
6. Providers and Tools
7. Settings

Each project workspace shall provide tabs or sections for:
- Overview.
- Files and Artifacts.
- AI Activity.
- Workflows.
- Timeline.
- Context and Decisions.
- Settings and Integrations.

The project workspace is the organizing boundary for project-specific context, activity, workflow runs, and artifacts. A workflow run shall belong to a project, and its model calls, tool invocations, and outputs shall be linked to that project where applicable.

---

## 21. Updated Core Data Model

The following entities extend Section 10.

### ProjectWorkspace
- `id`
- `project_id`
- `root_directory`
- `workspace_config`
- `created_at`
- `updated_at`

### ToolIntegration
- `id`
- `project_id`
- `application_name`
- `provider_name`
- `integration_type`
- `capabilities`
- `authorization_status`
- `created_at`

### AIActivity
- `id`
- `project_id`
- `tool_integration_id`
- `provider_name`
- `model_identifier`
- `activity_type`
- `title`
- `event_started_at`
- `event_completed_at`
- `captured_at`
- `duration_ms`
- `capture_method`
- `metadata_confidence`
- `content_reference`
- `content_checksum`

### ActivityFileReference
- `id`
- `activity_id`
- `file_path`
- `file_role`
- `content_checksum`
- `version_reference`

### WorkspaceVersion
- `id`
- `project_id`
- `file_path`
- `content_checksum`
- `source_activity_id`
- `version_reference`
- `created_at`

Activity records shall reference workflow, task, artifact, and project records using stable identifiers where applicable. Large content payloads should be stored as files or artifact blobs; SQLite should store metadata and references rather than unnecessarily duplicating large documents.

---

## 22. Updated MVP Scope and Release Priorities

The workspace foundation is part of the MVP, but broad automatic integrations are not.

### MVP additions
- Create and associate a local project workspace.
- Configurable default folder structure.
- Manual import of project files and AI conversation exports.
- Automatic capture of orchestrator-owned model calls, workflow events, and generated artifacts.
- Activity records with tool/model attribution and capture timestamps.
- Timeline and basic search/filtering.
- File checksums and basic conflict warnings.
- Explicit confirmation before replacing canonical files.
- Local export and backup.

### Post-MVP additions
- IDE extensions and richer coding-agent activity capture.
- Browser extension integrations.
- Official provider conversation integrations where supported.
- CLI capture.
- Advanced project context retrieval.
- More advanced Git-based version comparison and recovery.
- Optional synchronization across devices.

### MVP scope boundary

The initial release shall not promise universal automatic synchronization of private conversations across ChatGPT, Claude, Gemini, Cursor, or other third-party applications. Capture depends on each application's supported APIs, export formats, extension capabilities, and user authorization.

---

## 23. Updated Risks and Mitigations

**Risk: External AI applications do not expose conversation data through supported APIs.**  
Mitigation: Support manual imports and explicit capture first; add official integrations only where supported.

**Risk: Workspace becomes a duplicate and confusing copy of the source repository.**  
Mitigation: Distinguish canonical source directories from managed artifact/archive directories and use references where appropriate.

**Risk: Conflicting AI-generated code overwrites working code.**  
Mitigation: Use checksums, diffs, version references, and explicit approval before canonical replacement.

**Risk: Activity metadata is incomplete or inaccurate.**  
Mitigation: Preserve event time separately from capture time and label unknown, user-supplied, and verified metadata.

**Risk: Context retrieval transmits sensitive project data to hosted models.**  
Mitigation: Show selected context, apply provider data policies, and require user authorization for external transmission.

**Risk: File monitoring captures unrelated or sensitive files.**  
Mitigation: Require explicit directory selection, provide exclusion rules, and avoid monitoring outside approved project roots.

---

## 24. Updated Product Direction

AI Workflow Orchestrator shall combine two connected capabilities:

1. **Workflow orchestration:** Plan, execute, validate, recover, and observe multi-step AI work.
2. **Unified project workspace:** Consolidate project context, multi-tool contributions, code, documents, workflows, artifacts, and activity history.

The workspace becomes the persistent project memory and provenance layer. The orchestration engine uses that project context to execute future work while preserving traceability.

The product promise is:

**One project. One organized workspace. Multiple AI tools and models. Traceable contributions. Durable workflows. Verified results.**
