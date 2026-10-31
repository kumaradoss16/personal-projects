AI Workflow Orchestrator - Repository Assessment and Release Roadmap
BLOCKED - DOCUMENTATION OR SCOPE CONFLICTS
Read-only assessment

Assessment date: 31 October 2026
Primary implementation repository: kumaradoss16/AI-Workflow-Orchestrator
Related documentation repository: personal-projects / AI Workflow Orchestrator
1. Executive summary
The project is further along than the documentation-only baseline suggests.

The dedicated AI-Workflow-Orchestrator repository already contains:
- A Python 3.12 backend package.
- FastAPI API implementation.
- SQLite persistence and two migration files.
- Workflow and task domain models.
- DAG validation.
- A sequential execution engine.
- Retry, timeout, lease, heartbeat, and recovery modules.
- Unit and integration tests.
- A mock provider.
- Ruff and mypy configuration.
- Release planning and traceability documents.
However, the inspected repository does not yet contain the Tauri desktop application or TypeScript frontend. Nor does the source tree demonstrate the complete MVP described in SCOPE.md: real provider adapters, controlled developer tools, the complete approval system, and production Windows packaging remain outstanding or unverified.

My recommendation is to retain the existing seven-release sequence (v0.1 through v1.0) and organize it under the five phases you specified.

That is a better delivery strategy than replacing the existing release plan with a new five-release sequence. The existing seven releases provide more granular verification boundaries, particularly for provider integration, security-sensitive tools, desktop integration, and release hardening.

Important distinction: repository files demonstrate that implementation code exists. They do not, by themselves, prove that the code executes correctly or that its tests currently pass.

2. Repository assessment
2.1 Repository identity and source-of-truth findings
The prompt names two repositories. They are not identical in their current contents.
Area	Dedicated repository	personal-projects folder
Product documentation	Present	Present
AGENTS.md	Present	Present
AGENT_WORKFLOW.md	Present	Present
Backend source code	Present	Not present in project folder
Tests	Present	Not present in project folder
SQL migrations	Present	Not present in project folder
Python manifest	Present	Not present in project folder
Release documentation	Present	Present
Database specification	Different Git blob	Different Git blob
Implementation status	Different release status document	Different release status document

Most of the project specifications have matching Git blob identifiers across the two locations. However, the DATABASE.md, IMPLEMENTATION_STATUS.md, and RELEASE_ROADMAP.md files differ.

Recommended authority arrangement:
- Use AI-Workflow-Orchestrator as the primary implementation repository.
- Treat the personal-projects project folder as a related documentation snapshot, not as the implementation source of truth.
- Reconcile the differing specifications before using either repository to authorize schema or implementation changes.
- Record the canonical repository and documentation branch in the project governance documents.
This is a governance issue, not a reason to discard either repository.
2.2 Actual dedicated repository structure
The following is based on the inspected Git tree, not an assumed architecture.
AI-Workflow-Orchestrator/
│
├── .gitignore
├── 01.summary.md
├── AGENTS.md
├── AGENT_WORKFLOW.md
├── API_SPECIFICATION.md
├── ARCHITECTURE.md
├── DATABASE.md
├── DESIGN.md
├── DOCUMENTATION_AUDIT.md
├── EXECUTION_ENGINE.md
├── PRD.md
├── README.md
├── RULES.md
├── SCOPE.md
├── SECURITY.md
├── TECH_STACK.md
├── technical-architecture.md
│
├── RELEASES/
│   ├── IMPLEMENTATION_STATUS.md
│   ├── RELEASE_CHECKLIST.md
│   ├── RELEASE_RISK_REGISTER.md
│   ├── RELEASE_ROADMAP.md
│   ├── RELEASE_SPECIFICATIONS.md
│   ├── RELEASE_TRACEABILITY.md
│   ├── v0.2_RELEASE_NOTES.md
│   └── Readmd.md
│
├── examples/
│   └── three_task_workflow.json
│
├── migrations/
│   ├── 001_initial_schema.sql
│   └── 002_v02_durable_execution.sql
│
├── src/
│   └── orchestrator/
│       ├── api/
│       ├── db/
│       ├── domain/
│       └── engine/
│
├── tests/
│   ├── integration/
│   └── unit/
│
└── pyproject.toml


2.3 Existing implementation classification
The following classification is based on source files, migration files, and test files present in the remote repository. It is not a runtime verification.
Component	Repository evidence	Assessment
Python backend package	src/orchestrator/	Implemented structure
FastAPI application	api/app.py	Implemented structure
Bearer-token verification	api/security.py	Partial evidence
SQLite connection management	db/connection.py	Implemented structure
Migration runner	db/migrator.py	Implemented structure
Repository layer	db/repositories.py	Implemented structure
Domain models	domain/models.py	Implemented structure
State machine	domain/state_machine.py	Implemented structure
DAG validation	domain/dag.py	Implemented structure
Retry policy	domain/retry.py	Implemented structure
Sequential scheduler	engine/scheduler.py	Implemented structure
Lease management	engine/lease.py	Implemented structure
Startup recovery	engine/recovery.py	Implemented structure
Execution consistency	engine/consistency.py	Implemented structure
Structured logging	engine/logging.py	Implemented structure
Mock provider	engine/provider.py	Implemented structure
Ollama adapter	Not established by inspected source tree	Not demonstrated
Hosted provider adapter	Not established by inspected source tree	Not demonstrated
Controlled filesystem tools	Not established	Not demonstrated
Git tool adapter	Not established	Not demonstrated
Test-runner adapter	Not established	Not demonstrated
Complete approval subsystem	Not established	Not demonstrated
Tauri desktop shell	No src-tauri/	Not present
TypeScript frontend	No frontend source tree	Not present
Windows packaging	No packaging configuration demonstrated	Not demonstrated

The correct conclusion is that the repository has a meaningful backend foundation, not that it has a complete MVP.
2.4 Existing tests
The remote tree contains:

Unit tests
- test_dag.py
- test_log_redaction.py
- test_retry_policy.py
- test_state_machine.py
Integration tests
- test_api.py
- test_crash_recovery.py
- test_db_migrations.py
- test_sequential_execution.py
- test_wal_durability.py
The project status document reports 19 unit tests and 20 integration tests passing, as well as clean Ruff and mypy results.

Those are recorded claims in the repository's tracking document. I have not independently executed the tests against a local checkout, so I am not certifying those results as current.

3. Existing architecture summary
The architecture documented in the repository is appropriate for the intended MVP. I recommend preserving it.
Layer	Approved direction	Responsibility
Desktop	Tauri 2 + Rust	Desktop lifecycle and native integration
Frontend	TypeScript, HTML, CSS	Presentation and user interaction
Backend	Python 3.12 + FastAPI + Pydantic	Authoritative application and execution logic
Persistence	SQLite + standard-library sqlite3	Durable workflow and execution state
Artifacts	Local filesystem	Controlled project and execution artifacts
Local inference	Ollama HTTP API	Local model execution
Hosted inference	One approved provider adapter	Hosted model execution
Runtime	Local-first modular monolith	Single local backend process
Scheduling	Bounded in-process scheduler	Sequential dependency-aware execution

Core architectural rule
#chatgpt-mermaid-_r_1c2_{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,Segoe UI,Helvetica,Apple Color Emoji,Arial,sans-serif,Segoe UI Emoji,Segoe UI Symbol;font-size:16px;fill:rgb(237, 237, 237);}@keyframes edge-animation-frame{from{stroke-dashoffset:0;}}@keyframes dash{to{stroke-dashoffset:0;}}#chatgpt-mermaid-_r_1c2_ .edge-animation-slow{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 50s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-_r_1c2_ .edge-animation-fast{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 20s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-_r_1c2_ .error-icon{fill:rgb(48, 48, 48);}#chatgpt-mermaid-_r_1c2_ .error-text{fill:rgb(237, 237, 237);stroke:rgb(237, 237, 237);}#chatgpt-mermaid-_r_1c2_ .edge-thickness-normal{stroke-width:1px;}#chatgpt-mermaid-_r_1c2_ .edge-thickness-thick{stroke-width:3.5px;}#chatgpt-mermaid-_r_1c2_ .edge-pattern-solid{stroke-dasharray:0;}#chatgpt-mermaid-_r_1c2_ .edge-thickness-invisible{stroke-width:0;fill:none;}#chatgpt-mermaid-_r_1c2_ .edge-pattern-dashed{stroke-dasharray:3;}#chatgpt-mermaid-_r_1c2_ .edge-pattern-dotted{stroke-dasharray:2;}#chatgpt-mermaid-_r_1c2_ .marker{fill:rgb(175, 175, 175);stroke:rgb(175, 175, 175);}#chatgpt-mermaid-_r_1c2_ .marker.cross{stroke:rgb(175, 175, 175);}#chatgpt-mermaid-_r_1c2_ svg{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,Segoe UI,Helvetica,Apple Color Emoji,Arial,sans-serif,Segoe UI Emoji,Segoe UI Symbol;font-size:16px;}#chatgpt-mermaid-_r_1c2_ p{margin:0;}#chatgpt-mermaid-_r_1c2_ .label{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,Segoe UI,Helvetica,Apple Color Emoji,Arial,sans-serif,Segoe UI Emoji,Segoe UI Symbol;color:rgb(237, 237, 237);}#chatgpt-mermaid-_r_1c2_ .cluster-label text{fill:rgb(237, 237, 237);}#chatgpt-mermaid-_r_1c2_ .cluster-label span{color:rgb(237, 237, 237);}#chatgpt-mermaid-_r_1c2_ .cluster-label span p{background-color:transparent;}#chatgpt-mermaid-_r_1c2_ .label text,#chatgpt-mermaid-_r_1c2_ span{fill:rgb(237, 237, 237);color:rgb(237, 237, 237);}#chatgpt-mermaid-_r_1c2_ .node rect,#chatgpt-mermaid-_r_1c2_ .node circle,#chatgpt-mermaid-_r_1c2_ .node ellipse,#chatgpt-mermaid-_r_1c2_ .node polygon,#chatgpt-mermaid-_r_1c2_ .node path{fill:rgb(9, 23, 44);stroke:rgb(31, 78, 148);stroke-width:1px;}#chatgpt-mermaid-_r_1c2_ .rough-node .label text,#chatgpt-mermaid-_r_1c2_ .node .label text,#chatgpt-mermaid-_r_1c2_ .image-shape .label,#chatgpt-mermaid-_r_1c2_ .icon-shape .label{text-anchor:middle;}#chatgpt-mermaid-_r_1c2_ .node .katex path{fill:#000;stroke:#000;stroke-width:1px;}#chatgpt-mermaid-_r_1c2_ .rough-node .label,#chatgpt-mermaid-_r_1c2_ .node .label,#chatgpt-mermaid-_r_1c2_ .image-shape .label,#chatgpt-mermaid-_r_1c2_ .icon-shape .label{text-align:center;}#chatgpt-mermaid-_r_1c2_ .node.clickable{cursor:pointer;}#chatgpt-mermaid-_r_1c2_ .root .anchor path{fill:rgb(175, 175, 175)!important;stroke-width:0;stroke:rgb(175, 175, 175);}#chatgpt-mermaid-_r_1c2_ .arrowheadPath{fill:rgb(175, 175, 175);}#chatgpt-mermaid-_r_1c2_ .edgePath .path{stroke:rgb(175, 175, 175);stroke-width:1px;}#chatgpt-mermaid-_r_1c2_ .flowchart-link{stroke:rgb(175, 175, 175);fill:none;}#chatgpt-mermaid-_r_1c2_ .edgeLabel{background-color:rgb(0, 0, 0);text-align:center;}#chatgpt-mermaid-_r_1c2_ .edgeLabel p{background-color:rgb(0, 0, 0);}#chatgpt-mermaid-_r_1c2_ .edgeLabel rect{opacity:0.5;background-color:rgb(0, 0, 0);fill:rgb(0, 0, 0);}#chatgpt-mermaid-_r_1c2_ .labelBkg{background-color:rgba(0, 0, 0, 0.5);}#chatgpt-mermaid-_r_1c2_ .cluster rect{fill:rgb(48, 48, 48);stroke:rgba(255, 255, 255, 0.15);stroke-width:1px;}#chatgpt-mermaid-_r_1c2_ .cluster text{fill:rgb(237, 237, 237);}#chatgpt-mermaid-_r_1c2_ .cluster span{color:rgb(237, 237, 237);}#chatgpt-mermaid-_r_1c2_ div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,Segoe UI,Helvetica,Apple Color Emoji,Arial,sans-serif,Segoe UI Emoji,Segoe UI Symbol;font-size:12px;background:rgb(48, 48, 48);border:1px solid rgba(255, 255, 255, 0.15);border-radius:2px;pointer-events:none;z-index:100;}#chatgpt-mermaid-_r_1c2_ .flowchartTitleText{text-anchor:middle;font-size:18px;fill:rgb(237, 237, 237);}#chatgpt-mermaid-_r_1c2_ rect.text{fill:none;stroke-width:0;}#chatgpt-mermaid-_r_1c2_ .icon-shape,#chatgpt-mermaid-_r_1c2_ .image-shape{background-color:rgb(0, 0, 0);text-align:center;}#chatgpt-mermaid-_r_1c2_ .icon-shape p,#chatgpt-mermaid-_r_1c2_ .image-shape p{background-color:rgb(0, 0, 0);padding:2px;}#chatgpt-mermaid-_r_1c2_ .icon-shape .label rect,#chatgpt-mermaid-_r_1c2_ .image-shape .label rect{opacity:0.5;background-color:rgb(0, 0, 0);fill:rgb(0, 0, 0);}#chatgpt-mermaid-_r_1c2_ .label-icon{display:inline-block;height:1em;overflow:visible;vertical-align:-0.125em;}#chatgpt-mermaid-_r_1c2_ .node .label-icon path{fill:currentColor;stroke:revert;stroke-width:revert;}#chatgpt-mermaid-_r_1c2_ .node .neo-node{stroke:rgb(31, 78, 148);}#chatgpt-mermaid-_r_1c2_ [data-look="neo"].node rect,#chatgpt-mermaid-_r_1c2_ [data-look="neo"].cluster rect,#chatgpt-mermaid-_r_1c2_ [data-look="neo"].node polygon{stroke:url(#chatgpt-mermaid-_r_1c2_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-_r_1c2_ [data-look="neo"].swimlane.cluster rect{filter:none;}#chatgpt-mermaid-_r_1c2_ [data-look="neo"].node path{stroke:url(#chatgpt-mermaid-_r_1c2_-gradient);stroke-width:1px;}#chatgpt-mermaid-_r_1c2_ [data-look="neo"].node .outer-path{filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-_r_1c2_ [data-look="neo"].node .neo-line path{stroke:rgb(31, 78, 148);filter:none;}#chatgpt-mermaid-_r_1c2_ [data-look="neo"].node circle{stroke:url(#chatgpt-mermaid-_r_1c2_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-_r_1c2_ [data-look="neo"].node circle .state-start{fill:#000000;}#chatgpt-mermaid-_r_1c2_ [data-look="neo"].icon-shape .icon{fill:url(#chatgpt-mermaid-_r_1c2_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-_r_1c2_ [data-look="neo"].icon-shape .icon-neo path{stroke:url(#chatgpt-mermaid-_r_1c2_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-_r_1c2_ .node text{font-size:14px;font-weight:600;letter-spacing:normal;fill:rgb(153, 206, 255);}#chatgpt-mermaid-_r_1c2_ .edgeLabels text{font-size:13px;font-weight:600;letter-spacing:-0.08px;fill:rgb(153, 206, 255);}#chatgpt-mermaid-_r_1c2_ .node tspan[font-weight="normal"],#chatgpt-mermaid-_r_1c2_ .edgeLabels tspan[font-weight="normal"]{font-weight:600;}#chatgpt-mermaid-_r_1c2_ .edgeLabel .label rect{opacity:1;rx:13px;ry:13px;fill:rgb(0, 14, 26);stroke:rgb(26, 62, 95);stroke-width:1px;}#chatgpt-mermaid-_r_1c2_ .node rect,#chatgpt-mermaid-_r_1c2_ .node circle,#chatgpt-mermaid-_r_1c2_ .node ellipse,#chatgpt-mermaid-_r_1c2_ .node polygon,#chatgpt-mermaid-_r_1c2_ .node path{fill:rgb(0, 40, 77);stroke:rgba(255, 255, 255, 0.1);stroke-width:1px;}#chatgpt-mermaid-_r_1c2_ .node rect{rx:16px;ry:16px;}#chatgpt-mermaid-_r_1c2_ .mermaid-decision .label-container{fill:rgb(0, 14, 26);stroke:rgb(26, 62, 95);stroke-dasharray:2px,2px;}#chatgpt-mermaid-_r_1c2_ .edgePaths .flowchart-link{stroke:rgb(175, 175, 175);stroke-width:1px;stroke-linecap:round;stroke-linejoin:round;}#chatgpt-mermaid-_r_1c2_ .marker{fill:rgb(175, 175, 175);stroke:rgb(175, 175, 175);}#chatgpt-mermaid-_r_1c2_ :root{--mermaid-font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,Segoe UI,Helvetica,Apple Color Emoji,Arial,sans-serif,Segoe UI Emoji,Segoe UI Symbol;}Tauri + TypeScript UIRust Native Lifecycle LayerProtected FastAPI BackendDomain + Application LogicWorkflow Execution EngineSQLiteProject Files + ArtifactsProvider AdaptersControlled Tool AdaptersAuthenticated API requests





The backend must remain authoritative for:
- Workflow and task states.
- Scheduling and dependency resolution.
- Retry and timeout decisions.
- Authorization and approvals.
- Provider and tool execution.
- Persistence and recovery.
The frontend must never decide that a task succeeded simply because it received a model response.

4. Documentation conflicts and missing decisions
This is the main reason I would not authorize implementation immediately.
4.1 Material findings
ID	Finding	Impact	Required action
DOC-01	Two repositories contain different project snapshots	Risk of implementing against the wrong specification	Confirm canonical repository
DOC-02	AGENT_WORKFLOW.md says AGENTS.md was not present, although it exists	Conflicting contributor instructions	Reconcile the instructions
DOC-03	AGENT_WORKFLOW.md refers to lowercase prd.md and design.md, while actual files are uppercase	Broken case-sensitive links and ambiguity	Correct references after approval
DOC-04	technical-architecture.md overlaps with ARCHITECTURE.md and security design	Competing architecture interpretations	Keep ARCHITECTURE.md canonical; reconcile legacy document
DOC-05	RULES.md includes MCP/n8n implementation guidance while SCOPE.md excludes these from MVP	Agents could misinterpret future guidance as current authorization	Label those sections explicitly future-only
DOC-06	PRD §19 contains requirements not reflected in MVP scope	Scope expansion risk	Keep deferred until owner decision
DOC-07	Implementation status is inconsistent with the actual dedicated repository tree	Incorrect project progress reporting	Rebaseline status using source and test evidence
DOC-08	DATABASE.md differs between the two repositories	Schema and migration ambiguity	Select and reconcile canonical DB contract
DOC-09	Hosted provider remains undecided	Blocks hosted-provider release work	Owner selects provider and SDK
DOC-10	Several documents remain marked Proposed	Unclear approval state	Record explicit owner acceptance

4.2 Important inconsistency: AGENTS.md versus AGENT_WORKFLOW.md
AGENTS.md correctly identifies AGENT_WORKFLOW.md as the mandatory operating procedure and describes the domain authority hierarchy.

However, AGENT_WORKFLOW.md states that AGENTS.md was not present when it was written and instructs contributors not to assume it exists.

That statement is now stale.

The two documents should be reconciled before an AI coding agent is asked to implement anything.
4.3 Existing release status is not trustworthy enough
The dedicated repository's IMPLEMENTATION_STATUS.md contains contradictory status information:
- It says no application implementation is asserted.
- It reports v0.1 and v0.2 as completed.
- It records test and static-analysis results.
- It says implementation has not started.
- It describes the current release as v0.1 while also describing v0.2 as complete.
The source tree confirms that backend code, tests, and migrations exist. Therefore, the status document needs a new evidence-based baseline.

Do not use that document's release status as the sole indicator of implementation progress.

5. Recommended release roadmap
Recommendation: retain the existing seven-release sequence and group it into the requested five phases.

The five-phase model is useful for product-level communication. The seven-release model is more useful for engineering execution and verification.
5.1 Executive release summary
Phase	Version	Release name	Main objective	Release outcome
1	v0.1	Core Foundation	Backend, persistence, domain model, mock execution	Verifiable working execution foundation
1	v0.2	Durable Execution	Retry, timeout, pause, cancellation, recovery	Reliable backend execution lifecycle
2	v0.3	Provider Integration	Ollama and one hosted provider	Real AI inference
2	v0.4	Controlled Tools and Approvals	Tools, authorization, approvals, validation	Safe execution of approved developer actions
3	v0.5	Desktop Integration	Tauri shell and functional frontend	Usable desktop MVP interface
4	v0.9	Release Candidate	Security, packaging, backup, recovery, regression	Production-candidate evidence
4	v1.0	Production MVP	Final acceptance and release	First production MVP
5	v1.1+	Capability Expansion	Templates, interoperability, diagnostics	Controlled post-MVP evolution
5	v1.2–v2.x	Advanced Orchestration	Advanced execution and AI capabilities	Separately approved platform expansion

This is consistent with the existing RELEASES/RELEASE_ROADMAP.md rather than a replacement for it.
5.2 Phase 1 - Foundation and walking skeleton
Existing backend foundation identified
Releases: v0.1 and v0.2

Objective: establish a real, durable workflow execution engine before adding external AI providers, tools, and desktop complexity.
v0.1 - Core foundation
Workstream	Required deliverables
Backend	FastAPI application, configuration, health/readiness, request validation
Database	SQLite connection management, migration runner, initial persistence
Domain	Project, workflow, run, task, attempt, event models
Workflow	Structured workflow definition and DAG validation
Execution	Sequential scheduler and legal state transitions
Provider	Deterministic mock provider
API	Protected local API and durable run creation
CLI	Minimal interface for initial execution
Testing	Domain, migration, API, and sequential execution tests

Acceptance criteria
1. Create a local project.
2. Create a structured workflow with three dependent tasks.
3. Validate the workflow and reject cycles.
4. Execute tasks in dependency order.
5. Persist run, task, attempt, and event records.
6. Reject illegal state transitions.
7. Restart the backend and retrieve persisted execution records.
Do not add: Ollama, hosted provider calls, real tool execution, approvals, Tauri UI, parallel execution, or MCP.
v0.2 - Durable execution and recovery
The existing source tree already contains substantial implementation for this release.
Workstream	Required deliverables
Retry	Bounded retries and persisted retry scheduling
Timeout	Task execution timeout enforcement
Lifecycle	Pause, resume, and cancellation semantics
Lease	Task lease ownership, heartbeat, expiry
Recovery	Startup recovery and orphaned task detection
Reconciliation	Uncertain external outcome classification
State	NEEDS_ATTENTION and consistency checks
Observability	Structured and redacted execution logs

Acceptance criteria
- Retry scheduling survives process restart.
- Completed tasks are not needlessly repeated.
- Expired leases are detected.
- Crash recovery is deterministic and idempotent.
- Uncertain external side effects are not blindly retried.
- Cancellation and pause preserve valid persisted state.
- Critical recovery paths have integration tests.
Architecture review point: the existing implementation uses lease and heartbeat concepts. The execution specification and release roadmap must be reconciled on their exact MVP role. A lease mechanism must not accidentally imply that distributed workers or concurrent execution are approved.
5.3 Phase 2 - Real inference and controlled execution
Releases: v0.3 and v0.4

Objective: introduce real AI inference and controlled developer-tool capabilities without allowing model output to bypass authorization or execution policy.
v0.3 - Provider integration
Workstream	Required deliverables
Provider contract	Provider-neutral interface
Local inference	Ollama adapter
Hosted inference	Exactly one owner-approved hosted provider
Configuration	Provider and model configuration
Credentials	OS credential-store references
Validation	Request, response, and error normalization
Reliability	Timeouts, availability checks, provider failure handling
Privacy	Explicit local-versus-hosted transmission policy

Acceptance criteria
- Local and hosted provider paths work when configured.
- Provider errors are normalized.
- Timeouts are bounded.
- Secrets are never stored in plaintext database fields or logs.
- Hosted transmission is explicit.
- No implicit fallback from local to hosted inference occurs.
Blocker: hosted provider selection and exact SDK/version approval.
v0.4 - Controlled tools, approvals, and verification
Workstream	Required deliverables
Filesystem	Project-scoped read/write operations
Git	Read-oriented status, diff, and history
Test execution	Configured allowlisted test commands
Subprocess	shell=False, restricted environment, timeout and output limits
Approval	Action-specific approval requests and decisions
Integrity	Immutable action fingerprint and revalidation
Validation	Deterministic schema, artifact, and test validation
Audit	Tool invocation and approval event records

Acceptance criteria
- Unauthorized operations are rejected by the backend.
- Approvals are bound to the exact action, target, arguments, and context.
- Changed actions invalidate earlier approvals.
- Tool output is treated as untrusted.
- Filesystem operations cannot escape approved project boundaries.
- Arbitrary shell execution is unavailable.
- Validation evidence is distinct from model-reported success.
Important: this release is security-sensitive. Do not expose powerful tools first and implement approval controls afterward.
5.4 Phase 3 - Complete MVP desktop application
Release: v0.5

Objective: connect the stable backend to a usable Tauri desktop application.
Workstream	Required deliverables
Desktop shell	Tauri 2 application
Native layer	Rust-managed backend lifecycle
Frontend	TypeScript application shell
API client	Typed client against versioned backend contracts
Projects	Project list, creation, selection
Workflows	Workflow creation and configuration
Execution	Run details, task states, execution history
Approvals	Pending approval interface
Providers	Provider configuration
Observability	Logs, artifacts, validation, and errors
Refresh	Bounded polling through v1.0

Acceptance criteria
1. The desktop application starts and stops the backend safely.
2. The frontend communicates through the protected local API.
3. Project and workflow operations use persisted backend data.
4. Users can start and monitor workflows.
5. Users can review and resolve approval requests.
6. Real errors, loading states, denied actions, and unavailable services are visible.
7. The frontend does not simulate execution progress or authoritative state.
Explicit exclusion: do not add SSE merely because it appears in another API document. The existing roadmap records polling as the approved v1.0 approach.
5.5 Phase 4 - Release candidate and production MVP
Releases: v0.9 and v1.0

Objective: establish release evidence, not add new features.
v0.9 - Release candidate
- Full end-to-end regression.
- API and database verification.
- Recovery and crash testing.
- Provider failure testing.
- Approval and authorization testing.
- Filesystem and subprocess security testing.
- SQLite WAL backup/restore testing on Windows.
- Clean-clone build and test instructions.
- Windows packaging and clean-install testing.
- Upgrade and migration testing.
- Installation, configuration, permission, backup, and recovery documentation.
v1.0 - Production MVP
- Resolve release-blocking defects.
- Freeze MVP scope and dependencies.
- Verify all SCOPE.md §9 acceptance criteria.
- Confirm all mandatory security invariants.
- Verify backup and restore.
- Verify Windows installation and startup.
- Publish release notes and known limitations.
- Produce the approved Windows release artifact.
No new feature work belongs in v1.0.
5.6 Phase 5 - Post-MVP expansion
Candidate releases: v1.1 and v1.2–v2.x

This phase is not a commitment to implement every advanced capability.
Capability	Candidate release	Prerequisite
Workflow templates	v1.1	Stable workflow schema
Import/export	v1.1	Versioned definitions and import validation
Additional providers	v1.1	Provider adapter contract and provider approval
Better recovery diagnostics	v1.1	Stable recovery event model
Improved artifact management	v1.1	Artifact integrity and lifecycle policy
SSE updates	v1.1+	Measured need and API/security approval
Bounded parallelism	v1.2+	Concurrency, locking, recovery, and resource model
Conditional branching	v1.2+	Approved workflow semantics and state model
Provider routing/fallback	v1.2+	Policy, data-transmission, cost, and failure model
MCP	v1.2+	Tool trust, authorization, transport, and isolation design
Multi-agent coordination	v2.x	Explicit agent lifecycle and resource boundaries
Knowledge retrieval	v2.x	Ingestion, indexing, provenance, privacy design
Cloud sync/collaboration	Separate major-version proposal	New architecture and threat model


6. Feature traceability matrix
The matrix below assigns each major capability to one primary release. It is a roadmap-level mapping; the existing RELEASES/RELEASE_TRACEABILITY.md should remain the detailed requirement-to-test mapping authority.

Priority definitions: P0 = release-critical; P1 = important; P2 = deferrable; FUTURE = deferred.
Feature ID	Feature	Primary release	Priority	Dependencies	Acceptance evidence
BACK-001	FastAPI backend bootstrap	v0.1	P0	None	Backend starts and health checks respond
BACK-002	Protected local API	v0.1	P0	Backend bootstrap	Invalid session rejected
DB-001	SQLite initialization	v0.1	P0	Backend bootstrap	Fresh DB initializes
DB-002	Migration runner	v0.1	P0	DB-001	Migrations apply once and validate
DB-003	Execution persistence	v0.1	P0	DB-002	Run/task/event data survives restart
WF-001	Workflow domain model	v0.1	P0	DB-003	Valid definitions persist
WF-002	DAG validation	v0.1	P0	WF-001	Cycles and invalid dependencies rejected
EXEC-001	Sequential scheduler	v0.1	P0	WF-002	Dependency order enforced
EXEC-002	Legal state transitions	v0.1	P0	WF-001	Illegal transitions rejected
PROV-001	Mock provider	v0.1	P0	EXEC-001	Deterministic test execution
TEST-001	Backend unit/integration baseline	v0.1	P0	Backend/domain	Required tests execute
EXEC-003	Persisted retry policy	v0.2	P0	EXEC-001	Retry survives restart
EXEC-004	Timeout enforcement	v0.2	P0	EXEC-001	Timeout classified and persisted
EXEC-005	Pause/resume/cancel	v0.2	P0	State machine	Legal transitions verified
EXEC-006	Lease and heartbeat lifecycle	v0.2	P0	DB-003	Expired lease handled safely
EXEC-007	Startup recovery	v0.2	P0	EXEC-003–006	Recovery is idempotent
OBS-001	Structured execution logging	v0.2	P0	EXEC-001	Redaction and event tests pass
PROV-002	Provider adapter contract	v0.3	P0	EXEC-001	Adapter contract tests pass
PROV-003	Ollama adapter	v0.3	P0	PROV-002	Local inference success/failure tests
PROV-004	One hosted provider adapter	v0.3	P0	Owner approval	Hosted request/error/timeout tests
SEC-001	Credential-store integration	v0.3	P0	Provider config	Secret reference only
SEC-002	Provider transmission policy	v0.3	P0	Provider contract	Explicit transmission behavior
TOOL-001	Project-scoped filesystem tools	v0.4	P0	SEC-001, project boundaries	Escape attempts rejected
TOOL-002	Git inspection adapter	v0.4	P0	Project boundaries	Scoped read-only operations
TOOL-003	Allowlisted test runner	v0.4	P0	Subprocess policy	Unsafe command denied
APPROVAL-001	Approval request lifecycle	v0.4	P0	State machine	Approve/reject persisted
APPROVAL-002	Immutable action fingerprint	v0.4	P0	APPROVAL-001	Modified action rejected
SEC-003	Backend authorization enforcement	v0.4	P0	API/session	Negative permission tests pass
EXEC-008	Deterministic output verification	v0.4	P0	Provider/tool contracts	Invalid outputs cannot succeed
OBS-002	Artifact integrity evidence	v0.4	P0	Filesystem adapter	Hash and metadata verified
DESK-001	Tauri desktop shell	v0.5	P0	Stable backend/API	Windows shell starts/stops
DESK-002	Typed frontend API client	v0.5	P0	API contracts	Contract tests pass
DESK-003	Project/workflow/run screens	v0.5	P0	DESK-001, API	Core flows use backend data
DESK-004	Approval and provider screens	v0.5	P0	v0.3, v0.4	Real actions and state displayed
TEST-002	Full system E2E suite	v0.9	P0	v0.5	Complete workflow scenario passes
SEC-004	Security verification	v0.9	P0	v0.3–v0.5	Security test matrix passes
DB-004	Windows WAL backup/restore	v0.9	P0	DB-003	Restored DB integrity verified
REL-001	Windows packaging/install	v0.9	P0	DESK-001	Clean install validated
REL-002	v1.0 acceptance and release	v1.0	P0	v0.9	All MVP acceptance evidence approved
WF-003	Workflow templates	v1.1	P2	v1.0	Template validation passes
WF-004	Workflow import/export	v1.1	P2	Versioned schema	Unsafe imported permissions rejected
PROV-005	Additional providers	v1.1	P2	Provider adapter	Adapter conformance tests
EXEC-009	Bounded parallel execution	v1.2+	FUTURE	Concurrency architecture approval	Resource and recovery tests
EXEC-010	Branching and advanced scheduling	v1.2+	FUTURE	Workflow semantic design	State transition tests
PROV-006	Routing and fallback	v1.2+	FUTURE	Provider policy design	Policy and privacy tests
TOOL-004	MCP integration	v1.2+	FUTURE	Tool security architecture	Capability and isolation tests
AGENT-001	Multi-agent coordination	v2.x	FUTURE	Agent lifecycle architecture	Isolation and recovery tests
KNOW-001	Knowledge retrieval	v2.x	FUTURE	Ingestion and provenance design	Retrieval traceability tests


7. Technical dependency roadmap
The release sequence should follow this dependency chain:
#chatgpt-mermaid-_r_1c9_{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,Segoe UI,Helvetica,Apple Color Emoji,Arial,sans-serif,Segoe UI Emoji,Segoe UI Symbol;font-size:16px;fill:rgb(237, 237, 237);}@keyframes edge-animation-frame{from{stroke-dashoffset:0;}}@keyframes dash{to{stroke-dashoffset:0;}}#chatgpt-mermaid-_r_1c9_ .edge-animation-slow{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 50s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-_r_1c9_ .edge-animation-fast{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 20s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-_r_1c9_ .error-icon{fill:rgb(48, 48, 48);}#chatgpt-mermaid-_r_1c9_ .error-text{fill:rgb(237, 237, 237);stroke:rgb(237, 237, 237);}#chatgpt-mermaid-_r_1c9_ .edge-thickness-normal{stroke-width:1px;}#chatgpt-mermaid-_r_1c9_ .edge-thickness-thick{stroke-width:3.5px;}#chatgpt-mermaid-_r_1c9_ .edge-pattern-solid{stroke-dasharray:0;}#chatgpt-mermaid-_r_1c9_ .edge-thickness-invisible{stroke-width:0;fill:none;}#chatgpt-mermaid-_r_1c9_ .edge-pattern-dashed{stroke-dasharray:3;}#chatgpt-mermaid-_r_1c9_ .edge-pattern-dotted{stroke-dasharray:2;}#chatgpt-mermaid-_r_1c9_ .marker{fill:rgb(175, 175, 175);stroke:rgb(175, 175, 175);}#chatgpt-mermaid-_r_1c9_ .marker.cross{stroke:rgb(175, 175, 175);}#chatgpt-mermaid-_r_1c9_ svg{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,Segoe UI,Helvetica,Apple Color Emoji,Arial,sans-serif,Segoe UI Emoji,Segoe UI Symbol;font-size:16px;}#chatgpt-mermaid-_r_1c9_ p{margin:0;}#chatgpt-mermaid-_r_1c9_ .label{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,Segoe UI,Helvetica,Apple Color Emoji,Arial,sans-serif,Segoe UI Emoji,Segoe UI Symbol;color:rgb(237, 237, 237);}#chatgpt-mermaid-_r_1c9_ .cluster-label text{fill:rgb(237, 237, 237);}#chatgpt-mermaid-_r_1c9_ .cluster-label span{color:rgb(237, 237, 237);}#chatgpt-mermaid-_r_1c9_ .cluster-label span p{background-color:transparent;}#chatgpt-mermaid-_r_1c9_ .label text,#chatgpt-mermaid-_r_1c9_ span{fill:rgb(237, 237, 237);color:rgb(237, 237, 237);}#chatgpt-mermaid-_r_1c9_ .node rect,#chatgpt-mermaid-_r_1c9_ .node circle,#chatgpt-mermaid-_r_1c9_ .node ellipse,#chatgpt-mermaid-_r_1c9_ .node polygon,#chatgpt-mermaid-_r_1c9_ .node path{fill:rgb(9, 23, 44);stroke:rgb(31, 78, 148);stroke-width:1px;}#chatgpt-mermaid-_r_1c9_ .rough-node .label text,#chatgpt-mermaid-_r_1c9_ .node .label text,#chatgpt-mermaid-_r_1c9_ .image-shape .label,#chatgpt-mermaid-_r_1c9_ .icon-shape .label{text-anchor:middle;}#chatgpt-mermaid-_r_1c9_ .node .katex path{fill:#000;stroke:#000;stroke-width:1px;}#chatgpt-mermaid-_r_1c9_ .rough-node .label,#chatgpt-mermaid-_r_1c9_ .node .label,#chatgpt-mermaid-_r_1c9_ .image-shape .label,#chatgpt-mermaid-_r_1c9_ .icon-shape .label{text-align:center;}#chatgpt-mermaid-_r_1c9_ .node.clickable{cursor:pointer;}#chatgpt-mermaid-_r_1c9_ .root .anchor path{fill:rgb(175, 175, 175)!important;stroke-width:0;stroke:rgb(175, 175, 175);}#chatgpt-mermaid-_r_1c9_ .arrowheadPath{fill:rgb(175, 175, 175);}#chatgpt-mermaid-_r_1c9_ .edgePath .path{stroke:rgb(175, 175, 175);stroke-width:1px;}#chatgpt-mermaid-_r_1c9_ .flowchart-link{stroke:rgb(175, 175, 175);fill:none;}#chatgpt-mermaid-_r_1c9_ .edgeLabel{background-color:rgb(0, 0, 0);text-align:center;}#chatgpt-mermaid-_r_1c9_ .edgeLabel p{background-color:rgb(0, 0, 0);}#chatgpt-mermaid-_r_1c9_ .edgeLabel rect{opacity:0.5;background-color:rgb(0, 0, 0);fill:rgb(0, 0, 0);}#chatgpt-mermaid-_r_1c9_ .labelBkg{background-color:rgba(0, 0, 0, 0.5);}#chatgpt-mermaid-_r_1c9_ .cluster rect{fill:rgb(48, 48, 48);stroke:rgba(255, 255, 255, 0.15);stroke-width:1px;}#chatgpt-mermaid-_r_1c9_ .cluster text{fill:rgb(237, 237, 237);}#chatgpt-mermaid-_r_1c9_ .cluster span{color:rgb(237, 237, 237);}#chatgpt-mermaid-_r_1c9_ div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,Segoe UI,Helvetica,Apple Color Emoji,Arial,sans-serif,Segoe UI Emoji,Segoe UI Symbol;font-size:12px;background:rgb(48, 48, 48);border:1px solid rgba(255, 255, 255, 0.15);border-radius:2px;pointer-events:none;z-index:100;}#chatgpt-mermaid-_r_1c9_ .flowchartTitleText{text-anchor:middle;font-size:18px;fill:rgb(237, 237, 237);}#chatgpt-mermaid-_r_1c9_ rect.text{fill:none;stroke-width:0;}#chatgpt-mermaid-_r_1c9_ .icon-shape,#chatgpt-mermaid-_r_1c9_ .image-shape{background-color:rgb(0, 0, 0);text-align:center;}#chatgpt-mermaid-_r_1c9_ .icon-shape p,#chatgpt-mermaid-_r_1c9_ .image-shape p{background-color:rgb(0, 0, 0);padding:2px;}#chatgpt-mermaid-_r_1c9_ .icon-shape .label rect,#chatgpt-mermaid-_r_1c9_ .image-shape .label rect{opacity:0.5;background-color:rgb(0, 0, 0);fill:rgb(0, 0, 0);}#chatgpt-mermaid-_r_1c9_ .label-icon{display:inline-block;height:1em;overflow:visible;vertical-align:-0.125em;}#chatgpt-mermaid-_r_1c9_ .node .label-icon path{fill:currentColor;stroke:revert;stroke-width:revert;}#chatgpt-mermaid-_r_1c9_ .node .neo-node{stroke:rgb(31, 78, 148);}#chatgpt-mermaid-_r_1c9_ [data-look="neo"].node rect,#chatgpt-mermaid-_r_1c9_ [data-look="neo"].cluster rect,#chatgpt-mermaid-_r_1c9_ [data-look="neo"].node polygon{stroke:url(#chatgpt-mermaid-_r_1c9_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-_r_1c9_ [data-look="neo"].swimlane.cluster rect{filter:none;}#chatgpt-mermaid-_r_1c9_ [data-look="neo"].node path{stroke:url(#chatgpt-mermaid-_r_1c9_-gradient);stroke-width:1px;}#chatgpt-mermaid-_r_1c9_ [data-look="neo"].node .outer-path{filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-_r_1c9_ [data-look="neo"].node .neo-line path{stroke:rgb(31, 78, 148);filter:none;}#chatgpt-mermaid-_r_1c9_ [data-look="neo"].node circle{stroke:url(#chatgpt-mermaid-_r_1c9_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-_r_1c9_ [data-look="neo"].node circle .state-start{fill:#000000;}#chatgpt-mermaid-_r_1c9_ [data-look="neo"].icon-shape .icon{fill:url(#chatgpt-mermaid-_r_1c9_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-_r_1c9_ [data-look="neo"].icon-shape .icon-neo path{stroke:url(#chatgpt-mermaid-_r_1c9_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-_r_1c9_ .node text{font-size:14px;font-weight:600;letter-spacing:normal;fill:rgb(153, 206, 255);}#chatgpt-mermaid-_r_1c9_ .edgeLabels text{font-size:13px;font-weight:600;letter-spacing:-0.08px;fill:rgb(153, 206, 255);}#chatgpt-mermaid-_r_1c9_ .node tspan[font-weight="normal"],#chatgpt-mermaid-_r_1c9_ .edgeLabels tspan[font-weight="normal"]{font-weight:600;}#chatgpt-mermaid-_r_1c9_ .edgeLabel .label rect{opacity:1;rx:13px;ry:13px;fill:rgb(0, 14, 26);stroke:rgb(26, 62, 95);stroke-width:1px;}#chatgpt-mermaid-_r_1c9_ .node rect,#chatgpt-mermaid-_r_1c9_ .node circle,#chatgpt-mermaid-_r_1c9_ .node ellipse,#chatgpt-mermaid-_r_1c9_ .node polygon,#chatgpt-mermaid-_r_1c9_ .node path{fill:rgb(0, 40, 77);stroke:rgba(255, 255, 255, 0.1);stroke-width:1px;}#chatgpt-mermaid-_r_1c9_ .node rect{rx:16px;ry:16px;}#chatgpt-mermaid-_r_1c9_ .mermaid-decision .label-container{fill:rgb(0, 14, 26);stroke:rgb(26, 62, 95);stroke-dasharray:2px,2px;}#chatgpt-mermaid-_r_1c9_ .edgePaths .flowchart-link{stroke:rgb(175, 175, 175);stroke-width:1px;stroke-linecap:round;stroke-linejoin:round;}#chatgpt-mermaid-_r_1c9_ .marker{fill:rgb(175, 175, 175);stroke:rgb(175, 175, 175);}#chatgpt-mermaid-_r_1c9_ :root{--mermaid-font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,Segoe UI,Helvetica,Apple Color Emoji,Arial,sans-serif,Segoe UI Emoji,Segoe UI Symbol;}Repository and governancebaselineBackend bootstrap + protectedAPISQLite + migrations +repositoriesWorkflow domain + DAGvalidationSequential execution + mockproviderRetries + timeouts + lifecycleLeases + startup recoveryProvider adapter contractOllama + approved hostedproviderControlled tools +authorizationAction-scoped approvals +validatorsStable API + Tauri lifecycleTypeScript desktop UIEnd-to-end regression +securityWindows install +backup/restorev1.0 release





Testing and documentation should run continuously rather than being postponed until the final release.

Critical-path decision: provider integration and controlled tool execution can be developed as separate workstreams after the execution foundation is stable. Their integration must be verified before desktop MVP completion.

8. Database and migration roadmap
The existing DATABASE.md remains the schema authority. No schema changes are authorized by this roadmap.
Release	Database work	Migration implications	Required verification
v0.1	Initial project, workflow, version, run, task, attempt, event records	Initial schema migration	Fresh install and repository tests
v0.2	Retry, timeout, lease, heartbeat, pause/cancel, recovery fields	Existing 002_v02_durable_execution.sql is present	Upgrade from v0.1 and recovery tests
v0.3	Provider configuration and credential references, only as specified by canonical DB contract	Proposed change if not already represented	Secret reference and provider config tests
v0.4	Approval and operation/audit persistence, if not already represented	Requires explicit schema review	Fingerprint, immutability, and audit tests
v0.5	No automatic schema redesign for UI convenience	API and DB remain independently versioned	UI reads persisted state
v0.9	Backup, restore, migration upgrade and integrity verification	No feature schema expansion	Windows WAL backup/restore
v1.0	Freeze and verify	Release migration set	Clean install and upgrade
v1.1+	Each new capability reviewed independently	Approved migration and compatibility plan	Migration regression

Database controls
Preserve the documented principles:
- SQLite standard-library driver.
- Foreign-key enforcement.
- WAL behavior where approved.
- Explicit transaction boundaries.
- Short-lived or managed connections.
- No long-running write transaction around network calls, model execution, subprocesses, or approval waits.
- No ORM.
- Append-only execution history where specified.
- Explicit idempotency and consistency behavior.
- Controlled artifact references rather than storing arbitrary binary artifacts in workflow JSON.
Important finding: the dedicated repository already has two migration files, while the DATABASE.md in the other repository differs. Do not infer that the current migration set is fully aligned with the canonical schema until the two database specifications are reconciled.

9. Security roadmap
Security is not a post-MVP workstream. Foundational controls are required from v0.1.
Security control	Release	Required evidence
Loopback-only backend binding	v0.1	Network binding test
Session/bootstrap boundary	v0.1	Invalid/missing token tests
Host and Origin validation	v0.1	Rejected-origin tests
Request validation and size limits	v0.1	Invalid and oversized request tests
Backend-owned state transitions	v0.1	Illegal transition tests
Credential references	v0.3	No plaintext secret persistence
Local/hosted data policy	v0.3	Transmission policy tests
Project-scoped filesystem access	v0.4	Traversal and symlink tests
Restricted subprocess execution	v0.4	Unsafe command and environment tests
Action-scoped approvals	v0.4	Fingerprint tampering and replay tests
Model/tool output treated as untrusted	v0.3–v0.4	Injection and validation tests
Secret redaction	v0.2 onward	Log and error redaction tests
Windows install and backup protection	v0.9	Installation and recovery tests
Dependency vulnerability review	v0.9	Dependency inventory and review record

MVP security release blockers
The MVP must not be declared usable until:
1. The backend protects all application routes.
2. The frontend cannot bypass backend authorization.
3. No unrestricted shell or filesystem access is exposed.
4. Approval decisions are action-scoped and immutable.
5. Secrets are not exposed in plaintext storage or logs.
6. Hosted provider transmission is explicit and controlled.
7. Path traversal and symlink/reparse-point escapes are addressed.
8. Uncertain external outcomes are not treated as verified success.
9. Recovery preserves completed work and does not blindly repeat unsafe operations.

10. Testing and quality roadmap
Release	Test layers	Quality gate
v0.1	Unit, domain, migration, API, sequential execution	Dependency order and persisted results proven
v0.2	Retry, timeout, pause/cancel, lease, crash recovery, WAL	Recovery and retry invariants proven
v0.3	Provider contract, Ollama, hosted provider, secret handling	Provider failures and timeouts handled safely
v0.4	Filesystem, subprocess, Git, approvals, validators	Unauthorized and modified actions rejected
v0.5	Frontend/backend integration, desktop lifecycle, UI states	Core desktop flows use real backend data
v0.9	Full E2E, security, upgrade, backup/restore, Windows install	No unresolved release-blocking defects
v1.0	Full regression and acceptance suite	Every applicable MVP requirement evidenced

Required test categories
- Unit tests for domain and policy logic.
- API contract and authentication tests.
- SQLite constraints and migration tests.
- Integration tests for execution lifecycle.
- Provider adapter conformance tests.
- Negative-path security tests.
- Filesystem boundary tests.
- Subprocess restriction tests.
- Approval tampering and replay tests.
- Crash and recovery tests.
- Backup and restore tests.
- Frontend/backend contract tests.
- Tauri startup/shutdown tests.
- Windows clean-install and upgrade tests.
Do not use a generic code-coverage percentage as a substitute for testing the system's critical invariants.

11. Production-readiness checklist
The product should be called production-ready only when the following are evidenced.
Functional readiness
- [ ] Projects can be created and selected.
- [ ] Workflow definitions can be created and validated.
- [ ] Dependency order is enforced.
- [ ] Local and approved hosted providers work.
- [ ] Controlled tools operate within project boundaries.
- [ ] Approval requests and decisions work.
- [ ] Deterministic validation is available.
- [ ] Execution history and artifacts are inspectable.
Reliability
- [ ] Run and task states persist.
- [ ] Retry policy is bounded.
- [ ] Timeouts are enforced.
- [ ] Failure propagation is deterministic.
- [ ] Cancellation and pause semantics are tested.
- [ ] Startup recovery is idempotent.
- [ ] Uncertain side effects enter a safe state.
- [ ] Database and artifact consistency are checked.
Security
- [ ] Protected local API.
- [ ] Backend authorization.
- [ ] Credential-store integration.
- [ ] Secret redaction.
- [ ] Filesystem boundary enforcement.
- [ ] Symlink/reparse-point protection.
- [ ] Restricted subprocess execution.
- [ ] Immutable action-scoped approvals.
- [ ] Explicit hosted-data transmission policy.
- [ ] Dependency security review.
Operations and release
- [ ] Windows packaging.
- [ ] Clean installation.
- [ ] Upgrade and migration testing.
- [ ] Backup and restore.
- [ ] Application shutdown/startup testing.
- [ ] Diagnostics and logs.
- [ ] User documentation.
- [ ] Release notes and known limitations.
- [ ] Reproducible build and test instructions.

12. Risk register
Risk ID	Risk	Likelihood	Impact	Affected release	Mitigation	Approval required
R-01	Conflicting repository snapshots	High	High	All	Confirm canonical repository and DB specification	Yes
R-02	Stale implementation status	High	High	v0.1–v1.0	Rebaseline using source/test evidence	Yes
R-03	Conflicting agent instructions	High	Medium	All	Reconcile AGENTS and SOP	Yes
R-04	Hosted provider undecided	High	Medium	v0.3	Select provider and approve SDK	Yes
R-05	Retry duplicates external side effects	Medium	Critical	v0.2–v1.0	Operation records, idempotency, reconciliation	Design approval
R-06	Lease behavior implies unsupported concurrency	Medium	High	v0.2	Verify lease semantics against sequential MVP	Design approval
R-07	Unsafe project path handling	Medium	Critical	v0.4	Canonical path and reparse-point tests	Security approval
R-08	Subprocess execution escape	Medium	Critical	v0.4	Allowlist, shell=False, bounded environment	Security approval
R-09	Approval replay or tampering	Medium	Critical	v0.4	Immutable fingerprints and revalidation	Security approval
R-10	Hosted provider leaks local project data	Medium	Critical	v0.3	Explicit data transmission and redaction policy	Security approval
R-11	SQLite WAL backup inconsistency	Medium	High	v0.9	Windows backup/restore tests	DB design approval
R-12	Tauri/backend lifecycle instability	Medium	High	v0.5	Process ownership and shutdown tests	Architecture approval
R-13	Missing desktop implementation delays MVP	High	High	v0.5	Keep desktop work behind stable backend contracts	Release approval
R-14	Documentation duplication causes drift	High	Medium	All	Single authority per domain	Governance approval
R-15	Feature expansion delays production MVP	High	High	v1.0	Enforce release exclusions	Scope approval


13. Deferred features register
These capabilities should remain outside the MVP unless a separate scope change is approved.
Feature	Why deferred	Missing prerequisite	Candidate release
Parallel execution	Adds concurrency and recovery complexity	Concurrency and resource model	v1.2+
Automatic provider fallback	Different providers are not interchangeable	Provider policy and data rules	v1.2+
Intelligent routing	Adds cost, policy, and selection complexity	Reliable provider metadata and evaluation	v1.2+
MCP	Expands the external tool trust boundary	MCP security and capability design	v1.2+
n8n	Adds another execution platform	Demonstrated integration need	Future
Multi-agent coordination	Introduces agent lifecycle and shared context issues	Agent state and isolation model	v2.x
Knowledge retrieval	Requires ingestion, indexing, provenance	Data lifecycle and retrieval design	v2.x
Cloud synchronization	Changes the local-first trust model	Separate cloud architecture	Major-version proposal
Multi-user collaboration	Requires identity, permissions, and shared state	Multi-user threat model	Major-version proposal
Marketplace	Adds supply-chain and third-party trust risks	Extension signing and review model	Future

Deferred does not mean approved for a future release. Each capability requires a documented user problem, design, security analysis, and owner approval.

14. Ordered implementation backlog
This backlog is a planning-level sequence. It intentionally does not invent file paths for missing desktop modules.

The actual existing backend paths are referenced where they were observed. Proposed modules must be named only after a detailed implementation plan and design approval.
Phase 1 - v0.1 and v0.2
Task ID	Task	Files/modules affected	Dependencies	Priority	Verification
FND-001	Reconcile repository and specification authority	AGENTS.md, AGENT_WORKFLOW.md, README.md	Owner decision	P0	Authority review
FND-002	Rebaseline actual implementation status	RELEASES/IMPLEMENTATION_STATUS.md	Repository audit	P0	Evidence review
DB-001	Verify schema/migration alignment	DATABASE.md, migrations/	Canonical DB decision	P0	Migration tests
BACK-001	Verify backend lifecycle and API boundary	api/app.py, api/security.py	v0.1 scope	P0	API/auth tests
WF-001	Verify domain and DAG invariants	domain/	DB-001	P0	Unit tests
EXEC-001	Verify sequential scheduler behavior	engine/scheduler.py	WF-001	P0	Integration tests
EXEC-002	Verify persisted retry and timeout behavior	engine/scheduler.py, domain/retry.py	EXEC-001	P0	Failure tests
EXEC-003	Verify lease and recovery behavior	engine/lease.py, engine/recovery.py	EXEC-002	P0	Crash/recovery tests
OBS-001	Verify log redaction and consistency checks	engine/logging.py, engine/consistency.py	Backend baseline	P0	Security tests
TEST-001	Establish current executable baseline	Existing tests/, pyproject.toml	Repository access	P0	Actual test run

Phase 2 - v0.3 and v0.4
Task ID	Task	Files/modules affected	Dependencies	Priority	Verification
PROV-001	Approve provider and SDK	TECH_STACK.md	Owner decision	P0	Decision record
PROV-002	Implement provider contract	Existing engine provider boundary; proposed adapter module if required	v0.2	P0	Contract tests
PROV-003	Implement Ollama adapter	Proposed provider module	PROV-002	P0	Local inference tests
PROV-004	Implement approved hosted adapter	Proposed provider module	PROV-001, PROV-002	P0	Hosted tests
SEC-001	Integrate credential store	Proposed credential module	Provider decision	P0	Secret-handling tests
TOOL-001	Implement scoped filesystem adapter	Proposed tool module	Project boundaries	P0	Path security tests
TOOL-002	Implement Git inspection adapter	Proposed tool module	TOOL-001 boundaries	P0	Scoped Git tests
TOOL-003	Implement allowlisted test runner	Proposed tool module	Security approval	P0	Subprocess tests
APPROVAL-001	Implement approval lifecycle	Execution and persistence modules	DB/API approval	P0	Approval tests
APPROVAL-002	Implement immutable action binding	Approval and policy modules	APPROVAL-001	P0	Tampering/replay tests
EXEC-004	Implement deterministic output validation	Validation module	Provider/tool contracts	P0	Invalid output tests

Phase 3 - v0.5
Task ID	Task	Files/modules affected	Dependencies	Priority	Verification
DESK-001	Establish Tauri shell	Proposed src-tauri/	Stable API	P0	Windows startup
DESK-002	Establish TypeScript frontend	Proposed frontend directory	DESK-001	P0	Build/type checks
DESK-003	Implement typed API client	Proposed frontend API module	API contract	P0	Contract tests
DESK-004	Implement project and workflow screens	Proposed frontend views	DESK-003	P0	E2E tests
DESK-005	Implement run and task detail views	Proposed frontend views	DESK-003	P0	E2E tests
DESK-006	Implement approval and provider settings	Proposed frontend views	v0.3, v0.4	P0	Permission and provider tests
DESK-007	Implement polling and application states	Proposed frontend services/views	Stable API	P0	Integration tests

Phase 4 - v0.9 and v1.0
Task ID	Task	Files/modules affected	Dependencies	Priority	Verification
REL-001	Full end-to-end regression	Existing and new test suites	v0.5	P0	E2E evidence
SEC-002	Security verification	Backend, tools, providers, desktop	v0.4–v0.5	P0	Security matrix
DB-002	Verify Windows backup/restore	DB and release scripts	v0.2	P0	Restore integrity
REL-002	Windows packaging and clean install	Tauri build/package configuration	v0.5	P0	Clean VM/install
REL-003	Verify upgrade and migrations	Migration/release tooling	DB-002	P0	Upgrade tests
DOC-001	Final user and recovery documentation	README and release docs	All	P0	Documentation review
REL-004	Final MVP acceptance	SCOPE.md, release checklist	v0.9	P0	Owner approval

Phase 5 - Post-MVP
Create individual proposals for templates, import/export, additional providers, richer observability, parallel execution, routing, MCP, and advanced orchestration. Do not put them into the MVP backlog.

15. Proposed documentation structure
The repository already has substantial release planning documents. Do not create the proposed docs/roadmap/ and docs/releases/ structure wholesale.

That would create a second roadmap authority.
Existing document	Recommended role
RELEASES/RELEASE_ROADMAP.md	Master release roadmap
RELEASES/RELEASE_SPECIFICATIONS.md	Release-by-release specifications
RELEASES/RELEASE_TRACEABILITY.md	Requirement-to-release-to-test traceability
RELEASES/RELEASE_RISK_REGISTER.md	Risk register
RELEASES/RELEASE_CHECKLIST.md	Release gates and checklist
RELEASES/IMPLEMENTATION_STATUS.md	Evidence-based live status
AGENTS.md	Repository agent entry point
AGENT_WORKFLOW.md	Mandatory agent operating procedure
SCOPE.md	MVP scope authority
DATABASE.md	Persisted data authority
EXECUTION_ENGINE.md	Execution lifecycle authority
SECURITY.md	Security authority

The immediate requirement is reconciliation and correction, not adding more documentation files.

16. Production Definition of Done
Every feature must meet these conditions:
1. Requirement and acceptance criteria are documented.
2. Its primary release is identified.
3. The relevant domain design is approved.
4. Existing architecture and interfaces are preserved unless changes are approved.
5. Input validation and error handling are implemented.
6. Security implications are addressed.
7. Relevant positive and negative tests are added.
8. Tests are actually executed.
9. Existing regressions are checked.
10. Documentation is updated.
11. No unauthorized dependency or scope change is introduced.
12. Acceptance evidence is recorded.
13. Final review confirms the change matches the approved design.
A feature is not complete merely because its UI is visible or its code compiles.

17. Approval decisions required from the project owner
These decisions should be resolved before implementation proceeds.
Decision ID	Decision	Recommendation	Priority
ADR-001	Canonical implementation repository	Dedicated AI-Workflow-Orchestrator repository	Critical
ADR-002	Canonical database specification	Select one DB document and reconcile migrations	Critical
ADR-003	Release roadmap	Retain seven-release ladder under five phases	Critical
ADR-004	Agent instruction conflict	Reconcile AGENTS.md and AGENT_WORKFLOW.md	High
ADR-005	Hosted provider	Select one provider and approve its SDK/version	High
ADR-006	Lease semantics	Confirm leases are for local execution ownership/recovery, not distributed workers	High
ADR-007	PRD §19	Keep deferred unless explicitly approved	High
ADR-008	Architecture document	Keep ARCHITECTURE.md canonical; reconcile legacy technical-architecture.md	Medium
ADR-009	Test coverage threshold	Do not impose arbitrary percentage; use critical invariant coverage	Medium
ADR-010	Existing implementation status	Replace contradictory status claims with observed evidence	High

One technical point deserves special attention: the dedicated repository's current pyproject.toml uses version ranges rather than an exact lockfile-based dependency snapshot. The technology policy requires controlled dependency versions. The implementation plan should establish how reproducible builds and approved dependency pinning will be achieved. Do not upgrade packages simply to resolve this; first agree on the approved dependency-management approach.

18. Final architecture review
The proposed roadmap passes the high-level architecture test, subject to the unresolved documentation decisions.
Review area	Assessment
Modular monolith preserved	Yes
Backend authority preserved	Yes
SQLite retained for MVP	Yes
Sequential execution preserved	Yes
Recovery before advanced integrations	Yes
Provider adapters isolated	Yes
Tools protected by authorization	Yes
Approvals bound to exact actions	Yes
Windows desktop work sequenced after stable backend	Yes
Distributed infrastructure avoided	Yes
Advanced AI capabilities deferred	Yes
Release acceptance independently verifiable	Yes

Recommended next action
Do not start feature development yet. First approve the canonical repository, reconcile the database and agent-governance documents, and rebaseline the existing implementation with actual local test results. Then approve the v0.1/v0.2 baseline and the first bounded implementation plan.

Final status: 
BLOCKED - DOCUMENTATION OR SCOPE CONFLICTS

The roadmap structure is defensible, but the project should not be marked ready for implementation until the owner decisions in Section 17 are resolved. No files were modified, no dependencies were installed, and no implementation or release was started.
