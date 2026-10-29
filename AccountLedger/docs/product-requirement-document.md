# AccountLedger - Product Requirements Document (PRD)

**Document Version:** 1.0  
**Product:** AccountLedger  
**Product Type:** Google Chrome Extension  
**Status:** Proposed - MVP Definition  
**Platform:** Chrome Manifest V3  
**Architecture:** Local-first, privacy-focused  
**Primary Release:** v0.1 MVP

---

## 1. Product Overview

### 1.1 Product Vision

AccountLedger is a privacy-first Chrome extension that helps users maintain a centralized inventory of their online accounts and identify which email address and sign-in method they use for each website or platform.

The product is designed for users who maintain multiple personal, professional, educational, and project-related email identities and need a reliable way to organize their digital accounts.

### 1.2 Product Mission

Provide a simple, searchable, portable, and user-controlled record of online account associations without requiring users to store passwords or grant unnecessary access to their browsing activity.

### 1.3 Problem Statement

Users frequently register for online services using different email addresses and authentication methods. Over time, they lose track of those associations.

Existing solutions may focus on password storage, a specific authentication provider, or retrospective email analysis rather than a unified inventory spanning multiple email identities.

This creates five primary problems:

1. Users forget which email address they used to register for a website.
2. Account information is fragmented across browser bookmarks, password managers, spreadsheets, and memory.
3. Users struggle to identify duplicate or outdated registrations.
4. Account recovery becomes more difficult when the original email or sign-in method is unknown.
5. Users lack a portable inventory for reviewing and organizing their digital footprint.

### 1.4 Product Solution

AccountLedger will provide a browser-based account inventory that enables users to:

- Create and manage multiple email profiles.
- Associate websites with one or more email identities.
- Record usernames and sign-in methods without storing credentials.
- Search, filter, categorize, and update account records.
- Recognize saved websites when revisited.
- Import and export supported account data.
- Maintain control over locally stored information.

AccountLedger will not claim to discover every account associated with an email address automatically.

---

## 2. Target Audience

### 2.1 Primary Users

**Developers and technical professionals**

Users who maintain accounts across GitHub, cloud providers, developer tools, package registries, AI platforms, hosting providers, and documentation services.

Their primary need is a consolidated record of which identity they use for each service.

### 2.2 Secondary Users

**Students and job seekers**

Users managing educational services, coding platforms, certification portals, and job-search websites.

**Freelancers and independent developers**

Users maintaining separate personal, client, business, and project identities.

**Privacy-conscious individuals**

Users who want to audit their online registrations, identify outdated accounts, and organize their digital footprint.

### 2.3 User Personas

**Persona A - Multi-account developer**

- Uses several email addresses.
- Registers for new development platforms regularly.
- Forgets which identity was used for particular services.
- Needs fast lookup and reliable account mapping.

**Persona B - Student or job seeker**

- Uses personal, academic, and professional email addresses.
- Has accounts across learning and recruitment platforms.
- Needs a straightforward, low-maintenance inventory.

**Persona C - Independent professional**

- Separates personal and business services.
- Wants to identify inactive or duplicate registrations.
- Needs portable backups and structured categorization.

---

## 3. Product Goals and Non-Goals

### 3.1 Product Goals

1. Provide a unified inventory of online accounts across multiple email profiles.
2. Reduce the time required to identify the correct email and sign-in method for a website.
3. Enable reliable local storage, backup, restoration, and export.
4. Minimize permissions and avoid unnecessary data collection.
5. Deliver a useful MVP without requiring a backend or mandatory user registration.
6. Establish a foundation for optional account discovery and encrypted synchronization.

### 3.2 Non-Goals

The MVP will not:

- Store passwords, authentication tokens, or session cookies.
- Attempt to discover every account registered with an email address.
- Read Gmail or Outlook mailboxes.
- Automatically create or delete third-party accounts.
- Perform credential stuffing, authentication testing, or account takeover operations.
- Provide breach monitoring or guarantee account security.
- Require cloud synchronization or a user account.
- Include AI-based recommendations.
- Include subscription billing or team administration.

These exclusions are intentional scope controls.

---

## 4. Product Scope

### 4.1 MVP Scope - v0.1

The first release will include:

1. Email profile management.
2. Website account record management.
3. Sign-in method tracking.
4. Account categorization and lifecycle status.
5. Search, filtering, and duplicate detection.
6. Current-website account lookup.
7. JSON import and export.
8. CSV export.
9. Local data persistence.
10. Privacy controls and explicit deletion.
11. Input validation, error handling, and automated tests.

### 4.2 Future Scope

**v0.2 - Account Awareness**
- Optional detection of supported sign-in activity.
- Confirmation prompts for inferred records.
- Improved current-site matching.
- Detection confidence and source tracking.

**v0.3 - Account Audit**
- Inactive-account review workflows.
- Curated account deletion and privacy links.
- Enhanced import and migration tools.
- User feedback and retention validation.

**v1.0 - Optional Advanced Features**
- Optional encrypted cross-device synchronization.
- Email archive import or separately permissioned email integration.
- Advanced account audit reports.
- Additional supported browser integrations.
- Potential premium features, subject to validation.

Future features require separate scope approval and must not compromise the local-first baseline.

---

## 5. Functional Requirements

Priority definitions:

- **P0:** Mandatory for MVP release.
- **P1:** Important enhancement after the core workflow works.
- **P2:** Future consideration.

### FR-01: Email Profile Management

**Priority:** P0

The system shall allow users to create, view, edit, and delete email profiles.

Each profile shall support:

- Unique profile ID.
- Email address.
- Optional display name.
- Optional profile category.
- Creation timestamp.
- Last-modified timestamp.

Profile categories may include Personal, Work, Education, and Projects.

**Acceptance criteria:**
- Users can create multiple email profiles.
- Duplicate normalized email addresses are detected.
- Editing a profile does not unintentionally duplicate its account records.
- Deleting a profile requires confirmation and clearly explains what happens to associated account mappings.
- Invalid email addresses are rejected or flagged appropriately.

### FR-02: Website Account Management

**Priority:** P0

Users shall be able to create, view, update, and delete website account records.

Each record shall support:

- Unique record ID.
- Website domain.
- Service name.
- Associated email profile or profiles.
- Optional username.
- Sign-in method.
- Category.
- Lifecycle status.
- Notes.
- Source and confidence metadata.
- Creation and modification timestamps.

**Acceptance criteria:**
- Users can add an account manually.
- Records remain available after the extension is closed or restarted.
- Users can update or remove records.
- Invalid or unsafe URLs are rejected.
- Website identity is normalized to avoid common duplicate entries.

### FR-03: Sign-In Method Tracking

**Priority:** P0

Users shall be able to record the authentication method used by each account.

Supported initial values shall include:

- Email and password.
- Google.
- GitHub.
- Microsoft.
- Apple.
- Other.
- Unknown.

The system shall store the method label only, not the authentication secret.

**Acceptance criteria:**
- Users can select or edit the sign-in method.
- Unknown is a valid value.
- The application never requests a password for storage.
- Authentication-method labels are not presented as verified facts unless confirmed by the user or a documented detection process.

### FR-04: Account Categorization and Status

**Priority:** P0

Users shall be able to organize accounts using categories and lifecycle statuses.

Initial categories:

- Development.
- Cloud and Infrastructure.
- AI Tools.
- Education.
- Social.
- Productivity.
- Finance.
- Shopping.
- Other.

Initial statuses:

- Active.
- Unused.
- Review.
- Closed.

**Acceptance criteria:**
- Users can change categories and statuses.
- Filters update the displayed records correctly.
- Status changes do not delete account data.
- The application does not infer that an account is unused merely because it has not been visited recently.

### FR-05: Search and Filtering

**Priority:** P0

The application shall provide local search and filtering.

Users shall be able to search by:

- Website or service name.
- Domain.
- Email address.
- Username.

Users shall be able to filter by:

- Email profile.
- Category.
- Sign-in method.
- Lifecycle status.

**Acceptance criteria:**
- Search results update promptly for normal personal inventories.
- Search and filtering work without an internet connection.
- Clearing filters restores the full record list.
- Empty states explain how to add the first account.

### FR-06: Current-Website Lookup

**Priority:** P0

When a user opens the extension on a supported website, AccountLedger shall identify saved records that match the current website domain.

**Acceptance criteria:**
- Matching uses normalized hostnames rather than arbitrary substring comparisons.
- Subdomains are handled according to documented matching rules.
- Similar but unrelated domains are not treated as equivalent.
- Users can view associated email profiles and saved sign-in methods.
- No website data is transmitted to a remote server.

### FR-07: Duplicate Detection

**Priority:** P0

The system shall identify likely duplicate records based on normalized website identity and associated account identity.

**Acceptance criteria:**
- Duplicate candidates are flagged before a conflicting record is created or during import.
- Users can review candidates before merging.
- Records belonging to different email profiles are not automatically merged into one account.
- Duplicate detection never silently deletes user data.

### FR-08: Data Export

**Priority:** P0

Users shall be able to export account records.

Supported formats:

- JSON for full backup and restoration.
- CSV for spreadsheet-based review.

**Acceptance criteria:**
- Exports contain the documented fields.
- CSV values are escaped safely.
- Importable JSON exports include a schema version.
- Export failures are communicated clearly.
- Export operations do not transmit information to external services.

### FR-09: Data Import and Restoration

**Priority:** P0 for validated JSON import; CSV import may be deferred if necessary.

The system shall support restoration from compatible JSON exports.

**Acceptance criteria:**
- Imported data is validated before being committed.
- Unsupported schema versions are rejected with an explanatory error.
- Invalid records are reported without silently corrupting existing data.
- Users can review duplicate conflicts.
- Import operations do not overwrite existing data without explicit approval.
- Failed imports leave the existing inventory unchanged.

### FR-10: Local Data Persistence

**Priority:** P0

The extension shall persist profile and account records locally.

**Acceptance criteria:**
- Data survives normal browser restarts and extension popup closures.
- Storage errors are handled explicitly.
- The UI does not report a successful save until the storage operation succeeds.
- Data access is restricted to the extension's intended functionality.
- The application does not require a remote backend.

### FR-11: Privacy and Data Control

**Priority:** P0

Users shall be able to understand, export, and delete their stored information.

**Acceptance criteria:**
- The extension clearly describes what it stores.
- Users can delete individual records and profiles.
- A full local-data reset requires confirmation.
- The extension does not store passwords, session cookies, or authentication tokens.
- No analytics or remote telemetry is included in the MVP by default.
- Permissions are limited to those required for the implemented features.

---

## 6. Non-Functional Requirements

### NFR-01: Security

- Follow Chrome Manifest V3 requirements.
- Apply strict input validation and safe DOM rendering.
- Avoid unsafe dynamic HTML insertion.
- Prevent imported data from executing as code.
- Request the minimum necessary permissions.
- Never embed API keys or other secrets in the extension package.
- Test authorization boundaries between extension pages and website content.

### NFR-02: Privacy

- Local-first operation shall be the default.
- The core inventory shall work offline.
- The extension shall not collect browsing history as a product feature.
- Website recognition shall not require sending visited URLs to a server.
- Optional future integrations shall require explicit permission and separate disclosure.

### NFR-03: Performance

- The popup should open quickly under normal conditions.
- Search should remain responsive with thousands of account records.
- Large imports shall not block the UI unnecessarily.
- Performance targets shall be verified on supported Chrome versions and representative devices.

### NFR-04: Reliability and Data Integrity

- Use unique identifiers for profiles and account records.
- Validate imports before committing changes.
- Avoid partial updates when an operation fails.
- Handle malformed URLs, invalid records, and storage failures.
- Provide backup and restoration tests before release.

### NFR-05: Compatibility

- Target current stable Google Chrome releases supporting Manifest V3.
- Validate the extension on Windows and at least one additional supported desktop operating system.
- Test extension popup, options page, storage, and permission behavior.

### NFR-06: Maintainability

- Use modular components and clearly defined data contracts.
- Separate UI, application logic, persistence, validation, and browser integration.
- Use automated unit and integration tests.
- Maintain a documented schema migration strategy.

---

## 7. User Experience Requirements

### 7.1 Primary Navigation

The popup shall provide access to:

- Dashboard.
- Accounts.
- Email Profiles.
- Import and Export.
- Settings and Privacy.

### 7.2 Dashboard

The dashboard should display:

- Total account records.
- Number of email profiles.
- Account counts by category or status.
- Recently updated records.
- A primary action to add an account.

### 7.3 Empty States

When no records exist, the extension shall explain the value of creating an inventory and provide a clear Add Account action.

### 7.4 Error States

The UI shall distinguish between:

- Invalid input.
- Duplicate candidates.
- Storage failure.
- Invalid import.
- Unsupported data format.
- Permission unavailable.
- No matching website records.

### 7.5 Accessibility

- Support keyboard navigation.
- Use clear labels and focus indicators.
- Maintain adequate contrast.
- Avoid relying on color alone to convey status.
- Provide accessible names for interactive controls.

---

## 8. Technical Architecture

### 8.1 Proposed Technology Stack

- **Extension standard:** Chrome Manifest V3.
- **Language:** TypeScript.
- **UI:** HTML and CSS with lightweight JavaScript/TypeScript.
- **Persistence:** `chrome.storage.local` for lightweight datasets; IndexedDB if data volume or transaction requirements justify it.
- **Validation:** A runtime schema validation library or equivalent typed validation layer.
- **Testing:** Vitest or an equivalent unit-test framework, plus browser-level integration tests.
- **Build:** Vite or a similarly lightweight build tool.

A framework is optional. Avoid introducing a backend, database server, or complex UI framework without a demonstrated need.

### 8.2 Logical Architecture

The extension shall use five logical layers:

1. **Presentation Layer**
   - Popup.
   - Dashboard.
   - Account editor.
   - Profile manager.
   - Settings and import/export screens.

2. **Application Layer**
   - Account CRUD use cases.
   - Profile management.
   - Search and filtering.
   - Duplicate detection.
   - Import/export orchestration.

3. **Domain Layer**
   - Account and profile entities.
   - Domain normalization.
   - Validation rules.
   - Lifecycle and mapping rules.

4. **Persistence Layer**
   - Local repository.
   - Storage adapter.
   - Import transaction and recovery logic.
   - Schema migrations.

5. **Browser Integration Layer**
   - Manifest and permissions.
   - Current-tab domain retrieval.
   - Optional site matching.
   - Chrome extension lifecycle handling.

### 8.3 Data Relationships

- An email profile may be associated with multiple account records.
- A website may have multiple accounts belonging to different email profiles.
- Account mappings must remain distinct even when the website domain is the same.
- Account records must have stable identifiers independent of their display names.

---

## 9. Privacy and Permissions Strategy

The MVP shall use the narrowest viable permission model.

Potential permissions include:

- `storage` for local persistence.
- `activeTab` where appropriate for user-initiated current-tab recognition.

Host permissions should not be requested broadly unless a specific feature requires them.

The implementation must distinguish between a website's domain and its authentication provider. For example, a user signing in to a service with Google does not mean the service itself is a Google website.

The extension must not infer account ownership from a URL, login button, or browser session alone. Any detection-based association must be explicitly marked as inferred until confirmed.

Before Chrome Web Store submission, review the permissions, privacy disclosures, data-handling practices, and applicable store policies.

---

## 10. Success Metrics

The initial release shall be evaluated using the following proposed targets:

| Metric | Initial target |
|---|---:|
| First five account records entered | Under 3 minutes |
| Onboarding completion | At least 80% |
| Seven-day user return rate | At least 40% |
| Import/export functional test pass rate | 100% |
| Incorrect domain matching in test cases | 0 |
| Critical data-loss defects | 0 |
| Critical privacy defects | 0 |

These are validation targets rather than established industry benchmarks.

### Product Validation

Test with 10–15 users from the target audience over a two-week period.

Evaluate:

- Whether users can add records without assistance.
- Whether they return to look up account information.
- Which fields they actually maintain.
- Whether manual entry is too much friction.
- Whether automatic discovery is a genuine unmet need.
- Whether users trust the local-first privacy model.

Do not expand scope solely because users request more features. Prioritize requests supported by observed usage and recurring problems.

---

## 11. MVP Acceptance and Release Criteria

The MVP shall not be considered release-ready until:

1. All P0 functional requirements are implemented or explicitly deferred through an approved scope change.
2. Account and profile CRUD operations pass automated tests.
3. Search, domain matching, and duplicate detection pass edge-case tests.
4. JSON export and restoration work correctly.
5. Invalid imports cannot corrupt existing data.
6. Local data survives browser restarts.
7. Permission usage matches the documented requirements.
8. No passwords, cookies, tokens, or unintended browsing data are stored.
9. Core workflows pass browser-level integration tests.
10. The extension can be packaged and loaded in Chrome without critical runtime errors.
11. Privacy documentation and user-facing disclosures are complete.

A release candidate must have no unresolved critical security, privacy, or data-integrity defects.

---

## 12. Risks and Mitigations

### Risk 1: Weak Differentiation

Existing browser extensions already track sign-in methods and account-to-site associations.

**Mitigation:** Differentiate through multi-email organization, reliable import/export, account lifecycle management, and a coherent inventory workflow.

### Risk 2: Manual Entry Friction

Users may abandon the product if every account requires excessive data entry.

**Mitigation:** Minimize required fields, provide sensible defaults, and evaluate optional, consent-based detection after the core inventory is validated.

### Risk 3: Inaccurate Automatic Detection

Browser-visible information cannot reliably prove which email owns an account.

**Mitigation:** Maintain source and confidence metadata, request confirmation, and avoid unsupported ownership claims.

### Risk 4: Privacy and Permission Concerns

A browser extension that observes websites may be perceived as invasive.

**Mitigation:** Use least-privilege permissions, local processing, transparent disclosures, and no default telemetry.

### Risk 5: Weak Monetization

Users may prefer free password managers or existing account-memory extensions.

**Mitigation:** Validate retention before introducing paid plans. Test whether encrypted synchronization, account auditing, or team inventory workflows provide meaningful additional value.

---

## 13. Key Product Decisions

The following decisions define the proposed MVP baseline:

- Product name: AccountLedger.
- Primary platform: Google Chrome.
- Extension standard: Manifest V3.
- Architecture: Local-first.
- Initial audience: Developers and multi-account users.
- Core value: Searchable mapping of websites to email identities and sign-in methods.
- Data ownership: User-controlled local inventory.
- Backend: Not required for v0.1.
- Authentication secrets: Explicitly excluded.
- Automatic discovery: Limited to separately approved, supported detection workflows.
- Monetization: Deferred until demand is validated.

---

## 14. Definition of Done

A feature is complete only when:

- Its acceptance criteria are met.
- Validation and error handling are implemented.
- Automated tests cover expected and failure scenarios.
- Data-integrity and privacy implications have been reviewed.
- Documentation is updated.
- The feature works in the packaged Chrome extension, not just in the development environment.
- No critical regression is introduced.

## 15. Final Product Statement

AccountLedger is a privacy-first Chrome extension for managing the relationship between online services, email identities, and sign-in methods.

The MVP focuses on a reliable local inventory, fast lookup, multi-profile organization, duplicate prevention, and portable backups. Optional account discovery and synchronization are future capabilities that depend on demonstrated user demand and a clear privacy model.

The product's success will be measured not by the number of features implemented, but by whether users repeatedly rely on AccountLedger to answer a simple question: **Which account did I use for this website?**
