# AI Workflow Orchestrator - Technical Architecture

**Document type:** Technical Architecture and Security Design  
**Status:** Proposed baseline for implementation  
**Version:** 1.0  
**Related specification:** [prd.md](./prd.md)  
**Deployment:** Local-first Windows desktop MVP; extensible to macOS and Linux  
**Stack:** Tauri 2, TypeScript, Python, FastAPI, SQLite, local filesystem, hosted LLM APIs, Ollama

---

## 1. Executive Architecture Decision

Build a **local-first desktop application with a managed local orchestration service**. Use a modular monolith with clear domain boundaries and ports/adapters. Do not start with microservices, Redis, Kafka, Kubernetes, a graph database, or a hosted control plane.

The desktop UI is a control surface. The Python backend is the authority for workflow state, scheduling, authorization, provider calls, tool execution, persistence, usage accounting, and recovery.

### Core decisions

| Area | Decision |
|---|---|
| Desktop shell | Tauri 2 |
| UI | TypeScript, strict type checking |
| Backend | Python + FastAPI |
| Architecture | Layered modular monolith |
| Persistence | SQLite with migrations |
| Artifact storage | Local filesystem + database metadata |
| Workflow engine | Durable state machine + bounded scheduler |
| Model integration | Provider adapter interfaces |
| Local inference | Ollama adapter |
| Tool execution | Policy-enforced executor |
| Graph | Relational nodes/edges; UI visualization |
| Analytics | Immutable request usage records + rebuildable aggregates |
| Secrets | OS credential store |
| Testing | pytest, integration, security, recovery and UI tests |

### Architectural invariants

1. Backend domain logic owns all authoritative state transitions.
2. Model output is untrusted input and never grants permissions.
3. Every consequential tool action passes through backend policy evaluation.
4. State transitions and corresponding events are persisted transactionally.
5. Retries create new task-attempt records.
6. Arbitrary external side effects are not promised to execute exactly once.
7. Provider and tool adapters cannot bypass domain policies.
8. Unknown token, context, price, and provenance values remain unknown, not fabricated or silently set to zero.
9. External data transfer occurs only through explicitly configured integrations.
10. Project graphs and analytics are read models, not execution authorities.

---

## 2. System Context and Trust Boundaries

### 2.1 Logical system

~~~mermaid
flowchart LR
    U[User] --> UI[Tauri Desktop UI]
    UI --> API[Local FastAPI Service]
    API --> DB[(SQLite)]
    API --> FS[Project and Artifact Storage]
    API --> SEC[OS Credential Store]
    API --> LLM[Provider Adapters]
    LLM --> HOSTED[Hosted LLM APIs]
    LLM --> OLLAMA[Local Ollama]
    API --> EXEC[Policy-enforced Tool Executor]
    EXEC --> WORK[Approved Project Directories]
    EXEC --> GIT[Git and Test Tools]
    API --> READ[Graph and Analytics Queries]
    READ --> UI
~~~

### 2.2 Trust zones

| Zone | Components | Security assumption |
|---|---|---|
| Z0 | User, selected project directories | Project content may be malicious |
| Z1 | Tauri UI and frontend assets | All UI input is untrusted |
| Z2 | FastAPI, domain services, scheduler | Trusted application code; local access only |
| Z3 | SQLite, artifacts, logs, credential store | Must be protected from accidental disclosure |
| Z4 | Shell commands, Git, test runners, MCP tools | Tool input/output is untrusted |
| Z5 | Hosted providers and external APIs | External network and provider behavior are untrusted |

### 2.3 Boundary rules

- Never rely on frontend checks as the only authorization control.
- Validate all requests and imported workflow definitions at the backend boundary.
- Do not bind the local API to 0.0.0.0 by default.
- Loopback access alone is not authentication.
- LLM output cannot expand filesystem roots, command permissions, provider access, or approval scope.
- Treat repository files, prompts, imported conversations, model responses, tool output, and MCP responses as potentially adversarial.
- Separate trusted application policy from untrusted project and model content.

---

## 3. Component Architecture

### 3.1 Suggested repository structure

~~~text
src/
  desktop-ui/                 # Tauri + TypeScript presentation
  backend/
    app/
      api/                     # FastAPI routes and schemas
      application/             # Use cases and transaction orchestration
      domain/
        projects/
        workflows/
        tasks/
        policies/
        approvals/
        activities/
        artifacts/
        relationships/
        usage/
      infrastructure/
        persistence/
        providers/
        tools/
        filesystem/
        credentials/
        logging/
      runtime/
        scheduler/
        recovery/
        workers/
      security/
        authentication/
        authorization/
        validation/
        redaction/
        path_policy/
    migrations/
    tests/
~~~

### 3.2 Layer responsibilities

**Presentation**
- Project dashboard, workspace, activity timeline, graph, analytics, workflow editor, execution history, approvals, provider settings.
- Holds transient UI state only.
- Sends commands to the backend and renders authoritative responses.
- Never stores provider secrets or makes authorization decisions.

**API**
- Authenticates the local client; validates origin, request schema, content type, and payload size.
- Converts requests into application commands and queries.
- Returns versioned response schemas and safe error codes.
- Contains no workflow business logic.

**Application**
- Implements use cases: create project, validate workflow, start run, approve action, cancel run, import activity, query graph, query usage.
- Coordinates domain services and transactions.
- Emits durable events through the persistence boundary.

**Domain**
- Defines entities, invariants, state transitions, dependency rules, policy decisions, usage semantics, and relationship types.
- Has no dependency on FastAPI, SQLite, Tauri, provider SDKs, or shell commands.

**Infrastructure**
- Implements repositories, provider adapters, tool adapters, credential access, filesystem operations, migrations, and structured logging.

**Runtime**
- Schedules eligible tasks, manages bounded concurrency, cancellation, leases and heartbeats, and restart recovery.

**Security**
- Implements authentication, authorization, secret redaction, path validation, command policy, import validation, and security audit events.
- These controls are enforced by application services and executors, not merely exposed as UI options.

### 3.3 Request and execution sequence

1. User submits a workflow or action through the desktop UI.
2. FastAPI authenticates the local client and validates the request.
3. Application service loads the project and policy.
4. Domain validation checks workflow version, dependencies, and state transition.
5. Policy engine evaluates data classification, provider permissions, tool permissions, file scope, and approval requirements.
6. A short database transaction persists the command result, state, event, and any outbox record.
7. Scheduler leases eligible tasks and dispatches them to a bounded worker.
8. Provider or tool adapter performs the operation with timeout and capability restrictions.
9. Result is normalized, validated, redacted, and stored as an activity or artifact.
10. A transaction persists attempt outcome, usage record, events, and next eligible state.
11. UI receives updated state through polling initially.
12. Graph and analytics queries read persisted records and refresh their projections.

---

## 4. Workflow Runtime and Durable Execution

### 4.1 State machine

Implement explicit domain transition functions. Do not permit generic CRUD endpoints to update arbitrary workflow statuses.

~~~python
def transition_task(task, target_state, *, reason, occurred_at):
    if target_state not in ALLOWED_TRANSITIONS[task.status]:
        raise InvalidStateTransition(task.status, target_state)

    return task.with_transition(
        target_state=target_state,
        reason=reason,
        occurred_at=occurred_at,
    )
~~~

Persist the transition and its event in the same database transaction.

### 4.2 Scheduler

Use a single-process scheduler for the desktop MVP.

- Find tasks whose dependencies satisfy their completion conditions.
- Acquire a durable lease in a short SQLite transaction.
- Execute outside the transaction.
- Renew heartbeat/lease for long-running operations.
- Persist outcome and release lease.
- Enforce global and per-provider concurrency limits.
- Never hold a database transaction open while waiting on an LLM request or subprocess.

A lease includes owner/run ID, acquisition time, expiry, and heartbeat. On restart, expired leases are reconciled before work is re-dispatched.

### 4.3 Retry and idempotency

- Retry only errors classified as transient and permitted by task policy.
- Use bounded exponential backoff with jitter and maximum attempts.
- Persist each attempt separately.
- Generate idempotency keys where the destination supports them.
- Before retrying a side-effecting operation, inspect operation records or reconcile external state.
- Require approval when a prior side effect cannot be safely determined after a crash.
- Never automatically retry destructive or non-idempotent shell commands.

**Delivery semantics:** durable at-least-once task dispatch with idempotency and reconciliation where supported. No exactly-once guarantee for arbitrary external effects.

### 4.4 Cancellation

Cancellation is a persisted request, not proof that an external operation stopped.

- Stop scheduling new tasks.
- Signal supported adapters and subprocesses.
- Track cancellation requested, acknowledged, and completed separately.
- Terminate subprocess process groups after a configured grace period only where safe.
- Reconcile uncertain external operations; use NEEDS_ATTENTION when outcome cannot be established.
- Preserve completed task and attempt records.

### 4.5 Approval records

Bind each approval to:
- Exact action type and normalized parameters or parameter hash.
- Project, workflow, task, and attempt identifiers.
- Target files/resources and requested permissions.
- Expiry and one-time approval identifier.
- User decision, timestamp, and policy version.

Any material change to command, target path, parameters, or permissions invalidates the approval. Approval is never reusable for unrelated actions.

---

## 5. Model Provider and Routing Architecture

### 5.1 Provider interface

~~~python
from typing import Protocol

class ModelProvider(Protocol):
    async def get_capabilities(self, model_id: str) -> "ModelCapabilities": ...

    async def generate(
        self,
        request: "ModelRequest",
        *,
        timeout_seconds: float,
    ) -> "ModelResponse": ...

    async def health_check(self) -> "ProviderHealth": ...
~~~

Adapters return provider/model IDs, response content, structured output where supported, provider request ID, finish reason, normalized usage, versioned provider-specific usage metadata, latency, error classification, and capability information.

### 5.2 Capability-aware routing

Maintain a model catalog containing:
- Provider/model/version identifiers.
- Context window and output limit, source and verification date.
- Tool calling and structured output support.
- Vision/audio capabilities where applicable.
- Streaming support and usage metadata availability.
- Local/hosted classification and data policy restrictions.

The router evaluates task requirements, model capabilities, provider availability, user policy, context constraints, and budgets. It records candidate decisions and the selected model. Fallback must be explicitly permitted and must never silently move sensitive content from local to hosted inference.

### 5.3 Context assembly

Build each request from explicit, versioned inputs:
- Application-controlled system/developer policy.
- Project requirements and approved context references.
- Task inputs.
- Relevant artifacts or summaries.
- Tool results with provenance.
- Output schema and validation requirements.

Store a context manifest and assembly metadata. Prefer bounded excerpts and references over repeatedly loading entire repositories. Apply token budgeting before dispatch, reserving output allowance and a safety margin.

Do not claim to observe hidden provider-side context. Record only what the adapter can measure or estimate.

### 5.4 Ollama adapter

- Connect only to the configured local endpoint.
- Default to loopback and validate configured hosts against an explicit allowlist.
- Do not expose Ollama through a public tunnel or public interface by default.
- Enforce timeouts and bounded concurrency.
- Detect model availability and capabilities at runtime.
- Mark token counts estimated when the runtime does not provide authoritative counts.
- Never send local prompts to hosted fallback unless project data policy explicitly allows it.

### 5.5 Hosted providers

- Use TLS with certificate verification.
- Keep API keys in the OS credential store.
- Attach credentials only to the intended provider host.
- Never log authorization headers or full credential-bearing request objects.
- Apply provider-specific rate limits, timeouts, response-size limits, and retry policies.
- Validate response shape and finish reason.
- Treat text, tool calls, and structured output as untrusted.

---

## 6. Tool Execution and Sandboxing

### 6.1 Capability manifests

Every tool adapter declares:
- Tool identity and version.
- Permitted operations.
- Project scope.
- Read/write/network permissions.
- External side-effect classification.
- Cancellation support.
- Approval requirements.
- Input/output schemas.
- Timeout and resource limits.

The policy engine authorizes each invocation. Installing a tool does not grant broad access.

### 6.2 Filesystem protections

- Resolve project roots to canonical absolute paths.
- Reject paths escaping the approved root after normalization and symlink resolution.
- Handle traversal, alternate separators, symlinks, Windows junctions, drive paths, and UNC paths.
- Re-check the resolved target immediately before sensitive operations to reduce TOCTOU risk.
- Use atomic writes: temporary file in the same filesystem, validate, then replace after approval.
- Never overwrite protected user files without explicit approval.
- Store generated artifacts separately from canonical source until accepted.
- Record checksums and managed file versions.
- Do not follow arbitrary links from imported content without policy checks.

### 6.3 Subprocess execution

- Prefer argument arrays with shell=False.
- Never interpolate model-generated strings into shell commands.
- Use an allowlist of executable paths and command templates for MVP actions.
- Set explicit working directory, environment allowlist, timeout, output cap, and process-tree handling.
- Remove inherited secrets and unnecessary environment variables.
- Imported workflows cannot execute commands until reviewed and explicitly authorized.
- Treat stdout/stderr as untrusted and redact secrets before persistence.
- On Windows, manage child process trees using supported process-group or Job Object controls where available.
- Do not assume launching a subprocess automatically sandboxes it.

### 6.4 Sandboxing levels

**MVP:** Narrow command allowlist, approved project roots, no arbitrary administrator privileges, explicit approval for writes and side effects, bounded execution.

**Later:** Optional isolated worker process, restricted OS account, Windows Job Objects/AppContainer or equivalent controls, and container/VM execution where configured.

Do not require Docker for normal MVP operation. A container is not a complete security boundary unless mounts, capabilities, network, identity, and resource limits are restricted.

### 6.5 MCP and external tools

Treat each MCP server as a separate trust boundary:
- Register and approve each server explicitly.
- Record server identity, version, and transport.
- Expose only selected tools.
- Validate tool schemas and arguments.
- Require approval for consequential operations.
- Apply timeouts and response-size limits.
- Record invocation provenance.
- Never trust tool descriptions or responses as policy instructions.
- Never forward provider credentials to MCP servers without a documented need and explicit approval.

---

## 7. Desktop and Local API Security

### 7.1 Local service

The FastAPI service shall:
- Bind to loopback only by default.
- Use a dynamically allocated port where practical.
- Generate a high-entropy per-session or per-installation access token.
- Deliver the token to Tauri through a protected startup channel, not command-line arguments or logs.
- Require authentication for sensitive and state-changing endpoints.
- Validate expected Tauri Origin and reject unexpected browser origins.
- Apply strict CORS; CORS is not authentication.
- Validate content types and request/body sizes.
- Rate-limit sensitive endpoints and provider/tool dispatch.
- Shut down cleanly with the desktop app.
- Reject remote network clients.

Restrict local data directory permissions to the current user. Loopback and a random token do not protect against malware already running as the same user.

### 7.2 Tauri security

- Use restrictive Content Security Policy; avoid unsafe-eval and broad remote script sources.
- Keep Tauri capabilities and native command permissions minimal.
- Expose only narrowly scoped native commands.
- Validate arguments in native handlers and Python services.
- Avoid unrestricted shell, filesystem, and HTTP plugin permissions.
- Isolate remote content if ever supported.
- Render untrusted model/project HTML as sanitized Markdown with raw HTML disabled by default.
- Prevent external navigation from inheriting privileged application context.

### 7.3 Localhost threats

A malicious website may attempt to call a localhost service. Mitigations include a session token, strict Origin/Host validation, no permissive CORS, content-type validation, and avoiding cookie-based ambient authority. Test these controls in a real browser/webview environment.

---

## 8. Authorization, Credentials, and Privacy

### 8.1 Permission model

MVP is single-user but authorization remains explicit:
- Read project.
- Read/write artifacts.
- Invoke provider.
- Read/write files.
- Execute approved commands.
- Perform Git operations.
- Access network integrations.
- Approve a specific action.

Enforce policy in application services and tool executors. Do not introduce multi-user RBAC until collaboration is a real requirement.

### 8.2 Secret management

- Store provider keys in Windows Credential Manager or an OS-backed credential store through a minimal native bridge.
- Store only secret references in SQLite.
- Never put keys in workflow JSON, project files, source control, logs, crash reports, analytics, or exports.
- Redact known secret patterns and configured secret values before logging.
- Do not expose secrets back to the UI after initial entry.
- Scope each secret to provider and purpose.
- Support revocation and replacement.
- Never read arbitrary environment variables and forward them to child processes.

If secure OS credential storage is unavailable, fail closed for hosted calls rather than silently writing plaintext secrets.

### 8.3 Local data protection

Use OS disk encryption where available and document its role. Restrict application data permissions. Do not claim local-first protects data from malware or other processes running as the same user. Application-level encryption of database/artifacts is a separate feature requiring a documented key lifecycle.

---

## 9. Persistence and Data Model

### 9.1 SQLite

SQLite is authoritative for workflow metadata, execution events, relationships, approvals, integration metadata, and usage records.

Recommended after compatibility testing:
- WAL mode for concurrent readers and a single writer.
- Foreign-key enforcement on every connection.
- Configured busy timeout.
- Short transactions.
- Explicit migrations.
- Integrity checks in diagnostics.
- Regular backup/checkpoint strategy.

Use one controlled database access layer. Do not share raw SQLite connections arbitrarily across async tasks or threads. Never hold a transaction open during network calls or subprocess execution.

### 9.2 Core tables

| Table | Purpose |
|---|---|
| projects | Project identity and approved root |
| project_context_items | Requirements, decisions, context references |
| workflow_definitions | Versioned workflow definitions |
| workflow_runs | Durable workflow executions |
| task_definitions | Workflow dependency graph |
| task_runs | Task execution instances |
| task_attempts | Immutable attempt history |
| execution_events | Append-only state/activity events |
| approval_records | Action-scoped approvals |
| artifacts | Storage path, checksum, metadata |
| ai_activities | Captured model/tool activity and provenance |
| activity_file_references | Activity-to-file links |
| relationship_nodes | Typed graph nodes |
| relationship_edges | Explicit typed relationships |
| provider_configs | Non-secret config and secret references |
| model_catalog | Model capabilities/context metadata |
| ai_request_usage | Per-request usage observations |
| pricing_records | Versioned pricing data |
| usage_budgets | User-defined thresholds |
| integration_registry | Authorized external integrations |
| scheduler_leases | Durable task ownership/heartbeat |
| schema_migrations | Applied migration history |

Use normalized relational records for core entities. JSON fields may hold versioned provider-specific metadata or flexible task configuration. Fields used for constraints, joins, filters, and state transitions should be first-class columns.

### 9.3 Transactional events

Persist state changes and their execution event in one transaction. If a state change must trigger asynchronous processing, write an outbox record in that transaction. Consumers must be idempotent.

### 9.4 Artifact storage

- Store large outputs and binaries on disk; metadata and references in SQLite.
- Use opaque generated storage names.
- Record SHA-256, size, media type, source activity, and creation time.
- Validate paths remain within application data directory.
- Apply size limits and safe filename handling.
- Keep canonical source files distinct from generated artifacts.
- Never clean up artifacts referenced by active or retained workflows.

### 9.5 Backup and recovery

Use SQLite's backup API or a coordinated backup/checkpoint procedure. Include database, artifact metadata, and selected artifacts in a manifest. Verify checksums on restore. Restore into a new location first; do not overwrite active workspace without confirmation. Version export formats and migrations. OS credential-store secrets are not automatically part of project exports.

---

## 10. Project Graph and Provenance

Use relational relationship_nodes and relationship_edges tables for MVP. Do not introduce a graph database.

Node records contain stable ID, project ID, entity type/ID, safe display label, status, timestamps, and provenance reference.

Edge records contain stable ID, source and target node IDs, relationship type, provenance, confidence/source classification, and event timestamp where available.

Rules:
- Create relationships from explicit application events or user actions.
- Do not infer causality from time proximity.
- Enforce valid references and reject inappropriate self-edges.
- Query project-scoped neighborhoods, not all historical nodes.
- Filter and paginate server-side.
- Index project ID, entity type, relationship type, and endpoint IDs.
- Keep optional layout coordinates separate from domain relationship semantics.

The workflow definition remains authoritative for task dependencies. A graph is a visualization and navigation feature, not the state machine.

Activity provenance distinguishes event time from capture time and records source tool, provider/model, capture method, confidence, related project/workflow/task/files/artifacts, and content checksum/reference. Incomplete external capture must be disclosed.

---

## 11. Token, Context, and Cost Analytics

### 11.1 Usage pipeline

~~~mermaid
flowchart TD
    REQ[Model Request] --> ADAPTER[Provider Adapter]
    ADAPTER --> RESP[Normalized Response]
    RESP --> NORMALIZE[Usage Normalizer]
    NORMALIZE --> CLASSIFY[Measurement Classification]
    CLASSIFY --> STORE[(Immutable Request Usage)]
    STORE --> AGG[Rebuildable Aggregations]
    AGG --> DASH[Project and Global Dashboards]
~~~

### 11.2 Usage semantics

Capture when available:
- Input and output tokens.
- Total tokens.
- Cached input and reasoning tokens.
- Provider request ID.
- Provider/model/version and source tool.
- Project/workflow/task association.
- Measurement method: provider_reported, estimated, imported, unavailable.
- Tokenizer/version for estimates.
- Context limit and metadata source/version.
- Latency, outcome, timestamp, and cost method.

Preserve raw provider usage fields in a versioned structure. Normalize only fields whose semantics are understood.

### 11.3 Context calculation

Context utilization may be calculated only when the token count and model context capacity are comparable:

context utilization = comparable input tokens / verified context-window tokens × 100

Do not conflate prompt tokens, output allowance, actual output tokens, context capacity, and hidden provider-side tokens. If exact accounting is unavailable, label the calculation as an estimate and explain the basis. Context metadata must be model-version-aware and include source and verification date.

### 11.4 Cost

- Use versioned pricing records with effective dates.
- Preserve input/output/cached categories separately.
- Record currency and billing unit.
- Store calculation inputs and pricing record ID.
- Distinguish provider-reported billing from calculated estimates.
- Do not imply taxes, discounts, credits, subscriptions, or invoice totals are included unless supplied by provider data.

### 11.5 Aggregation and deduplication

Raw per-request usage is the source of truth. Aggregates must be rebuildable. Prefer provider request IDs for deduplication. If unavailable, use cautious fingerprints and flag uncertain matches for reconciliation. Do not sum incomparable token categories across providers without preserving provider grouping and measurement semantics.

---

## 12. API Design

### 12.1 Principles

- Version under /api/v1.
- Use strict Pydantic request/response schemas.
- Use stable error codes and correlation IDs.
- Paginate collections.
- Enforce body size, string length, array size, and nesting-depth limits.
- Avoid generic endpoints for arbitrary database updates.
- Use command endpoints for state transitions and approvals.
- Keep provider SDK objects out of public API contracts.

### 12.2 Initial endpoint groups

| Method and path | Purpose |
|---|---|
| POST /api/v1/projects | Create project |
| GET /api/v1/projects | List projects |
| GET /api/v1/projects/{id} | Read project |
| POST /api/v1/projects/{id}/workflows | Create workflow definition |
| POST /api/v1/workflows/{id}/validate | Validate workflow |
| POST /api/v1/workflows/{id}/runs | Start workflow run |
| GET /api/v1/runs/{id} | Read run status |
| POST /api/v1/runs/{id}/cancel | Request cancellation |
| POST /api/v1/approvals/{id}/decision | Decide a specific approval |
| GET /api/v1/projects/{id}/graph | Query project graph |
| GET /api/v1/projects/{id}/usage | Query project usage |
| GET /api/v1/usage/summary | Global usage summary |
| GET /api/v1/providers | List provider metadata |
| POST /api/v1/providers/{id}/health-check | Test provider connection |
| POST /api/v1/imports/validate | Validate import before acceptance |

Exact payloads, response models, and error codes must be specified before implementation.

### 12.3 Async execution

Starting a workflow returns a run ID and persisted state; it does not wait for completion. Use bounded polling initially. Add SSE/WebSockets only after reconnect, authorization, and backpressure behavior are specified.

---

## 13. Secure Coding Standards

### 13.1 Python

- Type public service boundaries and use strict Pydantic schemas.
- Keep domain code free of framework/provider SDK dependencies.
- Use parameterized SQL only.
- Use explicit transactions and rollback on failure.
- Avoid broad exception handlers that hide programming errors.
- Catch known exceptions at adapter/API boundaries and map to safe error codes.
- Never return stack traces, local paths, secrets, or raw provider errors to UI.
- Use async I/O for network operations; isolate blocking SDKs and subprocesses in bounded workers.
- Apply timeouts to every network and subprocess operation.
- Limit response and artifact sizes.
- Validate imported workflow schemas and deserialized JSON.
- Use safe temporary files and atomic replacement.
- Never use pickle or unsafe deserialization for imported/persisted user content.
- Avoid eval, dynamic code execution, and model-generated Python execution.
- Use structured logging with redaction.

### 13.2 TypeScript/UI

- Enable TypeScript strict mode.
- Validate backend responses at trust boundaries.
- Render model output as text or sanitized Markdown; raw HTML disabled by default.
- Avoid dangerouslySetInnerHTML unless a reviewed sanitizer is essential.
- Never place secrets in localStorage, sessionStorage, frontend environment variables, or URLs.
- Validate URL schemes and destinations before opening external links.
- Treat UI permission state as informational; backend enforces policy.
- Bound graph node counts and log rendering.
- Virtualize long lists and paginate histories.
- Debounce search/filter input and cancel stale requests.

### 13.3 Rust/Tauri

- Keep native commands narrow and typed.
- Validate paths and arguments in native code as well as Python.
- Do not expose generic shell execution to frontend.
- Restrict Tauri capabilities to required windows and commands.
- Review plugin permissions and dependencies.
- Never pass secrets in process arguments or diagnostic logs.
- Document and test any unsafe Rust blocks.

### 13.4 Configuration/imports

- Validate configuration against versioned schemas.
- Reject unknown privileged fields.
- Imported workflows start with no tool permissions and no approvals.
- Require preview and explicit activation.
- Resolve imported paths relative to approved project roots; reject external absolute paths unless approved.
- Never automatically execute scripts, hooks, package installation, or commands embedded in imported data.

---

## 14. Secure Performance and Code Optimization

Optimization must preserve authorization, auditability, and data integrity. Never weaken validation, redaction, approval gates, or transaction guarantees for speed.

### 14.1 Resource-aware MVP

Target modest developer hardware:
- One backend process and one scheduler.
- Bounded worker pools and queues.
- Low default concurrency for local inference.
- No mandatory Docker, Redis, Kafka, or external database.
- Lazy-load optional integrations.
- Do not load entire repositories into memory.
- Stream large responses and file operations where supported.
- Store large artifacts on disk, not SQLite blobs.
- Keep graph queries project-scoped and bounded.

### 14.2 Database optimization

- Add indexes based on measured query patterns.
- Index foreign keys and common filters: project, status, provider, model, timestamps.
- Use keyset pagination for large histories.
- Avoid N+1 query patterns.
- Batch inserts where safe.
- Keep write transactions short.
- Validate WAL backup/checkpoint and file-lock behavior on supported platforms.
- Inspect query plans with representative data.
- Retain raw usage/events; optimize aggregates rather than deleting audit history.

### 14.3 Backend optimization

- Use async HTTP clients with connection pooling.
- Reuse provider clients safely; do not share mutable request state.
- Apply semaphores per provider and global worker limits.
- Cache model capability metadata with TTL and source/version.
- Do not cache credentials or sensitive prompts.
- Use bounded queues and backpressure.
- Stream output to disk or bounded buffers.
- Use content hashes and versioned references to avoid repeated context serialization.
- Use incremental analytics aggregation or background rebuilds.
- Profile before introducing multiprocessing or additional services.

### 14.4 UI and graph optimization

- Fetch bounded graph neighborhoods.
- Filter and aggregate server-side.
- Use grouping for large graphs.
- Render visible/expanded nodes only where possible.
- Virtualize activity and log lists.
- Debounce search.
- Avoid full graph layout recalculation on every update.
- Cache layout by graph version and filter set.
- Provide list/table alternatives.

### 14.5 Performance budgets

Provisional targets, to be benchmarked on documented hardware:
- Dashboard local read p95 under 200 ms for ordinary project sizes.
- Workflow state read p95 under 100 ms for ordinary workloads.
- UI update visible within one second under normal local operation.
- No unbounded memory growth during long workflows.
- Cancellation request accepted promptly even if provider operation is pending.
- Startup and idle memory measured during release QA.

Do not claim these targets are achieved until tested with representative project data.

---

## 15. Threat Model and Controls

| Threat | Example | Controls |
|---|---|---|
| Prompt injection | Repository content asks agent to exfiltrate secrets | Treat content as untrusted; policy-separated instructions; no authority from content |
| Credential theft | API key appears in logs or child environment | OS credential store, redaction, scoped access, environment allowlist |
| Localhost abuse | Website calls local FastAPI service | Session token, Origin/Host checks, loopback-only, strict CORS |
| Path traversal | Model writes outside project | Canonical paths, symlink/junction checks, root allowlist, approval |
| Command injection | Model output interpolated into shell | Argument arrays, shell disabled, allowlist, approval |
| Malicious import | Workflow embeds command or external path | Schema validation, no permissions by default, preview |
| Tool compromise | MCP tool returns malicious output | Per-tool capabilities, validation, approvals, isolation |
| Replay/duplicate action | Retry repeats destructive operation | Operation IDs, idempotency, reconciliation, action-scoped approval |
| Database corruption | Crash during state update | Transactions, migrations, backups, integrity checks |
| Resource exhaustion | Huge response or graph payload | Size limits, bounded queues, pagination, timeouts |
| Supply-chain compromise | Malicious dependency/update | Lockfiles, review, scanning, signed release artifacts |
| Data leakage | Sensitive context sent to hosted provider | Project data policy, routing restrictions, explicit disclosure |

### Release security gates

1. Threat model reviewed for new features.
2. All state-changing endpoints authenticated and authorized.
3. Tool tests cover path traversal, command injection, and approval bypass.
4. Secret scanning run on repository and build artifacts.
5. Dependency and license review completed.
6. Import fuzzing and malformed-payload tests pass.
7. Recovery tests verify no unauthorized duplicate side effects.
8. Logs and exports checked for secret leakage.
9. Tauri capability and CSP review completed.
10. Known limitations documented.

---

## 16. Testing and Verification

### Unit tests
- Domain state transitions and dependency resolution.
- Retry classification and policy decisions.
- Token/cost calculations.
- Relationship validation.
- Schema and path validation.

### Integration tests
- SQLite repositories and migrations.
- Provider usage normalization using mocked responses.
- Ollama adapter with a local test instance where available.
- Tool executor in controlled temporary directories.
- Approval lifecycle.
- Artifact persistence and checksums.

### End-to-end tests
- Create project → define workflow → execute → validate → approve → complete.
- Restart backend during a run and verify recovery.
- Cancel a run with a pending provider request.
- Import workflow and verify no permissions before approval.
- Create graph relationships and navigate to source records.
- Record usage and verify dashboard aggregates.

### Security tests
- Path traversal, symlink and junction escape.
- Shell metacharacters and command injection.
- Unauthorized localhost request.
- Invalid Origin/Host and CORS behavior.
- Approval replay and altered action parameters.
- Secret redaction in logs and errors.
- Oversized request, response, and artifact.
- Malformed workflow and provider response.
- Prompt-injection fixtures.
- Untrusted Markdown/HTML rendering.
- Dependency vulnerability and secret scanning.

### Recovery tests
- Process termination between external operation and database completion.
- Expired scheduler lease and duplicate dispatch.
- SQLite busy/locked conditions.
- Interrupted artifact write.
- Database backup and restore.
- Migration from prior schema.

Use deterministic fake providers in CI. Never call paid APIs in ordinary unit tests. Never use real credentials in tests. Assert both expected results and absence of unauthorized side effects.

---

## 17. Packaging, Deployment, and Updates

- Package Tauri app and Python backend as a managed local process or bundled executable.
- Desktop app owns backend startup and shutdown.
- Use per-user application data directory.
- Keep project workspaces outside installation directory.
- Separate database, artifacts, logs, and cache.
- Do not require administrator privileges for normal use.
- Do not automatically open firewall ports.
- Do not install background services without explicit consent.
- Sign release artifacts where supported and verify update signatures.
- Back up before database migrations and define rollback/recovery.
- Never overwrite user project files during application updates.
- Provide an opt-in, previewable diagnostic bundle; exclude keys, full prompts, private source files, and credential-store contents by default.

---

## 18. Implementation Roadmap and Exit Gates

### Phase 0 - Architecture foundation

Deliver domain entities, state transitions, SQLite migrations, local API authentication, threat model, provider/tool contracts, and security test plan.

**Exit:** State and database invariants tested; local API design reviewed; no adapter can bypass domain boundaries.

### Phase 1 - Safe durable orchestration MVP

Deliver project management, versioned workflows, sequential scheduler, one hosted provider and Ollama, durable attempts/events, cancellation/recovery, basic approval gates, read-only Git status/diff, controlled test execution, artifact storage, validation, and token usage capture.

**Exit:** Restart/recovery and path/permission tests pass; secrets absent from logs/config; end-to-end workflow completes with human approval.

### Phase 2 - Workspace and visualization

Deliver unified workspace, manual external activity import, approved file monitoring, provenance, relationship persistence, workflow dependency visualization, project graph preview, and bounded graph queries.

**Exit:** Graph navigates to correct records; imported content cannot execute tools; missing capture coverage is disclosed.

### Phase 3 - Analytics and routing

Deliver immutable usage records, model catalog, context metadata provenance, project/global token analytics, provider/model/tool filters, versioned pricing, estimated costs, budgets, alerts, and configurable routing/fallback.

**Exit:** Reported and estimated values remain distinct; aggregates rebuild; budget controls only future eligible requests.

### Phase 4 - Integrations and hardening

Deliver IDE/CLI integrations, MCP capabilities, advanced graph exploration, optional isolated execution, additional provider adapters, and platform packaging.

**Exit:** Every integration has a documented permission/capture model; security and regression suites pass; performance is benchmarked.

---

## 19. Recommended Initial Dependencies

| Area | Suggested choice |
|---|---|
| Desktop | Tauri 2 |
| UI | TypeScript + lightweight component framework |
| Graph UI | Maintained node/edge visualization library, chosen after prototype |
| Backend API | FastAPI |
| Validation | Pydantic |
| Persistence | SQLite |
| Migrations | Alembic or explicit migration layer |
| Data access | SQLAlchemy 2 or repository layer |
| HTTP | httpx |
| Testing | pytest, pytest-asyncio, HTTPX test client |
| Python checks | Ruff, mypy or Pyright, Bandit as supplementary check |
| Frontend checks | TypeScript strict mode, ESLint, formatter, UI tests |
| Supply chain | Dependency audit and secret scanning in CI |

Do not add Redis, Kafka, Kubernetes, a graph database, or hosted control plane to MVP. Reconsider only when measured workload or multi-user requirements justify it.

---

## 20. Architecture Decision Records

Create and review:
- ADR-001: Modular monolith and process boundaries.
- ADR-002: Tauri-to-FastAPI authentication and lifecycle.
- ADR-003: SQLite concurrency, migration, and backup.
- ADR-004: Workflow delivery semantics, leases, retry, and reconciliation.
- ADR-005: Provider adapter and capability model.
- ADR-006: Tool permissions and subprocess restrictions.
- ADR-007: Secret storage and credential lifecycle.
- ADR-008: Artifact storage and canonical project file protection.
- ADR-009: Activity provenance and graph relationship semantics.
- ADR-010: Token/context measurement and pricing provenance.
- ADR-011: Packaging, signed updates, and migration rollback.

Each ADR records context, options, decision, consequences, security implications, and validation plan.

---

## 21. Architecture Readiness Checklist

Implementation is ready when:
- PRD requirements map to modules and data entities.
- API contracts and domain state transitions are defined.
- Database schema and migration strategy are reviewed.
- Local API and credential threat boundaries are documented.
- Tool permissions and path policy are testable.
- Provider capability and usage normalization contracts are defined.
- Recovery and idempotency behavior is documented.
- Initial performance baseline and supported hardware are recorded.
- Security tests are included in CI.
- MVP excludes unsupported assumptions such as unrestricted autonomous execution or automatic access to private third-party conversations.

**Architecture principle:** Build a small, durable, secure execution core first. Add visual graphs, analytics, integrations, and advanced automation as projections and adapters around that core-not as shortcuts around its security and state guarantees.
