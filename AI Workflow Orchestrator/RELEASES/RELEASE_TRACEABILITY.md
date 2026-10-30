# AI Workflow Orchestrator — Release Traceability

**Status:** Proposed mapping; source requirements remain authoritative.  
**Classification:** `MVP_REQUIRED`, `FOUNDATION_REQUIRED`, `DEFERRED_POST_MVP`, `EXPLICITLY_EXCLUDED`, `OWNER_DECISION_REQUIRED`.

## 1. Traceability rules

- Preserve source requirement identifiers where they exist.
- `P0-01`–`P0-10` are source identifiers from `SCOPE.md` §8.
- `RW-*` identifiers below are roadmap-generated work-package IDs, not source requirement IDs.
- Where a detailed section does not assign a unique requirement ID, cite the section and do not invent one.
- Acceptance evidence must be observed and attached to the release record; a planned test is not a passing test.

## 2. P0 milestone mapping

| Requirement | Source document | Source section | Target release | Dependency | Acceptance evidence | Status |
|---|---|---|---|---|---|---|
| P0-01 Application and backend foundation | SCOPE.md | §8 P0-01; §3.1 | v0.1 foundation; v0.5 desktop completion | Repository/toolchain readiness | Backend/desktop startup, secure local communication, clean shutdown tests | MVP_REQUIRED |
| P0-02 Persistence foundation | SCOPE.md | §8 P0-02; §3.8 | v0.1 initial persistence; v0.2 recovery semantics; v0.9 backup/restore | P0-01 | Fresh migrations, transactional repository tests, restart persistence, WAL backup/restore evidence | FOUNDATION_REQUIRED |
| P0-03 Workflow and task model | SCOPE.md | §8 P0-03; §3.3 | v0.1 | P0-02 initial persistence | Definition validation, version association, DAG tests, legal/illegal transition tests | MVP_REQUIRED |
| P0-04 Sequential execution engine | SCOPE.md | §8 P0-04; §3.4 | v0.1 core; v0.2 full reliability | P0-02, P0-03 | Dependency-order execution, attempt/event records, retry/timeout/cancel tests | MVP_REQUIRED |
| P0-05 Restart recovery | SCOPE.md | §8 P0-05; §3.4, §3.8 | v0.2; v0.9 integrated proof | P0-02, P0-04 | Crash/restart, orphan detection, uncertain-operation reconciliation tests | MVP_REQUIRED |
| P0-06 Provider adapters | SCOPE.md | §8 P0-06; §3.5 | v0.3 | P0-04, P0-05; hosted provider decision | Ollama and one hosted adapter success/failure/timeout; secret and data-disclosure tests | MVP_REQUIRED |
| P0-07 Tool and approval controls | SCOPE.md | §8 P0-07; §3.6, §3.7 | v0.4 | P0-04, P0-05, security foundations | Path/process boundary tests; action fingerprint/revalidation; backend denial tests | MVP_REQUIRED |
| P0-08 Validation and observability | SCOPE.md | §8 P0-08; §3.9, §3.10 | v0.4 foundation; v0.9 integrated verification | P0-04, P0-06, P0-07 | Deterministic validator evidence, artifact hash, event/log provenance and redaction tests | MVP_REQUIRED |
| P0-09 Desktop workflows | SCOPE.md | §8 P0-09; §3.11 | v0.5 | Stable API plus P0-03–P0-08 | Desktop end-to-end flows using real backend data; accessibility and error-state tests | MVP_REQUIRED |
| P0-10 Release verification | SCOPE.md | §8 P0-10; §3.12, §9 | v0.9 RC; v1.0 final | P0-01–P0-09 | Security report, full tests, Windows packaging/install/startup/shutdown, recovery documentation | MVP_REQUIRED |

## 3. Cross-document requirement mapping

| Roadmap ID | Requirement / capability | Source document | Source section | Target release | Dependency | Acceptance evidence | Classification |
|---|---|---|---|---|---|---|---|
| RW-001 | Local-first single-user Windows desktop product | SCOPE.md | §§2–3 | v0.1–v1.0 | — | Local install and documented local data paths | MVP_REQUIRED |
| RW-002 | Modular monolith and backend authority | ARCHITECTURE.md | architecture/layer boundaries | v0.1 | — | Architecture review; tests prove backend-only state transitions | FOUNDATION_REQUIRED |
| RW-003 | Approved runtime/toolchain and dependency pinning | TECH_STACK.md | technology registry | v0.1 | Owner/toolchain review | Reproducible environment and dependency inventory | FOUNDATION_REQUIRED |
| RW-004 | SQLite schema, migrations, constraints and transactions | DATABASE.md | schema/migrations/transactions | v0.1–v0.2 | RW-003 | Fresh migration, rollback, FK and transaction tests | FOUNDATION_REQUIRED |
| RW-005 | SQLite WAL and Windows backup/restore behavior | DATABASE.md; RULES.md | DB pragmas/recovery; WAL compatibility rule | v0.1 configure as specified; v0.9 prove | RW-004 | Windows backup/restore test evidence | MVP_REQUIRED |
| RW-006 | API authentication, versioning, errors, idempotent run creation | API_SPECIFICATION.md | API/security/run creation sections | v0.1 | RW-002, RW-004 | Contract, auth, duplicate request tests | FOUNDATION_REQUIRED |
| RW-007 | Workflow/task state machines and transition validation | EXECUTION_ENGINE.md | state machine sections | v0.1 | RW-004 | Full transition matrix tests | MVP_REQUIRED |
| RW-008 | Sequential DAG execution | SCOPE.md; EXECUTION_ENGINE.md | §3.3–3.4; DAG/scheduler sections | v0.1 | RW-007 | Three-task dependency-order test | MVP_REQUIRED |
| RW-009 | Retry, timeout, pause/cancel, leases and recovery | EXECUTION_ENGINE.md | retry/cancel/recovery sections | v0.2 | RW-007, RW-008 | Failure and restart suite | MVP_REQUIRED |
| RW-010 | Uncertain external side-effect handling | EXECUTION_ENGINE.md; SECURITY.md | recovery and prohibited guarantees | v0.2 foundation; v0.3–v0.4 integration | RW-009 | Timeout-after-side-effect reconciliation test | MVP_REQUIRED |
| RW-011 | Ollama adapter | SCOPE.md; TECH_STACK.md | §3.5; provider registry | v0.3 | RW-009 | Local inference success/unavailable/timeout tests | MVP_REQUIRED |
| RW-012 | One hosted provider adapter | SCOPE.md; TECH_STACK.md | §3.5; conditional dependency approval | v0.3 | Owner provider/SDK decision | Provider contract, auth, timeout and redaction tests | OWNER_DECISION_REQUIRED |
| RW-013 | Credential references and OS credential storage | SECURITY.md; TECH_STACK.md | secrets policy | v0.3 | RW-012 | No plaintext secret in DB/logs/frontend/exports | MVP_REQUIRED |
| RW-014 | External data transmission disclosure | SCOPE.md; SECURITY.md | §3.5; provider/data trust boundary | v0.3 | RW-011, RW-012 | User-visible destination disclosure and test | MVP_REQUIRED |
| RW-015 | Project-scoped filesystem operations | SCOPE.md; SECURITY.md | §3.6; filesystem boundaries | v0.4 | RW-009 | Traversal/symlink/junction/ADS negative tests | MVP_REQUIRED |
| RW-016 | Read-oriented Git inspection | SCOPE.md; SECURITY.md | §3.6; subprocess controls | v0.4 | RW-009 | Allowed Git commands and path boundary tests | MVP_REQUIRED |
| RW-017 | Configured test runner | SCOPE.md; SECURITY.md | §3.6; process execution | v0.4 | RW-009 | Allowlist, shell=False, env, timeout and output-limit tests | MVP_REQUIRED |
| RW-018 | Action-scoped approval and revalidation | SCOPE.md; EXECUTION_ENGINE.md; SECURITY.md | §3.7; approval lifecycle | v0.4 | RW-015–RW-017 | Changed fingerprint, stale/reused/rejected approval tests | MVP_REQUIRED |
| RW-019 | Deterministic validation and artifact integrity | SCOPE.md; EXECUTION_ENGINE.md | §§3.9–3.10 | v0.4 | RW-015–RW-017 | Validator pass/fail, atomic write and SHA-256 tests | MVP_REQUIRED |
| RW-020 | Desktop UI and backend lifecycle | DESIGN.md; ARCHITECTURE.md; SCOPE.md | UI requirements; §3.11 | v0.5 | Stable API, RW-018–RW-019 | UI/API contract and desktop integration tests | MVP_REQUIRED |
| RW-021 | Polling-based refresh | Owner decision; API_SPECIFICATION.md | live update contract | v0.5–v1.0 | Stable status endpoints | Bounded polling and stale/error state tests | FOUNDATION_REQUIRED |
| RW-022 | SSE | SCOPE.md §5; API_SPECIFICATION.md | deferred update route | Post-v1.0 (v1.1 candidate) | Evidence polling insufficient | Owner-approved scope change and performance/use-case evidence | DEFERRED_POST_MVP |
| RW-023 | PRD §19 Unified Project Workspace / Activity Consolidation | PRD.md; SCOPE.md | PRD §19 vs SCOPE §3/§8 | Deferred | Formal scope change and cross-document redesign | Explicit owner scope decision | DEFERRED_POST_MVP |
| RW-024 | Ruff + mypy | TECH_STACK.md; owner decision | quality tooling | v0.1 onward | — | CI/static-analysis output | FOUNDATION_REQUIRED |
| RW-025 | MCP integration | SCOPE.md; RULES.md | §4.3; future integration guidance | Post-v1.0 candidate only | Scope/security review | Approved scope change and threat-model review | DEFERRED_POST_MVP |
| RW-026 | n8n integration | SCOPE.md; RULES.md | §4.3; future integration guidance | Post-v1.0 candidate only | Scope/security review | Approved scope change and threat-model review | EXPLICITLY_EXCLUDED |
| RW-027 | Parallel/distributed execution | SCOPE.md | §§4.1, 5 | Post-v1.0 candidate; not committed | Sequential correctness/recovery proven; redesign approval | Concurrency invariants, resource limits, recovery and security review | EXPLICITLY_EXCLUDED |
| RW-028 | Autonomous multi-agent execution | SCOPE.md | §4.1 | Post-v1.0 candidate; not committed | Explicit use case and bounded autonomy design | Owner-approved scope and security review | EXPLICITLY_EXCLUDED |
| RW-029 | Docker runtime / distributed sandbox | SCOPE.md; TECH_STACK.md | exclusions | Not MVP | Explicit architecture and runtime decision | Owner-approved architecture change | EXPLICITLY_EXCLUDED |
| RW-030 | DLQ | SCOPE.md; EXECUTION_ENGINE.md | retry/recovery design | Not MVP | Demonstrated need; architecture decision | Owner-approved design; no substitution for current persisted states | EXPLICITLY_EXCLUDED |
| RW-031 | Automatic provider fallback / intelligent routing | SCOPE.md | §4.2 | Not MVP | Multiple providers and explicit semantics | Owner-approved scope/security/data policy | EXPLICITLY_EXCLUDED |
| RW-032 | Workflow templates/import-export/advanced graph | SCOPE.md | §5 | Post-v1.0 candidates | Stable workflow schema/versioning | Separate product decision and compatibility contract | DEFERRED_POST_MVP |

## 4. Requirement coverage summary

- P0 milestones: 10/10 mapped to a target release.
- Release ladder: all seven releases represented.
- Domain authorities: scope, product, architecture, stack, design, database, API, execution, security, agent workflow and rules included.
- Owner decision remains open: hosted provider and SDK.
- Prior owner decisions applied: Ruff/mypy approved; PRD §19 deferred; polling through v1.0; WAL Windows test gate; AGENT_WORKFLOW governs agent procedure.

This is planning coverage, not implementation completion. Every acceptance evidence cell remains pending until tests/reviews are actually run.
