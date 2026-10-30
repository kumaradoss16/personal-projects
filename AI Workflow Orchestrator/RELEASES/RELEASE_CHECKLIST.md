# AI Workflow Orchestrator — Reusable Release Checklist

**Use:** Copy this checklist for each release. Do not mark an item complete without evidence.

## 1. Release metadata

- Release:
- Target branch/commit:
- Release owner:
- Reviewer:
- Build environment:
- Supported Windows baseline:
- Release candidate artifact:
- Date:
- Gate status: `NOT_STARTED` / `IN_PROGRESS` / `BLOCKED` / `READY_FOR_REVIEW` / `APPROVED` / `RELEASED`

## 2. Scope and planning

- [ ] Release scope maps to `SCOPE.md` and approved roadmap.
- [ ] Every included requirement has source section and acceptance evidence.
- [ ] Deferred/excluded capabilities are not accidentally included.
- [ ] PRD §19 remains deferred unless separately approved.
- [ ] MCP and n8n remain excluded from MVP.
- [ ] No parallel/distributed execution or autonomous multi-agent capability is introduced.
- [ ] No Docker runtime, DLQ, SSE, model routing or automatic fallback is introduced without approval.
- [ ] Dependencies and affected documents are identified.
- [ ] Architecture/schema/API/security/scope changes have explicit approval where required.
- [ ] Local Git branch/status/diff are inspected before changes.

## 3. Architecture and implementation

- [ ] Backend remains authoritative for state and authorization.
- [ ] Modular monolith boundaries are preserved.
- [ ] Sequential execution remains the MVP runtime strategy.
- [ ] Provider/tool integrations use approved adapters.
- [ ] No speculative abstraction or unapproved dependency is added.
- [ ] No source paths are assumed without repository inspection.
- [ ] Error handling preserves failures and uncertainty.

## 4. Domain and execution correctness

- [ ] Legal state transitions pass.
- [ ] Illegal state transitions are rejected without durable mutation.
- [ ] Terminal states are not silently rewritten.
- [ ] Attempts and events are persisted consistently.
- [ ] Dependency DAG validation rejects cycles and unknown dependencies.
- [ ] Execution remains sequential.
- [ ] Model output is not treated as proof of success.
- [ ] `SUCCEEDED` and `VERIFIED` remain distinct.
- [ ] Cancellation semantics match the execution specification.
- [ ] Retry eligibility matches policy.
- [ ] Side-effect timeouts are treated as uncertain where outcome cannot be established.
- [ ] No exactly-once external execution claim is made.

## 5. Database and data integrity

- [ ] Fresh database initialization succeeds.
- [ ] Versioned migrations succeed from supported prior version.
- [ ] Foreign-key constraints are enabled and tested.
- [ ] Required per-connection pragmas are verified.
- [ ] Critical state changes use appropriate transactions.
- [ ] No write transaction is held across network/provider/subprocess/approval wait.
- [ ] WAL behavior is verified on Windows.
- [ ] Backup/restore procedure is tested and documented.
- [ ] Artifacts use atomic write behavior where specified.
- [ ] SHA-256 integrity evidence is verified where required.
- [ ] Recovery does not discard completed records or silently rewrite history.

## 6. API and local security

- [ ] API versioning and response/error contracts are stable.
- [ ] Authentication is required on protected endpoints.
- [ ] Loopback binding is enforced.
- [ ] Host and Origin validation is enforced.
- [ ] CORS is not wildcarded.
- [ ] Request body, list size and timeout limits match API specification.
- [ ] Idempotency behavior is tested.
- [ ] Packaged build exposes no unauthenticated debug/docs/diagnostic surface prohibited by SECURITY.md.
- [ ] Backend, not frontend, enforces authorization.
- [ ] Invalid/unavailable policy or credential checks fail closed.

## 7. Secrets and provider security

- [ ] Secrets are stored only through approved OS credential mechanism.
- [ ] Database stores credential references, not plaintext secrets.
- [ ] Secrets do not appear in logs, errors, URLs, frontend storage, prompts or exports.
- [ ] Provider configuration is explicit and validated.
- [ ] Hosted data transmission is disclosed.
- [ ] No automatic provider fallback occurs.
- [ ] Provider timeout and error classification tests pass.
- [ ] Unavailable usage/cost/health values are not fabricated.

## 8. Tool execution and approvals

- [ ] Only approved tool operations are exposed.
- [ ] Filesystem paths remain inside approved project roots.
- [ ] Traversal, symlink/junction escape and ADS cases are rejected.
- [ ] Subprocess executable identifiers and argument arrays are explicit.
- [ ] `shell=False` is used.
- [ ] Environment, working directory, timeout and output limits are controlled.
- [ ] Git capabilities remain within MVP read-oriented boundary.
- [ ] Test commands are explicitly configured/allowlisted.
- [ ] Approval is bound to canonical action fingerprint.
- [ ] Approval is single-use as specified.
- [ ] Changed target/arguments invalidate approval.
- [ ] Action is revalidated immediately before execution.
- [ ] Rejection, expiry, replay and missing authorization fail closed.
- [ ] Model output cannot approve its own action.

## 9. Provider/tool/database/API tests

- [ ] Unit tests pass.
- [ ] Integration tests pass.
- [ ] API contract tests pass.
- [ ] Database/migration/transaction tests pass.
- [ ] State-transition tests pass.
- [ ] Provider success/failure/timeout tests pass where applicable.
- [ ] Tool allowlist and boundary tests pass where applicable.
- [ ] Approval negative-path tests pass where applicable.
- [ ] Retry/idempotency tests pass where applicable.
- [ ] Restart/recovery tests pass where applicable.
- [ ] Regression tests pass.

## 10. UI and desktop

- [ ] UI uses real backend data.
- [ ] UI does not independently mutate authoritative state.
- [ ] Empty/loading/error/denied/unavailable states are present.
- [ ] Polling is bounded and does not imply unsupported live delivery.
- [ ] Keyboard and accessibility behavior is checked.
- [ ] No fake metrics, fabricated history, or decorative nonfunctional controls are presented.
- [ ] Tauri bootstrap and backend lifecycle are tested.
- [ ] Token remains in allowed frontend memory scope only.
- [ ] Windows startup/shutdown tests pass.

## 11. Static analysis and build

- [ ] Approved Ruff checks pass.
- [ ] Approved mypy checks pass.
- [ ] Approved pytest suites pass.
- [ ] Build commands are reproducible from a clean clone.
- [ ] Dependency inventory matches TECH_STACK.md.
- [ ] Lockfiles/pins are consistent with approved stack.
- [ ] No unapproved framework, runtime, service or SDK is present.

No numeric test coverage threshold is implied by this checklist. Apply an owner-approved threshold if one is added to the project specifications.

## 12. Packaging, installation and operations

- [ ] Windows package is generated.
- [ ] Clean installation is tested.
- [ ] Startup and shutdown are tested.
- [ ] Upgrade/migration path is tested.
- [ ] Application data path is documented.
- [ ] Backup and restore are tested.
- [ ] Recovery procedure is documented.
- [ ] Permissions and provider data destinations are documented.
- [ ] Known limitations and known issues are documented.
- [ ] Release notes are prepared.

## 13. Documentation and review

- [ ] Relevant specifications are updated only through approved changes.
- [ ] Roadmap/traceability reflects actual release content.
- [ ] No stale links or filename case mismatches are introduced.
- [ ] Test commands and results are recorded exactly.
- [ ] Unrun tests and limitations are explicit.
- [ ] Final diff is reviewed.
- [ ] Security/data-integrity risks are recorded.
- [ ] Release owner reviews evidence.
- [ ] Explicit approval is recorded.

## 14. Gate status definitions

| Status | Meaning | Minimum evidence |
|---|---|---|
| `NOT_STARTED` | Work has not begun | Release record exists |
| `IN_PROGRESS` | Work or verification is active | Current task list and owner |
| `BLOCKED` | A dependency, conflict, failed test or approval prevents progress | Blocker, impact, owner and resolution path |
| `READY_FOR_REVIEW` | Required work and evidence are complete | Test outputs, review artifacts, diff and risk report |
| `APPROVED` | Authorized reviewer/owner accepts the release gate | Explicit decision linked to evidence |
| `RELEASED` | Release artifact and record are complete | Artifact/version, release notes, final checklist and approval |

A status must not skip directly from `IN_PROGRESS` to `APPROVED` without review evidence. `RELEASED` is not synonymous with “code merged”.
