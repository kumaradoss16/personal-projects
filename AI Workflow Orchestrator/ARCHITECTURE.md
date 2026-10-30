# AI Workflow Orchestrator — System Architecture

**Document:** Architecture Specification  
**Version:** 1.0  
**Status:** Proposed  
**Date:** 2026-10-30  
**Architecture style:** Local-first modular monolith with durable asynchronous execution

## 1. Purpose and architectural direction

This document defines how to implement AI Workflow Orchestrator. The PRD defines product scope and behavior; design.md defines the desktop information architecture and visual system. This document turns those requirements into an implementation blueprint.

The product is a local-first desktop application for designing, executing, observing, validating, and recovering multi-step AI workflows across model providers and controlled developer tools. Its differentiator is durable, observable, verifiable execution—not another chatbot or unrestricted agent framework.

### 1.1 Non-negotiable principles

1. The backend owns workflow state, scheduling, authorization, approvals, persistence, and recovery.
2. Local execution and local persistence are defaults. External transmission must be explicit.
3. Implement durability and recovery before parallelism or distributed infrastructure.
4. Use a modular monolith. Do not introduce microservices without measured need.
5. Isolate providers and tools behind adapters.
6. Treat model output as untrusted input, never as authoritative state or authorization.
7. Prefer deterministic validation over model-based claims of success.
8. Require explicit, action-scoped approval for consequential operations.
9. The UI displays persisted and measured facts, never simulated progress or fabricated metrics.

### 1.2 MVP versus future scope

| Capability | MVP (P0) | Later |
|---|---|---|
| Desktop | Tauri 2, TypeScript UI | Additional platform polish |
| Backend | Local Python + FastAPI | Separate worker only if justified |
| Persistence | SQLite and local artifact storage | Backup/export enhancements |
| Workflow authoring | Structured editor/form | Advanced visual builder |
| Execution | Sequential dependency-aware scheduler | Bounded parallel execution |
| Models | One hosted provider + Ollama | Additional adapters and routing |
| Tools | Scoped file operations, Git inspection, test runner | MCP gateway and more tools |
| Approvals | Explicit approval gates | Advanced policy profiles |
| Recovery | Persisted state and reconciliation | Advanced worker leases |
| n8n | Not required | Optional external automation adapter |
| Collaboration/cloud | Out of scope | Separate architecture if validated |

The PRD places MCP after initial tool integrations. Do not make MCP or n8n a prerequisite for the MVP. Do not implement cloud synchronization, multi-tenancy, distributed queues, or a graph database in the initial release.

---

## 2. Recommended technology stack

| Layer | Recommendation | Reason |
|---|---|---|
| Desktop shell | Tauri 2 | Lightweight native desktop packaging and controlled native capabilities |
| Frontend | TypeScript, HTML, CSS | Typed contracts and precise control over the specified design |
| UI framework | Retain the framework already present; if none exists, choose a small maintained approach | Avoid unnecessary migration and dependency weight |
| Backend | Python 3.x + FastAPI | Strong AI integration ecosystem, typed APIs, async I/O |
| Boundary validation | Pydantic through FastAPI | Runtime validation of requests and integration data |
| Persistence | SQLite | Local transactions without a database server |
| Database access | Python sqlite3 behind repository interfaces initially | Minimal dependencies and explicit transaction behavior |
| Migrations | Versioned SQL migration runner | Transparent and testable schema changes |
| Local inference | Ollama HTTP adapter | Supports local models without coupling domain logic to Ollama |
| Hosted inference | One provider SDK behind a common adapter | Meets MVP requirement while preserving replaceability |
| Runtime | Bounded asyncio scheduler/worker | Adequate for local sequential execution |
| Artifact storage | Local filesystem | Efficient storage for larger files and checksum verification |
| Testing | pytest and existing frontend test tooling | Covers domain, API, persistence, UI, and recovery |
| Packaging | Tauri bundle with managed Python backend process | Single desktop installation |

### 2.1 Technology constraints

- Pin supported Python, Node, Tauri, and dependency versions in repository configuration.
- Use standard-library functionality when it is sufficient.
- Do not add Redis, Celery, Kafka, Kubernetes, Docker requirements, an ORM, or a graph database for MVP.
- Do not require a cloud account or hosted control plane.
- Keep provider SDKs and protocol-specific libraries inside infrastructure adapters.
- Use the existing frontend build system if one is already established; do not migrate it as unrelated work.
- Bundle fonts or use system fallbacks; never require remote font loading.
- Select runtime versions compatible with Tauri 2 and all pinned dependencies before implementation.

### 2.2 Why a modular monolith

The initial product has one local desktop user and one local backend. A modular monolith avoids distributed transactions, service discovery, network deployment, and operational overhead. Clear interfaces preserve future extraction options without pre-building distributed infrastructure.

---

## 3. System topology and component responsibilities

~~~text
+--------------------------------------------------------------+
|                    Tauri Desktop Application                 |
|  +----------------------+  +-------------------------------+ |
|  | TypeScript UI        |  | Narrow Native Capability Layer| |
|  | Overview / Projects  |  | Lifecycle / Secure Commands   | |
|  | Workflows / Approvals|  | Credential Store if required  | |
|  +----------+-----------+  +---------------+---------------+ |
+-------------|------------------------------|-----------------+
              | authenticated local API      |
              v                              v
+--------------------------------------------------------------+
|                  Python FastAPI Backend                      |
|  API / Session Auth / Request Validation                     |
|                         |                                    |
|                 Application Services                         |
|                         |                                    |
|              Domain State Machine                             |
|                         |                                    |
|       Runtime Scheduler / Attempts / Recovery                |
|              |                          |                    |
|        Provider Adapters            Tool Adapters             |
+--------------|--------------------------|--------------------+
               |                          |
        Ollama / Hosted LLM       Files / Git / Test Runner
               |
        +------+-----------------------------+
        | SQLite metadata/state              |
        | Local artifact and log directories |
        +------------------------------------+
~~~

This is a logical diagram. All backend modules run in one local process in MVP.

### 3.1 Frontend

The TypeScript UI owns navigation, forms, workflow definition editing, run monitoring, approval review, and presentation of events, logs, artifacts, and validation results.

The frontend must not:
- Schedule or execute workflow tasks.
- Mutate authoritative workflow state locally.
- Decide whether an action is authorized.
- Call hosted LLM APIs or Ollama directly.
- Access SQLite directly.
- Execute arbitrary shell commands.
- Store provider credentials in browser storage or frontend configuration.

### 3.2 Backend

The Python backend owns:
- Local API authentication and authorization.
- Request validation and application use cases.
- Project, workflow, task, and approval domain rules.
- Workflow versioning and dependency validation.
- Scheduling, attempts, cancellation, timeout, retry, and recovery.
- Provider/tool adapter invocation.
- State and event persistence.
- Artifact metadata, usage records, and safe diagnostics.

### 3.3 Tauri native layer

Keep native commands narrow. Use this layer for backend process lifecycle, secure session bootstrap, window lifecycle, and narrowly scoped OS integration. Do not expose generic commands for arbitrary shell execution, unrestricted filesystem access, or arbitrary network access. Backend authorization remains mandatory even for requests from the UI.

---

## 4. Backend module design

Enforce dependency direction: domain → application → infrastructure implementations at composition time. Domain code must not import FastAPI, Pydantic transport models, provider SDKs, SQLite drivers, or Tauri types.

### 4.1 Domain

Define framework-independent entities, enums, value objects, and invariants:

- Project
- WorkflowDefinition and WorkflowVersion
- WorkflowRun
- TaskDefinition
- TaskRun and TaskAttempt
- ExecutionEvent
- ApprovalRequest and ApprovalDecision
- Artifact
- ExecutionPolicy and ValidationResult

Keep state-transition rules and dependency semantics in domain services.

### 4.2 Application

Implement use cases:

- Create/update project.
- Create, validate, version, and activate workflow definitions.
- Start workflow run.
- Schedule and execute task attempt.
- Validate task output.
- Request and decide approval.
- Retry, pause, cancel, resume, and recover runs.
- Retrieve activity, usage, and artifact metadata.
- Configure and health-check providers.

Application services coordinate domain rules and ports. They must not contain provider-specific HTTP or UI logic.

### 4.3 Runtime

The runtime is a bounded in-process subsystem:
- Scheduler selects tasks whose dependencies satisfy completion conditions.
- Worker executes one task attempt through a configured executor.
- State coordinator persists transitions.
- Cancellation is propagated to active adapters.
- Recovery reconciles interrupted attempts at startup.
- Bounded queues and semaphores prevent unbounded resource use.

Run tasks sequentially by default in MVP. Model records and task contracts so bounded parallelism can be added later without changing entity identity or persistence semantics.

### 4.4 Infrastructure

Implement ports for:
- SQLite repositories and migrations.
- Artifact filesystem storage.
- Hosted provider and Ollama adapters.
- Project-scoped file operations.
- Git status/diff.
- Allowlisted test command execution.
- OS credential store.
- Structured logging, clock, and ID providers.

All external I/O belongs in infrastructure adapters or explicit application ports.

---

## 5. Workflow execution and data flow

### 5.1 Workflow creation

1. User submits a workflow definition through the UI.
2. FastAPI authenticates the session and validates the request schema.
3. Application service validates task IDs, dependencies, cycles, executor types, timeout/retry policy, and approval requirements.
4. Domain layer creates a versioned immutable definition.
5. Repository persists the definition, tasks, and dependency edges in one transaction.
6. API returns the persisted definition ID and version.
7. UI renders the persisted representation.

### 5.2 Workflow start

1. UI requests execution of a specific workflow version.
2. Backend verifies project access and runnable definition.
3. Backend creates workflow run and task-run records transactionally.
4. Backend records start events.
5. Scheduler identifies eligible tasks.
6. API returns the run ID immediately; execution continues asynchronously.
7. UI reads status through bounded polling in MVP. Add server-sent events only if measured update latency justifies them.

### 5.3 Model task

1. Scheduler claims an eligible task and persists a running attempt.
2. Application resolves provider policy and adapter.
3. Policy checks local-only restrictions, provider allowlist, model capabilities, context limits, and applicable usage limits.
4. Provider adapter constructs and sends provider-specific request.
5. Adapter enforces timeout/cancellation and normalizes response.
6. Raw provider metadata and usage provenance are retained where available.
7. Output is stored as an artifact or bounded database record.
8. Deterministic validators check the output contract.
9. State coordinator records success only when required validation passes.
10. Events, duration, provider/model identity, and safe usage metadata are persisted.

Hosted transmission must be explicit. Never silently fall back from a local model to a hosted provider.

### 5.4 Tool task

1. Resolve proposed operation against a registered adapter and schema.
2. Check project scope, operation class, path/destination restrictions, and approval policy.
3. Persist an approval request when required and wait for a user decision.
4. Display exact action, arguments, target, and risk in the UI.
5. Revalidate authorization and action fingerprint after approval.
6. Execute through the adapter with bounded timeout and resource limits.
7. Validate and store the result.
8. Run task completion validators and persist the final state.

MVP tools should be limited to project-scoped file operations, Git status/diff inspection, and approved test execution. Avoid unrestricted command execution.

### 5.5 Dependency and retry rules

- A task is eligible only when every required dependency meets its completion condition.
- Failed/skipped dependencies are not successful unless an explicit workflow condition defines an alternate path.
- Validate the dependency graph for cycles before activation.
- Persist scheduler decisions and transitions.
- Every retry creates a new attempt record.
- Retry only classified transient failures within configured bounds.
- Use idempotency keys where supported.
- Reconcile uncertain external side effects before retrying.
- Move ambiguous outcomes to NEEDS_ATTENTION.
- Never claim exactly-once execution for arbitrary external operations.

### 5.6 Cancellation and shutdown

Cancellation stops scheduling new tasks and signals active adapters. Each adapter must declare whether cancellation is immediate, cooperative, or unsupported. On shutdown, stop accepting new runs, persist state, close clients/connections, and terminate managed child processes. Do not mark an operation cancelled if an external side effect may still be running without recording uncertainty.

---

## 6. Workflow state and recovery

Use the canonical PRD workflow states:

- DRAFT
- READY
- RUNNING
- WAITING_FOR_APPROVAL
- PAUSED
- SUCCEEDED
- FAILED
- CANCELLED
- NEEDS_ATTENTION

Use the canonical task states:

- PENDING
- READY
- RUNNING
- WAITING_FOR_APPROVAL
- RETRY_WAIT
- SUCCEEDED
- FAILED
- SKIPPED
- CANCELLED
- NEEDS_ATTENTION

Rules:
- Implement transitions through one domain transition service.
- Reject invalid transitions.
- Persist each transition and its event atomically.
- Persist before reporting a transition as durable.
- Keep attempt history immutable.
- Bind each run to an immutable workflow version.
- Distinguish output produced, output validated, and workflow verified.
- Never allow model output to write authoritative state.

### 6.1 Startup recovery

1. Open database and apply pending migrations.
2. Run configured integrity checks.
3. Find runs and attempts left nonterminal.
4. Identify attempts whose worker ownership/process is no longer valid.
5. Reconcile external operations where adapters support status lookup.
6. Resume only safe, idempotent, policy-permitted work.
7. Move uncertain operations to NEEDS_ATTENTION.
8. Record recovery events and expose a recovery summary.

Never delete completed attempts or artifacts during recovery.

---

## 7. Database and persistence

### 7.1 SQLite ownership

SQLite is the source of truth for project metadata, workflow definitions, runs, task state, attempts, approvals, events, provider configuration metadata, usage records, and artifact references. Only the backend accesses it.

- Use foreign keys, parameterized SQL, and explicit transactions.
- Keep transactions short; never hold one across model/tool/network calls.
- Use a connection policy compatible with the chosen concurrency model.
- Enable WAL only after backup/restore and target-platform behavior are tested.
- Apply versioned migrations and back up before destructive migrations.
- Store large artifacts on disk and metadata in SQLite.

### 7.2 Logical schema

| Table | Purpose | Key fields |
|---|---|---|
| projects | Local project registry | id, name, working_directory, timestamps |
| workflow_definitions | Stable workflow identity | id, project_id, name, current_version |
| workflow_versions | Immutable snapshots | id, workflow_id, version, definition_json, checksum |
| task_definitions | Tasks within a version | id, workflow_version_id, task_key, config_json |
| task_dependencies | Explicit dependency edges | task_id, depends_on_task_id |
| workflow_runs | Execution instances | id, workflow_version_id, status, timestamps |
| task_runs | Task state within a run | id, workflow_run_id, task_definition_id, status |
| task_attempts | Immutable attempts | id, task_run_id, attempt_number, executor, status |
| execution_events | Append-oriented event history | id, run_id, task_run_id, event_type, metadata_json |
| approval_requests | Scoped action proposals | id, run_id, action_fingerprint, action_json, expires_at |
| approval_decisions | User decision audit | id, approval_request_id, decision, decided_at |
| artifacts | File metadata | id, run_id, relative_path, media_type, size_bytes, sha256 |
| provider_configs | Non-secret configuration | id, provider_type, endpoint, credential_reference, enabled |
| usage_records | Usage and cost provenance | id, attempt_id, token fields, measurement_source |
| schema_migrations | Applied migrations | version, applied_at, checksum |

Use stable UUID/ULID identifiers and one consistent UTC timestamp representation. Do not store credentials in provider configuration rows.

### 7.3 Transaction boundaries

Use transactions for workflow version creation, run initialization, task claiming, state/event persistence, approval creation, and final attempt/artifact metadata. Never hold a database transaction open during external execution.

### 7.4 Artifact storage

Use an application-managed data directory separate from project source directories. Store relative paths and SHA-256 checksums. For important writes: validate destination, write to temporary file, validate format/size, compute checksum, atomically move into place, persist metadata, and verify readability. Defend against path traversal and symlink/junction escape.

---

## 8. API architecture

Use a versioned local REST API with JSON contracts and FastAPI OpenAPI generation. Prefix routes with /api/v1. Keep transport schemas separate from domain entities. Do not expose generic database CRUD endpoints; mutations must map to application use cases.

### 8.1 Resource groups

| Resource | Operations |
|---|---|
| Health | readiness, version, runtime status |
| Projects | list, create, retrieve, update, archive |
| Workflows | list, create, validate, retrieve, update, activate |
| Workflow runs | start, retrieve, list, pause, resume, cancel |
| Tasks | state, attempts, outputs |
| Approvals | list pending, inspect, approve, reject |
| Providers | list, configure, test connection, capabilities |
| Tools | registered tools, permission view, availability |
| Activity | list/filter events |
| Artifacts | list, metadata, controlled retrieval |
| Usage | usage records and provenance-aware aggregates |
| Settings | non-secret application settings |

### 8.2 Long-running requests

Starting a workflow returns a run identifier and accepted state; it does not hold the HTTP request until completion. Use bounded polling in MVP. Add SSE only if necessary. Avoid WebSockets unless a concrete bidirectional requirement exists.

### 8.3 Error response

Return a stable machine-readable error code, safe human-readable message, request/correlation ID, optional field validation details, and retryable indicator only when reliable. Never expose stack traces, secrets, internal paths, SQL, or raw provider error bodies.

### 8.4 API controls

- Validate every request at the boundary.
- Enforce request size, content type, timeout, and rate limits.
- Paginate unbounded collections.
- Use idempotency keys for retryable state-changing operations.
- Validate Origin and Host.
- Restrict CORS to the trusted desktop origin.
- Bind to loopback only by default.
- Never put credentials in URLs.

---

## 9. Authentication and authorization

### 9.1 Local authentication

The MVP is a single-user desktop product and does not require account registration or a remote identity provider. Loopback alone is not authentication: local processes and browser pages may attempt to call the API.

Use a per-launch session/pairing mechanism:
1. Tauri starts the backend with a protected bootstrap secret or secure inherited channel.
2. Backend creates a short-lived session credential.
3. Desktop UI receives it through a channel unavailable to arbitrary web content.
4. Mutating API calls require the credential.
5. Backend validates Origin/Host and rejects untrusted browser origins.
6. Session is invalidated on shutdown/restart.

Do not place credentials in URLs, persistent browser storage, logs, or exported artifacts. Document and test the selected secure handoff mechanism against local cross-origin requests.

### 9.2 Authorization

Enforce capabilities in the backend by:
- Project scope.
- Operation type: read, write, execute, publish, network.
- Target path/resource.
- Provider and data-transmission policy.
- Approval requirement.
- Tool/adapter allowlist.

The UI may hide unavailable actions but is never the authorization enforcement point.

### 9.3 Approval semantics

Bind approval to an immutable fingerprint of operation, arguments, target, project, and execution context. Revalidate immediately before execution. If parameters change or approval expires, require a new decision. Agents cannot approve their own actions. Silence, timeout, and model output are not approval.

---

## 10. External services and integration boundaries

### 10.1 Hosted model provider

MVP includes one hosted provider adapter. Store credentials in the OS credential store. The adapter handles authentication, request/response normalization, streaming where needed, usage metadata, timeouts, rate limits, and provider-specific errors. Identify the destination before project data is transmitted. Never silently fall back to hosted inference.

### 10.2 Ollama

Use a dedicated adapter and configurable local endpoint. Detect available models/capabilities at runtime. Handle missing models, model loading, resource exhaustion, timeout, and unavailable service distinctly. Do not expose Ollama publicly or download models without user authorization and disk-space checks.

### 10.3 Git and test runner

MVP tools: Git status/diff, scoped file read/write, and allowlisted test execution. Use argument arrays and shell-disabled process creation. Never concatenate model output into shell commands. Bound output, duration, and process resources. Require approval for protected writes, commits, pushes, or destructive operations according to policy.

### 10.4 MCP — future integration

MCP is a follow-up integration, not an MVP dependency. When added:
- Connect only to explicitly registered servers.
- Put transport behind a dedicated gateway/adapter.
- Validate discovered tool schemas and register only approved capabilities.
- Apply per-server, per-tool, and per-project permissions.
- Treat descriptions, resources, and responses as untrusted.
- Require approval for high-impact actions.
- Authenticate remote connections; tightly configure local stdio process launches.
- Bound timeouts, payload/output sizes, and process resources.
- Record identity, arguments or safe references, result, duration, and correlation ID.

MCP must not bypass orchestrator policy or state management.

### 10.5 n8n — optional future integration

n8n is not required by the PRD MVP. If introduced, treat it as an external automation executor, never as the authoritative workflow engine:
- Trigger only allowlisted workflow IDs.
- Use dedicated least-privilege credentials.
- Authenticate callbacks and process them idempotently.
- Persist external execution IDs and reconcile uncertain outcomes.
- Validate input and callback schemas.
- Do not let n8n bypass approvals, project scope, or backend state transitions.

### 10.6 No mandatory cloud infrastructure

No cloud database, hosted queue, telemetry service, or SaaS control plane is required. Network access is limited to the selected hosted provider and integrations explicitly invoked by the user.

---

## 11. AI architecture

LLMs perform bounded reasoning such as planning, drafting, classification, and proposing structured actions. They do not own orchestration, authorization, persistence, or verification. MVP does not require a multi-agent framework: use direct provider calls through a common adapter and explicit workflow tasks.

### 11.1 Provider contract

Expose provider/model identity, capability metadata, request/response contracts, streaming where supported, timeout/cancellation, normalized errors, usage metadata/provenance, and health checks. Keep SDK-specific details inside adapters. Do not assume equivalent tool calling, structured output, context windows, or token accounting across providers.

### 11.2 Prompt and output management

- Version reusable prompt templates.
- Record template version, model, provider, and relevant generation parameters per attempt.
- Treat all model output as untrusted.
- Validate structured output against explicit schemas.
- Keep raw response references separate from normalized domain objects.
- Never embed credentials in prompts.
- Retain only necessary prompt/response data according to privacy policy.
- Use deterministic validation and human review for consequential outputs.

### 11.3 Routing and fallback

Provider routing and fallback are P1. Implement only after base adapters and data-transmission policy are stable. Routing may consider task type, capability, context constraints, availability, user preference, and reliable usage/cost data. Record every decision and attempt. Local-to-hosted fallback requires explicit authorization and must never happen silently.

### 11.4 Usage and cost

Record provider-reported, estimated, imported, and unavailable measurements distinctly. Preserve input/output/cached/reasoning categories where available. Version pricing with currency and effective dates. Label cost as estimated unless reconciled with billing. Never invent missing values.

---

## 12. Frontend architecture and design

### 12.1 Information architecture

Follow design.md:
- Overview
- Projects
- Workflows
- Activity
- Conversation Library
- Usage & Costs
- Approvals
- Providers & Tools
- Execution History
- Settings

Destinations may be marked planned until implemented. Never show dead controls as functional.

### 12.2 Frontend modules

Separate app shell/navigation, feature pages, shared components, API client/typed contracts, feature state, and formatting utilities. Keep server state separate from transient UI state. After uncertain mutation outcomes, query authoritative backend state before offering retry.

### 12.3 Visual system

The design specification is authoritative. Use graphite foundation and restrained electric violet:

| Token | Value |
|---|---|
| App background | #080B12 |
| Sidebar | #0A0E17 |
| Top bar | #0B101A |
| Surface 1 | #101622 |
| Surface 2 | #151D2B |
| Surface 3 | #1B2636 |
| Subtle border | #202B3B |
| Strong border | #334155 |
| Primary text | #E8EDF5 |
| Secondary text | #A1ADBD |
| Muted text | #718096 |
| Accent | #8B7CFF |
| Success | #35B887 |
| Warning | #E5A84B |
| Danger | #E16B75 |
| Info | #65A9E8 |

Layout baseline: 232 px sidebar (68 px collapsed), 56 px top bar, 1440 × 900 reference viewport, 12-column content grid, 16 px gutters, and 24 px page padding. Use a 4 px spacing scale; 6 px control, 10 px card, and 12 px modal radius.

Typography: Inter or system sans-serif for UI; JetBrains Mono or system monospace for technical values. Avoid remote font loading. Use sentence case, clear alignment, and dense but legible desktop typography.

Avoid neon glow, excessive gradients, glassmorphism, generic AI motifs, and fake real-time data. Never rely on color alone for status. Distinguish succeeded from verified. Support keyboard navigation, visible focus, reduced motion, and WCAG 2.1 AA contrast targets where practical. Graphs must have a list/table alternative.

---

## 13. Environment and configuration

Keep non-secret settings in validated application configuration. Keep credentials in the OS credential store. Provide placeholder-only example configuration. Validate security-critical settings at startup and fail closed when invalid. Never let model output or imported workflow data modify process-level configuration.

| Category | Examples | Storage |
|---|---|---|
| Application | log level, data directory, bind address | app config |
| Runtime | concurrency, shutdown timeout, request limits | app config |
| Database | SQLite path, backup policy | app config |
| Artifacts | storage root, maximum size | app config |
| Provider | provider type, model, endpoint, enabled state | SQLite/config |
| Credentials | API keys, tokens | OS credential store |
| Ollama | endpoint, request timeout | app config |
| Security | trusted origin, session policy, allowed roots | protected app config |
| Execution | timeouts, retry limits, tool allowlists | workflow policy/config |

Use platform-appropriate application data directories for database, artifacts, logs, backups, and temporary staging. Keep project source directories separate from app-managed artifacts. Never commit real .env files, keys, tokens, or machine-specific paths.

---

## 14. Error handling and observability

### 14.1 Error categories

Use stable categories such as:
- ValidationError
- AuthenticationError
- AuthorizationError
- NotFoundError
- ConflictError
- ProviderUnavailableError
- ProviderRateLimitError
- ProviderTimeoutError
- ToolUnavailableError
- ToolExecutionError
- ApprovalRequiredError
- PolicyDeniedError
- ArtifactIntegrityError
- PersistenceError
- CancellationError
- RecoveryRequiredError
- InternalError

Map infrastructure exceptions to application errors at adapter boundaries.

### 14.2 Error rules

- Never suppress exceptions silently.
- Do not expose stack traces, secrets, SQL, or internal paths to the UI.
- Preserve exception chaining in protected internal logs.
- Distinguish transient and permanent failures.
- Retry only when classification and operation semantics permit.
- A timeout does not prove an external operation failed.
- Reconcile uncertain side effects before retrying.
- Surface actionable failure reasons and next steps.

### 14.3 Observability

Assign correlation IDs to workflow runs and task attempts. Record UTC timestamps, state transitions, provider/model, tool identity, duration, retries, validation result, and safe error metadata. Use structured local logs. Redact credentials and sensitive payloads. Logs are diagnostic evidence, not authoritative workflow state. Do not introduce telemetry in MVP; any future telemetry must be optional and disclosed.

---

## 15. Security boundaries

### 15.1 Trust boundaries

1. Desktop UI ↔ backend: authenticate local requests, validate origin and request schema.
2. Backend ↔ filesystem: enforce approved roots, canonicalize paths, prevent symlink/junction escape, use atomic writes.
3. Backend ↔ OS processes: allowlist executables, use argument arrays, disable shell interpolation, sanitize environment, bound resources.
4. Backend ↔ model providers: explicit data destination, secure credentials, timeouts, output validation.
5. Backend ↔ tools/MCP/n8n: registry, schema validation, least privilege, approval, audit, reconciliation.
6. Imported content ↔ trusted instructions: imported files, conversations, tool output, and model responses are untrusted data.

### 15.2 Required controls

- Bind the local API to loopback by default.
- Authenticate local requests and validate Origin/Host.
- Restrict Tauri capabilities and native commands.
- Keep secrets out of frontend bundles, plaintext database fields, logs, and artifacts.
- Store credentials in OS secure storage.
- Restrict filesystem operations to explicit project roots.
- Resolve and validate symlinks/junctions before sensitive operations.
- Use shell-disabled subprocess execution and executable allowlists.
- Apply timeouts and output/resource limits.
- Enforce provider data-transmission policy.
- Require action-scoped approvals for consequential operations.
- Validate imported workflows before activation.
- Treat model and tool output as untrusted.
- Keep TLS certificate verification enabled for remote services.
- Review dependency licenses and supply-chain risks.

Local-first does not protect against compromise of the user's OS account, malware, malicious dependencies, or a compromised configured provider. Do not claim the application is a complete OS sandbox. Add OS-level sandboxing only where supported and tested.

---

## 16. Testing strategy

### 16.1 Backend

- Unit tests: state transitions, dependency rules, policy evaluation, retry classification, idempotency.
- Repository tests: transactions, foreign keys, migrations, integrity.
- API tests: schemas, authentication, authorization, errors, pagination.
- Provider tests: deterministic fakes/local test servers.
- Tool tests: path restrictions, subprocess arguments, timeouts, output limits, cancellation.
- Recovery tests: interrupted process and uncertain external outcomes.

### 16.2 Frontend

- Component tests for state labels, approval details, forms, and error states.
- API contract tests against typed schemas.
- End-to-end tests for critical project/workflow/approval journeys.
- Accessibility checks for keyboard navigation, focus, semantic controls, and contrast.
- Verify loading, empty, stale, offline, unavailable, and failed states.

### 16.3 Security and end-to-end scenarios

At minimum verify:
1. Create project and workflow.
2. Run a sequential multi-task workflow.
3. Persist state and recover after backend restart.
4. Block dependent tasks after dependency failure.
5. Request and approve an exact protected action.
6. Reject changed or expired approval.
7. Cancel a running task and record uncertain outcomes.
8. Prevent path traversal and symlink escape.
9. Prevent shell injection and unauthorized executable invocation.
10. Redact secrets from logs and exports.
11. Reject malformed provider/tool output.
12. Verify artifact checksum and read-back.
13. Reject unauthorized local API and cross-origin requests.

### 16.4 LLM evaluation

Do not make default tests depend on live LLM output. Use fixed fixtures and recorded regression cases. Validate structured responses against schemas. Record provider, model, prompt version, parameters, and available usage metadata in live evaluations. Use human review for ambiguous or high-impact outcomes.

### 16.5 Performance

The PRD's provisional targets are:
- Local dashboard interactions p95 below 200 ms, excluding external requests.
- Ordinary workflow-state reads p95 below 100 ms.
- Status updates visible within one second under normal conditions.

Benchmark on supported hardware before claiming targets are met.

---

## 17. Build, packaging, and deployment

### 17.1 MVP deployment

Install and run locally:
- Tauri desktop executable and bundled frontend.
- Packaged Python backend managed as a sidecar/process.
- Local SQLite database.
- Local artifact and log directories.
- Optional user-managed Ollama service.
- Optional network access to configured hosted provider.

No cloud runtime, remote database, external queue, or container platform is required.

### 17.2 Process lifecycle

1. Tauri starts backend and waits for readiness.
2. Backend binds to loopback on a safely allocated port or secured local IPC transport.
3. Desktop establishes authenticated session.
4. App monitors backend health and reports unavailable state.
5. On shutdown, UI requests graceful backend shutdown.
6. Backend stops scheduling, persists state, closes clients/connections, and terminates managed child processes.
7. Desktop cleans up backend process if graceful shutdown fails while preserving recovery evidence.

Do not assume a fixed port is safe from collision or hijacking. Validate the bound endpoint and authenticate the process/session.

### 17.3 Packaging and release gates

- Build and sign desktop artifacts according to the release process.
- Bundle only required runtime components.
- Keep credentials outside the package and user data outside installation directory.
- Test database migration and upgrade behavior.
- Verify clean install, upgrade, uninstall, and data-retention expectations.
- Never delete user data during upgrade/uninstall unless explicitly requested and disclosed.

A release must pass frontend build, backend tests, migration tests, Tauri packaging, critical end-to-end flows, security regression tests, dependency/license review, and manual first-run/provider/recovery checks.

---

## 18. Recommended repository structure

This is a target structure, not a mandate for a large refactor. Adapt names to existing conventions and introduce folders incrementally.

~~~text
AI-Workflow-Orchestrator/
├── apps/
│   └── desktop/
│       ├── src/
│       │   ├── app/                  # Shell, routes, app providers
│       │   ├── components/           # Shared UI components
│       │   ├── features/
│       │   │   ├── overview/
│       │   │   ├── projects/
│       │   │   ├── workflows/
│       │   │   ├── approvals/
│       │   │   ├── activity/
│       │   │   ├── conversations/
│       │   │   ├── usage/
│       │   │   ├── providers/
│       │   │   └── settings/
│       │   ├── lib/                  # API client and utilities
│       │   ├── styles/               # Design tokens/global styles
│       │   └── types/                # Transport contracts
│       ├── src-tauri/
│       │   ├── src/                  # Narrow native commands/lifecycle
│       │   ├── capabilities/
│       │   └── tauri.conf.json
│       ├── package.json
│       └── build configuration
├── backend/
│   ├── app/
│   │   ├── main.py                  # FastAPI composition root
│   │   ├── api/
│   │   │   ├── routes/
│   │   │   ├── schemas/
│   │   │   ├── dependencies.py
│   │   │   └── error_handlers.py
│   │   ├── domain/
│   │   │   ├── projects/
│   │   │   ├── workflows/
│   │   │   ├── tasks/
│   │   │   ├── approvals/
│   │   │   └── policies/
│   │   ├── application/
│   │   │   ├── projects/
│   │   │   ├── workflows/
│   │   │   ├── execution/
│   │   │   ├── approvals/
│   │   │   └── providers/
│   │   ├── runtime/
│   │   │   ├── scheduler.py
│   │   │   ├── worker.py
│   │   │   ├── recovery.py
│   │   │   └── cancellation.py
│   │   ├── infrastructure/
│   │   │   ├── database/
│   │   │   ├── filesystem/
│   │   │   ├── providers/
│   │   │   │   ├── base.py
│   │   │   │   ├── ollama/
│   │   │   │   └── hosted/
│   │   │   ├── tools/
│   │   │   │   ├── filesystem/
│   │   │   │   ├── git/
│   │   │   │   └── test_runner/
│   │   │   ├── secrets/
│   │   │   └── logging/
│   │   └── config/
│   ├── migrations/
│   ├── tests/
│   │   ├── unit/
│   │   ├── integration/
│   │   ├── security/
│   │   └── recovery/
│   └── pyproject.toml
├── docs/
│   ├── PRD.md
│   ├── DESIGN.md
│   ├── ARCHITECTURE.md
│   └── ADR/
├── scripts/
├── .gitignore
└── README.md
~~~

Do not create duplicate document roots if the repository already stores product documents elsewhere. Keep shared contracts explicit; frontend code must not import backend internals.

---

## 19. Implementation sequence

### Phase 0 — Foundation

- Confirm runtime versions and repository structure.
- Establish Tauri/TypeScript and FastAPI lifecycle.
- Define local API session security and data directories.
- Create SQLite connection policy and migration runner.
- Add structured logging and test harness.

**Exit:** Desktop starts/stops backend securely; database initializes and survives restart.

### Phase 1 — Durable workflow core (MVP)

- Project management.
- Workflow definition/versioning.
- Task/dependency validation.
- Sequential scheduler and state machine.
- Task attempts and execution events.
- SQLite persistence and recovery.
- Basic overview and run UI.

**Exit:** Multi-task workflow executes, persists, recovers after restart, and reports outcomes accurately.

### Phase 2 — Providers and safe tools (MVP)

- One hosted provider and Ollama adapters.
- Provider configuration and health checks.
- Project-scoped file operations.
- Git status/diff.
- Allowlisted test runner.
- Approval queue and action-scoped authorization.

**Exit:** Model/tool activity is traceable, policy-controlled, and recoverable.

### Phase 3 — Verification and release readiness (MVP)

- Deterministic validation framework.
- Cancellation and timeout handling.
- Error taxonomy and activity history.
- Artifact checksums and atomic writes.
- Recovery and security regression suite.
- Desktop packaging and release gates.

**Exit:** Critical PRD P0 acceptance criteria pass on supported Windows environment.

### Phase 4 — P1 capabilities

- Policy-driven provider routing and fallback.
- Workflow templates and import/export.
- Usage/cost analytics with provenance.
- Conversation import and context handoff.
- Additional provider adapters.

### Phase 5 — Optional integrations and scale

- MCP gateway.
- n8n adapter.
- Bounded parallel tasks.
- Project graph read model and visualization.
- Advanced policy profiles.

Each later phase requires user validation and architecture review. Do not pre-build infrastructure for hypothetical scale.

---

## 20. Future scalability without premature infrastructure

- **Parallel execution:** add bounded concurrency after dependency, cancellation, and recovery semantics are proven.
- **MCP:** add gateway and permission registry; orchestrator remains policy authority.
- **n8n:** add authenticated connector and callback reconciliation; never delegate authoritative state.
- **Event delivery:** add SSE if polling is insufficient; SQLite remains durable event source.
- **Project graph:** derive a read model from relational records; add graph storage only if proven query requirements justify it.
- **Provider routing:** add after capability metadata and data classifications are reliable.
- **Separate worker process:** extract only if isolation, resource control, or crash behavior requires it.
- **Cloud/team deployment:** requires a separate identity, tenant isolation, secrets, deployment, and threat-model design. It is not a configuration toggle for the local MVP.

---

## 21. Architecture decision summary

| Decision | Choice | Rationale |
|---|---|---|
| Deployment | Local desktop | Matches PRD privacy and product scope |
| Desktop | Tauri 2 | Lightweight native shell |
| UI | TypeScript | Typed contracts and design control |
| Backend | Python + FastAPI | AI ecosystem and local API |
| Architecture | Modular monolith | Simplest reliable deployment |
| Database | SQLite | Local transactional durability |
| Artifacts | Local filesystem | Efficient large-file storage |
| Runtime | Bounded in-process scheduler | No external queue required |
| Execution | Sequential first | Simpler correctness and recovery |
| Models | One hosted adapter + Ollama | Meets P0 requirements |
| Tools | Narrow adapters | Least privilege and auditability |
| MCP | Follow-up | PRD places it after initial tool integrations |
| n8n | Optional future adapter | Not required by PRD |
| Authentication | Local session/pairing | No account, but protects local API |
| Verification | Deterministic validators | Prevents unverified output being called success |

---

## 22. Architecture acceptance criteria

- [ ] Desktop/backend lifecycle is defined and tested.
- [ ] Backend is sole authority for workflow state and authorization.
- [ ] Workflow definitions are versioned and immutable per run.
- [ ] State transitions and events persist transactionally.
- [ ] Sequential dependency scheduling works without distributed queue.
- [ ] Restart recovery reconciles interrupted operations safely.
- [ ] One hosted provider and Ollama work through provider-neutral adapters.
- [ ] Hosted data transmission is explicit and local-only policy is enforceable.
- [ ] Tool execution is scoped, allowlisted, and approval-aware.
- [ ] SQLite migrations and artifact integrity checks are implemented.
- [ ] Local API authentication, origin validation, and loopback binding are tested.
- [ ] Critical failure, security, and recovery paths have automated tests.
- [ ] UI follows design.md and displays authoritative data only.
- [ ] Packaging and upgrade behavior preserve user data.

**Final architectural rule:** Build the smallest system that can execute a workflow, persist every meaningful transition, recover safely, enforce authorization, and prove whether its outputs passed validation. Add complexity only when a demonstrated product requirement requires it.
