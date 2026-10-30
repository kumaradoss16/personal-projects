# Execution Engine Specification - AI Workflow Orchestrator

**Document:** `EXECUTION_ENGINE.md`  
**Version:** 1.0  
**Status:** Proposed - authoritative execution contract after project-owner acceptance  
**Target:** MVP v1.0  
**Execution model:** Local, durable, sequential, dependency-aware orchestration  
**Last reviewed:** 2026-10-30

---

## 1. Purpose, authority, and non-negotiable invariants

This document defines the authoritative runtime behavior for workflow execution: state machines, dependency resolution, scheduling, retries, approvals, cancellation, persistence, idempotency, failure reconciliation, and restart recovery.

It must be read before implementing or modifying:

- Workflow or task state transitions.
- A scheduler, queue consumer, worker loop, cron job, webhook handler, or event-triggered execution path.
- Retry, timeout, cancellation, pause, resume, or recovery behavior.
- Approval gates or any state-mutating workflow function.
- Database writes that affect execution state, task attempts, events, or external operations.

Read this document with:

- `prd.md` - product goals and functional requirements.
- `ARCHITECTURE.md` - modular-monolith boundaries and execution design.
- `SCOPE.md` - approved MVP capabilities and exclusions.
- `TECH_STACK.md` - approved technologies and dependencies.
- `DATABASE.md`, if created - canonical schema and persistence constraints.

If these documents conflict, do not invent behavior. Stop, describe the conflict, and request an explicit decision. Update the affected documents before implementing the resolution.

### 1.1 Absolute execution invariants

1. **The backend is the sole authority** for workflow state, task state, scheduling, approvals, retries, persistence, and recovery.
2. **Strict determinism:** for the same persisted state, immutable workflow definition/version, explicit command, and recorded external results, the engine must make the same state-transition and scheduling decisions.
3. **Stateless between persistence boundaries:** execution logic must not depend on hidden in-memory progress, mutable global state, unrecorded timers, or process-local assumptions. Any fact required to continue execution must be persisted before crossing an asynchronous boundary or returning control.
4. The engine is a deterministic state machine plus side-effect adapters. Model output is untrusted data and must never directly mutate authoritative state.
5. Every state transition is validated against the transition rules in this document and committed transactionally before being reported as durable.
6. A task is never eligible to run until all required dependencies have succeeded and its execution policy permits it.
7. MVP execution is sequential: at most one task executor is active per workflow run. No task parallelism.
8. A retry is a new persisted attempt, not a reset or rewrite of the previous attempt.
9. External side effects are **not guaranteed exactly once**. The system must use stable operation identities, idempotency where supported, and reconciliation for uncertain outcomes.
10. Recovery must preserve completed work and must not silently convert an uncertain operation into success or failure.
11. A human approval authorizes only the exact action snapshot that was reviewed.
12. No queue broker, workflow framework, distributed worker, cron system, or orchestration framework may be introduced for MVP. Use the in-process Python `asyncio` scheduler and SQLite persistence defined by `TECH_STACK.md` and `ARCHITECTURE.md`.

## 2. Execution model and terminology

### 2.1 Entities

- **Workflow definition:** immutable, versioned specification of tasks, dependencies, inputs, policies, and validation requirements.
- **Workflow run:** one execution instance of a specific workflow definition version.
- **Task definition:** immutable task configuration within a workflow version.
- **Task run:** runtime instance of a task within one workflow run.
- **Attempt:** one actual execution attempt of a task. Each retry creates a new attempt record.
- **Operation:** a side-effecting external action initiated by an attempt, such as a file write or Git operation.
- **Approval request:** persisted request for a human decision over a specific action and immutable action fingerprint.
- **Execution event:** append-only record of a meaningful state transition or execution event.
- **Artifact:** output file or structured result stored outside or inside SQLite with a durable reference.

### 2.2 MVP execution topology

1. A local FastAPI application accepts authenticated commands.
2. The application service validates the command and persists the requested change.
3. A bounded in-process `asyncio` scheduler loads eligible work from SQLite.
4. The scheduler executes one task at a time for a workflow run.
5. Provider and tool adapters perform external I/O under explicit timeout and permission policies.
6. Results, artifacts, attempts, operation records, events, and state transitions are committed to SQLite/filesystem using the persistence protocol.
7. The UI reads backend-reported persisted state.

SQLite is the durable source of truth. An in-memory queue, task object, coroutine, or notification is only a wake-up optimization; it is never the authoritative record of pending work.

### 2.3 Explicit MVP limits

- Sequential execution only; concurrency limit per workflow run is **1**.
- No cron scheduling, recurring schedules, webhook triggers, remote event consumers, or background schedule service.
- A user command through the local application API is the MVP workflow trigger.
- Dependency graphs may be represented as a DAG, but the MVP scheduler resolves ready nodes sequentially.
- No distributed queue, DLQ service, lease-based multi-worker coordination, or cross-machine execution.
- No exactly-once guarantee for arbitrary provider/tool side effects.

## 3. State machine and legal transitions

### 3.1 State ownership

Only the backend execution domain may change workflow or task states. UI actions, provider responses, tool output, and model-generated text are inputs to validated commands; they are not state transitions themselves.

Every transition must record:

- Entity ID and run ID.
- Previous state and next state.
- Transition reason/event type.
- UTC timestamp.
- Initiating actor: system, user, recovery, or adapter result.
- Relevant attempt/operation/approval identifiers.
- Correlation/event identifier.
- Safe diagnostic details, with secrets redacted.

A transition that does not appear in the matrices below is illegal and must be rejected without changing durable state.

### 3.2 Workflow states

| State | Meaning | Terminal? |
|---|---|---|
| `DRAFT` | Definition is being edited and is not executable | No |
| `READY` | Definition passed validation and a run may be started | No |
| `RUNNING` | Run is active; scheduler may dispatch an eligible task | No |
| `WAITING_FOR_APPROVAL` | Progress is blocked on one or more required human decisions | No |
| `PAUSED` | User or system has suspended further scheduling | No |
| `SUCCEEDED` | All required tasks and validations completed successfully | Yes |
| `FAILED` | Run failed under its configured failure policy | Yes |
| `CANCELLED` | Cancellation completed and no task remains eligible to continue | Yes |
| `NEEDS_ATTENTION` | Safe automatic progress is impossible; explicit reconciliation or user action is required | No |

A workflow definition may be `DRAFT` or `READY`. Runtime states apply to a workflow run. A run must retain the exact immutable definition version from which it was created.

### 3.3 Workflow transition matrix

Rows are current states; columns are target states. Only marked transitions are legal.

| From  To | DRAFT | READY | RUNNING | WAITING_FOR_APPROVAL | PAUSED | SUCCEEDED | FAILED | CANCELLED | NEEDS_ATTENTION |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| DRAFT | - | ✓ | - | - | - | - | - | - | - |
| READY | ✓ | - | ✓ | - | - | - | - | - | - |
| RUNNING | - | - | - | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| WAITING_FOR_APPROVAL | - | - | ✓ | - | ✓ | - | ✓ | ✓ | ✓ |
| PAUSED | - | - | ✓ | ✓ | - | - | ✓ | ✓ | ✓ |
| SUCCEEDED | - | - | - | - | - | - | - | - | - |
| FAILED | - | - | - | - | - | - | - | - | - |
| CANCELLED | - | - | - | - | - | - | - | - | - |
| NEEDS_ATTENTION | - | - | ✓ | ✓ | ✓ | - | ✓ | ✓ | - |

Transition conditions:

- `DRAFT → READY`: complete schema, valid DAG, valid task configuration, supported executors, and valid policies.
- `READY → DRAFT`: edit the definition before a run starts; this does not mutate any existing run's pinned definition.
- `READY → RUNNING`: explicit user start command; create run record and initial events transactionally.
- `RUNNING → WAITING_FOR_APPROVAL`: at least one next action requires approval and no further task may proceed.
- `RUNNING → PAUSED`: explicit pause command accepted; no new task may be dispatched.
- `RUNNING → SUCCEEDED`: all required tasks are successful or validly skipped under explicit optional-task semantics, and all required validations pass.
- `RUNNING → FAILED`: unrecoverable task/workflow failure under the configured policy.
- `RUNNING → CANCELLED`: cancellation is complete; no active side effect remains unaccounted for.
- `RUNNING → NEEDS_ATTENTION`: outcome, integrity, permission, or recovery cannot be safely determined automatically.
- `WAITING_FOR_APPROVAL → RUNNING`: all blocking approval requests have been approved and revalidation succeeds.
- `WAITING_FOR_APPROVAL → PAUSED`: user pauses while waiting for approval.
- `WAITING_FOR_APPROVAL → FAILED`: approval is rejected and the workflow policy defines rejection as failure.
- `WAITING_FOR_APPROVAL → CANCELLED`: user cancels the run.
- `WAITING_FOR_APPROVAL → NEEDS_ATTENTION`: approval is expired, invalidated, or action fingerprint no longer matches and cannot be safely recreated automatically.
- `PAUSED → RUNNING`: explicit resume command and recovery/policy checks succeed.
- `PAUSED → WAITING_FOR_APPROVAL`: resume reveals a pending approval gate.
- `PAUSED → FAILED`: a persisted failure is discovered during resume/reconciliation.
- `PAUSED → CANCELLED`: user cancels.
- `PAUSED → NEEDS_ATTENTION`: unresolved operation or inconsistent state is detected.
- `NEEDS_ATTENTION → RUNNING`: an authorized reconciliation action resolves the blocker and all invariants pass.
- `NEEDS_ATTENTION → WAITING_FOR_APPROVAL`: reconciliation establishes that a fresh approval is required.
- `NEEDS_ATTENTION → PAUSED`: user chooses to keep the run suspended after reviewing the issue.
- `NEEDS_ATTENTION → FAILED`: explicit reconciliation determines the run cannot safely continue.
- `NEEDS_ATTENTION → CANCELLED`: cancellation is confirmed safe and complete.

Terminal workflow states are immutable. A new run must be created to execute a completed, failed, or cancelled workflow definition again.

### 3.4 Task states

| State | Meaning | Terminal? |
|---|---|---|
| `PENDING` | Waiting for dependencies or scheduler evaluation | No |
| `READY` | Dependencies succeeded and policy permits execution | No |
| `RUNNING` | An attempt is executing | No |
| `WAITING_FOR_APPROVAL` | The next protected action is awaiting human decision | No |
| `RETRY_WAIT` | A retry is scheduled for a persisted future time | No |
| `SUCCEEDED` | Execution and required validation passed | Yes |
| `FAILED` | Task exhausted retry policy or failed permanently | Yes |
| `SKIPPED` | Task was intentionally skipped under explicit optional/dependency policy | Yes |
| `CANCELLED` | Task did not complete because the run was cancelled | Yes |
| `NEEDS_ATTENTION` | Task has an uncertain or unsafe outcome requiring reconciliation | No |

### 3.5 Task transition matrix

| From  To | PENDING | READY | RUNNING | WAITING_FOR_APPROVAL | RETRY_WAIT | SUCCEEDED | FAILED | SKIPPED | CANCELLED | NEEDS_ATTENTION |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| PENDING | - | ✓ | - | - | - | - | - | ✓ | ✓ | ✓ |
| READY | - | - | ✓ | ✓ | - | - | - | ✓ | ✓ | ✓ |
| RUNNING | - | - | - | ✓ | ✓ | ✓ | ✓ | - | ✓ | ✓ |
| WAITING_FOR_APPROVAL | - | ✓ | - | - | - | - | ✓ | - | ✓ | ✓ |
| RETRY_WAIT | - | ✓ | - | - | - | - | ✓ | - | ✓ | ✓ |
| SUCCEEDED | - | - | - | - | - | - | - | - | - | - |
| FAILED | - | - | - | - | - | - | - | - | - | - |
| SKIPPED | - | - | - | - | - | - | - | - | - | - |
| CANCELLED | - | - | - | - | - | - | - | - | - | - |
| NEEDS_ATTENTION | - | ✓ | - | ✓ | - | - | ✓ | - | ✓ | - |

Transition conditions:

- `PENDING → READY`: every required dependency is `SUCCEEDED`, task configuration is valid, and run is `RUNNING`.
- `PENDING → SKIPPED`: task is explicitly optional and a defined skip policy applies; record reason.
- `PENDING → CANCELLED`: workflow cancellation is accepted.
- `PENDING → NEEDS_ATTENTION`: definition or dependency integrity is inconsistent.
- `READY → RUNNING`: scheduler atomically creates a new attempt and claims the task.
- `READY → WAITING_FOR_APPROVAL`: an approval gate must be satisfied before dispatch.
- `READY → SKIPPED`: explicit optional-task policy permits skip.
- `READY → CANCELLED`: run cancellation is accepted before dispatch.
- `READY → NEEDS_ATTENTION`: required executor, input, permission, or invariant is invalid.
- `RUNNING → WAITING_FOR_APPROVAL`: executor reaches an action-specific approval gate; current attempt is suspended at a persisted checkpoint.
- `RUNNING → RETRY_WAIT`: retryable failure and retry budget remains; persist next eligible time.
- `RUNNING → SUCCEEDED`: executor result and all required deterministic validations pass.
- `RUNNING → FAILED`: non-retryable failure or retry budget exhausted.
- `RUNNING → CANCELLED`: cancellation confirmed and active operation is stopped or safely accounted for.
- `RUNNING → NEEDS_ATTENTION`: external side effect may have occurred but its outcome is unknown, or integrity cannot be established.
- `WAITING_FOR_APPROVAL → READY`: approval is granted for the exact action fingerprint and preconditions still hold.
- `WAITING_FOR_APPROVAL → FAILED`: approval is rejected and policy says rejection fails the task.
- `WAITING_FOR_APPROVAL → CANCELLED`: run cancellation is accepted.
- `WAITING_FOR_APPROVAL → NEEDS_ATTENTION`: action changed, approval expired, or approval cannot be validated.
- `RETRY_WAIT → READY`: persisted retry time has arrived and the run is still `RUNNING`.
- `RETRY_WAIT → FAILED`: retry policy is exhausted or retry is administratively disabled.
- `RETRY_WAIT → CANCELLED`: run cancellation is accepted.
- `RETRY_WAIT → NEEDS_ATTENTION`: retry cannot be safely resumed due to uncertain prior side effect.
- `NEEDS_ATTENTION → READY`: explicit reconciliation establishes safe retry/re-execution eligibility.
- `NEEDS_ATTENTION → WAITING_FOR_APPROVAL`: reconciliation determines a new approval is required.
- `NEEDS_ATTENTION → FAILED`: explicit reconciliation establishes permanent failure.
- `NEEDS_ATTENTION → CANCELLED`: cancellation and side-effect status are confirmed.

Task terminal states are immutable. A retry does not transition `FAILED → READY`; it is represented by the policy-controlled transition from the active attempt to `RETRY_WAIT` before the task reaches terminal failure. Manual reruns create a new workflow run or an explicitly modeled new task-run revision; they do not rewrite history.

### 3.6 Workflow/task consistency rules

- A workflow cannot be `SUCCEEDED` while any required task is non-terminal or not `SUCCEEDED`.
- A workflow cannot be `RUNNING` while a blocking approval is unresolved.
- If any required task enters `NEEDS_ATTENTION`, the workflow must enter `NEEDS_ATTENTION`.
- If a required task enters `FAILED`, the workflow enters `FAILED` unless a future, explicitly approved failure-handling feature defines another policy.
- Optional tasks may be `SKIPPED` only when the immutable workflow definition declares the skip behavior.
- A dependent task must never execute when any required dependency is failed, cancelled, skipped without an allowed skip policy, or unresolved.
- State updates for a task and its parent workflow must be committed in one transaction when they are logically part of the same transition.

## 4. Scheduling and dependency resolution

### 4.1 Trigger model

MVP supports one trigger: an explicit user command to start a validated workflow run.

The following trigger types are **out of scope** and must not be implemented for MVP:

- Cron or recurring schedule.
- Webhook.
- External event bus or remote event consumer.
- File watcher or repository polling trigger.
- Automatic trigger based on model output.

A future trigger system must create a durable, authenticated start command and use the same validated run-creation path. It must not bypass the state machine or permission checks.

### 4.2 DAG validation

A workflow definition may express task dependencies as a directed acyclic graph (DAG).

Before a workflow becomes `READY`:

1. Every task ID must be unique within the workflow version.
2. Every dependency must reference an existing task in the same workflow version.
3. Self-dependencies are rejected.
4. Cycles are rejected using deterministic graph validation.
5. Duplicate dependency edges are normalized or rejected consistently.
6. All required task configuration and executor references must validate.
7. The definition version is frozen for any run created from it.

The graph is immutable during a run. Runtime AI output cannot add, remove, or rewrite dependency edges.

### 4.3 Readiness algorithm

For each scheduler iteration:

1. Load the workflow run and its pinned definition version from SQLite.
2. Confirm the workflow state is `RUNNING`.
3. Load persisted task states and dependency relationships.
4. Identify tasks in `PENDING` whose required dependencies are all `SUCCEEDED`.
5. Validate task policy, inputs, executor availability, and permissions.
6. Transition eligible tasks to `READY` transactionally.
7. Select the next eligible task using a stable ordering: workflow definition order, then stable task ID as tie-breaker.
8. Atomically claim the task, create its attempt record, transition it to `RUNNING`, and append the dispatch event.
9. Execute the attempt outside the database transaction.
10. Persist the outcome before considering another task.

Only one task may be `RUNNING` per workflow run. A task must never be dispatched based only on an in-memory queue entry; the database claim and state transition are authoritative.

### 4.4 Failure propagation

- A required dependency that enters `FAILED`, `CANCELLED`, or `NEEDS_ATTENTION` blocks its dependents.
- The workflow must transition to `FAILED` or `NEEDS_ATTENTION` as appropriate.
- A dependent task must not be silently marked successful or skipped.
- Optional dependencies and skip propagation are not inferred. They must be explicitly defined in the workflow schema and are not assumed to exist in MVP.

### 4.5 Ordering and fairness

- One workflow run executes at most one task at a time.
- Task selection is deterministic using the stable ordering above.
- No priority aging, work stealing, parallel dispatch, or cross-run global fairness guarantee is required for MVP.
- If multiple workflow runs are active, the application scheduler may process them in a stable FIFO order by persisted start time and run ID, with no starvation guarantee beyond that policy.
- The scheduler must re-read durable state before dispatching each task.

## 5. Resilience, retries, and timeouts

### 5.1 Retry policy

Retries apply only to failures classified as transient and safe to retry.

MVP default policy:

| Setting | Default |
|---|---|
| Maximum attempts | 3 total attempts (initial attempt + up to 2 retries) |
| Backoff | Exponential |
| Base delay | 1 second |
| Multiplier | 2 |
| Maximum delay | 30 seconds |
| Jitter | Uniform random value in ±20% of calculated delay |
| Retry counter | Persisted per task run |
| Retry scheduling | Persist `next_attempt_at` in UTC before releasing scheduler control |

Delay before retry (n), where (n=1) for the first retry:

[
d_n = \min(30, 1 \times 2^{n-1}) \text{ seconds}
]

Apply bounded jitter of ±20% to the calculated delay. The selected jittered delay and resulting absolute UTC `next_attempt_at` must be persisted once. Recovery must reuse that persisted timestamp; it must not recalculate jitter.

The workflow/task definition may specify a stricter retry count or delay within documented system bounds. MVP must not allow unbounded retries. Any policy override must be validated and persisted as part of the immutable run snapshot.

### 5.2 Retryable failures

Examples that may be retryable when the operation is known not to have committed:

- Temporary provider connection failure before request acceptance.
- HTTP 429 or transient 5xx response, subject to provider retry guidance and bounded timeout.
- Temporary local service unavailability.
- Temporary filesystem contention where no partial side effect remains.
- Explicit transient adapter errors with a known safe retry classification.

### 5.3 Non-retryable failures

Examples:

- Invalid workflow or task configuration.
- Authentication or authorization failure.
- Invalid credentials or unsupported model.
- Schema/contract violation that will not change on retry.
- Permission denial.
- Invalid path or path outside project boundary.
- Approval rejection when policy defines rejection as terminal.
- Deterministic validation failure that requires changed input or implementation.
- Unsupported operation or permanent provider error.

### 5.4 Unknown outcome rule

If a request may have reached an external system and the engine cannot establish whether its side effect occurred, it must not blindly retry.

Instead:

1. Persist the attempt and operation as outcome-unknown.
2. Transition the task and workflow to `NEEDS_ATTENTION`.
3. Preserve the idempotency key and external correlation ID, if any.
4. Attempt adapter-specific status lookup/reconciliation when supported.
5. Require explicit user action if the result remains ambiguous.

### 5.5 Timeouts

- Every provider call and tool operation must have a finite timeout.
- Default task attempt timeout: **10 minutes**.
- Maximum configurable task attempt timeout for MVP: **60 minutes**.
- Provider request timeout: bounded by the task's remaining deadline; adapters may use a shorter configured timeout.
- A task deadline is absolute and persisted as UTC when the attempt starts.
- Retries do not extend the overall task deadline unless a new task-run execution is explicitly created.
- A timeout requests cancellation of the active operation. It does not prove that an external side effect did not occur.
- If the adapter confirms safe termination and no side effect, classify as retryable if budget remains.
- If termination or side-effect outcome is uncertain, transition to `NEEDS_ATTENTION`.

### 5.6 Dead-letter handling

MVP does not introduce a separate DLQ service, broker, or queue table.

The durable failure record is represented by:

- Task state `FAILED` or `NEEDS_ATTENTION`.
- Attempt history and classified error.
- Operation records and reconciliation status.
- Append-only execution events.
- Safe diagnostic metadata and artifact references.

A task that exhausts retries transitions to `FAILED`. A task with an uncertain side effect transitions to `NEEDS_ATTENTION`. These records remain queryable and must not be deleted or automatically requeued.

A future DLQ feature requires an approved scope and architecture change; it must preserve original attempts and operation identities.

## 6. Interruption, pause, resume, cancellation, and approval

### 6.1 Pause semantics

Pause is a request to stop scheduling new tasks, not a promise that arbitrary external work can be interrupted instantly.

When a pause command is received:

1. Authenticate and authorize the user command.
2. Persist a pause-request event and pause intent.
3. Stop dispatching new tasks immediately.
4. If no task is active, transition workflow to `PAUSED`.
5. If a task is active, signal cooperative cancellation/pause to the executor where supported.
6. Persist the executor checkpoint and operation status.
7. Transition the task to a safe persisted state: `READY` if it has not begun a side effect, `RETRY_WAIT` only if a retry is already validly scheduled, or `NEEDS_ATTENTION` if outcome is uncertain.
8. Transition the workflow to `PAUSED` once no unaccounted active operation remains.

The system must not mark a partially completed task `SUCCEEDED` merely because pause was requested. A non-checkpointable operation may finish before pause completes; the UI must show that pause is pending.

### 6.2 Resume semantics

Resume is an explicit user command.

Before `PAUSED → RUNNING`:

- Re-read persisted workflow and task states.
- Reconcile any attempt that was active at interruption.
- Verify the workflow definition version and execution policy snapshot.
- Verify pending approval status.
- Verify required provider/tool configuration and permissions.
- Confirm no unresolved operation remains.
- Recompute task readiness from persisted dependencies.

If an approval is pending, transition to `WAITING_FOR_APPROVAL`. If an uncertain operation exists, transition to `NEEDS_ATTENTION`. Do not resume by simply restarting an old coroutine or replaying an in-memory queue.

### 6.3 Cancellation semantics

Cancellation is a durable command, not an immediate deletion.

1. Persist cancellation intent and event.
2. Stop dispatching new tasks.
3. Mark tasks that have not started as `CANCELLED`.
4. Signal active executors to stop using their supported cancellation mechanism.
5. Await bounded cooperative shutdown.
6. Reconcile any operation that may have committed an external side effect.
7. Persist final task/attempt/operation outcomes.
8. Transition workflow to `CANCELLED` only when no active or uncertain operation remains.

If an active operation cannot be stopped or its outcome cannot be determined, the workflow must remain `NEEDS_ATTENTION` rather than falsely reporting completed cancellation. The user may later confirm cancellation after reconciliation.

Cancellation does not roll back already completed tasks or external side effects. Compensating actions are not automatic in MVP.

### 6.4 Approval gate protocol

An approval gate is a durable execution checkpoint.

Before a protected operation:

1. Construct a canonical action description containing action type, target, normalized arguments, project boundary, relevant policy, and expected impact.
2. Compute and persist an action fingerprint from the canonical action representation.
3. Persist an approval request linked to workflow run, task run, attempt, and operation.
4. Transition the task to `WAITING_FOR_APPROVAL`.
5. Transition the workflow to `WAITING_FOR_APPROVAL`.
6. Do not execute the protected side effect while approval is pending.
7. Display the exact action and its scope to the user.

On approval:

- Persist actor, decision, timestamp, action fingerprint, and approval record.
- Revalidate permissions, target, arguments, and preconditions.
- If the fingerprint or material preconditions changed, invalidate the approval and request a new decision.
- Transition task to `READY` and workflow to `RUNNING` only when all blocking approvals are valid.
- Execute only the approved action snapshot.

On rejection:

- Persist rejection and reason.
- Apply the configured rejection policy. MVP default is to fail the task and workflow unless the workflow explicitly defines a safe optional skip.
- Never treat rejection as permission to find an alternate unapproved action.

Approval is single-action and single-use by default. It must not authorize later actions, different arguments, changed targets, or future workflow runs.

### 6.5 Application shutdown

On normal application shutdown:

- Stop accepting new workflow starts.
- Persist shutdown intent.
- Stop dispatching tasks.
- Signal active operations to stop where safe.
- Wait for bounded graceful shutdown.
- Persist final checkpoints and operation status.
- Leave unresolved operations marked for recovery/reconciliation.

Forced process termination may bypass graceful shutdown; restart recovery must handle that case.

## 7. Persistence boundaries and idempotency

### 7.1 Durable source of truth

SQLite is authoritative for:

- Workflow definitions and immutable versions.
- Workflow runs and pinned definition versions.
- Task runs and states.
- Attempts and attempt deadlines.
- Retry counts and persisted retry timestamps.
- Approval requests and decisions.
- Operation intents, idempotency keys, and reconciliation status.
- Execution events.
- Artifact metadata, checksums, and storage references.
- Provider/tool execution metadata and safe usage records.

Large artifact bytes may live in the local filesystem. SQLite must store a durable path/reference, checksum, size, and lifecycle status.

### 7.2 Required transaction boundaries

The following operations must be atomic database transactions:

| Boundary | Data committed together |
|---|---|
| Create run | Run record, pinned workflow version, initial task-run records, initial event |
| State transition | Previous/next state, transition metadata, event record |
| Dispatch task | Task claim, `RUNNING` state, new attempt record, attempt deadline, dispatch event |
| Schedule retry | Attempt outcome, retry count, `RETRY_WAIT` state, absolute next-attempt timestamp, event |
| Create approval | Approval request, action fingerprint, task/workflow waiting states, event |
| Resolve approval | Decision, actor/time, approval state, task/workflow transition or pending reconciliation event |
| Record operation | Stable operation ID, idempotency key, action fingerprint, intent state, event |
| Complete task | Attempt result, operation outcome, artifact metadata, validation results, task state, event |
| Fail/reconcile | Attempt/operation classification, task/workflow state, reconciliation event |
| Cancel | Cancellation intent, run event, all not-yet-started task cancellation transitions where possible |

Do not hold a SQLite transaction open while waiting for a model, network request, process, human approval, or long-running filesystem operation.

### 7.3 External side-effect protocol

For every side-effecting tool operation:

1. Derive a stable operation ID from the persisted run/task/attempt/action identity.
2. Create a canonical action fingerprint.
3. Persist an operation-intent record and idempotency key before invoking the adapter.
4. Commit the transaction.
5. Invoke the adapter with the idempotency key when the target supports it.
6. Persist the returned external operation/reference ID and outcome.
7. If the process dies between invocation and result persistence, recover by querying the target with the operation ID/idempotency key where supported.
8. If the target cannot determine the outcome, mark the operation uncertain and require reconciliation.

The idempotency key must remain stable across retries of the same logical operation. A genuinely changed action must receive a new operation identity and must pass approval again where required.

### 7.4 Exactly-once limitations

The orchestrator cannot guarantee exactly-once effects for arbitrary external systems.

- If an adapter/target supports idempotency keys, reuse the same key for retries of the same logical operation.
- If the target supports status lookup, reconcile before retrying.
- If neither idempotency nor status lookup is supported, do not automatically replay an operation whose outcome is unknown.
- Record uncertainty and require a human decision.
- Never claim that a database transaction makes a remote API call, Git push, or filesystem write exactly once.

### 7.5 Filesystem artifact protocol

Filesystem and SQLite changes cannot share one atomic transaction. Use a staged artifact protocol:

1. Write output to a uniquely named temporary file inside the controlled application/project storage area.
2. Flush and close the file; compute its checksum and size.
3. Atomically rename to the final artifact path when supported by the filesystem.
4. Commit artifact metadata and task completion in SQLite.
5. On startup, reconcile temporary files, final files without metadata, and metadata whose file is missing.
6. Never expose an artifact as complete until both file verification and metadata commit succeed.

For project-file modifications, preserve a recoverable backup or prior content reference where the operation is destructive and policy requires it. Require approval for protected writes. Do not overwrite a target based on stale assumptions; verify its expected precondition/checksum before commit.

### 7.6 Event history

- Execution events are append-only.
- Do not update historical event records to make the timeline appear successful.
- Corrections are new reconciliation events referencing the earlier event/operation.
- Persist events in the same transaction as the state transition they describe.
- Store timestamps in UTC using timezone-aware values.
- Never store credentials or unredacted secrets in event payloads.

## 8. Failure reconciliation and restart recovery

### 8.1 Startup recovery protocol

On every backend startup, before accepting new execution dispatch:

1. Open SQLite and verify database availability, schema version, and required integrity constraints.
2. Run pending migrations only through the approved migration mechanism.
3. Load non-terminal workflow runs and their task/attempt/operation records.
4. Identify tasks persisted as `RUNNING` whose process-local executor no longer exists.
5. Identify attempts with expired deadlines or missing completion records.
6. Identify operations in intent/submitted/in-progress states without a definitive outcome.
7. Reconcile staged artifacts and filesystem metadata.
8. Recompute workflow/task consistency from durable records.
9. Apply only deterministic recovery transitions allowed by the state matrices.
10. Persist recovery events and all resulting transitions.
11. Make only safe tasks eligible for dispatch.
12. Start the scheduler after recovery completes.

If recovery cannot establish consistency, do not dispatch the affected workflow. Transition it to `NEEDS_ATTENTION` and preserve diagnostic evidence.

### 8.2 Orphaned task handling

A task left in `RUNNING` after process death is not automatically safe to rerun.

Classify the interrupted attempt:

| Evidence | Recovery action |
|---|---|
| No operation intent was committed | Mark attempt interrupted; task may return to `READY` after policy checks |
| Operation intent committed, adapter never invoked is provable | Mark operation not-started; task may return to `READY` |
| Adapter supports idempotency/status lookup | Query/replay with the same operation identity according to adapter contract |
| Operation outcome is definitively successful | Persist reconciled result and complete task only after required validation |
| Operation outcome is definitively not applied | Retry only if policy permits and retry budget remains |
| Operation may have happened and cannot be queried | Mark task/workflow `NEEDS_ATTENTION`; require user reconciliation |
| Local file write is partial or artifact checksum fails | Quarantine incomplete artifact; restore/repair only through deterministic recovery rules |

A process-local lock or Python task object is not proof that an external operation did or did not happen.

### 8.3 Reconciliation commands

MVP reconciliation is user-initiated through the application and backend service. Supported outcomes should be narrowly defined:

- **Confirm applied:** user or adapter evidence establishes that the operation completed; persist the evidence and validate resulting state.
- **Confirm not applied:** evidence establishes no side effect occurred; make the task eligible for retry if policy permits.
- **Mark failed:** evidence establishes permanent failure; transition to `FAILED`.
- **Keep blocked:** retain `NEEDS_ATTENTION` until evidence is available.

Every reconciliation action must record the actor, evidence/reason, affected operation, and timestamp. Do not offer an “ignore and continue” path that bypasses dependency or permission invariants.

### 8.4 Compensating transactions

MVP does not implement automatic distributed sagas or generic compensation.

- Completed tasks and external side effects are not rolled back automatically when a later task fails.
- A compensation may be performed only as a separately defined, explicitly authorized tool action.
- Compensation must have its own operation ID, idempotency key, approval if required, attempt record, and audit event.
- The original operation remains in history; compensation does not erase it.
- If no safe compensation exists, report the partial completion and require user intervention.

### 8.5 Recovery invariants

- Never discard a completed attempt or event.
- Never silently reset a terminal task state.
- Never automatically repeat an uncertain non-idempotent side effect.
- Never dispatch a dependent task before dependency success is durable.
- Never resume while an approval, permission, or operation outcome is unresolved.
- Never treat a missing artifact as successful output.
- Recovery must be deterministic and repeatable: running recovery twice must not duplicate state transitions or external side effects.

## 9. Concurrency, locking, and transaction safety

MVP is a single-user local application with one backend process.

- Enforce one active scheduler owner per application instance.
- Enforce at most one running task per workflow run.
- Use SQLite transactions and conditional updates to claim tasks safely.
- A task claim must verify its current state and parent workflow state inside the transaction.
- Do not rely solely on in-memory mutexes for durable correctness.
- Use process-level single-instance protection where supported by the application lifecycle.
- Keep transactions short; never hold write locks across external I/O.
- Configure SQLite for safe local concurrency and explicitly handle busy/locked errors with bounded retries.
- Do not introduce distributed locks, leases, or multi-worker coordination in MVP.

If a second backend process is detected or database ownership cannot be established, refuse to dispatch work and report the conflict rather than risking duplicate execution.

## 10. Determinism and stateless execution contract

This section is mandatory for all runtime AI agents and human contributors.

### 10.1 Deterministic decision function

The orchestration core must behave as a deterministic function of:

- Persisted workflow definition version.
- Persisted workflow/task/attempt/operation states.
- Persisted policies and configuration snapshot.
- Explicit user/system command.
- Recorded adapter outcomes.
- Explicitly supplied clock/random values where needed.

The core must not make state decisions based on:

- Hidden process memory.
- Unpersisted mutable globals or singleton state.
- Model-generated claims of completion.
- Wall-clock reads scattered throughout business logic.
- Unrecorded random numbers or retry jitter.
- Network responses not captured as adapter results.
- UI-local state.
- Task execution order inferred from dictionary/set iteration or nondeterministic collection order.

Inject clock and jitter/random sources at defined boundaries. Persist the chosen timestamp and jitter result before scheduling a retry.

### 10.2 Statelessness between persistence boundaries

- Reconstruct execution decisions from SQLite after any process restart.
- Persist every checkpoint required to continue, pause, retry, or reconcile.
- Treat in-memory queues as disposable wake-up signals.
- Do not store the only copy of a task result, approval, retry deadline, or operation identity in memory.
- Adapter calls must receive explicit inputs and return explicit typed outcomes.
- Domain logic must not depend on FastAPI request objects, UI state, or provider SDK globals.
- No coroutine may be the sole record that work is pending or in progress.

### 10.3 AI-generated implementation constraints

AI coding agents must:

1. Read `EXECUTION_ENGINE.md` before implementing any scheduler, queue consumer, cron job, webhook, retry handler, approval flow, or state-mutating function.
2. Implement only the state transitions and execution semantics defined here.
3. Never add an orchestration framework or queue technology prohibited by `TECH_STACK.md`.
4. Never let an LLM directly set workflow/task status or authorize an operation.
5. Never add implicit retries, hidden background tasks, or non-durable scheduling.
6. Stop and request a specification change if a requirement cannot be represented by the defined states and transitions.
7. Add tests for every new transition and every crash boundary introduced by a change.
8. Report the exact invariants and failure paths covered by tests.

## 11. Required verification and test matrix

Execution-engine behavior is not complete until the corresponding tests exist and pass.

| ID | Scenario | Required assertion |
|---|---|---|
| EE-001 | Valid workflow definition | Valid DAG can transition `DRAFT → READY` |
| EE-002 | Cyclic dependency | Validation rejects cycle; no run is created |
| EE-003 | Missing dependency | Validation rejects unresolved task reference |
| EE-004 | Start workflow | Run, task records, and initial events commit atomically |
| EE-005 | Dependency ordering | Dependent task never starts before required dependency succeeds |
| EE-006 | Deterministic ordering | Same persisted state selects the same next task |
| EE-007 | Sequential limit | Never more than one running task per workflow run |
| EE-008 | Illegal transition | Invalid transition is rejected without partial writes |
| EE-009 | Retry scheduling | Retry count, attempt, delay, and next-attempt timestamp persist atomically |
| EE-010 | Retry jitter recovery | Restart reuses persisted next-attempt timestamp; jitter is not recalculated |
| EE-011 | Retry exhaustion | Task transitions to `FAILED` after configured attempts |
| EE-012 | Non-retryable error | No retry is scheduled |
| EE-013 | Unknown external outcome | Task/run enter `NEEDS_ATTENTION`; no blind replay occurs |
| EE-014 | Timeout | Deadline enforced; uncertain side effect is reconciled rather than blindly retried |
| EE-015 | Pause | No new task dispatch after pause intent is persisted |
| EE-016 | Resume | Scheduler recomputes readiness from durable state |
| EE-017 | Cancel | No new task dispatch; active operation is accounted for before final cancellation |
| EE-018 | Approval pending | Protected side effect is not invoked before approval |
| EE-019 | Approval fingerprint mismatch | Changed action invalidates approval and requires a new decision |
| EE-020 | Approval rejection | Configured rejection policy is applied and audited |
| EE-021 | Crash before adapter call | Recovery safely makes the task eligible again |
| EE-022 | Crash after adapter call | Recovery reconciles by operation identity or enters `NEEDS_ATTENTION` |
| EE-023 | Crash after result before DB commit | No duplicate side effect; reconciliation follows adapter capability |
| EE-024 | Artifact crash boundary | Temporary/final file and DB metadata are reconciled without false success |
| EE-025 | Repeated recovery | Running recovery twice creates no duplicate transitions or side effects |
| EE-026 | Event atomicity | State transition and event either both commit or neither commits |
| EE-027 | Terminal immutability | Terminal states cannot be silently reopened or rewritten |
| EE-028 | Dependent failure | Blocked dependent task never executes |
| EE-029 | Secret redaction | Credentials absent from events, logs, and error payloads |
| EE-030 | Concurrent dispatch attempt | Duplicate scheduler/claim cannot run two tasks for one workflow run |

### 11.1 Test design rules

- Use a temporary SQLite database and isolated filesystem directories.
- Use deterministic fake providers and tool adapters for ordinary tests.
- Inject a fake clock and deterministic random source for retry/backoff tests.
- Simulate process death at each side-effect persistence boundary.
- Assert both final state and complete event/attempt/operation history.
- Test negative behavior: no dispatch, no retry, no side effect, and no transition when policy denies it.
- Keep live provider calls opt-in and out of deterministic CI.

## 12. Operational diagnostics

For every active or failed run, the system should expose:

- Workflow/run ID and definition version.
- Workflow and task state.
- Current or last attempt ID.
- Attempt count and retry deadline.
- Dependency blockers.
- Pending approval details.
- Operation ID and external correlation reference, when available.
- Last durable event timestamp.
- Failure classification and safe diagnostic summary.
- Whether recovery or user reconciliation is required.

Diagnostics must be based on persisted records and must redact credentials, tokens, sensitive prompt content, and protected file contents according to the data policy.

## 13. MVP acceptance criteria

The execution engine is accepted only when:

- [ ] Workflow and task states match the enumerations in this document.
- [ ] Every legal transition is implemented and every illegal transition is rejected.
- [ ] DAG validation rejects cycles and missing dependencies.
- [ ] The scheduler executes eligible tasks in deterministic sequential order.
- [ ] No task dispatch occurs before dependency success is durable.
- [ ] Retry count, exponential backoff, jitter result, and retry deadline are persisted.
- [ ] Timeouts are finite and do not imply that external side effects were rolled back.
- [ ] Approval gates block side effects until exact action approval is validated.
- [ ] Pause, resume, and cancellation have durable and tested semantics.
- [ ] State, events, attempts, approvals, and operation intents are committed at required boundaries.
- [ ] Idempotency keys and reconciliation prevent blind replay of uncertain operations.
- [ ] Startup recovery handles orphaned attempts and artifact inconsistencies.
- [ ] Repeated recovery is deterministic and does not duplicate side effects.
- [ ] No separate queue, orchestration framework, cron service, or distributed infrastructure has been introduced.
- [ ] All required tests in Section 11 pass.

## 14. Final execution rule

The MVP execution engine is a **deterministic, stateless-between-persistence-boundaries, local sequential scheduler backed by SQLite**, with explicit state machines, action-scoped approvals, bounded retries, finite timeouts, idempotency controls, and restart reconciliation.

**No runtime agent or contributor may invent alternative state transitions, scheduling semantics, retry behavior, or recovery shortcuts. If the required behavior is not defined here, stop and obtain an approved specification change before implementation.**
