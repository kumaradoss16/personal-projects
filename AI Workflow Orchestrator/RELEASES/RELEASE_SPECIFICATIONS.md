# AI Workflow Orchestrator — Release Specifications

**Status:** Proposed; derived from current repository specifications  
**Release ladder:** v0.1, v0.2, v0.3, v0.4, v0.5, v0.9, v1.0

This document decomposes existing requirements into release work packages. It does not redefine schemas, API payloads, state matrices, security policy, or UI behavior. Those remain governed by their domain specifications.

## Common release contract

For every release:
- Read `AGENTS.md`, `AGENT_WORKFLOW.md`, `SCOPE.md`, `SECURITY.md`, then task-relevant domain specifications.
- Inspect actual repository state and preserve unrelated changes.
- Plan file/layer/data/API/security impact before implementation.
- Obtain approval for dependency, schema, API, security, architecture, and scope changes.
- Add tests before or alongside behavior; test negative and failure paths.
- Report exact commands and observed results; do not claim unrun tests passed.
- Update relevant documentation and stop at the release gate.

### v0.1 — Core Foundation and Mock Execution

#### A. Objective
Prove the smallest useful backend-authoritative sequential workflow execution slice using a mock provider and deterministic task behavior.

#### B. User-visible outcome
A developer can create a local project, define a small dependency-aware workflow, execute it through a minimal CLI, and inspect persisted run/task/event information.

#### C. Scope
**Included:** Python 3.12/FastAPI bootstrap; approved configuration; loopback-only API boundary and bootstrap authentication; SQLite initialization/migration runner; initial repositories; project/workflow/run/task/attempt/event records needed by the slice; workflow validation and DAG cycle detection; exact initial state transition commands; sequential asyncio scheduler; mock provider; deterministic transform; run creation/idempotency contract; minimal CLI; pytest tests; Ruff and mypy checks.

**Infrastructure:** SQLite standard library, local filesystem, approved pinned dependencies only.

**Excluded:** Ollama/hosted provider calls, real tool adapters, approvals, Tauri UI, advanced recovery/retries, MCP/n8n, SSE, Docker, DLQ, parallel execution, unrestricted shell.

#### D. Traceability
- `SCOPE.md` §3.1, §3.3, §3.4; P0-01, P0-02, P0-03, initial P0-04 slice.
- `ARCHITECTURE.md` application/domain/infrastructure boundaries and implementation sequence.
- `TECH_STACK.md` approved Python/FastAPI/Pydantic/SQLite/pytest/Ruff/mypy baseline.
- `DATABASE.md` connection, migration, transaction and initial entity contracts.
- `API_SPECIFICATION.md` API versioning, auth, run creation, idempotency and error envelope.
- `EXECUTION_ENGINE.md` workflow/task states, dependency validation and sequential scheduling.

#### E. Technical implementation breakdown
- **Presentation/CLI:** minimal command interface; no desktop frontend.
- **API/application:** request validation, authenticated routes, run-creation use case.
- **Domain:** workflow/task definitions, DAG validator, state transition rules.
- **Execution:** sequential scheduler and mock task/provider adapters.
- **Persistence:** SQLite migration runner and repositories; transaction boundaries from DATABASE.md.
- **Security:** loopback binding, bootstrap token boundary, Host/Origin and request protections as specified.
- **Data flow:** CLI → authenticated API/use case → domain validation → durable run/task creation → scheduler → mock adapter → transactional state/event persistence.
- **Failure behavior:** reject invalid definitions/transitions; do not mark success from untrusted model-like output; record failure and preserve durable history.

No source-code paths are prescribed here because implementation tree has not yet been created/inspected locally.

#### F. Dependencies
Stage 0 repository and documentation readiness → approved toolchain → database bootstrap → domain/state model → scheduler → CLI/API integration. Migration and state tests are critical-path prerequisites.

#### G. Testing
- Unit: domain validation, DAG cycle detection, state transition matrix, idempotency key validation.
- Integration: fresh database migration, repository persistence, API auth, run creation.
- Execution: three tasks with dependencies execute in required order; independent tasks remain sequential.
- Failure: malformed workflow, unknown dependency, illegal transition, provider mock failure, duplicate run request.
- Restart: persisted state/attempt/event records remain readable after process restart.
- Static: Ruff and mypy; tests via approved pytest stack.

#### H. Security/reliability gates
Loopback-only binding; protected routes require valid process-scoped authentication; validate Host/Origin; no wildcard CORS; body/list/timeout limits follow API spec; no secrets in logs; no frontend authority; use transactions for critical state changes; never hold write transactions across external calls.

#### I. Definition of Done
- A three-task workflow executes in dependency order.
- Workflow and task states are changed only through validated backend commands.
- Illegal transitions produce no durable state mutation.
- Run, task, attempt and event records persist.
- Fresh database migrations succeed.
- Duplicate run creation follows documented idempotency semantics.
- Protected API request without valid authentication is rejected.
- Relevant tests and Ruff/mypy checks pass.
- No unapproved provider/tool/UI feature is introduced.

#### J. Exit criteria
v0.1 evidence is reviewed and approved. v0.2 may begin only after state/persistence invariants are stable.

#### K. Risks and unresolved decisions
Initial state/schema mistakes can be expensive to migrate. Confirm exact initial schema mapping to DATABASE.md. Hosted provider choice is not a v0.1 blocker. Local branch/working-tree state must be inspected before implementation.

#### L. Deliverables
Backend bootstrap, configuration, migration/repository foundation, domain models/validators, sequential mock execution, CLI/API slice, tests, development instructions, release evidence.

---

### v0.2 — Durable Execution and Recovery

#### A. Objective
Complete the reliability semantics required for long-running workflows before real providers and side-effecting tools are introduced.

#### B. User-visible outcome
A user can observe bounded retry, timeout, pause/cancel behavior and recoverable execution history after interruption; uncertain external outcomes are surfaced for attention.

#### C. Scope
**Included:** persisted attempts/events; retry classification and bounded retry schedule; `RETRY_WAIT` and persisted `next_attempt_at`; operation/task timeouts; pause/resume/cancel semantics; leases/heartbeats; orphan detection; startup recovery; `NEEDS_ATTENTION`; consistency checks; redacted structured logs; recovery tests.

**Excluded:** provider integrations, tool integrations, approvals, UI, parallel execution, exactly-once external side effects, distributed queues, DLQ.

#### D. Traceability
- `SCOPE.md` §3.4, §3.8, §3.10; P0-04 and P0-05.
- `EXECUTION_ENGINE.md` retry, timeout, cancellation, lease, recovery and uncertain-outcome sections.
- `DATABASE.md` attempts/events/transaction and recovery contracts.
- `SECURITY.md` redaction and fail-closed requirements.
- `RULES.md` error handling and data integrity.

#### E. Technical implementation breakdown
- **Domain:** retry eligibility, state transitions, cancellation boundary, uncertain-outcome classification.
- **Execution runtime:** scheduler leases/heartbeats, timeout enforcement, startup recovery.
- **Persistence:** attempts, events, retry timestamps, leases and operation records per schema.
- **API:** expose only routes approved for the release; preserve stable error envelope.
- **Data flow:** scheduler claims task transactionally → executes outside write transaction → persists outcome/next retry → recovery reconciles stale work.
- **Failure:** transient safe failures may retry; policy denial/invalid input/deterministic validation failure do not retry by default; timeout after side effect may become `NEEDS_ATTENTION`.

#### F. Dependencies
v0.1 state machine and persistence. Recovery tests depend on stable attempt/event model. No external provider is required to test simulated interruption and uncertain outcomes.

#### G. Testing
- State-transition exhaustive legal/illegal tests.
- Retry classification and persisted schedule tests, including restart reuse of `next_attempt_at`.
- Timeout and cooperative cancellation tests.
- Crash between claim and completion; stale lease recovery.
- Uncertain side effect is not silently marked `SUCCEEDED` or blindly retried.
- Transaction rollback and consistency tests.
- Secret-redaction tests.

#### H. Security/reliability gates
No exactly-once claims; no write transaction during network/subprocess/approval wait; fail closed on missing policy; preserve append-only event history; no silent terminal-state rewrite.

#### I. Definition of Done
Persisted retry scheduling survives restart; timeout/cancel/pause semantics match EXECUTION_ENGINE.md; orphan work is detected; uncertain operations become reconciled or `NEEDS_ATTENTION`; tests prove no duplicate unsafe side effect.

#### J. Exit criteria
Recovery and retry evidence is approved before real provider/tool side effects are introduced.

#### K. Risks and unresolved decisions
Recovery correctness is high risk; lease expiry and reconciliation behavior must follow exact source contract. Any undocumented operational threshold is an owner decision, not an agent invention.

#### L. Deliverables
Retry/recovery services, lease and consistency logic, persisted scheduling, redacted logs, failure/restart test suite, updated operational documentation.

---

### v0.3 — Provider Integration

#### A. Objective
Add real inference while preserving provider neutrality and explicit control over local versus hosted data transmission.

#### B. User-visible outcome
A user can configure Ollama and one selected hosted provider, execute model-backed workflow tasks, and inspect provider/model outcome metadata.

#### C. Scope
**Included:** provider interface; Ollama adapter; one hosted provider adapter; configuration validation; credential references and OS credential storage; normalized request/response/error/timeout behavior; available usage metadata; explicit external data destination disclosure.

**Excluded:** multiple hosted providers, model routing, automatic fallback, consensus, cost optimization, MCP, n8n.

#### D. Traceability
- `SCOPE.md` §3.5; P0-06.
- `TECH_STACK.md` provider adapter and conditional SDK rules.
- `ARCHITECTURE.md` provider adapter boundary.
- `SECURITY.md` secrets and external transmission.
- `EXECUTION_ENGINE.md` provider timeout and uncertain operation behavior.

#### E. Technical implementation breakdown
- **Domain/application:** provider-neutral request and result contracts.
- **Infrastructure:** Ollama HTTP adapter and one hosted provider adapter.
- **Configuration:** explicit provider/model selection and validation.
- **Secrets:** OS credential store; database stores reference only.
- **Data flow:** workflow task → provider interface → selected adapter → remote/local endpoint → normalized result → durable attempt/event.
- **Failure:** normalize authentication, rate limit, timeout, unavailable, invalid response and provider errors; never silently switch provider.
- **Dependency:** hosted SDK choice and exact version require owner selection and TECH_STACK registry update before install.

#### F. Dependencies
v0.1 execution contract and v0.2 timeout/recovery semantics. Hosted provider work is blocked until provider/SDK decision. Ollama can proceed independently after provider contract approval.

#### G. Testing
Adapter contract tests; Ollama success/unavailable/timeout/malformed response; hosted provider auth/rate-limit/timeout; credential redaction; no secret in SQLite/logs/errors; no implicit fallback; explicit hosted-data disclosure.

#### H. Security/reliability gates
No credentials in source, DB plaintext, URLs, frontend storage, prompts, logs or exports. Backend owns provider calls. External transmission must be visible and deliberate.

#### I. Definition of Done
Ollama and exactly one selected hosted adapter pass success and failure tests; configuration is validated; secrets remain protected; errors and metadata are normalized; no unapproved routing/fallback.

#### J. Exit criteria
Both provider paths are verified and security review accepts credential/data-transmission handling.

#### K. Risks and unresolved decisions
Hosted provider and SDK are undecided. Provider rate-limit and usage metadata differ; preserve unavailable values rather than fabricate. External network behavior is not fully deterministic in CI.

#### L. Deliverables
Provider contract, Ollama adapter, selected hosted adapter, credential reference integration, configuration validation, provider tests and data-transmission documentation.

---

### v0.4 — Controlled Tools, Approvals and Verification

#### A. Objective
Introduce the MVP's narrow developer tool set only after authorization, retry and recovery foundations are established.

#### B. User-visible outcome
A workflow can perform explicitly allowed project operations, pause for action-specific human approval, and present deterministic validation/artifact evidence.

#### C. Scope
**Included:** project-scoped filesystem read/write; read-oriented Git status/diff/history; configured test-runner; tool adapter interface; allowlists; canonical path validation; subprocess controls; approval queue; action fingerprints; single-use action-scoped approvals; revalidation; deterministic validators; atomic artifact writes/checksums; success versus verified distinction.

**Excluded:** unrestricted shell, arbitrary executable invocation, general computer use, MCP, n8n, browser automation, autonomous tool authorization.

#### D. Traceability
- `SCOPE.md` §3.6–3.10; P0-07 and P0-08.
- `SECURITY.md` filesystem, process, authorization, approval and threat-boundary controls.
- `EXECUTION_ENGINE.md` approval lifecycle and uncertain outcome semantics.
- `API_SPECIFICATION.md` approved tool/approval routes and error contracts.
- `DATABASE.md` approval, operation, artifact and event persistence.

#### E. Technical implementation breakdown
- **Tool application services:** policy check before every operation.
- **Filesystem adapter:** canonical path under approved project root; reject traversal, symlink/junction escape and ADS; atomic writes.
- **Git adapter:** read-oriented inspection only for MVP.
- **Test runner:** configured allowlist; explicit executable identifiers and argument arrays; `shell=False`; sanitized environment; bounded time/output.
- **Approval service:** canonical action description/fingerprint, run/task/attempt/operation binding, expiry/invalidation rules per spec, revalidation immediately before execution.
- **Validation:** deterministic schema/field/file/test checks; evidence linked to task/run.
- **Failure:** changed action invalidates approval; missing/ambiguous authorization denies; side-effect timeout is uncertain and requires reconciliation.

#### F. Dependencies
v0.2 safe retry/recovery; v0.3 provider/task contract; stable DB/API approval contracts. Tool execution and approval tests must precede UI controls.

#### G. Testing
Path traversal, symlink/junction escape, ADS, project-root escape; process injection and shell metacharacters; unauthorized executable; timeout/output cap; approval fingerprint mutation; stale/reused approval; reject/timeout; backend policy bypass attempt; artifact atomicity/hash mismatch; validator false/true cases.

#### H. Security/reliability gates
Fail closed. No frontend-only authorization. No model self-approval. Approval covers only canonical action snapshot. No shell=True. No arbitrary path or command. Secrets and sensitive outputs redacted.

#### I. Definition of Done
Every tool action is allowlisted and backend-authorized; approvals are action-scoped and revalidated; no project-root escape passes tests; artifact integrity is recorded; deterministic validation is inspectable; `SUCCEEDED` and `VERIFIED` remain distinct.

#### J. Exit criteria
Security review and negative-path tests approve tool and approval boundaries.

#### K. Risks and unresolved decisions
Windows path canonicalization, junctions and process behavior require platform tests. Exact approval expiry behavior must follow the source specification. Any broader Git write operation requires scope/security review.

#### L. Deliverables
Tool adapters, policy/approval services, validator and artifact services, API/database integration, negative-path tests and permission documentation.

---

### v0.5 — Tauri Desktop Application

#### A. Objective
Deliver the approved functional desktop interface on top of stable backend contracts.

#### B. User-visible outcome
A user can manage projects/workflows, start and inspect runs, respond to approvals, configure providers/tools and inspect history/artifacts through the Windows desktop application.

#### C. Scope
**Included:** Tauri 2 shell; secure backend lifecycle/bootstrap; typed API client; vanilla TypeScript UI; functional DESIGN.md screens; bounded polling; accessibility; loading/empty/error/denied/unavailable states.

**Excluded:** nonfunctional mock controls, automatic conversation capture, SSE, visual node-canvas editor, cloud sync, non-MVP screens.

#### D. Traceability
- `SCOPE.md` §3.1, §3.2, §3.11; P0-01 and P0-09.
- `DESIGN.md` information architecture, components, tokens and interactions.
- `TECH_STACK.md` Tauri/Rust/TypeScript/Node baseline.
- `API_SPECIFICATION.md` routes and response contracts.
- `ARCHITECTURE.md` desktop/backend lifecycle and boundary.

#### E. Technical implementation breakdown
- **Native shell:** launch/monitor/stop backend; private bootstrap channel.
- **Frontend:** typed API client, rendering and user intent only.
- **Backend:** all state and authorization decisions remain server-side.
- **Polling:** bounded refresh; no SSE through v1.0 under current decision.
- **Data flow:** UI action → authenticated API → backend use case → persisted state → API response → UI refresh.
- **Failure:** backend unavailable, expired token, denied action, provider unavailable and stale data have distinct UI states.

#### F. Dependencies
v0.1 API/domain contracts; v0.3 provider configuration; v0.4 approvals/tools; stable DESIGN.md. UI may be prototyped against contract fixtures, but must not claim functional completion until integrated.

#### G. Testing
Typed API contract tests; UI state rendering; backend unavailable/auth expiry; denied approval; project/workflow/run end-to-end; keyboard/accessibility; polling behavior; startup/shutdown lifecycle; Windows integration.

#### H. Security/reliability gates
No token in persistent frontend storage; backend bootstrap protected; loopback and Origin/Host protections; no direct provider calls from UI; no UI-owned workflow status; no fake provider health or usage.

#### I. Definition of Done
Required screens support real operations; UI reflects persisted backend state; all error/empty/loading/denied states are implemented; Tauri lifecycle and API authentication work on Windows.

#### J. Exit criteria
Core user journeys pass desktop integration tests and design review.

#### K. Risks and unresolved decisions
Backend lifecycle and secure bootstrap are sensitive; API contract drift can cause UI/backend inconsistency. Avoid framework/dependency changes without approval.

#### L. Deliverables
Tauri shell, frontend components/screens, typed API client, UI tests, Windows integration evidence and updated user workflow docs.

---

### v0.9 — Production Readiness and Release Candidate

#### A. Objective
Prove the integrated MVP is safe to install, operate, restart, back up and recover on supported Windows environments.

#### B. User-visible outcome
A release candidate can be installed cleanly and used with documented limitations, permissions, backup and recovery procedures.

#### C. Scope
**Included:** full integration/regression; security and data-integrity review; WAL compatibility and backup/restore tests; crash/restart; provider/tool/approval failure tests; clean-clone CI; Windows packaging/install/startup/shutdown/upgrade; user documentation; release blocker triage.

**Excluded:** new feature expansion, Docker runtime, DLQ, SSE, MCP/n8n, parallel execution.

#### D. Traceability
- `SCOPE.md` §3.8, §3.10, §3.12 and §9; P0-05, P0-08, P0-10.
- `DATABASE.md` WAL, transactions, migrations, backup/recovery.
- `SECURITY.md` security tests and release gates.
- `TECH_STACK.md` CI and pinned toolchain.
- `RULES.md` definition of done and Git hygiene.

#### E. Technical implementation breakdown
Cross-layer test and packaging work. Verify all approved runtime dependencies and versions; ensure local data paths; test migration/upgrade and backup/restore; validate logs and redaction; inspect network exposure; package the Tauri/sidecar/backend arrangement as specified by architecture.

#### F. Dependencies
All v0.1–v0.5 capabilities integrated. Windows backup/restore verification is a release blocker for WAL acceptance.

#### G. Testing
Full pytest suites; API contract; DB migration and transaction; WAL backup/restore while active and after clean shutdown as specified by DB contract; crash/restart; uncertain side effect; provider timeout; path/process/approval negative paths; UI end-to-end; clean install and upgrade; clean-clone CI.

#### H. Security/reliability gates
No critical/high release-blocking security or data-integrity finding left unresolved without explicit owner risk acceptance. Validate local API exposure, token handling, secrets, filesystem/process boundaries, approvals, recovery and artifact integrity.

#### I. Definition of Done
Release candidate artifact is reproducible; clean install/startup/shutdown/upgrade tested; backup/restore proven; all critical-path tests pass; release blockers resolved; docs are complete.

#### J. Exit criteria
Owner/reviewer approves RC evidence and v1.0 acceptance checklist is fully traceable.

#### K. Risks and unresolved decisions
Windows packaging and WAL backup behavior are major risks. CI environment and supported Windows versions must match TECH_STACK/architecture; do not invent broader support.

#### L. Deliverables
RC build, CI results, test/security reports, backup/restore evidence, installation/configuration/recovery docs, known-issues list and release checklist.

---

### v1.0 — Production MVP

#### A. Objective
Release the smallest complete product that satisfies the approved MVP boundary.

#### B. User-visible outcome
The user can create and manage a local project, define and execute sequential AI workflows with Ollama and one hosted provider, use approved project-scoped tools, authorize consequential actions, inspect deterministic verification and recover execution safely.

#### C. Scope
**Included:** only the approved `SCOPE.md` §3 capabilities and required v0.9 remediation.

**Excluded:** deferred and explicitly excluded scope remains excluded. No new feature work during release stabilization.

#### D. Traceability
All P0-01–P0-10 milestones and `SCOPE.md` §9 acceptance checklist; PRD requirements admitted by SCOPE; domain acceptance contracts in ARCHITECTURE, TECH_STACK, DATABASE, API_SPECIFICATION, EXECUTION_ENGINE, SECURITY and DESIGN.

#### E. Technical implementation breakdown
Release integration and defect correction only. No new architectural subsystem or dependency.

#### F. Dependencies
Approved v0.9 release candidate and owner sign-off.

#### G. Testing
Full clean-clone CI; full unit/integration/contract/state/DB/recovery/security/UI/packaging regression; end-to-end MVP scenario; installation and restore evidence.

#### H. Security/reliability gates
All security controls remain enabled in packaged builds; no critical unresolved data-loss, unauthorized execution, secret exposure or recovery defect; known limitations documented.

#### I. Definition of Done
Every applicable `SCOPE.md` §9 item has evidence; all release gates approved; artifact and release notes published through the approved release process.

#### J. Exit criteria
Owner explicitly approves production MVP and release record is complete.

#### K. Risks and unresolved decisions
Known non-blocking issues must be documented. Owner must accept any residual release risk explicitly.

#### L. Deliverables
Versioned Windows release artifact, release notes, installation/user documentation, test/security/recovery evidence, known-issues record and final status update.
