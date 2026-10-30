# API Specification — AI Workflow Orchestrator

**Document:** `API_SPECIFICATION.md`  
**Version:** 1.0  
**Status:** Proposed — authoritative API contract after project-owner acceptance  
**Target:** MVP v1.0  
**API style:** Versioned REST over HTTP + Server-Sent Events (SSE) for read-only live updates  
**Backend:** Python 3.12 + FastAPI  
**Primary client:** Tauri 2 desktop application with TypeScript frontend  
**Last reviewed:** 2026-10-30

---

## 1. Purpose, authority, and mandatory pre-flight

This document defines the contract between the local FastAPI backend and the Tauri desktop frontend. It specifies routes, methods, authentication, authorization, request and response representations, errors, pagination, event streaming, idempotency, and compatibility rules.

All developers and AI coding agents MUST read this document before:

- Adding, removing, renaming, or changing an API route or HTTP method.
- Writing or changing FastAPI routers, request/response models, transport serializers, or client API calls.
- Changing authentication, local session bootstrap, authorization, CORS/origin validation, or network binding.
- Adding WebSocket/SSE channels, polling behavior, or external API consumers.
- Changing status codes, error shapes, pagination, or idempotency semantics.

Read it together with:

- `prd.md` — product requirements and client use cases.
- `ARCHITECTURE.md` — modular-monolith boundaries and backend authority.
- `SCOPE.md` — approved MVP capabilities and exclusions.
- `TECH_STACK.md` — approved frameworks, runtimes, and dependencies.
- `DATABASE.md` — canonical persistence model.
- `EXECUTION_ENGINE.md` — state transitions, execution, approvals, and recovery.

If documents conflict, stop and raise the conflict. Do not silently change a contract or implement an undocumented endpoint.

### 1.1 Non-negotiable API invariants

1. The backend is authoritative for workflow state, task state, scheduling, validation, authorization, approvals, persistence, and recovery.
2. The frontend is a presentation/control client. It cannot write SQLite, call providers directly, or decide state transitions.
3. Every state-changing request is validated by an application use case and persisted transactionally before success is returned.
4. Model/provider output is untrusted input; it never authorizes an API operation.
5. The API is local-only in MVP. It is not a public service or a supported third-party integration surface.
6. No endpoint may expose secrets, raw credential values, unrestricted filesystem paths, or internal stack traces.
7. Responses must reflect persisted facts. Do not fabricate progress, usage, execution status, or artifact availability.
8. Do not add routes, fields, or alternate API paradigms without explicit human approval.

---

## 2. API style, transport, and versioning

### 2.1 Architectural style

Use **REST over HTTP/1.1** with JSON request/response bodies.

- Base path: `/api/v1`.
- Content type for JSON: `application/json`.
- Character encoding: UTF-8.
- Resource identifiers: UUID strings.
- Timestamps: UTC ISO 8601 strings with timezone, e.g. `2026-10-30T09:30:00.000Z`.
- Live updates: Server-Sent Events at a versioned read-only endpoint where supported.
- No GraphQL, gRPC, tRPC, WebSocket, or RPC framework in MVP.
- No public API or third-party client compatibility commitment in MVP.

SSE is a transport for notifications, not an alternate source of truth. Clients must be able to recover by fetching the corresponding REST resource.

### 2.2 Versioning policy

- All application routes MUST be under `/api/v1`.
- Do not use header-only or query-parameter API versioning.
- Additive optional response fields may be introduced within v1 if clients are required to ignore unknown fields.
- Do not change the meaning, type, nullability, or required status of an existing field in v1.
- Do not remove or rename routes or fields in v1.
- Breaking changes require a separately approved `/api/v2`, a migration plan, and a compatibility period appropriate to the desktop release model.
- Do not create `/api/latest` or unversioned aliases.
- API version and application version are separate concepts.

### 2.3 Local binding and network exposure

- Bind the backend to loopback only (127.0.0.1 or an equivalent explicitly configured loopback address).
- Do not bind to `0.0.0.0`, a LAN interface, or a public interface by default.
- Select an available local port at startup and communicate the chosen origin to the Tauri shell through the approved native bootstrap channel.
- Do not assume a fixed port is a security boundary.
- The Tauri native layer owns backend process lifecycle and session bootstrap.
- The backend must validate the Host header and allowed Origin values. Reject unexpected hosts, origins, and cross-origin browser requests.
- CORS must not use wildcard origins or wildcard credential access.
- The frontend must not derive or guess the backend port independently.

### 2.4 Request limits and timeouts

MVP defaults:

| Setting | Default |
|---|---:|
| JSON request body limit | 1 MiB |
| List page size | 50 |
| Maximum page size | 100 |
| Default API request timeout | 30 seconds |
| Provider/tool execution timeout | Defined by execution policy, not HTTP request lifetime |
| SSE heartbeat | 15 seconds |
| SSE reconnect | Client-managed with bounded backoff |

Large files and artifacts must use dedicated staged upload/download flows only if explicitly approved; do not place large binary payloads in JSON bodies.

Long-running workflow execution is asynchronous. The start endpoint returns after the run has been durably created and accepted, not after all tasks complete.

---

## 3. Authentication and authorization

### 3.1 Security model

The MVP is a single-user local desktop application. It does not require user accounts, passwords, OAuth, JWT issuance, or role management.

Use a **per-application-launch opaque bearer session token**:

1. The Tauri native layer starts the backend through a controlled process lifecycle.
2. A cryptographically secure random session token is generated for that backend process.
3. The token is delivered to the frontend through a private Tauri IPC/bootstrap mechanism, not through a URL, command-line argument, log, environment dump, or public endpoint.
4. The frontend keeps the token in memory only and sends it in the `Authorization` header.
5. The backend compares tokens using constant-time comparison.
6. The token is invalidated when the backend process exits or the session is explicitly revoked.
7. A new backend process creates a new token. Persisted workflow data is independent of the session.

The precise IPC implementation must use the already approved Tauri native integration. Do not add an authentication SDK or JWT dependency for this single-user local session.

### 3.2 Request authentication

All application endpoints except the narrowly scoped bootstrap/health checks described below require:

`Authorization: Bearer <opaque-session-token>`

Rules:

- Never put tokens in query parameters, URL fragments, cookies, localStorage, sessionStorage, logs, crash reports, or analytics.
- Do not expose a general unauthenticated endpoint that returns or exchanges the session token.
- Missing, malformed, expired, or invalid tokens return `401`.
- Do not reveal whether a token was close to valid.
- Tokens must be redacted from request logs and exception traces.
- Use HTTPS only if a future deployment crosses a machine boundary; local loopback HTTP is permitted only with the protected launch/session model and strict origin/host controls described here.

### 3.3 Bootstrap and health exceptions

The following endpoints may be unauthenticated, but must return no sensitive data:

- `GET /health/live` — process liveness only.
- `GET /health/ready` — readiness and non-sensitive component status only.
- `GET /api/v1/meta` — API version and non-sensitive capability metadata only, if needed before authenticated client initialization.

Do not expose provider names, project paths, database paths, configuration values, or detailed diagnostics through unauthenticated endpoints.

### 3.4 Authorization model

MVP has one local application principal. RBAC is not applicable. Authentication does not imply unrestricted access.

The backend enforces capability and resource authorization for every operation:

| Capability | Permitted operation | Enforcement |
|---|---|---|
| Read project | List/get project | Authenticated session; project exists |
| Manage project | Create/update/archive project | Authenticated session; path validation |
| Read workflow | List/get definitions and versions | Authenticated session; project ownership |
| Manage workflow | Create/update/validate/version definition | Authenticated session; definition not immutable in-use version |
| Execute workflow | Start run | Authenticated session; definition READY; project valid |
| Control execution | Pause/resume/cancel | Authenticated session; legal transition and run ownership |
| Read execution | Get run/tasks/events/attempts | Authenticated session; parent project/run relationship |
| Resolve approval | Approve/reject exact action | Authenticated session; pending decision and fingerprint match |
| Manage provider config | Create/update/enable/disable/test provider | Authenticated session; secret reference only |
| Read artifact | Get metadata/download artifact | Authenticated session; run ownership and artifact-root validation |

No client-supplied project ID, task ID, run ID, or artifact ID is trusted without verifying its full parent relationship.

### 3.5 Approval is not authentication

A valid session token does not approve consequential actions. Approval gates are enforced by the execution engine. Approval requests are bound to an immutable action fingerprint and execution context. The backend must reject stale, modified, or already-resolved approvals.

---

## 4. Resource model and endpoint catalogue

All routes below are relative to `/api/v1`. Unless explicitly marked public, they require bearer authentication.

### 4.1 System and capabilities

| Method | Route | Purpose |
|---|---|---|
| GET | `/meta` | API version and supported non-sensitive capabilities |
| GET | `/health/live` | Process liveness; unauthenticated |
| GET | `/health/ready` | Readiness; unauthenticated, non-sensitive |
| GET | `/system/status` | Authenticated application/runtime status |
| GET | `/system/diagnostics` | Sanitized diagnostics; no secrets or unrestricted paths |

### 4.2 Projects

| Method | Route | Purpose |
|---|---|---|
| GET | `/projects` | List projects |
| POST | `/projects` | Create project |
| GET | `/projects/{project_id}` | Get project |
| PATCH | `/projects/{project_id}` | Rename/update description |
| POST | `/projects/{project_id}/archive` | Archive project |
| POST | `/projects/{project_id}/restore` | Restore archived project |

Project paths must be canonicalized and validated by the backend. Do not provide an API to delete arbitrary directories or recursively remove user project contents.

### 4.3 Workflow definitions

| Method | Route | Purpose |
|---|---|---|
| GET | `/projects/{project_id}/workflows` | List workflow definitions |
| POST | `/projects/{project_id}/workflows` | Create workflow definition |
| GET | `/workflows/{workflow_id}` | Get current definition |
| PATCH | `/workflows/{workflow_id}` | Update editable definition |
| GET | `/workflows/{workflow_id}/versions` | List immutable versions |
| GET | `/workflows/{workflow_id}/versions/{version}` | Get a specific version |
| POST | `/workflows/{workflow_id}/validate` | Validate workflow structure and DAG |
| POST | `/workflows/{workflow_id}/publish` | Create/publish an immutable version |

The API must not allow mutation of a workflow version already referenced by a run. Publishing creates a new version.

### 4.4 Workflow execution

| Method | Route | Purpose |
|---|---|---|
| POST | `/workflows/{workflow_id}/runs` | Create and start a run from a selected READY version |
| GET | `/runs` | List runs with optional filters |
| GET | `/runs/{run_id}` | Get workflow run |
| POST | `/runs/{run_id}/pause` | Request pause |
| POST | `/runs/{run_id}/resume` | Resume a paused run |
| POST | `/runs/{run_id}/cancel` | Request cancellation |
| GET | `/runs/{run_id}/tasks` | List task runs |
| GET | `/runs/{run_id}/events` | Read paginated execution events |
| GET | `/runs/{run_id}/artifacts` | List run artifacts |

The start request must be idempotent when an `Idempotency-Key` is supplied. The endpoint returns `202 Accepted` once the run and initial task state are committed.

### 4.5 Task and attempt inspection

| Method | Route | Purpose |
|---|---|---|
| GET | `/task-runs/{task_run_id}` | Get task runtime state |
| GET | `/task-runs/{task_run_id}/attempts` | List task attempts |
| GET | `/attempts/{attempt_id}` | Get attempt details |
| GET | `/attempts/{attempt_id}/events` | Read attempt-related events |

No direct endpoint may set a task status. Task state transitions are performed by execution-engine commands only.

### 4.6 Human approvals

| Method | Route | Purpose |
|---|---|---|
| GET | `/approvals` | List pending/recent approvals |
| GET | `/approvals/{approval_id}` | Get approval details |
| POST | `/approvals/{approval_id}/decision` | Approve or reject the exact pending action |

There is no generic approval bypass endpoint. A decision cannot alter the action fingerprint or approve a modified action.

### 4.7 Providers

| Method | Route | Purpose |
|---|---|---|
| GET | `/providers` | List configured providers and safe status |
| POST | `/providers` | Add provider configuration |
| GET | `/providers/{provider_id}` | Get safe provider configuration |
| PATCH | `/providers/{provider_id}` | Update non-secret configuration |
| POST | `/providers/{provider_id}/enable` | Enable provider |
| POST | `/providers/{provider_id}/disable` | Disable provider |
| POST | `/providers/{provider_id}/test` | Test provider connectivity/capability |

Credential values are supplied through a secure native credential-entry flow and stored in the approved OS credential store. API requests and responses carry only an opaque `credential_ref` where necessary. Never accept or return plaintext credentials through ordinary JSON provider endpoints.

### 4.8 Artifacts

| Method | Route | Purpose |
|---|---|---|
| GET | `/artifacts/{artifact_id}` | Get artifact metadata |
| GET | `/artifacts/{artifact_id}/content` | Download an authorized artifact |

Artifact download must resolve a validated application-managed storage reference. The backend must reject path traversal, symlinks escaping the artifact root, and access to arbitrary local files. No generic filesystem browsing or arbitrary file download endpoint is permitted.

### 4.9 Live updates

| Method | Route | Purpose |
|---|---|---|
| GET | `/events/stream` | Authenticated SSE stream for relevant application events |

SSE events are notifications. Clients must refetch canonical resources after reconnect or a detected event gap. The stream must not expose secrets or unbounded prompt/output content. SSE does not authorize state changes.

---

## 5. Request and response schemas

### 5.1 Common conventions

- JSON object field names use `snake_case`.
- Unknown request fields are rejected for security-sensitive commands and otherwise follow strict Pydantic model policy; do not silently accept privileged fields.
- Required fields must be present and non-null.
- Optional fields may be omitted; nullable fields explicitly accept `null`.
- Empty strings are invalid for required names and identifiers.
- UUIDs use canonical string form.
- Timestamps use UTC ISO 8601 strings.
- Enums are uppercase strings matching domain specifications.
- Responses include only fields the caller is authorized to see.
- The backend must never serialize ORM/database rows directly; use explicit response models.

### 5.2 Common response envelope

Successful resource responses use the resource object directly. Collection responses use:

```json
{
  "items": [],
  "page": {
    "limit": 50,
    "next_cursor": null,
    "has_more": false
  }
}
```

Command responses use:

```json
{
  "request_id": "6d33f4e8-0a7e-4a4b-8e58-2ad19c8b0f00",
  "accepted": true
}
```

Do not wrap every resource in an additional `data` property. Keep the contract consistent across endpoints.

### 5.3 Project schemas

Create request:

```json
{
  "name": "Example Project",
  "description": "Optional project description",
  "working_directory": "C:\\Users\\Developer\\Projects\\example"
}
```

| Field | Type | Required | Rules |
|---|---|---|---|
| `name` | string | Yes | Trimmed, 1–120 chars |
| `description` | string | No | Max 2,000 chars; default empty |
| `working_directory` | string | Yes | Absolute path; backend canonicalizes and validates |

Project response:

```json
{
  "id": "UUID",
  "name": "Example Project",
  "description": "Optional project description",
  "working_directory_display": "C:\\Users\\Developer\\Projects\\example",
  "created_at": "2026-10-30T09:30:00.000Z",
  "updated_at": "2026-10-30T09:30:00.000Z",
  "archived_at": null
}
```

The display path may be redacted or shortened by UI policy. Do not expose internal application storage paths.

### 5.4 Workflow definition schemas

Create/update request:

```json
{
  "name": "Code Review Workflow",
  "description": "Review and validate a code change",
  "tasks": [
    {
      "task_key": "inspect",
      "name": "Inspect changes",
      "description": "Inspect the selected project changes",
      "executor_type": "provider",
      "configuration": {
        "provider_id": "UUID",
        "model": "configured-model",
        "prompt_template": "Review the supplied context"
      },
      "validation_policy": {},
      "retry_policy": {},
      "timeout_seconds": 300,
      "is_optional": false,
      "position": 0,
      "depends_on": []
    }
  ]
}
```

Task fields:

| Field | Type | Required | Rules |
|---|---|---|---|
| `task_key` | string | Yes | Stable unique key within version; 1–80 chars |
| `name` | string | Yes | 1–160 chars |
| `description` | string | No | Max 4,000 chars |
| `executor_type` | string | Yes | Must match approved executor registry |
| `configuration` | object | Yes | Executor-specific Pydantic schema |
| `validation_policy` | object | No | Defaults to `{}`; validated policy |
| `retry_policy` | object | No | Defaults to system policy |
| `timeout_seconds` | integer | Yes | 1–3600 |
| `is_optional` | boolean | No | Defaults to false |
| `position` | integer | Yes | Non-negative unique ordering value |
| `depends_on` | string[] | No | Task keys within the same definition; defaults to empty |

Workflow response includes `id`, `project_id`, `name`, `version`, `status`, `tasks`, `created_at`, and `updated_at`. Secrets must be represented by credential references, never values.

Validation response:

```json
{
  "valid": false,
  "errors": [
    {
      "code": "WORKFLOW_DEPENDENCY_CYCLE",
      "message": "The task dependency graph contains a cycle.",
      "path": "tasks"
    }
  ],
  "warnings": []
}
```

### 5.5 Start workflow run

Request:

```json
{
  "workflow_version": 1,
  "execution_policy": {
    "require_approval_for": ["filesystem_write", "git_mutation"],
    "allow_network": true
  },
  "input": {
    "instruction": "Review the current changes"
  }
}
```

| Field | Type | Required | Rules |
|---|---|---|---|
| `workflow_version` | integer | Yes | Positive; must exist and be READY |
| `execution_policy` | object | No | Validated snapshot; defaults to approved project policy |
| `input` | object | No | Bounded JSON object; no secrets unless a dedicated secure flow is used |

Response (`202 Accepted`):

```json
{
  "request_id": "UUID",
  "run": {
    "id": "UUID",
    "workflow_definition_id": "UUID",
    "workflow_version": 1,
    "status": "READY",
    "created_at": "2026-10-30T09:30:00.000Z"
  }
}
```

The backend commits the run, task-run rows, policy snapshot, and initial event before returning. Execution proceeds asynchronously.

### 5.6 Workflow run and task response schemas

Workflow run:

```json
{
  "id": "UUID",
  "project_id": "UUID",
  "workflow_definition_id": "UUID",
  "workflow_version": 1,
  "status": "RUNNING",
  "created_at": "2026-10-30T09:30:00.000Z",
  "started_at": "2026-10-30T09:30:01.000Z",
  "finished_at": null,
  "pause_requested_at": null,
  "cancel_requested_at": null
}
```

Task run:

```json
{
  "id": "UUID",
  "workflow_run_id": "UUID",
  "task_definition_id": "UUID",
  "status": "RUNNING",
  "input_reference": null,
  "output_reference": null,
  "error_code": null,
  "created_at": "2026-10-30T09:30:00.000Z",
  "started_at": "2026-10-30T09:30:01.000Z",
  "finished_at": null,
  "updated_at": "2026-10-30T09:30:01.000Z"
}
```

Task state values and legal transitions are defined only by `EXECUTION_ENGINE.md`. Clients must treat state values as finite enums and unknown future values as non-actionable.

### 5.7 Pause, resume, cancel commands

Request body is optional:

```json
{
  "reason": "User requested pause"
}
```

Response:

```json
{
  "request_id": "UUID",
  "run_id": "UUID",
  "status": "PAUSED",
  "accepted": true
}
```

The response status reflects the persisted state after the command. If the operation is only a request and the engine must reach a safe boundary first, return the current persisted status plus `accepted: true`; do not claim the run is paused/cancelled before it is.

### 5.8 Approval schemas

Decision request:

```json
{
  "decision": "APPROVED",
  "action_fingerprint": "sha256-hex-digest",
  "reason": "Reviewed and authorized"
}
```

| Field | Type | Required | Rules |
|---|---|---|---|
| `decision` | enum | Yes | `APPROVED` or `REJECTED` |
| `action_fingerprint` | string | Yes | Must exactly match pending action fingerprint |
| `reason` | string | No | Max 2,000 chars |

Approval response includes approval ID, decision, decision timestamp, and resulting run/task status. The backend revalidates the action and its preconditions before continuing execution.

### 5.9 Provider schemas

Create/update provider request:

```json
{
  "provider_key": "ollama",
  "display_name": "Local Ollama",
  "base_url": "http://127.0.0.1:11434",
  "configuration": {
    "default_model": "granite4.2:3b"
  },
  "credential_ref": null
}
```

Only provider-specific, non-secret configuration is accepted. Hosted provider credential setup must use the secure credential-entry channel. Provider response must omit secret material and include only safe fields such as ID, provider key, display name, endpoint display value, enabled status, connection status, and timestamps.

### 5.10 Error response schema

All API errors use this shape:

```json
{
  "error": {
    "code": "WORKFLOW_STATE_CONFLICT",
    "message": "The workflow cannot be resumed from its current state.",
    "details": [
      {
        "field": "status",
        "issue": "Expected PAUSED; received RUNNING."
      }
    ],
    "request_id": "UUID"
  }
}
```

- `code` is stable and machine-readable.
- `message` is safe for user display and must not contain stack traces or secrets.
- `details` is optional and bounded.
- `request_id` is present for correlation.
- Do not return raw exception strings, SQL statements, filesystem paths outside authorized display, or provider credential details.

---

## 6. HTTP status codes and error catalogue

### 6.1 Status code policy

| Status | Meaning | Use |
|---|---|---|
| 200 | OK | Successful read or completed command |
| 201 | Created | Resource created synchronously |
| 202 | Accepted | Workflow run or deferred command durably accepted |
| 204 | No Content | Successful operation with no response body, only where documented |
| 400 | Bad Request | Malformed request or invalid command envelope |
| 401 | Unauthorized | Missing/invalid/expired session |
| 403 | Forbidden | Authenticated request lacks capability or action authorization |
| 404 | Not Found | Resource absent or intentionally concealed across ownership boundary |
| 409 | Conflict | Invalid state transition, duplicate identity, stale version, or idempotency conflict |
| 413 | Content Too Large | Body exceeds configured limit |
| 422 | Unprocessable Entity | Pydantic/domain validation failure |
| 429 | Too Many Requests | Bounded local protection/rate limit triggered |
| 500 | Internal Server Error | Unexpected server fault; safe generic message |
| 503 | Service Unavailable | Backend not ready, shutting down, or required local subsystem unavailable |
| 504 | Gateway Timeout | Only for a bounded synchronous adapter health/test operation |

### 6.2 Stable error codes

| Code | HTTP | Meaning |
|---|---:|---|
| `AUTH_REQUIRED` | 401 | Missing bearer token |
| `AUTH_INVALID` | 401 | Invalid or expired token |
| `ACCESS_DENIED` | 403 | Operation not authorized |
| `RESOURCE_NOT_FOUND` | 404 | Resource does not exist or is not visible |
| `VALIDATION_ERROR` | 422 | Request/domain validation failed |
| `PROJECT_PATH_INVALID` | 422 | Project path invalid or inaccessible |
| `WORKFLOW_NOT_READY` | 409 | Workflow version cannot be executed |
| `WORKFLOW_STATE_CONFLICT` | 409 | Illegal workflow transition |
| `TASK_STATE_CONFLICT` | 409 | Illegal task operation |
| `WORKFLOW_DEPENDENCY_CYCLE` | 422 | DAG contains a cycle |
| `APPROVAL_REQUIRED` | 409 | Action is awaiting approval |
| `APPROVAL_STALE` | 409 | Approval fingerprint or precondition no longer matches |
| `IDEMPOTENCY_CONFLICT` | 409 | Key reused with a different request |
| `RESOURCE_ARCHIVED` | 409 | Archived resource cannot be used for requested operation |
| `PROVIDER_UNAVAILABLE` | 503 | Provider unavailable |
| `PROVIDER_CONFIGURATION_INVALID` | 422 | Provider configuration invalid |
| `ARTIFACT_NOT_AVAILABLE` | 404 | Artifact missing/quarantined/not accessible |
| `RATE_LIMITED` | 429 | Request rate exceeded |
| `BACKEND_NOT_READY` | 503 | Backend is not ready |
| `INTERNAL_ERROR` | 500 | Unexpected safe-to-report failure |

Do not make clients parse human-readable messages to determine behavior.

---

## 7. Query parameters, filtering, and pagination

### 7.1 Common pagination

Collection endpoints use cursor pagination:

- `limit`: optional integer, default 50, range 1–100.
- `cursor`: opaque server-generated cursor; clients must not construct or interpret it.
- `sort`: only documented fields and directions are accepted.
- Stable ordering must include a unique tie-breaker such as UUID.
- A cursor is bound to its resource/filter/sort context. Reusing it with different filters returns `400`.

Example:

`GET /api/v1/runs?limit=25&status=RUNNING&project_id=<uuid>&cursor=<opaque>`

Response:

```json
{
  "items": [],
  "page": {
    "limit": 25,
    "next_cursor": null,
    "has_more": false
  }
}
```

### 7.2 Approved filters

| Resource | Filters |
|---|---|
| Projects | `archived` |
| Workflows | `project_id`, `status` |
| Runs | `project_id`, `workflow_id`, `status`, `created_after`, `created_before` |
| Task runs | `status` |
| Approvals | `decision`, `workflow_run_id` |
| Events | `task_run_id`, `event_type`, `after` |
| Providers | `enabled`, `provider_key` |

Reject unknown filter names rather than silently ignoring misspellings. Validate date ranges and maximum range sizes.

### 7.3 Event and attempt pagination

Events and attempts must use stable keyset/cursor pagination. Do not return the full lifetime event history by default. Event order is chronological with a stable unique tie-breaker.

---

## 8. Idempotency, concurrency, and state mutation

### 8.1 Idempotency-Key

Require `Idempotency-Key` for commands that create a workflow run or may trigger an external side effect. The key must be a high-entropy client-generated value (UUID is acceptable) and is scoped to the authenticated local session/application.

For a repeated request:

- Same key + same canonical request body: return the original accepted result and resource reference.
- Same key + different canonical request body: return `409 IDEMPOTENCY_CONFLICT`.
- A retry after client timeout must not create a second workflow run.
- The key and result mapping must be persisted before returning success.

The idempotency implementation must use the existing approved persistence design. If a dedicated API idempotency table is not present in `DATABASE.md`, do not silently add one. Submit a schema change proposal and obtain approval; until then, use the approved durable operation/run identity mechanism only where it fully satisfies the contract.

### 8.2 Conditional updates and optimistic concurrency

- State-changing endpoints must validate the current persisted state inside the transaction.
- Clients must not send arbitrary `status` fields to update run/task records.
- Use an `ETag` / `If-Match` precondition for mutable workflow definition updates if version concurrency is implemented. If not implemented in the MVP, serialize updates through the application service and return `409` for stale version edits.
- A state transition command is accepted only if the transition is legal under `EXECUTION_ENGINE.md`.
- Duplicate pause/resume/cancel commands must be handled deterministically and must not duplicate side effects.

### 8.3 External operation safety

Persist operation intent and stable idempotency identity before invoking a side-effecting adapter. API success does not mean an external action succeeded; it means the command was accepted or the operation reached its documented terminal result. Uncertain external outcomes must surface as `NEEDS_ATTENTION`/reconciliation state, not be blindly replayed.

---

## 9. Live event stream contract

### 9.1 Transport

Use SSE at:

`GET /api/v1/events/stream`

Headers:

- `Accept: text/event-stream`
- `Authorization: Bearer <opaque-session-token>`
- Optional `Last-Event-ID` for reconnect.

SSE is read-only. All commands remain REST requests.

### 9.2 Event format

```text
id: 01J... 
event: workflow.status_changed
data: {"event_id":"UUID","workflow_run_id":"UUID","task_run_id":null,"occurred_at":"2026-10-30T09:30:00.000Z","payload":{"previous_status":"READY","status":"RUNNING"}}

```

MVP event types:

- `workflow.created`
- `workflow.status_changed`
- `task.status_changed`
- `task.attempt_started`
- `task.attempt_finished`
- `approval.created`
- `approval.decided`
- `artifact.available`
- `run.recovery_required`
- `system.status_changed`

Events carry identifiers, timestamps, safe summaries, and state facts only. Do not stream raw prompts, full model outputs, secrets, or arbitrary artifact contents.

### 9.3 Delivery semantics

- Delivery is best-effort and may be duplicated after reconnect.
- Event IDs are stable and ordered according to persisted event records.
- The client must tolerate duplicate events.
- If an event gap is detected or replay is unavailable, refetch the canonical run/task resources and event page.
- SSE disconnection must never pause or cancel a workflow.
- The event stream is not a durable queue and must not be used to trigger execution.

---

## 10. Rate limiting, resource protection, and abuse resistance

This is a local single-user API, not a public multi-tenant service. Apply bounded protection rather than an elaborate rate-limiting subsystem.

MVP controls:

- Limit request body size to 1 MiB.
- Bound page sizes to 100.
- Bound concurrent provider health checks and other expensive diagnostic operations.
- Prevent duplicate workflow-start commands through idempotency.
- Bound SSE client connections to the expected desktop client count.
- Apply execution-policy timeouts to provider/tool calls independently of HTTP timeouts.
- Reject excessive or malformed filter/query parameters.
- Avoid unbounded response bodies; artifact content uses streaming with size limits and authorization.
- Return `429 RATE_LIMITED` for a bounded protection limit, with optional `Retry-After`.

Do not add Redis or a distributed rate-limit service.

---

## 11. Security, privacy, and error handling

- Enforce bearer authentication on every protected endpoint.
- Validate Origin and Host; do not treat CORS as authentication.
- Bind loopback only and protect backend launch/session bootstrap.
- Use Pydantic request models and explicit response models.
- Parameterize all database queries.
- Apply project-root and artifact-root filesystem boundaries.
- Do not expose arbitrary filesystem, shell, process, or database operations as generic endpoints.
- Keep API keys in the approved OS credential store; JSON carries opaque references only.
- Redact bearer tokens, credential references where sensitive, prompt content, PII, and filesystem details from logs.
- Never return Python tracebacks, SQL errors, raw provider errors, or secret-bearing exception text.
- External provider transmission must be explicit in workflow/provider configuration and disclosed in the UI.
- Validate artifact download authorization and content type; set safe download headers.
- Avoid permissive CORS, wildcard hosts, open redirects, and user-controlled callback URLs.
- Apply bounded request parsing and reject oversized/deeply nested JSON.
- Diagnostics must be opt-in or user-invoked where they may contain project metadata.

---

## 12. OpenAPI and client contract generation

FastAPI's OpenAPI schema is the machine-readable representation of this contract.

- Keep OpenAPI generation enabled for development and automated contract testing.
- Serve interactive docs only to the local authenticated development context or disable them in production builds.
- Do not expose unauthenticated Swagger/ReDoc endpoints containing sensitive internal details in packaged production mode.
- Define request/response Pydantic models explicitly.
- Generate or validate the TypeScript client types from the backend OpenAPI schema using an already approved tool only. Do not add a code-generation dependency without approval.
- The generated client must not become an independent source of truth.
- CI must detect unintended OpenAPI route/schema drift.
- Breaking schema changes require review and API versioning.

---

## 13. Testing and verification requirements

API implementation is incomplete until the following are covered by automated tests.

| ID | Scenario | Required assertion |
|---|---|---|
| API-001 | Route inventory | Every implemented route is documented and versioned |
| API-002 | Authentication missing | Protected route returns 401 |
| API-003 | Invalid token | Protected route returns 401 without leaking details |
| API-004 | Token redaction | Session token never appears in logs/errors |
| API-005 | Host/Origin validation | Unapproved host/origin is rejected |
| API-006 | Loopback binding | Packaged backend does not bind to LAN/public interface |
| API-007 | Project create | Valid project returns 201 and persists |
| API-008 | Invalid project path | Rejected without filesystem escape |
| API-009 | Workflow validation | Valid DAG accepted; cyclic DAG rejected |
| API-010 | Immutable version | Run-referenced version cannot be mutated |
| API-011 | Start run | Returns 202 only after durable run/task/event commit |
| API-012 | Start idempotency | Same key/body returns same run |
| API-013 | Idempotency conflict | Same key/different body returns 409 |
| API-014 | State transition | Illegal pause/resume/cancel transition rejected |
| API-015 | Backend authority | Client cannot directly set workflow/task status |
| API-016 | Approval fingerprint | Stale/modified action approval rejected |
| API-017 | Provider secret | Plaintext credentials never appear in JSON response |
| API-018 | Artifact traversal | Path traversal and symlink escape rejected |
| API-019 | Artifact authorization | Cross-run artifact access denied/not found |
| API-020 | Pagination | Cursor ordering stable; page limit bounded |
| API-021 | Unknown filters | Unknown query parameters rejected |
| API-022 | Error envelope | Errors use stable schema and code |
| API-023 | Exception safety | No traceback, SQL, token, or secret leakage |
| API-024 | SSE auth | Unauthenticated stream rejected |
| API-025 | SSE reconnect | Duplicate/reconnect handling is safe |
| API-026 | SSE authority | Disconnect does not change workflow state |
| API-027 | Request size | Oversized JSON returns 413 |
| API-028 | Concurrency | Competing state-changing commands preserve invariants |
| API-029 | OpenAPI drift | Contract schema changes are reviewed |
| API-030 | Shutdown | Backend shutdown does not falsely report unfinished work as complete |

Use isolated test databases and temporary filesystem roots. Test both successful and adversarial requests. Contract tests must verify status codes, field names/types, nullability, and stable error codes.

---

## 14. Change control and implementation rules

Any proposed API change must include:

1. Business/client requirement and affected user flow.
2. Proposed route, method, request, response, and error behavior.
3. Authentication and authorization impact.
4. Database and execution-engine impact.
5. Security/privacy and local-boundary impact.
6. Compatibility and versioning impact.
7. Test plan and migration/release plan.
8. Explicit human approval.

After approval:

1. Update this document.
2. Update relevant PRD/architecture/database/execution documents.
3. Implement backend use case and route.
4. Add request/response models and contract tests.
5. Update the TypeScript client.
6. Verify generated OpenAPI and route inventory.
7. Run security, integration, and regression tests.

AI coding agents must not add routes, fields, frameworks, transports, authentication mechanisms, or client-facing capabilities based solely on convenience or model suggestions. If an operation is not specified here, stop and request approval.

---

## 15. Final API rule

The MVP API is a **versioned, authenticated, loopback-only REST interface with read-only SSE notifications**, implemented by the existing FastAPI backend. It serves the Tauri desktop client and is not a public integration platform.

The backend remains the sole authority for state, permissions, approvals, persistence, execution, and recovery. Every endpoint must enforce the contracts in this document and the invariants in `DATABASE.md` and `EXECUTION_ENGINE.md`.

**No developer or AI coding agent may silently add or change an endpoint, transport, authentication mechanism, response contract, or authorization rule. Read this document first, propose the change, and obtain explicit human approval before implementation.**
