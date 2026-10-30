# Database and Persistence Specification — AI Workflow Orchestrator

**Document:** `DATABASE.md`  
**Version:** 1.0  
**Status:** Proposed — authoritative data contract after project-owner acceptance  
**Target:** MVP v1.0  
**Database:** SQLite  
**Driver:** Python 3.12 standard-library `sqlite3`  
**Persistence model:** Local relational database + controlled local filesystem  
**Last reviewed:** 2026-10-30

---

## 1. Purpose, authority, and mandatory AI pre-flight

This document is the canonical specification for the application's relational data model, persistence boundaries, constraints, indexes, transactions, migrations, backup, recovery, and data lifecycle.

It must be read **before** any developer or AI coding agent:

- Proposes or implements a schema migration.
- Writes, modifies, or reviews raw SQL, query builders, or persistence/repository code.
- Adds, removes, or changes a table, column, index, constraint, trigger, relationship, or persisted enum.
- Changes transaction boundaries, SQLite configuration, database paths, backup behavior, or artifact persistence.
- Changes retention, deletion, export, or recovery behavior.

The pre-flight review must also read the relevant requirements in:

- `prd.md` — product requirements and initial logical data entities.
- `ARCHITECTURE.md` — modular-monolith boundaries and local persistence topology.
- `SCOPE.md` — approved MVP capabilities and exclusions.
- `TECH_STACK.md` — approved database engine, driver, and migration tooling.
- `EXECUTION_ENGINE.md` — execution state machines, atomicity, idempotency, and recovery.

### 1.1 Authority and change control

1. This document is the single source of truth for the **approved persisted data model**.
2. Do not invent tables, columns, relationships, indexes, triggers, or persistence abstractions without explicit human approval.
3. A logical entity mentioned in the PRD is not permission to create an unreviewed schema.
4. If implementation, another document, or an existing database conflicts with this specification, stop and report the discrepancy. Do not silently mutate the schema to make the conflict disappear.
5. Schema changes require a documented proposal, human approval, an updated `DATABASE.md`, a migration, and tests.
6. `TECH_STACK.md` controls permitted technology. This document does not authorize an ORM, database server, migration framework, or additional persistence dependency.

### 1.2 MVP persistence principles

- SQLite is the durable source of truth for structured application and execution state.
- The local filesystem stores project files and large workflow artifacts.
- The backend is the only authority allowed to mutate persisted workflow state.
- Persisted state must be sufficient to reconstruct execution after restart.
- No database server, Redis, message broker, ORM, or distributed persistence layer is required or approved for MVP.
- Secrets are not ordinary workflow data and must never be stored in plaintext database fields.
- Data integrity and recoverability take priority over speculative scale optimization.

---

## 2. Database engine and configuration

### 2.1 Approved engine and driver

| Concern | Approved choice | Policy |
|---|---|---|
| Database | SQLite | Embedded, local relational database |
| Minimum SQLite capability | SQLite 3.37.0 or later | Verify runtime version at startup; use features supported by this baseline |
| Python runtime | Python 3.12.x | As pinned by `TECH_STACK.md` |
| Driver | Python standard-library `sqlite3` | Parameterized SQL and explicit transaction control |
| Database location | Application-owned local data directory | Never place the live database inside a user project repository by default |
| Artifact storage | Local filesystem | Store metadata and integrity references in SQLite |
| ORM | None | Do not add an ORM without approval |
| Connection pool | No external pool | One short-lived connection per unit of work, or a carefully managed backend-owned connection strategy; never share a connection across concurrent threads/tasks without explicit serialization |
| Extensions | None required | Do not load SQLite extensions in MVP |

The exact SQLite runtime version must be recorded in diagnostic information. Startup must fail safely with a clear compatibility error if the runtime is below the minimum or required database features are unavailable.

### 2.2 Required SQLite settings

Apply and verify the following settings for every database connection where applicable:

| Setting | Required value | Rationale |
|---|---|---|
| `journal_mode` | `WAL` | Allows readers while a writer is active |
| `foreign_keys` | `ON` | Enforces declared FK relationships |
| `busy_timeout` | 5000 ms default | Bounded wait for transient lock contention |
| `synchronous` | `FULL` | Favor durability for workflow state and operation records |
| `temp_store` | `MEMORY` | Keep temporary query structures local to process |
| `trusted_schema` | `OFF` where supported | Reduce risk from untrusted schema objects |
| `recursive_triggers` | `OFF` unless a reviewed migration requires them | Avoid unexpected recursive trigger behavior |

Configuration requirements:

- Set and verify connection-local pragmas on every new connection; do not assume they persist across connections.
- Use WAL checkpointing during controlled maintenance and shutdown. Do not delete or manually manipulate `-wal` or `-shm` files while the database is open.
- Use explicit transaction control. Repository methods must not depend on implicit transaction behavior that varies by Python version.
- Use `BEGIN IMMEDIATE` for short read-modify-write critical sections such as task claims, approval decisions, and state transitions.
- Never hold a write transaction while performing network I/O, model inference, human approval waits, subprocess execution, or long filesystem operations.
- Handle `SQLITE_BUSY` and `SQLITE_LOCKED` with bounded retry and clear diagnostics. Do not retry indefinitely.
- Use UTC timestamps in ISO 8601 format with timezone information. Persist a consistent canonical representation.
- Use integer UTC epoch milliseconds instead only if the entire codebase adopts that representation through an approved schema decision. Do not mix formats within a table.

### 2.3 Database and filesystem paths

- Keep the live database, WAL files, backups, and application-managed artifacts under an application-owned data directory.
- Resolve and validate paths before use; do not trust paths supplied by model output or imported workflow definitions.
- Project working directories remain user-selected external paths. The database stores their canonical references, not copies of entire project trees.
- Store artifact references as application-managed relative paths where possible. Resolve them against a configured storage root and reject path traversal.
- Never place credentials, raw API keys, or secret values in database filenames, paths, metadata, or logs.

### 2.4 Encryption and local security

SQLite does not provide transparent encryption at rest in the approved baseline.

Therefore:

- Protect the application data directory using operating-system account permissions and full-disk/device encryption where available.
- Store provider credentials only through an approved OS credential store or secure Tauri/native credential facility after its exact implementation is approved in `TECH_STACK.md`.
- Persist only a non-secret credential reference in SQLite.
- Never add SQLCipher or another encrypted SQLite build without explicit technology and compatibility approval.
- Do not claim the database itself is encrypted unless an approved, tested encrypted database implementation is actually deployed.
- Do not log PII, secrets, raw credentials, sensitive prompt content, or full source-code contents by default.
- Minimize sensitive data retained in execution history and diagnostic metadata.

---

## 3. Schema and entities

### 3.1 Schema conventions

- Use `snake_case` for table and column names.
- Use singular table names consistently.
- Primary keys are application-generated UUIDs stored as canonical lowercase text (`TEXT`, 36 characters). Do not use SQLite rowids as public identifiers.
- Foreign keys use the same UUID text representation as their referenced primary keys.
- Boolean values use `INTEGER NOT NULL CHECK (column IN (0, 1))`.
- Timestamps use canonical UTC ISO 8601 text, for example `2026-10-30T09:30:00.000Z`.
- Structured JSON documents use `TEXT` with application validation and, where supported, `CHECK (json_valid(column))`. JSON is not a substitute for relational keys or constraints.
- Enumerated states use `TEXT` plus explicit `CHECK` constraints matching `EXECUTION_ENGINE.md`.
- All tables must declare explicit primary keys and appropriate `NOT NULL`, `UNIQUE`, `CHECK`, and foreign-key constraints.
- Do not use SQLite's dynamic typing as a reason to omit application validation.
- Avoid storing duplicated derivable state. When a denormalized field is necessary for a measured query, document its source of truth and update transaction.

### 3.2 MVP core entities

The following tables are the approved logical MVP schema baseline. Exact physical SQL definitions must be implemented through reviewed migrations and must preserve the listed semantics. Any additional table or field requires approval.

#### 3.2.1 `project`

Stores a local workspace managed by the application.

| Column | Type | Rules |
|---|---|---|
| `id` | TEXT | PK, UUID, NOT NULL |
| `name` | TEXT | NOT NULL, trimmed, length 1–120 |
| `description` | TEXT | NOT NULL, default empty string |
| `working_directory` | TEXT | NOT NULL, canonical absolute path |
| `created_at` | TEXT | NOT NULL, UTC timestamp |
| `updated_at` | TEXT | NOT NULL, UTC timestamp |
| `archived_at` | TEXT | NULL, UTC timestamp when archived |

Constraints:
- `id` unique primary key.
- The same canonical working directory must not be registered twice unless a future approved product requirement permits aliases.
- Project name is not globally unique.
- Archive is a lifecycle state; do not hard-delete a project that still has retained workflow history.

#### 3.2.2 `workflow_definition`

Stores versioned workflow definitions. A definition version is immutable once referenced by a run.

| Column | Type | Rules |
|---|---|---|
| `id` | TEXT | PK, UUID, NOT NULL |
| `project_id` | TEXT | FK → `project.id`, NOT NULL |
| `name` | TEXT | NOT NULL, length 1–160 |
| `version` | INTEGER | NOT NULL, CHECK > 0 |
| `definition_json` | TEXT | NOT NULL, valid JSON object |
| `created_at` | TEXT | NOT NULL, UTC timestamp |
| `updated_at` | TEXT | NOT NULL, UTC timestamp |
| `archived_at` | TEXT | NULL, UTC timestamp |

Constraints:
- `UNIQUE(project_id, name, version)`.
- A version's task definitions and configuration are immutable after a run is created from it.
- Editing a published definition creates a new version; it must not rewrite historical run inputs.
- `definition_json` must not contain secret values. It may contain references to separately stored credentials.

#### 3.2.3 `task_definition`

Stores normalized task definitions belonging to a workflow version.

| Column | Type | Rules |
|---|---|---|
| `id` | TEXT | PK, UUID, NOT NULL |
| `workflow_definition_id` | TEXT | FK → `workflow_definition.id`, NOT NULL |
| `task_key` | TEXT | NOT NULL, stable within workflow version |
| `name` | TEXT | NOT NULL, length 1–160 |
| `description` | TEXT | NOT NULL, default empty string |
| `executor_type` | TEXT | NOT NULL, validated against approved executor registry |
| `configuration_json` | TEXT | NOT NULL, valid JSON object |
| `validation_policy_json` | TEXT | NOT NULL, valid JSON object; default `{}` |
| `retry_policy_json` | TEXT | NOT NULL, valid JSON object; default system policy |
| `timeout_seconds` | INTEGER | NOT NULL, CHECK between 1 and 3600 |
| `is_optional` | INTEGER | NOT NULL, default 0, boolean CHECK |
| `position` | INTEGER | NOT NULL, CHECK >= 0 |

Constraints:
- `UNIQUE(workflow_definition_id, task_key)`.
- `UNIQUE(workflow_definition_id, position)`.
- The task graph must be acyclic; enforce through application-level DAG validation before a definition is marked executable.
- No secret values in task configuration. Use credential references.
- Task definitions cannot be modified after their version has been used by a workflow run.

#### 3.2.4 `task_dependency`

Stores explicit DAG edges; avoids opaque dependency arrays as the only persisted representation.

| Column | Type | Rules |
|---|---|---|
| `workflow_definition_id` | TEXT | FK → `workflow_definition.id`, NOT NULL |
| `task_id` | TEXT | FK → `task_definition.id`, NOT NULL |
| `depends_on_task_id` | TEXT | FK → `task_definition.id`, NOT NULL |

Constraints:
- Composite PK: `(task_id, depends_on_task_id)`.
- `CHECK(task_id <> depends_on_task_id)`.
- Both referenced tasks must belong to the same `workflow_definition_id`; enforce in the application and with composite foreign keys if the physical schema includes a corresponding unique parent key.
- DAG cycle detection is application-level validation; a normal FK cannot enforce acyclicity.
- Dependency rows are immutable for a published workflow version.

#### 3.2.5 `workflow_run`

Stores one execution of a pinned workflow definition version.

| Column | Type | Rules |
|---|---|---|
| `id` | TEXT | PK, UUID, NOT NULL |
| `workflow_definition_id` | TEXT | FK → `workflow_definition.id`, NOT NULL |
| `project_id` | TEXT | FK → `project.id`, NOT NULL |
| `workflow_version` | INTEGER | NOT NULL, CHECK > 0 |
| `status` | TEXT | NOT NULL, default `READY`, workflow-state CHECK |
| `execution_policy_json` | TEXT | NOT NULL, valid JSON object snapshot |
| `created_at` | TEXT | NOT NULL, UTC timestamp |
| `started_at` | TEXT | NULL, UTC timestamp |
| `finished_at` | TEXT | NULL, UTC timestamp |
| `pause_requested_at` | TEXT | NULL, UTC timestamp |
| `cancel_requested_at` | TEXT | NULL, UTC timestamp |

Constraints:
- The referenced definition must belong to the same project and match `workflow_version`; enforce through application validation and an appropriate composite unique/FK design.
- `status` must be one of `READY`, `RUNNING`, `WAITING_FOR_APPROVAL`, `PAUSED`, `SUCCEEDED`, `FAILED`, `CANCELLED`, `NEEDS_ATTENTION`.
- Definition-level `DRAFT` is not a workflow-run status.
- A run's definition version and execution policy snapshot are immutable.

#### 3.2.6 `task_run`

Stores the runtime state for one task in one workflow run.

| Column | Type | Rules |
|---|---|---|
| `id` | TEXT | PK, UUID, NOT NULL |
| `workflow_run_id` | TEXT | FK → `workflow_run.id`, NOT NULL |
| `task_definition_id` | TEXT | FK → `task_definition.id`, NOT NULL |
| `status` | TEXT | NOT NULL, default `PENDING`, task-state CHECK |
| `input_reference` | TEXT | NULL, safe reference to persisted input |
| `output_reference` | TEXT | NULL, safe reference to persisted output |
| `error_code` | TEXT | NULL, normalized error category |
| `created_at` | TEXT | NOT NULL, UTC timestamp |
| `started_at` | TEXT | NULL, UTC timestamp |
| `finished_at` | TEXT | NULL, UTC timestamp |
| `updated_at` | TEXT | NOT NULL, UTC timestamp |

Constraints:
- `UNIQUE(workflow_run_id, task_definition_id)`.
- The task definition must belong to the workflow definition version pinned by the parent run.
- State values must match `EXECUTION_ENGINE.md`: `PENDING`, `READY`, `RUNNING`, `WAITING_FOR_APPROVAL`, `RETRY_WAIT`, `SUCCEEDED`, `FAILED`, `SKIPPED`, `CANCELLED`, `NEEDS_ATTENTION`.
- A task's attempt count is derived from its attempt records; do not use a mutable `attempt_number` column as the authoritative history.

#### 3.2.7 `task_attempt`

Stores each actual execution attempt. Retries create new rows and never overwrite earlier attempts.

| Column | Type | Rules |
|---|---|---|
| `id` | TEXT | PK, UUID, NOT NULL |
| `task_run_id` | TEXT | FK → `task_run.id`, NOT NULL |
| `attempt_number` | INTEGER | NOT NULL, CHECK > 0 |
| `status` | TEXT | NOT NULL, constrained attempt lifecycle |
| `provider_id` | TEXT | NULL, non-secret provider identifier |
| `model_name` | TEXT | NULL |
| `started_at` | TEXT | NOT NULL, UTC timestamp |
| `deadline_at` | TEXT | NOT NULL, UTC timestamp |
| `finished_at` | TEXT | NULL, UTC timestamp |
| `input_reference` | TEXT | NULL, safe reference |
| `output_reference` | TEXT | NULL, safe reference |
| `error_code` | TEXT | NULL |
| `error_summary` | TEXT | NULL, redacted and bounded |
| `duration_ms` | INTEGER | NULL, CHECK >= 0 |
| `created_at` | TEXT | NOT NULL, UTC timestamp |

Constraints:
- `UNIQUE(task_run_id, attempt_number)`.
- Attempt status values must be a separately documented, finite set such as `RUNNING`, `SUCCEEDED`, `FAILED`, `INTERRUPTED`, `TIMED_OUT`, `CANCELLED`, `OUTCOME_UNKNOWN`. The application must not confuse attempt status with task state.
- A completed attempt is immutable except for additive reconciliation metadata introduced through an approved schema change.

#### 3.2.8 `execution_event`

Append-only audit timeline for workflow and task execution.

| Column | Type | Rules |
|---|---|---|
| `id` | TEXT | PK, UUID, NOT NULL |
| `workflow_run_id` | TEXT | FK → `workflow_run.id`, NOT NULL |
| `task_run_id` | TEXT | FK → `task_run.id`, NULL |
| `attempt_id` | TEXT | FK → `task_attempt.id`, NULL |
| `event_type` | TEXT | NOT NULL |
| `actor_type` | TEXT | NOT NULL, CHECK in `SYSTEM`, `USER`, `RECOVERY`, `ADAPTER` |
| `occurred_at` | TEXT | NOT NULL, UTC timestamp |
| `correlation_id` | TEXT | NULL |
| `payload_json` | TEXT | NOT NULL, valid JSON object; secrets redacted |

Constraints:
- Events are insert-only for application code; no update or delete path in ordinary runtime.
- Event payloads must be bounded and must not contain credentials, tokens, passwords, raw secret material, or unredacted PII.
- A state transition and its corresponding event must commit in the same transaction.
- Events reference the affected run and, where applicable, task and attempt.

#### 3.2.9 `approval_record`

Stores action-scoped human decisions.

| Column | Type | Rules |
|---|---|---|
| `id` | TEXT | PK, UUID, NOT NULL |
| `workflow_run_id` | TEXT | FK → `workflow_run.id`, NOT NULL |
| `task_run_id` | TEXT | FK → `task_run.id`, NOT NULL |
| `attempt_id` | TEXT | FK → `task_attempt.id`, NULL |
| `action_type` | TEXT | NOT NULL |
| `action_summary` | TEXT | NOT NULL, bounded and safe for display |
| `action_fingerprint` | TEXT | NOT NULL, cryptographic digest of canonical action |
| `decision` | TEXT | NOT NULL, `PENDING`, `APPROVED`, `REJECTED`, `INVALIDATED` |
| `decision_reason` | TEXT | NULL, bounded and redacted |
| `decided_at` | TEXT | NULL, UTC timestamp |
| `created_at` | TEXT | NOT NULL, UTC timestamp |

Constraints:
- A decision is immutable once recorded; invalidation is represented as an explicit state/event, not by rewriting prior decision history.
- Approval applies only to the exact action fingerprint and execution context.
- Enforce that only one unresolved approval exists for a given operation/action identity through a unique key once the operation table is introduced and approved.

#### 3.2.10 `artifact`

Stores metadata for output files and generated artifacts. Artifact bytes live in the filesystem.

| Column | Type | Rules |
|---|---|---|
| `id` | TEXT | PK, UUID, NOT NULL |
| `workflow_run_id` | TEXT | FK → `workflow_run.id`, NOT NULL |
| `task_run_id` | TEXT | FK → `task_run.id`, NULL |
| `artifact_type` | TEXT | NOT NULL |
| `storage_reference` | TEXT | NOT NULL, application-managed relative path/reference |
| `checksum_sha256` | TEXT | NOT NULL, 64-character lowercase hex |
| `size_bytes` | INTEGER | NOT NULL, CHECK >= 0 |
| `media_type` | TEXT | NULL |
| `created_at` | TEXT | NOT NULL, UTC timestamp |
| `lifecycle_status` | TEXT | NOT NULL, `STAGED`, `AVAILABLE`, `MISSING`, `QUARANTINED` |

Constraints:
- Storage reference must resolve within the configured artifact root.
- Checksum and size must be verified before an artifact is marked `AVAILABLE`.
- An artifact record must not claim availability when the corresponding file is absent or invalid.
- Do not store large binary payloads as SQLite BLOBs in MVP unless a specific approved requirement justifies it.

#### 3.2.11 `operation_record`

Durable intent and reconciliation record for side-effecting operations. This table is required by `EXECUTION_ENGINE.md` idempotency and crash-recovery requirements.

| Column | Type | Rules |
|---|---|---|
| `id` | TEXT | PK, UUID, NOT NULL; stable operation ID |
| `workflow_run_id` | TEXT | FK → `workflow_run.id`, NOT NULL |
| `task_run_id` | TEXT | FK → `task_run.id`, NOT NULL |
| `attempt_id` | TEXT | FK → `task_attempt.id`, NOT NULL |
| `operation_type` | TEXT | NOT NULL |
| `idempotency_key` | TEXT | NOT NULL, stable for the logical operation |
| `action_fingerprint` | TEXT | NOT NULL |
| `status` | TEXT | NOT NULL, finite operation lifecycle |
| `external_reference` | TEXT | NULL, safe external correlation/reference ID |
| `request_metadata_json` | TEXT | NOT NULL, valid JSON object with secrets removed |
| `result_metadata_json` | TEXT | NULL, valid JSON object with secrets removed |
| `created_at` | TEXT | NOT NULL, UTC timestamp |
| `updated_at` | TEXT | NOT NULL, UTC timestamp |
| `reconciled_at` | TEXT | NULL, UTC timestamp |

Constraints:
- `UNIQUE(idempotency_key)` within the application's operation namespace.
- Operation state must distinguish at least `INTENT_RECORDED`, `SUBMITTED`, `SUCCEEDED`, `FAILED`, `NOT_APPLIED`, `OUTCOME_UNKNOWN`, and `RECONCILED`.
- Never reuse an idempotency key for a materially different action.
- Operation records are durable evidence; do not delete them as part of ordinary workflow cleanup.

#### 3.2.12 `provider_config`

Stores non-secret provider configuration and references to secrets.

| Column | Type | Rules |
|---|---|---|
| `id` | TEXT | PK, UUID, NOT NULL |
| `provider_key` | TEXT | NOT NULL, stable adapter identifier |
| `display_name` | TEXT | NOT NULL |
| `base_url` | TEXT | NULL, validated URL |
| `credential_ref` | TEXT | NULL, opaque OS credential-store reference only |
| `configuration_json` | TEXT | NOT NULL, valid JSON object; no secret values |
| `is_enabled` | INTEGER | NOT NULL, default 0, boolean CHECK |
| `created_at` | TEXT | NOT NULL |
| `updated_at` | TEXT | NOT NULL |

Constraints:
- `UNIQUE(provider_key, display_name)` unless a future approved provider-profile identity model replaces it.
- No API key, bearer token, password, or private key may be stored in any column.
- Provider configuration must be validated by the corresponding adapter before use.

### 3.3 Schema additions requiring explicit approval

The following are not part of the MVP schema unless separately approved:

- User/account/team/organization tables.
- Conversation ingestion, message indexing, embeddings, vector tables, or knowledge graphs.
- Scheduler/cron definitions, webhook subscriptions, external event inbox/outbox, or broker queues.
- Distributed worker leases, job queues, or cross-machine locks.
- Billing, subscription, telemetry, or analytics event warehouses.
- Generic plugin marketplace or third-party extension registry.
- Secret values or credential vault tables.
- Generic key-value storage intended to bypass typed schema review.

### 3.4 JSON column policy

JSON is permitted for versioned workflow definitions, task configuration, policies, and bounded event metadata. It must not become an unstructured dumping ground.

- Validate each JSON document against a versioned Pydantic schema before persistence.
- Include a schema version in each long-lived document where evolution is expected.
- Reject unknown privileged fields rather than silently accepting them.
- Keep query-critical attributes relational and indexed.
- Do not place secret values, binary blobs, or unbounded logs in JSON columns.
- Do not add JSON expression indexes until a real query and benchmark justify them.

---

## 4. Relationships and keys

### 4.1 Relationship map

| Parent | Child | Cardinality | FK |
|---|---|---|---|
| `project` | `workflow_definition` | 1:N | `workflow_definition.project_id` |
| `workflow_definition` | `task_definition` | 1:N | `task_definition.workflow_definition_id` |
| `task_definition` | `task_dependency` | 1:N as dependency source | `task_dependency.task_id` |
| `task_definition` | `task_dependency` | 1:N as dependency target | `task_dependency.depends_on_task_id` |
| `workflow_definition` | `workflow_run` | 1:N | `workflow_run.workflow_definition_id` |
| `project` | `workflow_run` | 1:N | `workflow_run.project_id` |
| `workflow_run` | `task_run` | 1:N | `task_run.workflow_run_id` |
| `task_definition` | `task_run` | 1:N across runs | `task_run.task_definition_id` |
| `task_run` | `task_attempt` | 1:N | `task_attempt.task_run_id` |
| `workflow_run` | `execution_event` | 1:N | `execution_event.workflow_run_id` |
| `task_run` | `execution_event` | 1:N | `execution_event.task_run_id` |
| `task_attempt` | `execution_event` | 1:N | `execution_event.attempt_id` |
| `workflow_run` | `approval_record` | 1:N | `approval_record.workflow_run_id` |
| `task_run` | `approval_record` | 1:N | `approval_record.task_run_id` |
| `workflow_run` | `artifact` | 1:N | `artifact.workflow_run_id` |
| `task_run` | `artifact` | 1:N | `artifact.task_run_id` |
| `task_attempt` | `operation_record` | 1:N | `operation_record.attempt_id` |
| `workflow_run` | `operation_record` | 1:N | `operation_record.workflow_run_id` |
| `task_run` | `operation_record` | 1:N | `operation_record.task_run_id` |

### 4.2 Key and referential integrity rules

- Every FK must reference a declared PK or UNIQUE key.
- Enable `PRAGMA foreign_keys = ON` for every connection.
- Use `ON DELETE RESTRICT` for execution history, attempts, events, approvals, artifacts, and operation records.
- Use `ON DELETE CASCADE` only for unpublished definition children when a reviewed lifecycle explicitly permits physical deletion. Do not cascade-delete execution history.
- Avoid `ON UPDATE CASCADE`; UUID primary keys are immutable.
- Validate that workflow run project, definition, and version agree.
- Validate that task run definition belongs to the workflow definition version pinned by its parent run.
- Validate that both endpoints of a dependency edge belong to the same workflow definition.
- Validate that approval, attempt, operation, and event references all belong to the same workflow run/task lineage.
- Where SQLite composite foreign keys are used, declare matching composite UNIQUE constraints on parent tables. Otherwise enforce cross-parent consistency in the repository transaction and test it.
- Use UUID generation in the application layer. Do not depend on SQLite-specific UUID extensions.

### 4.3 Relationship cardinality

- Project to workflow definitions: one-to-many.
- Workflow definition to task definitions: one-to-many.
- Task definition to other task definitions through dependency edges: many-to-many, represented by `task_dependency`.
- Workflow definition to workflow runs: one-to-many.
- Workflow run to task runs: one-to-many.
- Task run to attempts, events, approvals, artifacts, and operations: one-to-many.
- No user/team many-to-many relationships exist in MVP.

---

## 5. Constraints and validation

### 5.1 Database-level constraints

All migrations must use database constraints as the final integrity boundary.

Required classes of constraints:

- `PRIMARY KEY` for every entity.
- `FOREIGN KEY` for every relationship.
- `NOT NULL` for required identity, state, timestamp, and ownership fields.
- `UNIQUE` for natural uniqueness and idempotency identities.
- `CHECK` for enums, booleans, positive versions, bounded timeouts, non-negative sizes/durations, and valid checksums.
- `DEFAULT` for safe, deterministic defaults only.
- `json_valid` checks for JSON columns if JSON1 is available in the approved SQLite runtime; startup/migration must verify support before relying on it.

Never rely on application validation alone for an invariant that SQLite can enforce.

### 5.2 Application-level validation

The backend must validate before every persistence write:

- UUID format and identity ownership.
- String normalization, maximum lengths, and non-empty required values.
- Canonical project paths and artifact-root containment.
- Pydantic schema and schema-version compatibility for JSON documents.
- Workflow/task status transitions against `EXECUTION_ENGINE.md`.
- DAG references, same-version membership, and cycle detection.
- Cross-table run/project/definition consistency.
- Retry counts, deadlines, and timeout bounds.
- Provider identifiers and executor types against approved registries.
- Approval action fingerprint and current preconditions.
- Idempotency-key stability and uniqueness.
- Artifact checksum, size, and storage path.
- Secret redaction before logs, event payloads, and persisted metadata.

### 5.3 State constraints

The workflow run status CHECK must use exactly:

- `READY`
- `RUNNING`
- `WAITING_FOR_APPROVAL`
- `PAUSED`
- `SUCCEEDED`
- `FAILED`
- `CANCELLED`
- `NEEDS_ATTENTION`

The task run status CHECK must use exactly:

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

Definition editability (`DRAFT` / `READY`) is separate from workflow-run status. Attempt and operation statuses are separate lifecycles and must not be conflated with task state.

### 5.4 Append-only and immutable records

- `execution_event` is append-only.
- Attempt history is not overwritten by retries.
- Operation intent and reconciliation history must remain auditable.
- Approval decisions are immutable historical facts; invalidation is an additional event/state.
- Workflow definitions used by runs are immutable snapshots.
- A correction must be represented by a new record/event, not by rewriting history to appear as if the original event never occurred.

---

## 6. Indexes and performance

### 6.1 Index policy

SQLite B-tree indexes are the approved default. Do not add Hash or GIN indexes; those are not SQLite index types. Do not add FTS5, R-tree, or other virtual-table extensions without an approved use case and technology review.

Every index must correspond to a demonstrated query pattern. Avoid indexing every column: indexes increase write amplification and backup size.

### 6.2 Required indexes

| Table | Index | Purpose |
|---|---|---|
| `workflow_definition` | UNIQUE `(project_id, name, version)` | Definition lookup and version uniqueness |
| `task_definition` | UNIQUE `(workflow_definition_id, task_key)` | Task identity within a version |
| `task_definition` | UNIQUE `(workflow_definition_id, position)` | Deterministic task ordering |
| `task_dependency` | PK `(task_id, depends_on_task_id)` | Dependency edge uniqueness and outgoing lookup |
| `task_dependency` | `(depends_on_task_id, task_id)` | Reverse dependency lookup |
| `workflow_run` | `(status, created_at)` | Active/recent workflow dashboard |
| `workflow_run` | `(project_id, created_at DESC)` | Project execution history |
| `workflow_run` | `(workflow_definition_id, workflow_version, created_at DESC)` | Runs by definition version |
| `task_run` | UNIQUE `(workflow_run_id, task_definition_id)` | One task run per definition per workflow run |
| `task_run` | `(workflow_run_id, status)` | Scheduler and workflow progress reads |
| `task_attempt` | UNIQUE `(task_run_id, attempt_number)` | Attempt order and uniqueness |
| `task_attempt` | `(task_run_id, started_at DESC)` | Attempt history |
| `execution_event` | `(workflow_run_id, occurred_at, id)` | Run timeline pagination |
| `execution_event` | `(task_run_id, occurred_at, id)` | Task event history |
| `approval_record` | `(decision, created_at)` | Pending approval queue |
| `approval_record` | `(workflow_run_id, created_at)` | Approval history by run |
| `artifact` | `(workflow_run_id, created_at DESC)` | Run artifacts |
| `artifact` | `(task_run_id, created_at DESC)` | Task artifacts |
| `operation_record` | UNIQUE `(idempotency_key)` | Prevent duplicate logical operation identity |
| `operation_record` | `(status, updated_at)` | Startup reconciliation of incomplete operations |
| `provider_config` | UNIQUE `(provider_key, display_name)` | Provider profile lookup |

The primary-key and UNIQUE constraints above create their own indexes; do not create duplicate indexes over the same columns.

### 6.3 Query and pagination rules

- Use parameterized SQL for all values.
- Avoid `SELECT *` in repository code; select required columns explicitly.
- Use stable keyset pagination for long event/attempt histories (timestamp plus UUID tie-breaker).
- Do not use unbounded event or artifact queries on dashboard paths.
- Use `EXPLAIN QUERY PLAN` when adding or changing a performance-sensitive query.
- Keep scheduler queries bounded to non-terminal runs and eligible task states.
- Avoid N+1 query patterns when loading workflow summaries and task status.
- Do not add full-text search, vector search, or JSON expression indexes for MVP without an approved feature and benchmark.

### 6.4 Performance targets

The PRD's provisional local targets remain the acceptance baseline:

- Ordinary workflow-state reads: p95 under 100 ms.
- Dashboard interactions: p95 under 200 ms excluding external requests.
- Status changes visible to the UI within one second under normal conditions.

These are targets to measure, not guaranteed performance claims. Benchmark with representative local datasets before adding indexes or changing the engine.

---

## 7. Transactions and concurrency

### 7.1 Transaction model

Use SQLite transactions with explicit boundaries. The default isolation behavior is SQLite's serializable transaction semantics; WAL improves reader/writer coexistence but does not permit multiple concurrent writers.

Use short transactions and commit before any external I/O.

### 7.2 Mandatory atomic transactions

The following logical operations must commit atomically:

| Operation | Records committed together |
|---|---|
| Create project | Project row |
| Publish workflow version | Definition version, task definitions, dependency edges |
| Create workflow run | Run row, pinned policy/version, all task-run rows, initial event |
| Claim task | Task state, attempt row, attempt deadline, dispatch event |
| Transition state | State update, transition event, parent workflow update when applicable |
| Schedule retry | Attempt result, retry count/next time, task state, event |
| Request approval | Approval row, task/workflow waiting state, event |
| Resolve approval | Decision, action validation result, corresponding state/event |
| Record side-effect intent | Operation row, stable idempotency key, event |
| Complete task | Attempt result, operation result, artifact metadata, validation result, task state, event |
| Reconcile interrupted operation | Operation classification, attempt/task/workflow changes, reconciliation event |
| Cancel run | Cancellation intent, not-started task state updates, event |

Do not hold an open transaction while calling a hosted model, Ollama, a subprocess, Git, filesystem operations of unbounded duration, or waiting for user approval.

### 7.3 Critical write locking

Use `BEGIN IMMEDIATE` for critical read-modify-write operations:

- Claiming the next eligible task.
- Applying workflow/task state transitions.
- Creating a retry schedule.
- Resolving an approval.
- Creating an operation intent/idempotency key.
- Applying recovery reconciliation.

Within the transaction:

1. Read current durable state.
2. Validate the expected prior state and ownership.
3. Validate the requested transition and related invariants.
4. Apply conditional updates.
5. Insert the corresponding event/history record.
6. Commit.

If the expected state changed, reject or re-evaluate from current persisted state. Never apply a stale UI or scheduler snapshot blindly.

### 7.4 Concurrency rules

- One backend process owns execution dispatch for the application instance.
- At most one task may be in `RUNNING` per workflow run in MVP.
- SQLite permits only one active writer at a time; keep write transactions short.
- Use conditional updates and transaction checks, not only Python locks, to protect durable invariants.
- A process-level mutex may reduce contention but is not the source of truth.
- Do not share a single `sqlite3.Connection` across concurrent tasks/threads without an explicitly serialized access strategy.
- Do not add connection-pool dependencies or distributed locks.
- On lock contention, wait only for the configured bounded timeout, then return a safe retryable error or surface a recoverable condition.
- A second backend process must not dispatch tasks against the same database.

### 7.5 Filesystem transaction boundary

SQLite cannot atomically commit with filesystem writes. Follow the staged artifact protocol in `EXECUTION_ENGINE.md`:

- Write to a unique temporary path.
- Flush/close and calculate checksum.
- Atomically rename to final path where supported.
- Commit artifact metadata and task completion.
- Reconcile orphan temporary/final files during startup.
- Never mark an artifact available before the file and metadata are both verified.

---

## 8. Migrations strategy

### 8.1 Approved migration mechanism

Use **versioned SQL migration files and a small project-owned Python migration runner built on the standard-library `sqlite3` module**.

- No Alembic, Prisma, Flyway, Django ORM, SQLAlchemy, or other migration framework is approved by this document.
- No ORM is approved by default.
- The migration runner itself must remain small, deterministic, testable, and documented.
- The migration mechanism must be consistent with `TECH_STACK.md`; changing it requires explicit technology approval.

### 8.2 Migration file conventions

- Store migrations in a single documented directory, e.g. `backend/migrations/`.
- Name files with a monotonically increasing sequence and descriptive slug, e.g. `0001_initial_schema.sql`.
- A migration is immutable once applied. Never edit a migration that may have run on a user's machine.
- Add a new forward migration to correct or extend an existing schema.
- Each migration must have a unique version and checksum recorded in the migration history table.
- Use explicit SQL transaction boundaries. A migration must either fully apply or leave the database at the prior valid version, except for SQLite operations that have documented transactional limitations.
- The runner must reject duplicate versions, checksum mismatches, gaps where ordering is required, and unsupported schema versions.
- Record application timestamp and migration identifier.
- Run `PRAGMA foreign_key_check` and relevant integrity checks after migrations.

### 8.3 Migration workflow

Before writing a migration:

1. Read this document and all relevant architecture/execution specifications.
2. Submit a schema change proposal describing the requirement and alternatives.
3. Identify affected tables, FKs, constraints, indexes, queries, data conversion, and rollback/recovery implications.
4. Obtain explicit human approval.
5. Update `DATABASE.md` first.
6. Write a forward-only migration and update repository persistence code.
7. Add migration tests against both an empty database and a copy of the prior schema with representative data.
8. Verify foreign keys, uniqueness, state constraints, and data preservation.
9. Run the full backend test suite and recovery tests.
10. Document release and backup requirements.

Never run destructive migrations automatically on startup without a verified backup and an explicit release policy.

### 8.4 Rollback policy

- Prefer forward-fix migrations over down migrations.
- Do not promise rollback when a migration destroys or transforms data irreversibly.
- Before a migration that may lose or rewrite data, create and verify a backup.
- If migration fails, preserve the original database and report the error; do not delete or recreate it silently.
- Schema downgrade is not supported as a general automatic operation in MVP.

---

## 9. Backup, recovery, and data lifecycle

### 9.1 Data classification

| Data | Classification | Default handling |
|---|---|---|
| Provider credentials | Secret | OS credential store only; never SQLite/logs |
| Project paths and names | User data | Local, access-controlled |
| Workflow definitions/configuration | User/project data | Local; exclude secret values |
| Task inputs and outputs | Potentially sensitive | Store only required data/references; local by default |
| Events and diagnostics | Operational data | Redacted, bounded, local |
| Approval decisions | Security/audit-relevant | Retain with execution history |
| Artifacts | Potentially sensitive | Local filesystem, path-restricted, checksummed |
| Provider usage metadata | Operational data | Store minimal non-secret metadata |

### 9.2 Backup policy and targets

This is a local desktop application; there is no remote backup service in MVP. Targets below are product defaults and must be exposed clearly to the user.

| Item | MVP policy |
|---|---|
| Backup frequency | Automatic daily backup while the application is running, plus before every schema migration |
| Additional backup | Before major application upgrades and before user-initiated destructive data operations |
| Backup method | SQLite online backup API or a verified consistent snapshot; do not copy only the main DB file while WAL is active |
| Artifact backup | Include application-managed artifacts referenced by retained records, or clearly report that artifacts are excluded |
| Backup location | Application-managed backup directory, with optional user-selected export destination |
| Retention | Keep the most recent 7 daily backups and 4 weekly backups; older backups may be pruned only after successful verification of newer backups |
| Integrity | Verify backup can be opened, run `PRAGMA integrity_check`, and run `PRAGMA foreign_key_check` |
| RPO target | 24 hours for automatically backed-up database state; zero loss is not guaranteed for changes since the last backup |
| RTO target | Restore the database and make the application usable within 30 minutes on supported local hardware, excluding manual artifact recovery or unusually large data sets |

Because the application is local-first, backups are not automatically off-device. Users must be told that device loss can destroy both the live database and local backups. Provide a documented manual export/copy workflow to a user-controlled external location; cloud sync is not part of MVP.

### 9.3 Restore procedure

1. Stop the backend and ensure no process holds the database.
2. Preserve the current database and WAL files for diagnosis; do not overwrite them immediately.
3. Select a verified backup and validate its integrity.
4. Restore into a new database path.
5. Run schema version checks, foreign-key checks, and application-level consistency checks.
6. Reconcile artifact references against the filesystem.
7. Reconcile interrupted workflow operations using `EXECUTION_ENGINE.md`.
8. Open the restored database only after validation succeeds.
9. Record restore metadata and any missing artifact/operation findings.
10. If consistency cannot be established, preserve the backup and require user intervention rather than silently discarding records.

### 9.4 Retention and deletion rules

MVP defaults:

- Workflow definitions and versions: retain until the user explicitly archives/deletes them; definitions referenced by runs cannot be physically deleted.
- Workflow runs, task runs, attempts, events, approvals, and operation records: retain until explicit user deletion/export policy is implemented; do not auto-delete operational history in MVP.
- Artifacts: retain while referenced by retained runs; deletion must be explicit and must leave a durable tombstone or clearly recorded artifact-unavailable state.
- Backups: retain according to the rotation policy in Section 9.2.
- Temporary files: remove only after startup reconciliation proves they are abandoned and not part of an active operation.
- Secrets: never copy into backups, exports, events, or logs. Credential lifecycle is managed by the OS credential store.
- Project files: the application does not own or delete user project files as part of database cleanup.

Use archive/soft-delete semantics for projects and workflow definitions where history must remain accessible. Do not add generic soft-delete columns to every table without a concrete lifecycle requirement. For append-only history, preserve records rather than marking each row deleted.

Physical deletion is allowed only through an explicit user action with a clear impact summary, a verified backup where applicable, and FK-safe handling. A deletion operation must not leave dangling artifact references or silently erase audit evidence.

### 9.5 Privacy and regulatory posture

The product is local-first and does not require an account, but this alone does not establish GDPR/CCPA compliance.

- Collect and retain only data required for the user's workflows.
- Provide a user-visible way to inspect and export local data before destructive deletion.
- Do not transmit project or execution data to a hosted provider unless the workflow explicitly selects that provider and the UI discloses the destination.
- Never log PII, secrets, tokens, passwords, or unredacted sensitive prompts by default.
- Redact diagnostic fields before persistence, not only at display time.
- Avoid collecting telemetry in MVP. Any future telemetry must be opt-in, documented, and separately approved.
- Deletion/export behavior must account for database rows, filesystem artifacts, backups, and OS credential-store references.
- A user deleting the live database may still have retained backups. The UI/documentation must explain that backup copies need separate deletion to complete a full local-data purge.
- Do not claim legal compliance without a jurisdiction-specific legal review and a verified data-subject workflow.

---

## 10. Persistence and execution-engine integration

The database schema must implement the durability contract in `EXECUTION_ENGINE.md`.

### 10.1 Required persistence boundaries

- Create run: workflow run, pinned version/policy, task-run rows, initial event in one transaction.
- Claim task: task state, attempt, deadline, and dispatch event in one transaction.
- Retry: attempt result, retry count, persisted absolute retry timestamp, task state, and event in one transaction.
- Approval: approval record, action fingerprint, waiting states, and event in one transaction.
- Operation: persist operation intent and idempotency key before external invocation.
- Completion: persist attempt/operation result, artifact metadata, validation result, task state, and event atomically.
- Recovery: persist reconciliation evidence and resulting state changes atomically.

### 10.2 Idempotency

- Every side-effecting operation must have a stable operation ID and unique idempotency key.
- Reuse the same key when retrying the same logical operation, only when the adapter/target contract supports it.
- Do not generate a new key merely because the process restarted.
- Do not reuse a key for changed action arguments or a changed target.
- If the target has no idempotency or status lookup support, uncertain operations must be blocked for reconciliation; never blindly replay them.
- Database uniqueness prevents duplicate operation identity records but cannot guarantee exactly-once behavior in external systems.

### 10.3 Append-only history

Execution events, attempts, approvals, and operation reconciliation evidence must preserve historical facts. State tables may update current status; history tables record what happened. A correction is an additional reconciliation record/event.

---

## 11. Security requirements

- Use parameterized SQL for every query. Never interpolate user/model-controlled values into SQL strings.
- Do not expose SQLite files or database paths through the local API.
- Validate project and artifact paths against configured boundaries.
- Apply least privilege to the application data directory and backup directory.
- Keep secrets out of SQLite, JSON configuration, exception text, logs, events, crash reports, and exported files.
- Redact sensitive fields before persistence and before display.
- Do not log raw SQL parameter values when they may contain user content or sensitive data.
- Do not load arbitrary SQLite extensions.
- Do not accept imported database files without integrity and schema validation.
- Ensure local API authentication/authorization is enforced before repository mutation.
- Backups must receive the same filesystem access protection as the live database.
- Do not claim encryption at rest unless it is provided by a verified OS/device control or separately approved encrypted database implementation.

---

## 12. Required verification and test matrix

Database work is incomplete until the following tests exist and pass.

| ID | Scenario | Required assertion |
|---|---|---|
| DB-001 | Fresh database | All approved tables, constraints, and indexes are created |
| DB-002 | Runtime version | Startup rejects unsupported SQLite version |
| DB-003 | Foreign keys | FK enforcement is enabled on every connection |
| DB-004 | Invalid FK | Orphan references are rejected |
| DB-005 | Duplicate definition version | Unique constraint rejects duplicate project/name/version |
| DB-006 | Duplicate task key | Unique constraint rejects duplicate task key per definition |
| DB-007 | Cross-version dependency | Application rejects dependency edges across workflow versions |
| DB-008 | DAG cycle | Application validation rejects cyclic workflow |
| DB-009 | Invalid state | CHECK constraint rejects unsupported workflow/task state |
| DB-010 | Invalid boolean/timeout | CHECK constraints reject invalid values |
| DB-011 | JSON validation | Invalid JSON is rejected where JSON1 support is required |
| DB-012 | Task claim atomicity | State, attempt, deadline, and event commit or roll back together |
| DB-013 | Event atomicity | Failed state write does not leave an orphan transition event |
| DB-014 | Attempt immutability | Retry creates a new attempt; earlier attempt remains unchanged |
| DB-015 | Idempotency uniqueness | Duplicate idempotency key is rejected |
| DB-016 | Approval fingerprint | Changed action cannot reuse prior approval |
| DB-017 | WAL recovery | Database reopens consistently after process interruption |
| DB-018 | Busy handling | Lock contention is bounded and does not corrupt state |
| DB-019 | Migration from empty | All migrations apply in order |
| DB-020 | Migration from prior schema | Existing data is preserved and upgraded correctly |
| DB-021 | Migration checksum | Modified applied migration is rejected |
| DB-022 | Migration failure | Database remains recoverable; no silent recreation |
| DB-023 | Backup consistency | Backup opens and passes integrity/FK checks |
| DB-024 | Restore | Restored database passes schema and application consistency checks |
| DB-025 | Artifact reconciliation | Missing, staged, and orphan artifacts are classified correctly |
| DB-026 | Secret redaction | Secrets are absent from persisted event and error metadata |
| DB-027 | Delete restrictions | Referenced execution history cannot be cascade-deleted |
| DB-028 | Query plans | Critical scheduler/dashboard queries use expected indexes |
| DB-029 | Concurrent claims | Competing claims cannot start two tasks for one workflow run |
| DB-030 | Repeat recovery | Repeated recovery does not duplicate events or operations |

Use temporary SQLite databases and isolated filesystem directories in tests. Include migration fixtures with representative historical data. Test crash boundaries and lock contention deterministically where possible.

---

## 13. Implementation and release acceptance checklist

- [ ] `DATABASE.md` has been reviewed and accepted by the project owner.
- [ ] SQLite runtime compatibility is checked and reported.
- [ ] WAL, foreign keys, synchronous mode, and busy timeout are configured and verified.
- [ ] All approved tables and constraints match this document.
- [ ] All FK relationships and cross-entity ownership checks are tested.
- [ ] Required indexes exist; duplicate/redundant indexes are avoided.
- [ ] State values match `EXECUTION_ENGINE.md`.
- [ ] Execution state and event history are committed atomically.
- [ ] Attempts and operation records preserve idempotency/recovery evidence.
- [ ] Secrets are stored outside SQLite and redacted before persistence/logging.
- [ ] Versioned SQL migrations and migration history are implemented.
- [ ] Backup and restore are tested, including WAL-aware backup.
- [ ] Artifact/database reconciliation is tested.
- [ ] Retention and deletion operations preserve FK and audit invariants.
- [ ] All applicable tests in Section 12 pass.
- [ ] No unapproved ORM, migration framework, database engine, extension, or persistence dependency has been introduced.

## 14. Final database rule

The MVP persistence layer is **SQLite through Python's standard-library `sqlite3`, plus application-managed local filesystem artifacts**. It is intentionally local, relational, explicit, and dependency-light.

**No AI coding agent or developer may add a table, relationship, column, index, migration framework, ORM, or alternate database without reading this document and receiving explicit human approval.** If a required data shape or persistence behavior is not defined here, stop and submit a schema change proposal before implementation.
