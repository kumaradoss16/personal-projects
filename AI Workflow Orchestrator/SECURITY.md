# Security Specification — AI Workflow Orchestrator

**Document:** `SECURITY.md`  
**Version:** 1.0  
**Status:** Proposed — binding after project-owner acceptance  
**Target:** MVP v1.0  
**Deployment:** Windows-first, local-first desktop application  
**Last reviewed:** 2026-10-30

---

## 1. Purpose, authority, and mandatory AI security checkpoint

This document defines the mandatory security baseline for the AI Workflow Orchestrator: threat model, trust boundaries, authentication, authorization, secret management, input validation, data protection, logging, tool execution, and prohibited practices.

**Mandatory pre-flight:** Every human developer and AI coding agent MUST read this document before:

- Implementing or modifying authentication, authorization, sessions, permissions, or security controls.
- Creating or changing an API endpoint, request model, response model, network listener, or transport.
- Modifying database access, migrations, persistence, backup, or artifact storage.
- Configuring provider adapters, MCP/tool adapters, filesystem access, Git operations, test runners, or autonomous execution.
- Changing process spawning, shell invocation, sandboxing, environment variables, credential storage, logging, or telemetry.
- Adding dependencies that affect cryptography, authentication, parsing, networking, process execution, or security.

Read this document with `prd.md`, `ARCHITECTURE.md`, `SCOPE.md`, `TECH_STACK.md`, `DATABASE.md`, `EXECUTION_ENGINE.md`, and `API_SPECIFICATION.md`. The API contract is named `API_SPECIFICATION.md` in this repository; a reference to `API.md` does not authorize creating a duplicate specification.

If documents conflict, stop and report the exact conflict. Do not silently weaken controls, invent a security model, or implement the easiest interpretation.

### 1.1 Security principles

1. **Local-first is not automatically secure.** Local APIs, files, processes, and model runtimes remain attack surfaces.
2. **Least privilege:** every component receives only the access required for its current operation.
3. **Backend authority:** the backend alone authorizes actions and changes workflow/task state.
4. **Zero trust for generated content:** model output, tool output, imported project content, and external provider responses are untrusted.
5. **Explicit consent:** external data transmission and consequential actions require clear user awareness and, where policy requires, action-scoped approval.
6. **Deny by default:** unknown tools, operations, paths, providers, permissions, and protocol origins are rejected.
7. **Defense in depth:** validation, authorization, isolation, persistence controls, and audit events are independent layers.
8. **Fail closed:** when security checks, policy evaluation, or approval verification fail, the operation does not proceed.
9. **No silent security downgrade:** a failing test or implementation obstacle is never justification for bypassing a control.
10. **Secure defaults:** development convenience must not leak into packaged production configuration.

---

## 2. Threat model and trust boundaries

### 2.1 System context

The MVP is a Windows-first desktop application composed of:

- Tauri 2 native desktop shell and TypeScript frontend.
- Local Python 3.12 + FastAPI backend.
- SQLite database and application-managed local artifact storage.
- Hosted LLM provider APIs and local inference through Ollama.
- Restricted adapters for project-scoped filesystem operations, Git inspection, and configured test execution.

The backend is a local modular monolith. It is not a public multi-tenant service. Future integrations such as MCP, remote workers, browser automation, and collaboration are not implicitly approved by this document or the MVP scope.

### 2.2 Trust classification

| Component / input | Trust level | Required treatment |
|---|---|---|
| Frontend requests and UI state | Untrusted | Authenticate, validate, authorize server-side |
| Tauri IPC messages | Untrusted at the boundary | Validate command names, arguments, origin, and caller context |
| Local loopback HTTP traffic | Untrusted transport input | Session authentication, Host/Origin checks, bounded parsing |
| User-provided prompts and workflow definitions | Untrusted | Schema validation, policy checks, size limits |
| Project files, source code, repository metadata | Untrusted content | Treat as data; do not execute or follow embedded instructions |
| LLM-generated text, plans, code, and tool arguments | Untrusted | Parse against schemas; independently authorize each action |
| Hosted provider responses | Untrusted external data | Validate status, size, schema, encoding, and content |
| Ollama responses | Untrusted local service output | Validate as external output; do not assume local means safe |
| MCP/tool responses if later approved | Untrusted | Capability-scoped adapter boundary and output validation |
| Filesystem paths and symlinks | Untrusted | Canonicalize and enforce approved roots |
| Git repository content/hooks/configuration | Untrusted | Never run hooks or commands implicitly |
| OS process results and environment | Partially trusted | Explicit executable allowlist, bounded output, controlled environment |
| SQLite database and artifact files | Integrity-sensitive | Parameterized queries, transaction boundaries, file permissions, backups |
| Credential store | Sensitive trusted storage | Access only through approved native credential abstraction |

### 2.3 Trust boundaries

1. **Frontend ↔ backend:** Every request crosses an authentication and validation boundary. The frontend is not an authorization authority.
2. **Tauri native layer ↔ OS:** Native commands are privileged. Expose only narrow, typed commands; do not create a generic shell or filesystem bridge.
3. **Backend ↔ provider:** Provider requests may disclose project context externally. Provider responses may contain malicious or malformed content.
4. **Backend ↔ local tools:** Filesystem, Git, and test-runner adapters can affect user data or execute code. Each adapter requires explicit capabilities and bounded execution.
5. **Backend ↔ persistence:** Database and artifact writes must follow the canonical schema, transaction, and storage boundaries.
6. **Workflow engine ↔ model output:** A model may propose actions but cannot approve, authorize, or directly execute them.
7. **Application ↔ project workspace:** Project files are potentially adversarial and must remain inside an explicitly selected project boundary unless separately approved.

### 2.4 Primary threats and mitigations

| Threat | Example | Mandatory mitigation |
|---|---|---|
| Local API abuse | Another local process sends commands | Per-launch bearer token, loopback-only binding, Host/Origin validation |
| Session theft | Token appears in logs or process arguments | In-memory frontend token, private IPC bootstrap, redaction |
| Prompt injection | README instructs agent to exfiltrate secrets | Treat retrieved content as data; policy engine independently authorizes tools |
| Tool abuse | Model requests arbitrary shell command | No generic shell tool; allowlisted executable and argument policy |
| Path traversal | Artifact path escapes project root | Canonical path checks, symlink containment, root-scoped operations |
| Command injection | User/model input becomes shell syntax | No shell interpolation; argument-array process invocation |
| SQL injection | User input concatenated into SQL | Parameterized SQL exclusively |
| Secret exfiltration | Prompt or tool output includes API key | Secret redaction, scoped context, no secret injection into model context |
| Malicious provider response | Oversized or malformed response | Timeouts, response-size limits, schema validation |
| XSS / UI injection | Model output contains HTML/script | Render as text or sanitized Markdown; no raw HTML execution |
| Approval replay | Old approval reused for changed action | Immutable action fingerprint, one-time decision, revalidation |
| Duplicate side effect | Retry repeats external operation | Persist operation identity; idempotency/reconciliation |
| Data tampering | Untrusted process modifies DB/artifacts | OS file permissions, integrity checks, controlled storage |
| Denial of service | Huge prompt/output or endless tool loop | Input/output limits, deadlines, bounded retries and execution budgets |

### 2.5 Threat model maintenance

Reassess the threat model whenever a new provider, tool, integration, network listener, file capability, execution mode, or deployment target is proposed. New attack surfaces require explicit human review before implementation.

---

## 3. Authentication, session management, and authorization

### 3.1 Identity model

MVP is a single-user local desktop application. It does not implement user accounts, password authentication, OAuth, multi-tenant identity, or RBAC roles.

The authenticated principal is the active local application session. This does not imply that every operation is permitted: capability checks, project boundaries, execution policies, and human approval remain mandatory.

Do not introduce user/role tables or an identity provider without an approved scope and architecture change.

### 3.2 Session protocol

Follow `API_SPECIFICATION.md`:

1. The Tauri native layer controls backend process startup.
2. Generate a cryptographically secure, unpredictable, per-process opaque bearer token using an OS-backed cryptographic random source.
3. Deliver it only through the private, approved Tauri IPC/bootstrap channel.
4. Keep it in frontend memory only; do not persist it to disk, browser storage, URLs, command-line arguments, or logs.
5. Send it using `Authorization: Bearer <token>` for protected API calls.
6. Validate it on every protected request using constant-time comparison.
7. Invalidate it when the backend process exits or the session is revoked.
8. Never provide a general unauthenticated token-issuance or token-retrieval endpoint.

Missing, malformed, invalid, or expired tokens return `401` using the documented error envelope. Do not reveal token validity details.

### 3.3 Local API protections

- Bind only to loopback; never bind to `0.0.0.0`, LAN, or public interfaces by default.
- Validate Host and Origin against the expected Tauri application origin and configured loopback origin.
- Do not use wildcard CORS origins or wildcard credential access.
- CORS is not authentication and must not replace bearer-token validation.
- Do not assume an unpredictable port alone provides security.
- Do not expose debug consoles, interactive API documentation, profiler endpoints, or diagnostic endpoints to unauthenticated callers in packaged builds.
- Health endpoints may be unauthenticated only if they expose non-sensitive liveness/readiness information.

### 3.4 Authorization and capability enforcement

No RBAC roles are required for MVP. Enforce authorization through a backend policy layer:

- **Project scope:** every project, workflow, run, task, and artifact access must verify the complete parent-resource relationship.
- **Filesystem scope:** operations are restricted to explicitly selected project roots and application-managed storage roots.
- **Tool scope:** each adapter exposes a finite set of named capabilities; no wildcard capability.
- **Execution scope:** workflow policy is snapshotted for each run and cannot be silently widened mid-run.
- **Approval scope:** approval authorizes one immutable action with a specific target, arguments, and execution context.
- **Provider scope:** provider credentials are accessible only to the relevant provider adapter; never to general workflow prompts or unrelated tools.

The backend must deny unknown operations and missing permissions. The frontend may hide unavailable controls for usability but this is not a security control.

### 3.5 Human approval

Require explicit, action-scoped approval for consequential operations as defined by the execution policy, including filesystem writes, Git mutations, and other irreversible or externally visible actions.

Approval records must bind to:

- Exact action type and adapter.
- Canonical target and bounded arguments.
- Project and workflow-run identity.
- Relevant policy/version context.
- A cryptographic action fingerprint.
- Expiry/validity and one-time decision state.

Before execution, recompute and compare the fingerprint and revalidate authorization and preconditions. Reject stale, modified, expired, duplicated, or already-consumed approvals. A model response is never an approval.

---

## 4. Secret handling and permissions

### 4.1 Secret classification

Treat the following as secrets or sensitive material:

- Hosted provider API keys and access tokens.
- Application session bearer token.
- Any future integration credentials.
- Private keys, signing material, recovery codes, and credential-store references where disclosure enables access.
- Sensitive prompt/context content and private project data, even when not credentials.

### 4.2 Storage

- Store provider credentials in the approved OS credential store through a narrow native abstraction.
- Store only an opaque credential reference and non-secret metadata in SQLite.
- Never store plaintext API keys in SQLite, project files, workflow definitions, logs, crash reports, source control, or ordinary configuration files.
- Do not add a new secrets manager or encryption dependency without approval.
- Application-managed files must use restrictive per-user OS permissions.
- Do not claim that SQLite encryption at rest is enabled unless an approved, implemented encryption mechanism exists. The baseline relies on OS account/file protections and encrypted device storage where available.
- Backups containing sensitive data must receive equivalent access protection. Do not export credentials.

### 4.3 Secret use

- Retrieve a credential only immediately before the provider call that requires it.
- Keep secret lifetime in memory as short as practical; do not retain it in global state or task outputs.
- Never place credentials in prompts, model context, tool arguments, artifacts, environment dumps, exception messages, or response bodies.
- Do not pass all process environment variables to child processes. Construct a minimal allowlisted environment.
- Do not include secrets in URLs, query strings, Git remotes, command-line arguments, or filenames.
- Do not allow model-generated code or shell commands to read the credential store.
- Provider adapters must not expose credential values through debug or diagnostic methods.

### 4.4 Rotation and revocation

- Provide a user-controlled credential replacement/removal flow.
- Revoke or replace credentials at the provider when compromise is suspected.
- Remove the stored credential and invalidate its reference when the user disconnects a provider.
- A credential rotation must not silently rewrite historical execution records; historical records may retain a non-secret credential identifier/version for audit only.
- Do not invent automatic key rotation where the provider does not support it.

### 4.5 Least privilege

- Run the application as the current unprivileged desktop user. Never request administrator privileges for routine operation.
- Do not install a privileged background service for MVP.
- Backend modules receive only required dependencies/capabilities; provider adapters receive only their own credential access.
- SQLite access is confined to the application data directory and the backend persistence layer.
- Frontend code must not access the database file or credential store directly.
- Child processes must run with the current user's permissions, restricted working directory, minimal environment, explicit executable path, bounded resources, and timeout.
- Do not use administrator/root execution to bypass filesystem or process restrictions.

---

## 5. Input validation and data protection

### 5.1 General validation

Validate every input at the backend boundary, regardless of frontend validation.

- Use explicit Pydantic request models and strict types.
- Reject unknown fields for security-sensitive commands.
- Enforce length, range, nesting-depth, collection-size, and body-size limits.
- Normalize identifiers and canonicalize paths before authorization checks.
- Validate enum values against explicit allowlists.
- Reject malformed Unicode, unexpected encodings, invalid content types, and oversized payloads.
- Do not trust MIME types or filenames supplied by clients.
- Avoid unsafe deserialization formats such as pickle for untrusted input.
- Validate provider responses and tool outputs before they enter domain logic or persistence.

### 5.2 SQL injection

- Use parameterized SQL for all values.
- Never concatenate user/model/provider-controlled strings into SQL statements.
- Dynamic table names, columns, sort keys, and SQL fragments must come from fixed internal allowlists.
- Do not expose raw SQL execution endpoints or model-accessible database tools.
- Database schema and queries must follow `DATABASE.md`.
- Test injection payloads against all repository and API query paths.

### 5.3 XSS and content rendering

- Treat prompts, model outputs, Markdown, filenames, logs, and provider content as untrusted display data.
- Render text using safe framework APIs; do not inject untrusted strings as raw HTML.
- If Markdown rendering supports HTML, disable raw HTML or apply a reviewed sanitizer with a strict allowlist.
- Block script execution, event-handler attributes, dangerous URL schemes, and embedded active content.
- Apply a restrictive Content Security Policy appropriate to the Tauri application.
- Do not use `eval`, `new Function`, or dynamic code execution to render content.
- Validate links before opening them externally; do not automatically navigate to model-provided URLs.

### 5.4 Prompt injection and model-output safety

Prompt injection cannot be reliably solved by prompt wording alone. Enforce security in deterministic code:

- Project documents, source files, issue text, provider responses, and tool outputs are data, not policy or instructions.
- Never allow retrieved content to override system policy, tool permissions, approval requirements, or user intent.
- Do not treat model-generated JSON as trusted merely because it parses.
- Validate every proposed tool call against a typed schema, allowlisted operation, project boundary, execution policy, and approval requirement.
- Model output cannot change workflow state, create permissions, approve actions, access credentials, or select arbitrary executable paths.
- Do not place secrets in model context. Redact detected secrets before content is persisted, displayed, or sent to another provider.
- Apply output-size and recursion limits to agent-generated plans and tool-call chains.
- If policy cannot determine whether an action is safe, reject or request human review; never default to allow.

### 5.5 Filesystem and path safety

- Require explicit project selection and a configured project root.
- Canonicalize paths before checking containment; string-prefix checks alone are insufficient.
- Resolve and validate symlinks/reparse points so they cannot escape approved roots.
- Reject path traversal, device paths, alternate data stream abuse, reserved device names, and unexpected path forms on Windows.
- Use safe file APIs and atomic writes where appropriate.
- Do not follow model-provided paths outside approved roots.
- Separate read and write capabilities. Read permission does not imply write permission.
- Do not overwrite, delete, or move user data without the relevant approval policy.
- Artifact download must resolve only application-managed artifact references, never arbitrary local paths.

### 5.6 Process and command execution

- Do not provide an unrestricted shell, terminal, PowerShell, or command-execution tool to the model.
- Prefer fixed, purpose-built adapters for Git inspection and configured test execution.
- Invoke processes with an explicit executable and argument array; never interpolate untrusted input into a shell command string.
- Use a fixed executable allowlist and operation-specific argument validation.
- Set an explicit working directory inside the approved project root.
- Construct a minimal environment; remove inherited secrets and unnecessary variables.
- Enforce timeout, output-size, and cancellation limits.
- Do not execute project scripts, Git hooks, package lifecycle scripts, or downloaded binaries implicitly.
- Require approval for operations that modify files, repository state, or external systems.
- A process timeout does not prove that an external side effect did not occur; reconcile before retrying.

### 5.7 Encryption in transit and at rest

**In transit**
- Hosted provider traffic must use HTTPS with certificate validation enabled.
- Never disable TLS verification or accept invalid certificates as a workaround.
- Local loopback HTTP is permitted only for the protected desktop session model documented in `API_SPECIFICATION.md`; the service must remain loopback-bound and authenticated.
- Do not send credentials or project data to unapproved endpoints.

**At rest**
- Use OS account access controls for the application data directory, SQLite database, and artifact directory.
- Recommend/require full-disk encryption on devices storing sensitive project data where supported by the operating system.
- Keep provider credentials in the OS credential store.
- Do not claim database-level encryption unless explicitly implemented and tested with an approved technology.
- Backups and exports must not weaken the source data's access controls.

### 5.8 External data transmission

- The UI must clearly identify when workflow context is sent to a hosted provider.
- Send only the minimum context required for the task.
- Local Ollama inference must remain local unless the user explicitly configures a remote endpoint and approves that deployment model.
- Do not expose Ollama or the application backend through a public tunnel as part of MVP.
- Never transmit credential values, unrelated project files, or hidden application data to providers.

---

## 6. Logging, telemetry, and audit restrictions

### 6.1 Data that MUST NOT be logged

Under no circumstances may ordinary application logs, debug logs, analytics, telemetry, crash reports, or exception messages contain:

- Plaintext passwords, API keys, access tokens, bearer tokens, session tokens, private keys, or recovery codes.
- Authorization headers, cookies, credential-store contents, or secret-bearing environment variables.
- Full prompt content, private source files, complete model outputs, or arbitrary tool input/output by default.
- Sensitive personal data, payment data, authentication data, or unrelated user data.
- Full filesystem paths where they expose private directory/user information unnecessarily.
- Database connection strings or secret-bearing provider URLs.
- Raw HTTP request/response bodies from providers or clients when they may contain sensitive data.
- Unredacted exception objects from providers, OS processes, or credential APIs.
- Contents of environment variables or process command lines that may contain secrets.

Do not add a “verbose” or “debug” mode that bypasses these restrictions.

### 6.2 Permitted operational logging

Logs may contain only bounded operational metadata needed for diagnosis, such as:

- Correlation/request ID.
- Workflow/run/task/attempt UUIDs.
- Event type and state transition.
- Provider identifier and model name, if non-sensitive.
- Adapter name and operation category.
- Duration, retry count, timeout category, and safe error code.
- Sanitized validation result and artifact identifier.
- Application version and sanitized platform/runtime details.

Use structured logs. Redact sensitive fields before serialization, not after writing. Prefer allowlisted fields over attempting to maintain an exhaustive denylist.

### 6.3 Audit events

Security-relevant events must be recorded using the approved persistence/event model, including:

- Session start/end and authentication failures (without token values).
- Provider added, disabled, credential replaced, or disconnected.
- Approval created, approved, rejected, expired, or invalidated.
- Tool operation requested, authorized, denied, executed, or reconciled.
- Project-root or execution-policy changes.
- Security validation failures and suspicious path/command attempts.

Audit events must not store secrets or full sensitive payloads. Apply the retention policy in `DATABASE.md`; do not create new tables without approval.

---

## 7. Prohibited operations and development practices

The following are prohibited in all production code and must not be introduced as temporary shortcuts:

### 7.1 Code execution and process control

- Using `eval()`, `exec()`, `new Function()`, dynamic imports from untrusted paths, or equivalent arbitrary code execution on user/model-controlled content.
- Running un-sandboxed arbitrary OS commands through agentic tools.
- Exposing generic shell, PowerShell, command prompt, Python REPL, or terminal execution to the model.
- Building shell command strings from user/model/provider input.
- Executing downloaded binaries, repository scripts, package scripts, Git hooks, or project-provided automation without explicit allowlisting and approval.
- Running the application or child processes with administrator privileges by default.
- Allowing a model to select arbitrary executable paths, environment variables, working directories, or process privileges.

### 7.2 Network and browser security

- Binding the local API to `0.0.0.0`, LAN, or public interfaces in MVP.
- Disabling Host/Origin checks or using wildcard CORS to make a request work.
- Disabling TLS certificate validation.
- Exposing bearer tokens in URLs or accepting credentials from query parameters.
- Creating unauthenticated state-changing API endpoints.
- Disabling CSP or permitting unrestricted inline script execution.
- Adding public tunnels, remote access, webhooks, or external listeners without approved scope and threat-model updates.

### 7.3 Data and credential handling

- Storing plaintext credentials in SQLite, JSON, source control, logs, prompts, or artifacts.
- Logging authorization headers, tokens, private prompts, or sensitive provider payloads.
- Returning secrets or unrestricted local paths in API responses.
- Using string-concatenated SQL with untrusted values.
- Deserializing untrusted pickle or equivalent executable object formats.
- Reading/writing/deleting outside approved project and application storage roots.
- Disabling database constraints, foreign keys, or transaction protections to bypass an error.

### 7.4 Authorization and workflow policy

- Treating model output as user consent, authorization, or approval.
- Allowing a model/tool to grant itself permissions or alter policy.
- Bypassing, auto-approving, or weakening a required approval gate.
- Replaying stale approvals or accepting a changed action under an old approval.
- Allowing frontend-only authorization checks.
- Allowing clients to write workflow/task status directly.
- Retrying uncertain external side effects without reconciliation.
- Failing open when the policy engine, credential store, validation, or approval service is unavailable.

### 7.5 Testing and delivery shortcuts

- Disabling authentication, authorization, input validation, CSP, TLS checks, or security tests to make a test pass.
- Marking a failing security test as skipped or weakening its assertion without documented human approval.
- Adding broad exception handlers that hide security failures and continue execution.
- Committing secrets, test credentials, private user data, or production database files.
- Introducing security-sensitive dependencies without review and approval.
- Suppressing dependency/security scanner findings without triage and documented disposition.

---

## 8. Dependency and secure configuration rules

- Follow `TECH_STACK.md`; do not add or substitute frameworks or security libraries without explicit approval.
- Before adding a dependency, document its purpose, maintainer/release health, license, transitive dependencies, vulnerability history, data handling, and maintenance burden.
- Pin resolved versions through approved lockfiles and review dependency updates.
- Do not install packages dynamically at runtime.
- Do not fetch executable code or configuration from untrusted remote locations.
- Keep development-only tooling out of production packages.
- Separate development and packaged production configuration.
- Disable debug mode, reloaders, tracebacks, and development API docs in production builds.
- Use explicit configuration allowlists; reject insecure or unknown security-sensitive settings.
- Never make a security control optional through a casual feature flag or environment variable.

---

## 9. Security verification and release gates

A release is blocked if a critical security invariant is unverified or a known critical/high vulnerability is unresolved without an explicitly approved risk disposition.

Minimum automated verification:

| ID | Security test | Required result |
|---|---|---|
| SEC-001 | Missing/invalid bearer token | Protected API returns 401 |
| SEC-002 | Token leakage scan | No token in logs, URLs, errors, or artifacts |
| SEC-003 | Non-loopback bind attempt | Packaged configuration refuses LAN/public binding |
| SEC-004 | Invalid Host/Origin | Request rejected |
| SEC-005 | Wildcard CORS regression | Configuration test fails |
| SEC-006 | SQL injection payloads | Inputs remain data; no query manipulation |
| SEC-007 | XSS payload rendering | No script execution or unsafe HTML |
| SEC-008 | Prompt injection in project file | Cannot override policy or authorize a tool |
| SEC-009 | Forged tool call | Backend denies non-allowlisted operation |
| SEC-010 | Approval replay | Reused/stale fingerprint rejected |
| SEC-011 | Path traversal | Access outside allowed roots denied |
| SEC-012 | Symlink/reparse escape | Escape from allowed root denied |
| SEC-013 | Shell metacharacters | Never interpreted as shell syntax |
| SEC-014 | Executable allowlist | Unknown executable rejected |
| SEC-015 | Environment isolation | Child process receives only approved variables |
| SEC-016 | Provider secret serialization | Credential never appears in API response |
| SEC-017 | Provider TLS | Invalid certificate is rejected |
| SEC-018 | Oversized request/output | Bounded rejection or truncation policy applied |
| SEC-019 | Database access | Foreign keys and constraints remain enabled |
| SEC-020 | Log redaction | Sensitive fixture values absent from all logs |
| SEC-021 | Approval unavailable | Consequential operation fails closed |
| SEC-022 | Policy engine unavailable | Tool operation is denied |
| SEC-023 | Process timeout | No false success; outcome reconciled |
| SEC-024 | Artifact authorization | Cross-project/run access denied |
| SEC-025 | Dependency review | New dependency has recorded approval |
| SEC-026 | Production config | Debug endpoints and unsafe settings disabled |
| SEC-027 | Secret storage | Credentials absent from SQLite and project files |
| SEC-028 | Backup handling | Backup permissions and restore behavior verified |

Tests must use synthetic credentials and data. Never use real provider keys in automated tests or commit history.

### 9.1 Vulnerability handling

- Triage findings by exploitability and impact, not scanner severity alone.
- Block release for exploitable critical/high issues in exposed attack paths.
- Record accepted risks with affected component, rationale, compensating controls, owner, and review date.
- Do not suppress or downgrade findings solely to achieve a green pipeline.
- Re-run relevant security and regression tests after remediation.

---

## 10. Security incident and recovery expectations

If credential leakage, unauthorized tool execution, data exposure, or suspected tampering occurs:

1. Stop the affected workflow and disable the affected adapter/provider if safe to do so.
2. Revoke/rotate exposed credentials at the issuing provider.
3. Preserve sanitized operational evidence without copying secret-bearing payloads.
4. Identify affected projects, runs, artifacts, and external operations.
5. Reconcile uncertain side effects before retrying.
6. Restore from a verified backup if database or artifact integrity is compromised.
7. Document root cause, scope, remediation, and regression tests.
8. Update this threat model and relevant specifications before re-enabling the capability.

The application must not automatically resume a workflow that has unresolved security-relevant uncertainty.

---

## 11. Security change control

Any security-sensitive change proposal must include:

1. The business requirement and affected workflow.
2. Threats introduced or mitigated and updated trust boundaries.
3. Authentication, authorization, and permission impact.
4. Secret/data handling and logging impact.
5. API, database, execution-engine, and dependency impact.
6. Abuse cases and failure-mode analysis.
7. Tests, release gates, rollback, and recovery plan.
8. Explicit human approval.

After approval, update this document and all affected specifications before implementation. Security-sensitive changes require review of the resulting diff and test evidence.

**AI coding agents must never weaken security controls to make a test pass, bypass a complex implementation, or satisfy a model-generated plan.** If the secure implementation is unclear or blocked, stop, explain the risk and alternatives, and request a human decision.

---

## 12. Final security rule

AI Workflow Orchestrator is a local-first execution system that handles private project data, external provider credentials, and potentially consequential developer operations. Local deployment does not remove the need for authentication, authorization, isolation, validation, auditability, and explicit approval.

**Treat every client request, project file, model response, provider response, and tool output as untrusted. The backend must independently validate and authorize every operation. Deny by default, fail closed, preserve evidence, and never silently weaken a security control.**
