# AI Workflow Orchestrator — Dashboard Design System

**Document:** Product UI / UX Design Specification  
**Version:** 1.0  
**Status:** Proposed  
**Scope:** Desktop dashboard and core application shell  
**Platform:** Windows-first desktop application; Tauri 2  
**Frontend:** TypeScript  
**Backend:** Python + FastAPI  
**Design direction:** Premium dark engineering workspace  
**Related documents:** `prd.md`, `technical-architecture.md`

---

## 1. Design vision

Design AI Workflow Orchestrator as a premium engineering operations console: calm, precise, information-dense, and trustworthy. It should feel like a serious desktop tool for controlling long-running AI work—not a generic chatbot, analytics template, or neon cyberpunk dashboard.

The interface must make five things immediately understandable:

1. **What is happening?** Active workflows, task states, progress, and current blockers.
2. **Why is it happening?** Workflow structure, dependencies, model/tool activity, and provenance.
3. **Can I trust the result?** Validation outcomes, approvals, retries, and evidence.
4. **What did it consume?** Token, context, latency, and estimated cost measurements with clear provenance.
5. **Where is the work?** Project workspaces, conversations, files, artifacts, decisions, and graph relationships.

The UI is a control and observability surface. It must never become the authority for workflow state, permissions, approvals, or execution policy. The backend remains authoritative as specified in the technical architecture.

### Product character

- **Technical:** clear labels, precise states, timestamps, identifiers, and useful diagnostics.
- **Controlled:** consequential actions are explicit and reviewable.
- **Quietly premium:** restrained color, strong typography, clean spacing, subtle borders.
- **Data honest:** reported, estimated, imported, and unavailable values are visibly distinct.
- **Local-first:** communicate local storage and external data transmission without alarmist language.
- **Progressive:** show essential information first; reveal deeper execution details on demand.

### Avoid

- Neon glow, excessive gradients, glassmorphism, and glowing borders.
- Generic AI sparkle motifs, robot illustrations, or decorative circuit backgrounds.
- Large empty hero sections that waste desktop space.
- KPI cards that show numbers without context or useful actions.
- Fake real-time data, fabricated usage values, or misleading progress.
- Color as the only indicator of state.
- An unrestricted node-canvas builder in the MVP.

---

## 2. Information architecture

### Primary navigation

Use a persistent left sidebar with these destinations:

| Navigation | Purpose |
|---|---|
| Overview | Global operational dashboard |
| Projects | Project workspaces and context |
| Workflows | Definitions, templates, and workflow runs |
| Activity | Unified AI, model, tool, and file activity |
| Conversation Library | Imported and captured cross-platform conversations |
| Usage & Costs | Token, context, latency, and cost analytics |
| Approvals | Actions waiting for user authorization |
| Providers & Tools | Provider health, model catalog, integrations, and tool permissions |
| Execution History | Searchable run and task history |
| Settings | Application, privacy, storage, security, and appearance |

**MVP navigation rule:** destinations may be present but marked as planned only when their implementation is not available. Do not render dead controls that appear functional.

### Application shell

- Left sidebar: fixed width 232 px, collapsible to 68 px.
- Top application bar: 56 px high.
- Main content: fluid, maximum comfortable reading width where appropriate.
- Right inspector: optional 320–380 px panel for selected workflow, task, graph node, conversation, or usage record.
- Toasts: bottom-right, non-blocking, short-lived for non-critical feedback.
- Critical execution and approval states remain visible until resolved.

### Sidebar structure

Top:
- Product mark and name: **Orchestrator**
- Workspace/project switcher

Navigation:
- Overview
- Projects
- Workflows
- Activity
- Conversations
- Usage

Operations:
- Approvals with pending count
- Execution History

Bottom:
- Provider health summary
- Local runtime status
- Settings
- User profile is not required for the local-first MVP

The current project context should be visible in the top bar and should be preserved when navigating between project-scoped views.

---

## 3. Visual design system

### 3.1 Color palette

Use a neutral graphite foundation with a restrained electric-violet accent. The accent is for primary actions, selected navigation, focus, and active visualization—not for every card or border.

| Token | Hex | Usage |
|---|---|---|
| `--bg-app` | `#080B12` | Main application background |
| `--bg-sidebar` | `#0A0E17` | Sidebar background |
| `--bg-topbar` | `#0B101A` | Top bar |
| `--surface-1` | `#101622` | Main cards and panels |
| `--surface-2` | `#151D2B` | Hovered/raised surfaces |
| `--surface-3` | `#1B2636` | Selected rows, nested panels |
| `--border-subtle` | `#202B3B` | Default borders |
| `--border-strong` | `#334155` | Emphasis and separators |
| `--text-primary` | `#E8EDF5` | Main text |
| `--text-secondary` | `#A1ADBD` | Secondary text |
| `--text-muted` | `#718096` | Metadata and low emphasis |
| `--accent` | `#8B7CFF` | Primary accent |
| `--accent-hover` | `#A397FF` | Hover accent |
| `--accent-soft` | `#211D3B` | Selected/active background |
| `--success` | `#35B887` | Verified success |
| `--warning` | `#E5A84B` | Waiting, caution |
| `--danger` | `#E16B75` | Failed, destructive |
| `--info` | `#65A9E8` | Informational |
| `--local` | `#35B887` | Local execution indicator |
| `--hosted` | `#8B7CFF` | Hosted provider indicator |

**Color rules**
- Use accent violet sparingly; avoid full-card accent fills.
- Status colors must always be paired with text labels or icons.
- Do not use green to imply a task is complete unless validation passed; distinguish `Succeeded` from `Verified`.
- Use neutral gray for unknown or unavailable measurements.
- Provider type (local/hosted) should use a small label, not a large colored badge.
- Ensure text and controls meet WCAG 2.1 AA contrast targets where practical.

### 3.2 Typography

Preferred type stack:
- UI: Inter, with system sans-serif fallback.
- Technical values and IDs: JetBrains Mono or system monospace.
- Do not require remote font loading; bundle fonts or use local/system fallback.

| Style | Size | Weight | Use |
|---|---:|---:|---|
| Page title | 24 px | 600 | Main page heading |
| Section title | 16 px | 600 | Card and panel headings |
| Subsection | 14 px | 600 | Table groups and inspectors |
| Body | 13 px | 400 | Standard interface text |
| Dense metadata | 12 px | 400 | Timestamps, labels, descriptions |
| Technical value | 12–13 px | 500 | IDs, model names, token values |
| KPI value | 26–30 px | 600 | Dashboard metrics |

Use sentence case. Avoid oversized marketing typography inside the application.

### 3.3 Spacing, radius, and elevation

Use a 4 px spacing scale:
- 4 px: icon/text micro-gap
- 8 px: compact control spacing
- 12 px: dense content spacing
- 16 px: standard component padding
- 20–24 px: section separation
- 28–32 px: major page separation

Radius:
- Inputs and compact controls: 6 px
- Cards and panels: 10 px
- Modals and large overlays: 12 px
- Pills: fully rounded only for compact state labels

Borders are the primary separation mechanism. Use shadows sparingly and never use glow effects as decoration.

### 3.4 Iconography

Use one consistent outline icon family (Lucide-style). Default icon size 16 px in controls and 18 px in navigation. Avoid mixing filled and outline icon families. Icons supplement labels; they do not replace critical text.

---

## 4. Dashboard — Overview

### Purpose

Give the user a fast, actionable view of the system and current project operations. The first viewport should answer: active work, items requiring attention, recent outcomes, provider availability, and usage.

### Desktop layout

At 1440 × 900, use a 12-column grid with 16 px gutters and 24 px page padding.

**Header row**
- Page title: `Overview`
- Supporting text: “Monitor workflows, providers, and project activity.”
- Current project selector or “All projects”
- Date/time range filter for analytics only
- Primary action: `New workflow`
- Secondary action: `New project`

**Operational summary row**
Four compact cards:
1. Active workflows
2. Pending approvals
3. Completed and verified runs
4. Provider availability

Each card includes:
- Current value
- Short descriptive label
- Optional comparison only when a valid comparison period exists
- Click-through to the relevant filtered view

Do not invent percentage changes or display placeholder numbers as real data.

**Main content grid**
- Left 8 columns: Active workflows and recent execution activity.
- Right 4 columns: Provider status and usage snapshot.
- Lower row: Recent projects and recent artifacts.

### Dashboard components

#### Active workflows panel

Each row shows:
- Workflow name
- Project name
- Current state
- Progress as completed/total tasks
- Current task or waiting reason
- Last update time
- Overflow actions

Use a thin progress bar with a textual value, e.g. “3 of 5 tasks”. If execution is waiting for approval, show the approval state instead of suggesting continuous progress.

#### Attention required panel

Prioritize:
- Pending approvals
- Workflows needing attention
- Failed validation
- Provider unavailable when a queued workflow depends on it
- Recovery reconciliation required

Each item includes severity, concise reason, affected workflow/task, timestamp, and a direct action. Do not hide failures in a generic notification feed.

#### Provider status panel

For each configured provider:
- Provider name and local/hosted designation
- Health state: Available, Unavailable, Not checked, or Configuration required
- Model count where known
- Last health check
- Test connection action

A successful health check must not imply that every model or operation is available.

#### Usage snapshot

Show:
- Input tokens
- Output tokens
- Total tokens
- Estimated cost only when pricing data exists
- Measurement provenance (reported / estimated / imported / unavailable)
- Time range

Provide a link to Usage & Costs for detailed analysis. Never combine incompatible provider usage categories into an unqualified total.

#### Recent activity

Display a chronological stream of meaningful events:
- Workflow started/completed/failed
- Task attempt started/retried
- Approval requested/decided
- Provider request completed
- Artifact created
- Conversation imported
- Project file changed by an approved operation

Each activity item links to its source record. Show event time and capture time separately when they differ.

---

## 5. Projects workspace

### Projects list

Use a table or compact card/list hybrid. Each project row contains:
- Name and description
- Working directory (truncated, copyable)
- Active workflow count
- Last activity
- Storage/health indicators only when measurable
- Last opened time

Actions:
- Open project
- Create project
- Import project context
- Open working directory
- Project settings

Creation flow should be a short, focused modal:
1. Project name
2. Optional description
3. Existing directory or create new workspace directory
4. Review path and permissions
5. Create project

Never silently move, overwrite, or recursively modify an existing directory.

### Project overview

Project header:
- Project name and description
- Local path
- Project status
- Quick actions: New workflow, Import conversation, Add context

Tabs:
- Overview
- Workflows
- Activity
- Conversations
- Project Graph
- Files & Artifacts
- Context & Decisions
- Usage
- Settings

Overview content:
- Active workflows
- Recent execution history
- Recent conversations
- Pinned requirements and decisions
- Latest artifacts
- Project-level usage snapshot

The project workspace is the organizing boundary for related work, but must preserve source provenance and avoid copying canonical source files unnecessarily.

---

## 6. Workflow management and execution

### Workflow list

Columns:
- Workflow
- Project
- Version
- Latest run state
- Last execution
- Updated
- Actions

Filters:
- Project
- State
- Template
- Date range
- Provider/model (where applicable)

Primary action: `Create workflow`.

### Workflow definition editor

MVP should use a structured editor or form-based builder, not a complex freeform canvas.

Sections:
1. Workflow identity and description
2. Task sequence and dependencies
3. Executor/provider assignment
4. Inputs and output references
5. Validation requirements
6. Retry and timeout policy
7. Approval policy
8. Review and activate

Show a compact dependency preview. The workflow definition remains authoritative for dependency semantics.

### Workflow run detail

Header:
- Workflow name and run ID
- Project
- State
- Started / elapsed time
- Actions: Pause (if supported), Cancel, View logs

Main layout:
- Left: task execution list / dependency view
- Center: selected task details
- Right: execution inspector or collapsible details

Task row:
- Task name and order
- State label
- Attempt number
- Executor/provider/model
- Duration
- Validation outcome
- Dependency state

Task detail tabs:
- Overview
- Inputs
- Output
- Attempts
- Logs
- Validation
- Tool activity

Display the state machine states consistently:
- Draft
- Ready
- Running
- Waiting for approval
- Paused
- Succeeded
- Failed
- Cancelled
- Needs attention

For tasks, also support Pending, Retry wait, and Skipped.

Do not use color alone to encode states. A completed task with failed validation must not appear as verified success.

### Approval experience

Approval queue should be an explicit operational destination.

Each approval request shows:
- Requested action
- Workflow and task
- Target resource (file, command, Git operation, publication, etc.)
- Exact scope and parameters
- Risk explanation
- Requested time
- Whether the request has expired or changed

Actions:
- Approve this exact action
- Reject
- Inspect details

Approval must be action-scoped. If the action parameters change, require a new approval. Never use a broad “approve all future actions” control in the MVP.

---

## 7. Project graph visualization

### Purpose

Provide a navigable visual map of the project's work history and relationships. The graph is for exploration and provenance—not a replacement for workflow execution state or the workflow editor.

### Node types

- Project
- Conversation
- Message / prompt
- Model request
- Provider / model
- Workflow definition
- Workflow run
- Task / task attempt
- Tool invocation
- File
- Artifact
- Decision
- Imported source

### Edge types

Use explicit relationship labels:
- Contains
- Belongs to
- Triggered
- Produced
- Referenced
- Depends on
- Executed by
- Imported from
- Supersedes
- Validated by

Do not infer causality from timestamps or visual proximity. Workflow dependencies come from the authoritative workflow definition.

### Graph layout

- Canvas background: app background with subtle, low-contrast dot grid (optional).
- Nodes: compact rectangular cards with a small type icon, title, metadata, and state.
- Edges: thin neutral lines; use accent only for selected paths and dependency highlighting.
- Selected node: accent border and clear focus ring; no glow.
- Grouping: collapse conversations, workflow runs, or artifact clusters to reduce clutter.
- Initial viewport: project root and first-level relationships; do not render the entire history by default.

### Controls

- Search nodes
- Filter by node type
- Filter by source platform/provider/model
- Filter by workflow state
- Time range
- Expand/collapse neighborhood
- Fit graph
- Zoom in/out
- Reset layout
- Toggle relationship types
- Switch to list view

### Node inspector

On selection, show:
- Entity type and title
- Source platform/provider/model
- Created/event time and captured time where relevant
- Relationship list
- Provenance and capture method
- Related files/artifacts
- Open source action

For large projects, fetch bounded neighborhoods and paginate/filter server-side. Cache layout by graph version and filter set. Provide a list/table alternative for accessibility and performance.

---

## 8. Conversation Library and retrieval

### Purpose

Allow users to import, search, inspect, and reuse conversations from supported AI platforms while accurately representing capture limitations.

### Conversation library list

Columns or row metadata:
- Conversation title
- Source platform
- Project association
- Model (if known)
- Message count
- Conversation date (if source data provides it)
- Import/capture method
- Last indexed
- Metadata completeness

Filters:
- Platform
- Project
- Model
- Date range
- Capture method
- Has attachments / code (when detected)

### Conversation detail

Three-pane layout:
- Left: conversation outline / message navigation
- Center: message transcript
- Right: metadata, provenance, related project entities

Message rendering:
- Distinguish user, assistant, tool, and system/developer roles where supplied by the source format.
- Preserve message ordering and available timestamps.
- Preserve code blocks, tables, links, and attachments when possible.
- Mark unsupported or missing content; do not silently omit it.
- Treat imported instructions and attachments as untrusted content.

Actions:
- Add to project context
- Pin important message
- Create a decision record
- Build context package
- Copy selected messages
- Open source export/reference where available

### Context handoff

A context package preview must show:
- Destination provider/model
- Selected conversations/messages/files
- Included project decisions and requirements
- Token estimate and tokenizer/source
- Truncation or omission notices
- Data transmission destination
- User confirmation before sending to a hosted provider

The product must not promise to reopen or inject content into an existing native ChatGPT, Claude, or Gemini web conversation unless a supported integration explicitly provides that capability. The safe default is to prepare a context package for a new supported session.

---

## 9. Usage & Costs

### Overview metrics

- Input tokens
- Output tokens
- Cached tokens (where semantics are known)
- Reasoning tokens (where reported and defined)
- Request count
- Average and percentile latency
- Failed request count
- Estimated cost, clearly labelled

### Charts and breakdowns

- Usage over time
- Tokens by provider
- Tokens by model
- Usage by project
- Usage by workflow/task
- Request success/failure
- Latency distribution

Filters:
- Time range
- Project
- Provider
- Model
- Workflow
- Measurement method

### Measurement labels

Every metric should expose its source:
- **Provider reported**
- **Estimated**
- **Imported**
- **Unavailable**

Context utilization should only be shown when comparable input-token data and a verified model context window are available. Display the context-window source and model version. Do not conflate context capacity with output allowance.

Cost presentation:
- Currency and pricing basis
- Pricing record/version and effective date
- Input/output/cached categories where applicable
- Estimated vs provider-reported billing
- Explicit notice that taxes, credits, discounts, subscriptions, or invoice totals may be excluded

Raw per-request usage is the source of truth; dashboard aggregates are rebuildable.

---

## 10. Providers & Tools

### Provider cards/list

Show:
- Provider name
- Local or hosted
- Connection state
- Configured models
- Capabilities (streaming, tool calling, structured output, usage reporting) when verified
- Last health check
- Credential status without revealing the credential
- Configure / test connection

Never display API keys, even partially, in the UI after initial entry. Use OS credential storage and backend-mediated access.

### Tool integrations

Each tool entry shows:
- Tool identity and version
- Source (built-in, MCP, external)
- Granted capabilities
- Allowed project roots
- Approval policy
- Availability
- Recent invocation history

The UI must not imply that hiding a button is an authorization control. Backend policy enforcement is authoritative.

---

## 11. Global interaction patterns

### Search

Global search should search:
- Projects
- Workflows and runs
- Conversations and messages
- Files and artifacts
- Decisions
- Activity events

Results must indicate entity type and source. Apply project scope and permission checks before returning results.

### Filters

- Filters should be composable and visible.
- Show active filters as removable chips.
- Provide a clear-all action.
- Persist useful filters per view where appropriate.
- Avoid excessive filter controls on small screens.

### Tables

- Sticky header for long tables.
- Sortable columns only when sorting semantics are clear.
- Server-side pagination for large histories.
- Row selection and bulk actions only where bulk operations are safe.
- Use monospace for IDs and token counts.
- Provide empty, loading, and error states.

### Inspectors

Use a right-side inspector for contextual details without navigating away. Inspector content must be independently addressable through a full detail view for deep inspection.

### Feedback and destructive actions

- Use inline confirmation for low-risk actions.
- Use a confirmation dialog for delete, overwrite, revoke, cancel, and destructive tool operations.
- Explain the target and consequence precisely.
- Do not use ambiguous labels such as “Continue” when a specific action is being authorized.

---

## 12. Responsive and window behavior

The first release targets desktop. Prioritize usable layouts from 1280 px width upward.

- At 1280 px: collapse secondary panels and allow sidebar collapse.
- At 1440 px: use the full dashboard grid.
- At 1920 px: cap content width or increase column spacing; do not stretch text lines excessively.
- At narrow window widths: convert multi-pane views to tabs/drawers and retain all critical actions.
- Support keyboard navigation and visible focus.
- Respect OS window scaling and high-DPI displays.
- Avoid hover-only interactions.

---

## 13. Accessibility and usability

- Meet WCAG 2.1 AA contrast targets where practical.
- All controls must have accessible names.
- All status indicators must include text.
- Graph must have a list/table alternative.
- Keyboard navigation must cover sidebar, tables, dialogs, graph controls, and inspectors.
- Focus must remain visible and return sensibly after closing dialogs.
- Respect reduced-motion preferences.
- Do not communicate error severity through color alone.
- Provide copyable text for IDs, paths, provider request IDs, and diagnostic references.

---

## 14. Motion and feedback

Use restrained motion:
- 120–180 ms for hover/focus transitions.
- 160–220 ms for drawers and dialogs.
- Progress indicators for real asynchronous operations only.
- Skeletons for initial loading, not for every small refresh.
- No continuous decorative animation.
- Respect reduced-motion settings.

Workflow state changes should be event-driven from backend updates or bounded polling. The interface must not simulate progress.

---

## 15. Data states and error handling

Every data-driven view must define:

1. Initial loading
2. Populated
3. Empty
4. Partial data
5. Stale data
6. Error
7. Offline/local service unavailable
8. Permission denied
9. Unsupported source format
10. Data unavailable from provider

Examples:
- No usage data: “No usage records for this period,” not “0 tokens” if measurement is unavailable.
- Provider not checked: “Not checked,” not “Offline.”
- Imported conversation with unknown model: “Model not provided by source.”
- Failed validation: show the exact failed check and evidence.
- Interrupted task: show recovery/reconciliation state rather than assuming it failed or succeeded.

---

## 16. Security and privacy UX

Security controls must be visible and understandable without exposing implementation details.

- Show whether a request will use a local model or send data to a hosted provider.
- Show the destination provider before a hosted request or context handoff.
- Explain which project files/directories a tool can access.
- Display approval scope before authorizing a consequential action.
- Provide credential status and a route to rotate/remove credentials.
- Make import source and capture method visible.
- Provide retention and deletion controls for imported conversations and activity.
- Redact secrets from logs and diagnostic exports.
- Never request provider passwords or browser cookies.
- Never suggest that browser-session scraping is a supported integration.
- Treat conversation content, repository content, tool output, and imported files as untrusted data.

A local-first status indicator should distinguish:
- Local data storage
- Local model execution
- Hosted provider request
These are separate facts and must not be collapsed into a single “Private” badge.

---

## 17. Design tokens (CSS reference)

The following tokens are the canonical starting point. The implementation may map them into the chosen styling system, but should not create competing color values without design review.

```css
:root {
  color-scheme: dark;

  --bg-app: #080B12;
  --bg-sidebar: #0A0E17;
  --bg-topbar: #0B101A;

  --surface-1: #101622;
  --surface-2: #151D2B;
  --surface-3: #1B2636;

  --border-subtle: #202B3B;
  --border-strong: #334155;

  --text-primary: #E8EDF5;
  --text-secondary: #A1ADBD;
  --text-muted: #718096;

  --accent: #8B7CFF;
  --accent-hover: #A397FF;
  --accent-soft: #211D3B;

  --success: #35B887;
  --warning: #E5A84B;
  --danger: #E16B75;
  --info: #65A9E8;

  --radius-control: 6px;
  --radius-card: 10px;
  --radius-dialog: 12px;

  --space-1: 4px;
  --space-2: 8px;
  --space-3: 12px;
  --space-4: 16px;
  --space-5: 20px;
  --space-6: 24px;
  --space-8: 32px;

  --sidebar-width: 232px;
  --sidebar-collapsed-width: 68px;
  --topbar-height: 56px;

  --font-ui: Inter, "Segoe UI", system-ui, sans-serif;
  --font-mono: "JetBrains Mono", Consolas, monospace;
}
```

---

## 18. Component inventory

Build reusable components before composing pages:

**Application shell**
- AppSidebar
- WorkspaceSwitcher
- AppTopbar
- Breadcrumbs
- GlobalSearch
- ConnectionStatus

**Data display**
- MetricCard
- StatusLabel
- ProviderLabel
- MeasurementSourceLabel
- DataTable
- ActivityFeed
- EmptyState
- ErrorState
- LoadingSkeleton
- Timestamp
- CopyableIdentifier

**Workflow**
- WorkflowList
- WorkflowState
- TaskRow
- DependencyPreview
- RunProgress
- AttemptTimeline
- ValidationResult
- ApprovalRequestCard
- ExecutionLogViewer

**Project and retrieval**
- ProjectCard / ProjectRow
- ProjectTabs
- ConversationList
- ConversationTranscript
- MessageRoleLabel
- ProvenancePanel
- ContextPackagePreview
- ArtifactList
- DecisionRecord

**Graph and analytics**
- ProjectGraphCanvas
- GraphNode
- GraphEdgeLegend
- GraphFilterBar
- GraphNodeInspector
- UsageSummary
- UsageTimeSeries
- ProviderBreakdown
- CostEstimateDetails

**Forms and overlays**
- Modal
- Drawer
- ConfirmActionDialog
- FormField
- Select
- DateRangePicker
- Toast
- Tooltip

All components must support loading, empty, error, and disabled states as applicable.

---

## 19. MVP screen delivery order

Build in this order to keep the UI aligned with the implementation roadmap:

### UI-01 — Application shell
- Sidebar, topbar, routing, theme tokens, keyboard focus.
- Local service connection state.

### UI-02 — Overview dashboard
- Active workflows, attention required, provider health, recent activity.
- Usage snapshot only when usage records exist.

### UI-03 — Projects
- Project list, create/open project, project overview, files/artifacts.

### UI-04 — Workflow definition and execution
- Structured workflow editor.
- Run detail, task states, attempts, logs, validation.

### UI-05 — Approvals
- Approval queue, exact action scope, approve/reject flow.

### UI-06 — Providers and tools
- Provider setup/status, credential status, capability display, tool permissions.

### UI-07 — Activity and conversation import
- Manual import, import validation, conversation library, transcript and provenance.
- Do not imply unsupported automatic capture.

### UI-08 — Project graph
- Bounded graph query, filters, node inspector, list alternative.

### UI-09 — Usage & costs
- Token and request analytics, context metadata, pricing provenance, estimates.

This order follows the PRD and architecture roadmap. Graph and advanced analytics are not blockers for the durable orchestration MVP.

---

## 20. Acceptance criteria

The dashboard design is ready for implementation when:

- [ ] The visual system is consistent across all primary screens.
- [ ] Navigation matches the PRD and does not expose dead controls as functional.
- [ ] Dashboard metrics are sourced from backend records and link to relevant detail views.
- [ ] Workflow/task states match the authoritative backend state model.
- [ ] Waiting, failed, cancelled, and needs-attention states are visually distinct.
- [ ] Approval details show the exact action scope and target.
- [ ] Project graph distinguishes dependency edges from provenance/association edges.
- [ ] Graph has a keyboard-accessible list alternative.
- [ ] Conversation imports show platform, capture method, and metadata completeness.
- [ ] Context handoff previews destination and data being transmitted.
- [ ] Usage distinguishes provider-reported, estimated, imported, and unavailable measurements.
- [ ] Context utilization is hidden or qualified when comparable measurements are unavailable.
- [ ] Cost estimates expose currency and pricing basis.
- [ ] Provider credentials are never displayed or stored in frontend state.
- [ ] Local versus hosted execution is clearly communicated.
- [ ] Empty, loading, stale, offline, partial, and error states are designed.
- [ ] No UI action bypasses backend authorization or workflow state transitions.
- [ ] Desktop layouts work at 1280 px and 1440 px widths.
- [ ] Contrast, keyboard focus, and reduced-motion behavior are reviewed.

---

## 21. Final design principles

1. **Execution before decoration:** prioritize operational clarity over visual effects.
2. **Evidence over claims:** show validation results, provenance, and measurement quality.
3. **Backend authority:** UI state is a projection, never the source of truth.
4. **Explicit control:** approvals and destructive actions must be scoped and understandable.
5. **Progressive disclosure:** show the important state first; expose technical depth on demand.
6. **Honest analytics:** never fabricate tokens, context capacity, costs, or provider health.
7. **Local-first transparency:** distinguish local storage from local inference and hosted transmission.
8. **Performance-aware visualization:** graph and activity views must remain bounded and responsive.

**Design outcome:** a restrained, premium dark engineering workspace that makes AI-assisted work understandable, controllable, recoverable, and verifiable.
