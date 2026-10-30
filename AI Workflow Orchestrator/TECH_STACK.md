# Approved Technology Stack - AI Workflow Orchestrator

**Document:** `TECH_STACK.md`  
**Version:** 1.0  
**Status:** Proposed baseline - binding after project-owner acceptance  
**Target:** MVP v1.0  
**Deployment:** Windows-first, local-first desktop application  
**Last reviewed:** 2026-10-30

---

## 1. Purpose and authority

This document is the authoritative registry of technologies approved for AI Workflow Orchestrator. It exists to keep implementation consistent, minimize operational complexity, and prevent framework creep, dependency bloat, and unreviewed technology substitutions.

Read this document together with:

- `prd.md` - product requirements and user outcomes.
- `ARCHITECTURE.md` - system boundaries, module responsibilities, and data flow.
- `SCOPE.md` - approved MVP scope and exclusions.
- `design.md` - user interface and visual design requirements.
- `AGENTS.md`, if present - repository-specific contributor instructions.

When technology choices conflict, stop and request a decision. Do not resolve conflicts by silently choosing a new framework or dependency.

### 1.1 Registry rules

1. Technologies listed as **Approved** may be used within the stated responsibility and version policy.
2. Technologies listed as **Conditional** may be used only when the stated condition is met.
3. Technologies listed as **Not approved** must not be introduced into MVP.
4. A technology appearing in an example, tutorial, generated plan, issue, or future roadmap is not thereby approved.
5. This registry controls technology selection; `SCOPE.md` still controls whether the capability itself is in MVP.
6. Existing repository configuration must be inspected before changing versions or build systems. Do not perform a migration merely to match a preferred version.
7. Exact resolved dependency versions must be committed through the appropriate lockfiles. Version ranges in this document define the permitted compatibility line, not permission to install arbitrary latest releases.

**Important status note:** This document records the approved target baseline derived from the current PRD, architecture, and scope. Where the repository has not yet established an implementation or lockfile, the version line below is the target to adopt during foundation setup-not a claim that the dependency is already installed or tested. The project owner must accept this baseline before it is treated as ratified.

## 2. Core stack and versions

### 2.1 Approved platform stack

| Layer | Approved technology | Version policy | Status | Responsibility |
|---|---|---|---|---|
| Desktop shell | Tauri | Major version 2.x; pin exact compatible release | Approved | Native desktop packaging, window lifecycle, narrow OS capabilities |
| Native toolchain | Rust | Stable channel, exact toolchain pinned in `rust-toolchain.toml` | Approved | Tauri host and narrowly scoped native commands |
| Frontend language | TypeScript | 5.x; exact version locked | Approved | UI, typed client contracts, presentation logic |
| Frontend runtime/tooling | Node.js | 22.x LTS; pin an exact patch release in project tooling | Approved | Frontend dependency installation and build/test scripts |
| Frontend platform | HTML5, CSS3, browser Fetch API | Standards-based; no framework requirement | Approved | UI structure, styling, local API communication |
| Backend language | Python | 3.12.x; pin exact patch in development/CI/runtime setup | Approved | Orchestration service and domain/application logic |
| Backend web framework | FastAPI | 0.115.x or later compatible 0.x release; exact version locked | Approved | Local HTTP API and request lifecycle |
| API/data validation | Pydantic | 2.x; exact version locked | Approved | Boundary validation and serialization |
| Database | SQLite | Runtime bundled/provided by the selected Python distribution; record runtime version in diagnostics | Approved | Durable local relational state |
| Database driver | Python standard-library `sqlite3` | Python 3.12 standard library | Approved | Parameterized SQL and explicit transaction control |
| Artifact storage | Local filesystem | OS-supported filesystem APIs | Approved | Workflow artifacts and larger payloads |
| Local inference | Ollama HTTP API | Support the documented API; validate runtime/model availability at connection time | Approved | Local model inference through a backend adapter |
| Hosted inference | One provider API and its official SDK, behind an adapter | Exact provider and SDK version must be approved and locked before installation | Conditional | Initial hosted model integration |

### 2.2 Architectural constraints

- The product is a **local-first modular monolith**.
- The Python backend is authoritative for workflow state, scheduling, authorization, approvals, provider/tool execution, persistence, usage accounting, and recovery.
- The frontend is a presentation and control surface. It must not own authoritative state transitions.
- The frontend must not call hosted model APIs or Ollama directly.
- The backend must bind to a protected local interface only; do not expose the service to the LAN or public internet by default.
- Provider and tool integrations must remain behind narrow adapter interfaces.
- SQLite and the local filesystem are the MVP persistence boundary.
- Execute tasks sequentially in MVP. Do not introduce distributed workers or parallel execution infrastructure.
- Do not add a second backend service, message broker, database server, or cloud control plane.

### 2.3 Frontend framework decision

The architecture deliberately avoids mandating a frontend framework before the existing frontend implementation is inspected.

- **Approved baseline:** TypeScript with semantic HTML and CSS.
- **Existing framework rule:** If the repository already contains a working UI framework, retain it. Do not migrate it as unrelated work.
- **New framework rule:** If no framework exists, vanilla TypeScript is the default. A framework may be proposed only through the dependency approval process in Section 6.
- **Not approved by default:** React, Vue, Angular, Svelte, Solid, or another component framework. None may be added or substituted without explicit approval.

This is intentional. Tauri 2 does not require a large UI framework, and the MVP prioritizes reliable execution and a lean desktop footprint over framework-specific abstractions.

### 2.4 Version pinning policy

- Pin exact resolved versions in committed lockfiles and toolchain files.
- Use compatible patch/minor updates within the approved major/minor line only after automated tests pass.
- Do not use floating `latest`, unbounded wildcards, or unpinned Git dependencies.
- Pin the Python interpreter used by development and CI; declare supported Python compatibility in project metadata.
- Pin Node through the repository's chosen version file/tooling and CI configuration.
- Pin Rust through `rust-toolchain.toml`; do not rely on an undocumented machine-global toolchain.
- Commit the frontend package-manager lockfile and Python dependency lockfile.
- Keep development, CI, packaging, and user runtime versions consistent.
- A major-version upgrade is a technology change and requires approval under Section 6.

## 3. Approved libraries and tooling

This section distinguishes approved dependencies from standard-library capabilities. Do not add a library simply because it is commonly used for a similar task.

### 3.1 Frontend

| Function | Approved choice | Rule |
|---|---|---|
| Language/compiler | TypeScript 5.x | Required for frontend application code |
| UI structure | HTML5 | Use semantic elements and accessible controls |
| Styling | CSS3 | Use the existing CSS organization; no utility CSS framework by default |
| API requests | Native Fetch API | Use a small typed API client; do not add a request framework without approval |
| Tauri integration | Official Tauri 2 JavaScript API and only required official plugins | Add plugins only for a concrete approved native capability |
| State management | Local component/module state and explicit application stores using TypeScript | No external state-management package approved for MVP |
| UI components | Project-owned components using the design system | No third-party component suite approved by default |
| Routing | Tauri window/navigation structure and existing frontend routing, if present | Do not add a router solely for hypothetical future screens |
| Icons | Existing icon assets or one consistent lightweight icon source | Do not install multiple icon libraries |
| Formatting/linting | Existing repository tooling; if absent, propose one formatter/linter pair before adding | Do not introduce overlapping tools |

### 3.2 Backend

| Function | Approved choice | Rule |
|---|---|---|
| HTTP API | FastAPI | Local application API only |
| Request/response schemas | Pydantic 2.x | Boundary schemas; keep domain entities framework-independent |
| ASGI server | Uvicorn | Required runtime server for FastAPI; pin compatible version |
| Database access | Standard-library `sqlite3` | Explicit SQL and transaction boundaries |
| Schema migrations | Versioned SQL migration files plus a small project-owned migration runner | No ORM or migration framework in MVP |
| Async orchestration | Python `asyncio` | Bounded in-process sequential scheduler and cancellation |
| HTTP client | HTTPX | Use for async provider/Ollama HTTP adapters where needed; avoid multiple HTTP client libraries |
| Serialization | Python standard library plus Pydantic at API boundaries | Do not duplicate serialization frameworks |
| Logging | Python `logging` with structured fields through a small project-owned adapter | Redact secrets; do not add an observability platform |
| Configuration | Environment variables and typed application configuration | Secrets must not be embedded in source, frontend bundles, or ordinary config files |
| IDs/time | Standard-library `uuid` and timezone-aware `datetime` | Centralize creation through small interfaces where deterministic tests require it |
| Filesystem | Python standard library | Enforce canonical project-root boundaries and safe file operations |
| Process execution | Python standard library `subprocess` / asyncio subprocess APIs | Strict allowlist, argument arrays (not shell interpolation), timeout, output limits, and approval policy |

### 3.3 Database and persistence

- Use SQLite as the only MVP database.
- Use normalized relational tables for projects, workflow definitions/versions, runs, tasks, attempts, events, approvals, artifacts, and usage records as required by the approved data model.
- Use explicit transactions for related state changes.
- Enable and verify appropriate SQLite safety settings, including foreign-key enforcement.
- Use parameterized SQL exclusively; never interpolate untrusted values into SQL statements.
- Use versioned SQL migrations. Migrations must be deterministic, ordered, testable, and safe to diagnose.
- Store large artifacts on the local filesystem and keep metadata/checksums/references in SQLite.
- Do not store API secrets in workflow definitions, artifacts, logs, or exported files.
- Do not introduce an ORM, graph database, vector database, or external database server for MVP.

### 3.4 Testing and quality

| Area | Approved choice | Rule |
|---|---|---|
| Backend tests | pytest | Required for unit, integration, and recovery tests |
| Async backend tests | pytest-asyncio | Add only where async tests require it; pin compatible version |
| Frontend tests | Existing test runner, if present | Preserve current tooling; if absent, propose a minimal runner before installation |
| Browser/UI automation | Not required for MVP baseline | Any browser automation dependency requires a separately approved test need |
| Static typing | TypeScript compiler and Python type annotations | Keep strict TypeScript settings; use existing Python type-check tooling if present |
| Lint/format | Existing configured tools | Do not add duplicate formatters or linters |
| Security checks | Dependency audit and focused tests for path boundaries, local API auth, secrets, and approval enforcement | Use existing CI capabilities before adding a new scanner |

Required test layers:

1. Domain tests for state transitions, dependencies, policy, and invariants.
2. Application tests for workflow use cases and orchestration.
3. Repository tests for SQLite transactions and migrations.
4. Adapter tests for provider and tool boundaries using deterministic fakes.
5. Recovery tests for interruption, restart, and uncertain external operations.
6. Security tests for authorization, path traversal, secret redaction, and command execution policy.
7. Frontend tests for critical forms, state rendering, and API error handling using the repository's approved test tooling.

Tests must not make real hosted-provider calls by default. Live provider tests must be opt-in, explicitly configured, and excluded from ordinary deterministic CI.

### 3.5 Development tools

- Git and GitHub for source control and code review.
- Repository lockfiles and version files for reproducible setup.
- pytest for backend testing.
- Existing package manager and scripts must be retained if already established.
- Use the official Tauri CLI/tooling compatible with Tauri 2.
- Use the official Python packaging/build tooling already present; do not add a second packaging system.
- Prefer standard OS diagnostics and application logs over a new telemetry stack.

## 4. Infrastructure and hosting

### 4.1 MVP deployment target

| Concern | Approved target |
|---|---|
| Application form | Windows desktop application |
| Desktop packaging | Tauri 2 bundle |
| Backend hosting | Python FastAPI process managed by the desktop application lifecycle |
| API exposure | Localhost/loopback only, protected by an application-scoped authentication/session mechanism |
| Database hosting | Embedded SQLite database stored in the application data directory |
| Artifact hosting | Local filesystem under controlled application/project storage paths |
| Model inference | Ollama on the local machine, or one explicitly configured hosted provider |
| User identity | No account or remote identity service required |
| Cloud services | None required for core operation |
| CI | GitHub Actions on a Windows runner for Windows build/package verification; other runners only when a concrete cross-platform check is needed |

### 4.2 Infrastructure explicitly not required for MVP

The following are not approved MVP dependencies:

- Docker or Docker Compose as a required runtime or installation prerequisite.
- Kubernetes or container orchestration.
- Redis, Celery, Kafka, RabbitMQ, or another message broker.
- PostgreSQL, MySQL, MongoDB, or a separate database server.
- Cloud hosting, remote workers, hosted control plane, or cloud synchronization.
- External telemetry/analytics infrastructure.
- A vector database or graph database.
- n8n or another automation platform.
- A public-facing backend endpoint or reverse proxy.

Docker may be used by an individual developer for an isolated experiment only if it does not become part of the product's required build, runtime, or release path. Such use must not change the approved architecture.

### 4.3 CI/CD requirements

- GitHub Actions is the approved CI platform.
- The primary release verification job must run on a Windows runner and validate the actual desktop packaging path.
- CI must install pinned toolchains and dependencies from committed lockfiles.
- CI must run backend tests, frontend checks that exist in the repository, and the Tauri build/package verification.
- CI must not require real provider API keys for ordinary pull-request validation.
- Secrets must be stored in GitHub Actions secrets only when a workflow genuinely requires them; never commit them.
- Release artifacts must be traceable to a commit and build run.
- Do not add a second CI platform without approval.

## 5. Permitted alternatives and flexibility

Flexibility is deliberately narrow. An alternative is pre-approved only within the conditions below; all other substitutions require the approval process in Section 6.

| Area | Default | Permitted alternative | Conditions |
|---|---|---|---|
| Frontend UI | Vanilla TypeScript + HTML/CSS or existing framework already in repository | Retain the repository's existing framework | No migration or additional framework without approval |
| Frontend package manager | Existing repository package manager | npm if no package manager has been established | One package manager and one lockfile only |
| Python dependency manager | Existing repository manager | pip with a committed, reproducible lock/requirements strategy if no manager exists | Do not maintain competing dependency manifests |
| SQLite migrations | Versioned SQL + project-owned runner | A lightweight migration tool may be proposed | Requires approval; no ORM bundled by default |
| Provider transport | HTTPX or official provider SDK | Official SDK for the single selected hosted provider | Must remain inside its adapter and be approved before installation |
| Ollama transport | HTTPX against Ollama API | Official maintained Ollama Python client | Only if it materially reduces adapter complexity; approval required |
| Logging | Python standard logging with structured fields | A lightweight structured logging library | Requires approval and must not introduce a remote logging service |
| Frontend tests | Existing runner | Vitest if a frontend test runner is needed and none exists | Requires dependency approval; do not add a second runner |
| Icons | Existing icon assets/source | Lucide icon package | One icon family only; approval required if adding a package |
| Native Tauri plugins | Minimal official Tauri 2 APIs | Specific official Tauri 2 plugin | Only for a documented required OS capability; review permissions and plugin maintenance |

### 5.1 No implicit alternatives

- “Permitted alternative” does not mean an agent can install it without review where the table explicitly requires approval.
- A library that is already transitive is not automatically approved as a direct application dependency.
- Replacing an existing working implementation is a migration and requires approval unless it is a compatible patch update within the pinned line.
- A new provider SDK does not authorize adding a new provider; provider count remains controlled by `SCOPE.md`.
- Future roadmap technologies are not approved for MVP.

## 6. Dependency rules and approval process

### 6.1 Mandatory pre-installation check

Before running any of the following, an AI coding agent or developer must inspect this document and the existing manifests/lockfiles:

- `npm install`, `npm add`, `pnpm add`, `yarn add`
- `pip install`, `uv add`, Poetry add commands, or equivalent Python dependency operations
- Cargo dependency additions or Tauri plugin installation
- Any command that changes a package manifest, lockfile, compiler, runtime, framework, or build system

The same rule applies before writing code that introduces a new framework, library, runtime, architectural paradigm, external service, or native capability.

**Do not install first and seek approval later.** Dependency installation is not a discovery mechanism for unapproved choices.

### 6.2 What requires approval

Explicit project-owner approval is required before:

- Adding any new direct runtime or development dependency not listed as approved.
- Adding a new Tauri plugin or native capability.
- Adding a new frontend framework, component library, state manager, router, styling framework, or UI toolkit.
- Adding an ORM, database engine, migration framework, queue, cache, broker, or persistence service.
- Adding a new provider SDK or hosted provider.
- Adding a new test runner, linter, formatter, type checker, build tool, or security scanner.
- Changing a foundational language, runtime, framework, database, package manager, or build system.
- Upgrading a foundational dependency to a new major version.
- Adding a dependency that introduces a native binary, background service, telemetry, network listener, or additional permission surface.
- Adding a dependency with unclear maintenance status, license, provenance, or transitive dependency impact.

Patch/minor updates within the approved version policy may proceed through normal review and CI, provided compatibility and security checks pass and no architecture or permission boundary changes.

### 6.3 Required approval workflow

1. **Inspect first.** Read `TECH_STACK.md`, `SCOPE.md`, `ARCHITECTURE.md`, the relevant PRD requirements, and current package manifests/lockfiles.
2. **State the need.** Identify the concrete requirement that cannot be met reasonably with the approved stack or standard library.
3. **Check existing capability.** Search the repository and approved dependencies to ensure the capability is not already available.
4. **Compare alternatives.** Include at least one no-new-dependency option and explain why it is insufficient, if applicable.
5. **Assess impact.** Document bundle/runtime size, transitive dependencies, license, maintenance, security history, permissions, platform compatibility, performance, migration effort, and long-term maintenance.
6. **Propose exact change.** Name the package, purpose, proposed version range, direct/transitive status, affected files, and tests.
7. **Request explicit human approval.** The project owner must approve the specific dependency and intended use. AI-generated plans, inferred preferences, silence, or ambiguous feedback are not approval.
8. **Update this registry.** Add the approved package/version and rationale to the correct section before installation.
9. **Install and lock.** Add the dependency through the selected package manager and commit the updated manifest and lockfile.
10. **Verify.** Run relevant tests, security/license review, and build checks. Report the result and any residual risk.

### 6.4 Dependency request template

| Field | Required detail |
|---|---|
| Request ID | Unique identifier |
| Package/technology | Exact name and source |
| Proposed version | Exact version or bounded range |
| Purpose | Specific requirement addressed |
| Existing capability | Why current stack/standard library is insufficient |
| Alternatives | No-dependency and existing-approved options considered |
| Dependency impact | Direct/transitive tree and size/runtime impact |
| Security and privacy | Permissions, network behavior, data collection, vulnerability posture |
| License and maintenance | License, maintainer activity, release history, support status |
| Architecture impact | Modules and boundaries affected |
| Test plan | Tests/build/security checks required |
| Recommendation | Approve / reject / defer |
| Owner decision | Explicit decision and date |

### 6.5 Outcomes

- `APPROVE` - exact package/technology and intended use are authorized; registry update is required before installation.
- `REJECT` - do not add or use it.
- `DEFER` - keep the proposal out of the current implementation.
- `REPLACE` - approve a defined migration that removes an existing technology; update architecture and migration plan first.

Approval is limited to the named package, version policy, and purpose. It is not blanket approval for related packages or a new framework ecosystem.

## 7. Explicitly prohibited framework and dependency creep

Unless the project owner approves a documented scope and stack change, do not introduce:

### 7.1 Frontend/framework substitutions

- React, Vue, Angular, Svelte, Solid, or another UI framework where none is already established.
- Tailwind CSS, Bootstrap, Material UI, Ant Design, shadcn/ui, or another styling/component system.
- Redux, Zustand, MobX, Pinia, or another external state-management library.
- A second router, component system, icon system, or CSS framework.
- A framework migration justified only by preference, familiarity, generated code, or trend.

**Foundational technologies must not be substituted.** Do not swap Tauri for Electron, TypeScript for JavaScript, Python/FastAPI for Node/Express, or SQLite for a server database without explicit approval and updated architecture/PRD/scope decisions.

### 7.2 Backend/infrastructure additions

- SQLAlchemy, SQLModel, Django ORM, or another ORM.
- PostgreSQL, MySQL, MongoDB, Neo4j, or a vector database.
- Redis, Celery, Kafka, RabbitMQ, or distributed job infrastructure.
- LangChain, LangGraph, CrewAI, AutoGen, or another agent framework in MVP.
- Docker as a required product runtime.
- Kubernetes, service mesh, microservices, or remote worker infrastructure.
- Multiple overlapping HTTP, logging, validation, serialization, or configuration libraries.

### 7.3 General dependency prohibitions

- No dependency added solely for syntactic convenience when the standard library or existing approved stack is sufficient.
- No duplicate libraries that solve the same problem.
- No abandoned, unmaintained, or unverified package.
- No dependency with incompatible or unclear licensing.
- No package installation directly from arbitrary Git branches or unverified URLs.
- No secrets, telemetry, or external network behavior hidden inside a dependency.
- No unbounded or floating dependency versions.

## 8. AI coding agent and contributor enforcement rules

Every AI coding agent and human contributor must follow these rules:

1. Read `TECH_STACK.md` before installing packages, changing manifests, selecting a framework, or introducing a new technical paradigm.
2. Read `SCOPE.md` before implementing a feature or integration.
3. Inspect the actual repository and current lockfiles; do not assume a technology is installed because it appears in documentation.
4. Do not substitute foundational technologies or frameworks.
5. Do not add major dependencies without explicit project-owner approval.
6. Do not run package installation commands before approval for an unlisted dependency.
7. Do not silently add transitive dependencies through a new framework, plugin, or SDK.
8. Prefer standard-library functionality and existing approved dependencies.
9. Keep provider SDKs, HTTP clients, database code, and OS-specific functionality behind the appropriate adapters.
10. If the task cannot be completed within the approved stack, stop and submit a dependency/stack change request. Do not improvise a substitute.
11. Update manifests and lockfiles together; never leave dependency state ambiguous.
12. Report dependency changes, version changes, tests run, and any security/license findings in the completion summary.

### 8.1 Required pre-change checklist

Before any dependency or framework change, confirm:

- [ ] I read `TECH_STACK.md`.
- [ ] I read the relevant scope and architecture sections.
- [ ] I inspected the current manifests and lockfiles.
- [ ] The required capability is not already available.
- [ ] The change has explicit project-owner approval.
- [ ] The approved version and purpose are recorded in this document.
- [ ] The dependency has passed license, maintenance, and security review.

If any required approval item is unchecked, do not install or introduce the dependency.

## 9. Change control and maintenance

- The project owner is the final authority for technology decisions.
- Update this document whenever an approved technology, version policy, package manager, build system, or deployment requirement changes.
- Update `ARCHITECTURE.md` when a technology change affects system boundaries, runtime topology, persistence, security, or module responsibilities.
- Update `SCOPE.md` when the change also expands MVP capabilities or infrastructure scope.
- Update `prd.md` when the change affects product requirements or user-visible behavior.
- Record material architectural changes in an Architecture Decision Record (ADR).
- Keep the registry synchronized with manifests, lockfiles, CI configuration, and actual implementation.
- A dependency appearing in a lockfile without an approved registry entry must be reviewed; do not assume it was intentionally approved.

## 10. Final rule

The approved MVP stack is:

**Tauri 2 + Rust (native host) + TypeScript + Python 3.12 + FastAPI + Pydantic 2 + SQLite (`sqlite3`) + local filesystem + Ollama adapter + one explicitly approved hosted-provider adapter.**

Use a lean TypeScript UI without adding a new frontend framework by default. Use Python's standard library and explicit SQL wherever practical. Run the backend as a protected local process and package the desktop application for Windows.

**No foundational technology substitution, framework addition, major dependency, or infrastructure expansion is permitted without explicit human approval and an update to this registry.**
