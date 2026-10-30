# AI Agent Workflow — Standard Operating Procedure

**Document:** `AGENT_WORKFLOW.md`  
**Version:** 1.0  
**Status:** Proposed — binding after project-owner acceptance  
**Target:** MVP v1.0  
**Repository:** `kumaradoss16/personal-projects/AI Workflow Orchestrator`  
**Last reviewed:** 2026-10-30

---

## 1. Purpose, authority, and mandatory adherence

This document defines the required operating procedure for every AI coding agent and human developer performing implementation work in this repository. It governs the complete task lifecycle: intake, context loading, planning, approval, implementation, verification, and handoff.

**Mandatory adherence:** AI coding agents MUST treat this document as their operating manual. They must follow it for every task, including bug fixes, refactoring, documentation changes, dependency updates, tests, and configuration changes. An agent must not skip a step because a task appears small or familiar. If a step is not applicable, state why in the final handoff.

This document governs agent workflow; it does not override the project's product, architecture, scope, technology, database, execution, API, or security specifications. Those documents remain authoritative within their respective domains.

### 1.1 Source-of-truth hierarchy

Use the following authority order when interpreting project requirements:

1. Explicit, current instruction or decision from the project owner for the task.
2. `SCOPE.md` for approved MVP boundaries and scope-change control.
3. `SECURITY.md` for security requirements and prohibited practices.
4. Domain specifications for their respective contracts:
   - `PRD.md` / `prd.md`: product requirements and acceptance outcomes.
   - `ARCHITECTURE.md`: system structure and component boundaries.
   - `TECH_STACK.md`: approved technologies, versions, and dependencies.
   - `DATABASE.md`: persisted data model and database behavior.
   - `EXECUTION_ENGINE.md`: workflow/task states, scheduling, retries, approvals, and recovery.
   - `API_SPECIFICATION.md`: backend/frontend API contract.
   - `design.md`: UX and visual design.
5. Existing implementation and tests, which provide evidence of current behavior but do not silently override approved specifications.

`AGENTS.md` may contain repository-level instructions. Read it if present. It was not present at the time this document was prepared; do not assume it exists or invent its contents. If it is later added, read it during every task's context-loading phase and reconcile any conflict before proceeding.

If two authoritative documents conflict, do not guess which one is newer or more convenient. Stop and ask the project owner to resolve the conflict, then update the affected documents before implementation.

### 1.2 Non-negotiable operating principles

- **Scope before code:** a requested or suggested feature is not automatically approved for MVP.
- **Understand before changing:** inspect existing implementation and tests before editing.
- **Plan before complex work:** communicate the intended changes and obtain approval at the required gates.
- **Small, reviewable changes:** prefer incremental, focused diffs over broad rewrites.
- **Backend authority:** frontend code must not take over backend responsibilities for state, authorization, scheduling, or persistence.
- **Deterministic execution:** workflow decisions must follow `EXECUTION_ENGINE.md`; do not introduce hidden in-memory progress or nondeterministic state mutation.
- **Security by default:** follow `SECURITY.md`; never bypass controls to make implementation easier.
- **Verify before claiming:** report only commands actually run and results actually observed.
- **No fabricated completion:** do not claim a test, build, migration, commit, or deployment succeeded unless its result confirms success.
- **Communicate uncertainty:** ask rather than making risky assumptions.

---

## 2. Task intake and context-loading order

No code, schema, dependency, configuration, or contract changes may begin until the applicable context-loading steps are complete.

### 2.1 Required sequence

**Step 1 — Read the task request**

Identify:

- The requested outcome and reason for the change.
- Explicit acceptance criteria and expected deliverables.
- Target platform, affected user flow, and constraints.
- Whether the task requests implementation, investigation, documentation, or a proposal.
- Any paths, issue identifiers, branch constraints, or commit instructions provided by the owner.

Rewrite ambiguous requests as a concise understanding statement. Ask a clarifying question if the intended behavior or acceptance criteria cannot be established safely.

**Step 2 — Check repository instructions**

- Read root-level `AGENTS.md`, if present.
- Read any nested `AGENTS.md` that applies to directories being changed.
- Read relevant contributor instructions, if present.
- Treat these as additional instructions, not permission to violate project specifications or security rules.

**Step 3 — Read scope and security boundaries**

Read:

1. `SCOPE.md`
2. `SECURITY.md)

Determine whether the requested work is in MVP scope, deferred, explicitly excluded, or security-sensitive. If out of scope, do not implement it. Follow the scope-change process and request an explicit decision.

**Step 4 — Read product and architecture documents**

Read:

1. `prd.md` (or the repository's canonical PRD filename)
2. `ARCHITECTURE.md)
3. `design.md`, when the task affects UI, interaction, or visual behavior.

Identify the product outcome, component ownership, data flow, module boundaries, and architectural constraints relevant to the task.

**Step 5 — Read the approved technology registry**

Read `TECH_STACK.md` before:

- Running `npm install`, `pip install`, or another package installation command.
- Updating lockfiles or build tooling.
- Introducing a library, framework, runtime, service, or new implementation paradigm.
- Replacing an existing technology.

Inspect current manifests and lockfiles as well. The registry's permitted version range is not permission to upgrade existing dependencies indiscriminately.

**Step 6 — Read the domain contract(s) relevant to the change**

Read the relevant specifications before touching their domain:

| Change area | Required specification |
|---|---|
| Tables, columns, SQL, repositories, migrations, persistence | `DATABASE.md` |
| Workflow/task states, scheduler, retries, cancellation, approvals, recovery | `EXECUTION_ENGINE.md` |
| Routes, request/response models, authentication transport, SSE, API client | `API_SPECIFICATION.md` |
| Authentication, secrets, tool execution, process spawning, file access, logging | `SECURITY.md` |
| UI layout, interaction, accessibility, visual behavior | `design.md` |
| Dependency or framework selection | `TECH_STACK.md` |
| Product behavior and acceptance outcomes | `prd.md` and `SCOPE.md` |

For cross-cutting changes, read every applicable specification. Do not rely on a summary or memory when the repository document is available.

**Step 7 — Inspect the existing implementation**

Before editing:

- Inspect the repository tree and relevant source files.
- Trace the current call path across UI, API, application/domain logic, adapters, and persistence as applicable.
- Read nearby tests, fixtures, schemas, types, and configuration.
- Check current Git status and diff. Do not overwrite pre-existing user changes.
- Search for existing utilities or abstractions before creating new ones.
- Identify compatibility constraints and callers of any interface being changed.

Do not assume that a file, route, function, or dependency exists based only on a document or prior conversation.

**Step 8 — Establish a baseline**

Run the narrowest relevant existing checks when practical. Record pre-existing failures separately from failures introduced by the task. Do not modify unrelated code to make the baseline appear clean.

### 2.2 Context-loading completion record

Before planning, the agent must be able to state:

- Which requirements and specifications apply.
- Whether the task is in scope.
- Which modules and contracts are affected.
- Which tests and checks are relevant.
- What assumptions or unresolved questions remain.

If this cannot be established, pause and ask for clarification.

---

## 3. Planning and human approval process

### 3.1 Required implementation plan

Before complex implementation, provide a concise plan containing:

1. **Objective:** what will change and what will not change.
2. **Scope:** the relevant approved scope items and acceptance criteria.
3. **Current behavior:** the existing implementation path and observed limitation.
4. **Proposed approach:** ordered, incremental steps.
5. **Files/components:** expected files to add, modify, or remove.
6. **Data/API impact:** schema, migration, API, persisted data, and compatibility impact.
7. **Security impact:** trust boundaries, permissions, secrets, process execution, and logging impact.
8. **Dependencies:** any proposed package or tooling changes, with justification.
9. **Verification:** tests, lint, type checks, build, and manual checks.
10. **Risks and rollback:** likely failure modes and how to revert safely.

Keep the plan proportional to the task, but never omit relevant security, persistence, or contract impact.

### 3.2 Changes that require explicit approval before implementation

The agent MUST present its plan and wait for explicit human approval before:

- Adding, removing, or changing database tables, columns, constraints, indexes, relationships, migrations, or persisted enum values.
- Changing architecture, module boundaries, execution model, state machines, scheduling, retry, recovery, or concurrency behavior.
- Adding or replacing a major dependency, framework, runtime, database, provider, or integration.
- Introducing a new API route or changing an API contract, authentication flow, authorization policy, or network exposure.
- Changing secret storage, encryption, logging, telemetry, sandboxing, process execution, filesystem permissions, or security controls.
- Introducing a new tool capability or expanding the set of actions available to an AI agent.
- Changing data retention, deletion, backup, export, or recovery behavior.
- Expanding MVP scope or implementing a deferred/out-of-scope capability.
- Performing destructive, irreversible, externally visible, or production-impacting operations.
- Rewriting a subsystem or performing a broad refactor that affects multiple modules or contracts.

A task request that clearly and explicitly authorizes a specific change may count as approval for that stated change only. It does not authorize adjacent scope expansion or unmentioned architectural decisions. If the authorization is ambiguous, ask.

### 3.3 Approval protocol

1. Present the plan and identify the approval gate(s).
2. State the material risks, alternatives, and unresolved decisions.
3. Stop implementation and wait.
4. Treat approval as valid only when the project owner explicitly approves the described plan or a clearly identified subset.
5. If the owner changes the requirement, update the plan and obtain approval for the revised high-impact work.
6. Record material decisions in the relevant specification or decision record before implementing them.

Silence, inactivity, a vague acknowledgement, an AI-generated recommendation, or a passing test is not approval.

### 3.4 Low-risk work

A narrowly scoped documentation correction, isolated bug fix, or test addition may proceed without a separate approval round when it is clearly in scope, does not change a protected contract, and has no material security or data risk. The agent must still state its intended approach and follow the verification and handoff requirements.

---

## 4. Implementation boundaries

### 4.1 Incremental implementation

- Work in small, logically complete steps.
- Make the smallest change that satisfies the approved acceptance criteria.
- Preserve existing public interfaces unless a contract change has been approved.
- Reuse established project patterns and approved dependencies.
- Keep domain logic independent of UI and provider-specific details.
- Keep provider and tool integrations behind the approved adapter boundaries.
- Keep backend state transitions and authorization in the backend.
- Keep persistence changes consistent with `DATABASE.md`.
- Keep workflow execution consistent with `EXECUTION_ENGINE.md`.
- Keep API behavior consistent with `API_SPECIFICATION.md`.
- Keep security controls consistent with `SECURITY.md`.

### 4.2 Strict prohibitions during implementation

Agents MUST NOT:

- Add features, UI sections, settings, integrations, or abstractions not required by the approved task.
- Substitute foundational technologies or introduce framework creep.
- Install a package before checking `TECH_STACK.md` and obtaining required approval.
- Create a database entity or relationship without approval.
- Change an API contract informally to make frontend/backend integration easier.
- Bypass validation, authorization, approval gates, transaction rules, or security tests.
- Add unrestricted shell execution or broaden tool permissions for convenience.
- Hardcode secrets, credentials, user-specific paths, or machine-specific configuration.
- Fabricate data, metrics, progress, provider availability, or test results.
- Add placeholder implementations, fake success paths, or controls that appear functional but do nothing.
- Rewrite unrelated files, reformat the entire repository, or remove user changes.
- Suppress warnings or failures without understanding and documenting their cause.
- Make unrelated dependency upgrades or lockfile churn.
- Change generated files manually when they should be regenerated through the approved build process.
- Commit, push, deploy, delete, migrate, or perform other external actions unless the task explicitly requests or authorizes them.

### 4.3 Dependency discipline

Before any package installation or dependency modification:

1. Read `TECH_STACK.md`.
2. Inspect the existing manifest and lockfile.
3. Search for an existing standard-library or approved project solution.
4. Explain why the dependency is necessary and why existing capabilities are insufficient.
5. Identify license, maintenance, transitive dependency, security, platform, and bundle-size implications.
6. Obtain explicit approval where required by `TECH_STACK.md` or this SOP.
7. Install the smallest compatible version and update the correct lockfile.
8. Run dependency and regression checks.

Do not use installation commands as exploratory discovery.

### 4.4 Database and migration discipline

Before changing persistence:

1. Read `DATABASE.md` and `EXECUTION_ENGINE.md` when execution state is involved.
2. Describe the schema and data migration impact.
3. Obtain explicit approval.
4. Update the canonical schema document first.
5. Create a forward migration using the approved migration workflow.
6. Preserve existing data and define rollback/recovery behavior.
7. Test fresh database creation, upgrade from the previous schema, constraints, and recovery.
8. Never silently edit a production or user database to make a test pass.

### 4.5 Security discipline

Before security-sensitive implementation, read `SECURITY.md`. Apply least privilege, deny-by-default validation, safe output rendering, secret redaction, and explicit authorization. If a secure implementation is blocked, stop and escalate; do not weaken the control.

### 4.6 Git and change hygiene

- Inspect `git status` before editing and before handoff.
- Never discard or overwrite unrelated user changes.
- Keep changes focused and reviewable.
- If commits are requested, create small, descriptive commits after verification.
- Do not amend, force-push, rebase shared history, or push to a protected branch unless explicitly authorized.
- Never commit credentials, local databases, generated secrets, private project data, or machine-specific artifacts.
- Report the exact commit hash and branch only after Git confirms them.

---

## 5. Verification and quality gates

A task is not complete merely because code was written or the application appears to work once. The agent must verify the change and report the evidence.

### 5.1 Required verification sequence

**Step 1 — Review the diff**

- Inspect every changed file.
- Check for unrelated modifications, accidental deletions, generated noise, debug code, TODO placeholders, secrets, and machine-specific paths.
- Confirm the diff matches the approved plan.

**Step 2 — Static checks**

Run the repository-approved checks that apply:

- Formatter.
- Linter.
- Type checker.
- Static analysis/security checks, where configured.

Do not invent tooling or add a new checker dependency without approval.

**Step 3 — Automated tests**

Run, as applicable:

- Focused unit tests for changed logic.
- Integration tests for affected boundaries.
- API contract tests for changed endpoints.
- Database and migration tests for persistence changes.
- Execution-state, retry, approval, cancellation, and recovery tests for orchestration changes.
- Frontend component or interaction tests for UI changes.
- Relevant regression suite.

Use the test framework and commands already defined by repository configuration and `TECH_STACK.md`.

**Step 4 — Build and packaging checks**

For changes affecting runtime, dependencies, native integration, frontend bundling, or packaging, run the relevant build. Verify that the packaged configuration does not accidentally enable debug behavior or broaden network exposure.

**Step 5 — Security and data-integrity review**

For applicable changes, verify:

- Authorization occurs in the backend.
- Inputs are validated at trust boundaries.
- Secrets are not exposed in logs, responses, artifacts, or process arguments.
- Filesystem paths remain inside authorized roots.
- Tool operations respect allowlists and approvals.
- Database transactions and constraints remain valid.
- Failure and cancellation paths do not report false success.
- External side effects are reconciled rather than blindly repeated.

**Step 6 — Regression and acceptance review**

Map each acceptance criterion to a test, command, or explicit manual verification. Confirm no unrelated behavior changed.

### 5.2 Failure handling

- If a check fails, inspect and report the actual failure.
- Distinguish pre-existing failures from task-introduced regressions using the recorded baseline.
- Fix failures caused by the change within the approved scope.
- If the fix requires scope, architecture, schema, dependency, or security changes beyond approval, stop and request approval.
- Never disable, skip, weaken, or delete a test solely to obtain a green result.
- If a test cannot be run because of missing secrets, unavailable hardware, network access, or environment limitations, state that clearly and provide the exact reason.
- Do not claim full verification when only a subset was run.

### 5.3 Definition of done

A task may be marked complete only when:

- The requested outcome and acceptance criteria are satisfied.
- The implementation follows approved specifications and the approved plan.
- No unapproved scope, architecture, dependency, schema, or API change was introduced.
- Relevant automated checks pass, or limitations are explicitly documented.
- The final diff has been reviewed.
- Documentation and tests are updated where behavior or contracts changed.
- No known critical security or data-integrity regression remains.
- The handoff accurately describes what changed and what was verified.

---

## 6. Handoff requirements

Every completed task must end with a concise, evidence-based handoff. Use this format:

### Task summary
- **Status:** Complete / Partially complete / Blocked
- **Objective:** One-sentence description of the requested outcome.
- **Result:** What now works or what was documented.

### Files changed
| File | Change | Reason |
|---|---|---|
| `path/to/file` | Added / Modified / Deleted | Short explanation |

List only files actually changed.

### Implementation notes
- Important design or behavior decisions.
- Compatibility, migration, or configuration notes.
- Any approved deviation from the original plan.

### Verification
| Check | Command or procedure | Result |
|---|---|---|
| Lint/type check | Exact command | Passed / Failed / Not run |
| Tests | Exact command | Passed / Failed / Not run |
| Build | Exact command | Passed / Failed / Not run |
| Security/manual checks | Exact procedure | Passed / Failed / Not applicable |

Do not use “passed” unless the command or procedure was actually executed and its result observed.

### Risks and follow-up
- Remaining limitations, known defects, unverified scenarios, or deferred work.
- Any required human decision.
- Any migration or user action needed.

### Git status
- **Branch:** Exact branch, if verified.
- **Commit:** Exact commit hash and message, if committed.
- **Push:** Confirmed / Not requested / Not performed.
- Never imply that a change was pushed merely because a commit was created locally.

### Suggested commit message
When a commit was not requested or not created, provide a concise proposed commit message. Do not create a commit unless authorized.

---

## 7. Escalation and mandatory halt conditions

The agent MUST stop the affected work immediately and contact the project owner when any of the following occurs:

1. **Conflicting requirements:** PRD, scope, architecture, security, database, execution, API, or design documents disagree.
2. **Missing approval:** a required approval gate has not been explicitly satisfied.
3. **Scope uncertainty:** the request may be deferred or out of scope, or acceptance criteria are ambiguous.
4. **Architecture ambiguity:** module ownership, execution responsibility, state authority, or integration boundary is unclear.
5. **Schema uncertainty:** a change appears to require an undocumented table, relationship, migration, destructive data operation, or persistence behavior.
6. **Security uncertainty:** safe handling of credentials, permissions, paths, process execution, untrusted content, or network exposure cannot be established.
7. **Missing secrets or access:** required credentials, permissions, provider access, signing material, or environment configuration are unavailable. Never ask the user to paste secrets into chat or source files.
8. **Unexpected database behavior:** corruption, failed migration, constraint violations, unexpected data loss, locking anomalies, or an unexplained persistence error occurs.
9. **Uncertain external side effect:** a provider/tool/process may have completed an operation but the result is unknown. Do not retry blindly.
10. **Unexpected destructive behavior:** a command or migration may delete, overwrite, publish, or expose user data.
11. **Pre-existing user changes:** the requested edit conflicts with uncommitted or untracked user work and cannot be safely isolated.
12. **Dependency conflict:** the required implementation appears to need an unapproved dependency, framework substitution, or runtime change.
13. **Verification gap:** a critical test cannot be run or a security/data-integrity assertion cannot be established.
14. **Environment mismatch:** the current OS, runtime, database, or toolchain differs materially from the supported project target.
15. **Potential secret or data exposure:** a credential, private prompt, personal data, or project content may have been exposed.
16. **Tooling or repository uncertainty:** a requested Git operation, deployment, or external action cannot be performed with a verified target and safe authorization.

### 7.1 Escalation format

Report:

- **Halt reason:** the exact condition encountered.
- **Evidence:** file, symbol, command, error, or conflicting statements.
- **Impact:** what could go wrong if work continued.
- **Options:** safe alternatives, including the recommended option.
- **Decision required:** one precise question for the project owner.

Do not continue implementation in the affected area while awaiting a decision. Safe, independent investigation may continue only if it cannot modify state, expose data, or pre-empt the decision.

### 7.2 Security incident response

If a secret, private data, or security boundary may have been compromised:

- Stop the affected operation.
- Do not copy the sensitive value into the handoff.
- Preserve only sanitized evidence.
- Follow `SECURITY.md` incident-response requirements.
- Ask the owner to revoke/rotate credentials when appropriate.
- Do not resume the affected workflow until uncertainty is resolved.

---

## 8. Documentation and specification maintenance

When an approved change alters product behavior, architecture, dependencies, schema, API, execution semantics, or security posture:

1. Identify the authoritative document(s) affected.
2. Obtain approval for the change where required.
3. Update the specification as part of the same task or before implementation when it is a prerequisite.
4. Keep examples, names, version numbers, and cross-references consistent.
5. Update tests and implementation to match the specification.
6. Report any unresolved discrepancy.

Do not create duplicate specifications under alternate names. The canonical API contract is `API_SPECIFICATION.md`; use that name rather than creating `API.md` unless the project owner explicitly approves a rename.

If `AGENTS.md` is later introduced, keep its repository-level instructions consistent with this SOP. It should point agents to this document and the domain specifications rather than duplicating and diverging from them.

---

## 9. Standard task lifecycle

The mandatory lifecycle is:

1. **Intake** — understand the request and acceptance criteria.
2. **Load context** — read applicable repository instructions and specifications in the prescribed order.
3. **Inspect** — understand the current implementation, tests, Git state, and baseline.
4. **Classify** — determine scope, risk, and approval requirements.
5. **Plan** — present implementation steps, impact, risks, and verification.
6. **Obtain approval** — wait at mandatory human approval gates.
7. **Implement** — make the smallest approved incremental changes.
8. **Verify** — run relevant static checks, tests, builds, and security reviews.
9. **Review** — inspect the complete diff and confirm acceptance criteria.
10. **Handoff** — report files, decisions, tests, limitations, and Git status accurately.

An agent must return to the planning/approval step if implementation reveals a material change in scope, risk, architecture, dependencies, or data behavior.

---

## 10. Final operating rule

The purpose of an AI coding agent in this repository is to deliver **small, correct, secure, testable, and reviewable changes** within the approved product boundary.

Do not optimize for writing the most code or completing a task at any cost. Optimize for preserving project integrity and making progress that the human owner can understand and verify.

**When uncertain, stop and ask. When approval is required, wait. When a check fails, report it. Never invent evidence, silently expand scope, or weaken a security or data-integrity control.**
