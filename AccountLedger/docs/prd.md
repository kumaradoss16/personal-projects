# Product Requirements Document (PRD)

# AccountLedger

Local-first browser extension for managing email-linked website accounts and account activity

Draft v1.0

Chrome Extension · Manifest V3

Local-first

## 1. Document Overview

| Field              | Details                                                |
| ------------------ | ------------------------------------------------------ |
| Product name       | AccountLedger (working title)                          |
| Document           | Product Requirements Document                          |
| Version            | 1.0                                                    |
| Date               | 24 October 2026                                        |
| Author             | Product owner / project team                           |
| Status             | Draft - pending requirements approval                  |
| Product type       | Google Chrome browser extension                        |
| Primary platform   | Desktop Chrome                                         |
| Architecture       | Local-first, no mandatory backend                      |
| Target release     | MVP followed by v1 and v2                              |
| Estimated delivery | 6–8 weeks for the initial scope, subject to validation |

Project description: A privacy-focused Chrome extension that helps users maintain an organized inventory of websites and services associated with their email addresses, record account creation and authentication activity, and associate recorded events with device profiles and optional network/location metadata.

Core product principle: AccountLedger is an account inventory and activity-tracking tool, not a credential manager and not a universal authentication-monitoring system. Its interface must distinguish confirmed events from inferred events and unavailable information.

## 2. Executive Summary

AccountLedger helps users understand which websites and online services are associated with their email addresses and maintain a centralized inventory of those accounts. It combines manually managed account records with best-effort detection of account creation, sign-in, and sign-out activity on supported websites. Users can review an account activity timeline, associate events with named device profiles, and optionally attach public-IP and approximate location information when technically available and explicitly enabled. By default, account records and activity history remain in local browser storage, reducing dependence on third-party services. The product is designed to improve account visibility and organization without collecting passwords, authentication tokens, cookies, or unrelated browsing history.

## 3. Problem Statement

### 3.1 Current problems

Users often create accounts across many websites using multiple email addresses. Over time, they lose track of which email was used for which service, whether an account was created recently, and which devices were used to access it.

| Problem                                 | User impact                                                                                                          | Product opportunity                                                             |
| --------------------------------------- | -------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| Fragmented account records              | Users forget which services are linked to each email address                                                         | Maintain a searchable account inventory                                         |
| Multiple email identities               | Users cannot easily filter accounts by email                                                                         | Support multiple email profiles                                                 |
| Incomplete activity history             | Users have no convenient personal record of account creation and authentication activity                             | Provide a timestamped event timeline                                            |
| Unclear device attribution              | Users may not remember which device was used for an event                                                            | Associate events with user-defined device profiles                              |
| Limited visibility into network context | Users may want to know the public IP or approximate location associated with an event                                | Support optional, explicitly configured metadata enrichment                     |
| Unreliable automated detection          | Website implementations differ, and authentication may occur through redirects, pop-ups, or single-page applications | Use evidence-based detection and explicit confidence labels                     |
| Privacy concerns                        | Centralized account inventories may expose sensitive behavioral information                                          | Keep data local by default and provide export, retention, and deletion controls |

### 3.2 Opportunity

Create a lightweight extension that combines an account directory with a transparent personal activity ledger. The product should provide immediate value through manual account management even when automatic detection is unavailable.

### 3.3 Core design decision

Account inventory and authentication activity must be treated as separate but related datasets.

- An account record represents a user's relationship with a website or service.
- An activity event represents an observed, inferred, provider-reported, or user-confirmed occurrence.
- An event source describes how the event was obtained.
- A verification status communicates the strength of the evidence.

The extension must never present an inferred sign-in or sign-out as verified fact.

## 4. Goals & Success Metrics

The following targets are proposed product acceptance criteria, not existing performance results. Baselines must be measured during testing.

| ID     | Goal / KPI                 | Proposed target                                                                                         | Measurement                  |
| ------ | -------------------------- | ------------------------------------------------------------------------------------------------------- | ---------------------------- |
| KPI-01 | Account capture usability  | Add an account manually in under 30 seconds in usability testing                                        | Timed usability tests        |
| KPI-02 | Account inventory accuracy | 100% of saved records persist correctly across extension restarts in release tests                      | Automated persistence tests  |
| KPI-03 | Search responsiveness      | Filter 1,000 local account records in under 300 ms at the 95th percentile on the reference test machine | Performance tests            |
| KPI-04 | Event traceability         | 100% of stored events contain event type, event time, recorded time, source, and verification status    | Schema validation            |
| KPI-05 | Detection transparency     | 100% of inferred events are visibly labeled as inferred                                                 | UI and integration tests     |
| KPI-06 | Privacy                    | Zero network transmission of inventory or activity data in default local-only mode                      | Network inspection           |
| KPI-07 | Data portability           | Export and re-import supported records without silent loss of supported fields                          | Round-trip tests             |
| KPI-08 | Permission minimization    | No unjustified permissions or broad host access                                                         | Manifest and security review |
| KPI-09 | Accessibility              | Core inventory and timeline journeys operable using keyboard navigation                                 | Accessibility tests          |
| KPI-10 | User control               | Users can disable history recording and delete stored account and activity data                         | Functional acceptance tests  |

### Product adoption metrics for post-launch

Track only metrics that can be collected without violating the local-first privacy model. The MVP should not transmit analytics by default.

- Time to first saved account.
- Number of accounts managed locally, available only as an on-device diagnostic if needed.
- Successful import/export completion rate in testing.
- False-positive and false-negative rates in the supported detection test suite.
- Crash, data corruption, and failed migration rates.

Any future telemetry must be opt-in, separately documented, and reviewed before implementation.

## 5. Non-Goals

The following capabilities are explicitly excluded from the MVP unless separately approved.

| ID    | Non-goal                                                                       | Reason                                                                              |
| ----- | ------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------- |
| NG-01 | Storing or autofilling passwords                                               | Avoid duplicating password-manager functionality and reduce credential exposure     |
| NG-02 | Capturing passwords, OTPs, cookies, session IDs, or access tokens              | Prohibited by the product's security model                                          |
| NG-03 | Guaranteed detection across every website                                      | Website authentication flows vary and may not expose reliable signals               |
| NG-04 | Retrieving historical activity from before installation automatically          | Requires provider records, imports, or user-entered history                         |
| NG-05 | Determining exact physical location from an IP address                         | IP geolocation is approximate and may be incorrect                                  |
| NG-06 | Reliably reading a computer's hostname through standard extension APIs         | Browser extension APIs do not provide a universal hostname interface                |
| NG-07 | Discovering every account associated with an email address across the internet | No universal, reliable, authorized account-discovery interface exists               |
| NG-08 | Monitoring all browsing activity                                               | The extension should observe only supported, relevant account-related interactions  |
| NG-09 | Automatically signing users out of websites                                    | This would require website-specific behavior and could interfere with user sessions |
| NG-10 | Operating a mandatory cloud synchronization service                            | Local-only operation is the default                                                 |
| NG-11 | Performing covert device discovery or local-network scanning                   | Unnecessary for the product's core purpose                                          |
| NG-12 | Claiming an account is compromised based only on an IP or location difference  | Activity metadata alone cannot establish account compromise                         |

## 6. Target Users & Personas

[Remote Working with Laptop. Woman office work personal profile. Flat Cartoon Vector Illustration, icon Stylish abstract design landing page web](https://images.openai.com/static-rsc-4/mC4sdZDteQkmF8dH6OYF3nU4uMeI_ONc73HuQiubuEQozncNypkjN5HUOydfc_Msi4m-1f4zHr9mYVheY4Q4hHlDU1eGojqWMQuEe-mj31W_y9Arurtvc2OvOIGPPe021URDWo4j1M11ASnZ2x8U2d_k5v0pYVUrqgSmWk-4D7A6sHtKvRKMPciL5qivF-QO?purpose=inline)

### Persona A - The multi-account user

**Primary audience:** Uses several email addresses across shopping, education, development, productivity, and social platforms.

**Needs:** A searchable account directory, email-based filtering, duplicate prevention, and easy account maintenance.

[Programmer illustration Images - Free Download on Magnific (formerly Freepik)](https://images.openai.com/static-rsc-4/Ifv5eo6rh_4Rx3Y8U_LSghinZlux5kWtmBAOHVF_R7xu2UXeOexN3y0yHnFLzKc9mejRbXpBsbUxe7DvqDWhQ6jbtAvp4sYpjE1MH1yT41kqOxSLdUAz0xlORG9LKn357zxytov_zlvw0FjIs5IAOhOFjRpRoDGvB_ZVNeTgW4k?purpose=inline)

### Persona B - The technical user

**Secondary audience:** Uses many developer platforms, cloud dashboards, repositories, and technical services.

**Needs:** Detailed activity records, device attribution, exportable data, and transparent detection evidence.

[Girl Is Pointing To Privacy Illustrations - Free Download in SVG, PNG](https://images.openai.com/static-rsc-4/f0UHSZGNyAKnwnXn8zZRj51iGi-8fRQ4TdOt6TdvvAHNyXmfb44Xd3-WUHDKutcDRTbnhqmfQz_JCExTJh93MFfDv5JnTV4nH4yQsXygDZFvXghEP1SHr3Q1X-jsnJk_FK9JFcLlx3nkF6UEhCtELr-szgbSIfE8z014Q55WeYM?purpose=inline)

### Persona C - The privacy-conscious user

**Privacy-sensitive audience:** Wants better account visibility but does not want an account inventory sent to a cloud provider.

**Needs:** Local storage, narrow permissions, explicit consent for network enrichment, and reliable deletion.

### User assumptions

- The user has Google Chrome installed on a desktop operating system.
- The user may have multiple email addresses.
- The user may use a website across several devices.
- The user is willing to enter some account details manually when automatic detection is not possible.
- Users should not need to create a cloud account to use the MVP.

## 7. User Stories & Use Cases

| ID    | User story                                                                                                                                             | Priority    |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------- |
| US-01 | As a user, I want to add multiple email profiles so that I can organize accounts by email address.                                                     | Must-have   |
| US-02 | As a user, I want to add an existing website account manually so that my inventory is useful immediately.                                              | Must-have   |
| US-03 | As a user, I want to see all accounts associated with a selected email so that I can identify where that email is used.                                | Must-have   |
| US-04 | As a user, I want to search and filter accounts so that I can find a specific service quickly.                                                         | Must-have   |
| US-05 | As a user, I want the extension to suggest that an account may have been created so that I can record new accounts without relying entirely on memory. | Should-have |
| US-06 | As a user, I want sign-in activity recorded when there is sufficient evidence so that I can review my account history.                                 | Should-have |
| US-07 | As a user, I want sign-out activity recorded separately from sign-in activity so that I can review session-related events.                             | Should-have |
| US-08 | As a user, I want to confirm or reject a suggested event so that inaccurate records do not become trusted history.                                     | Must-have   |
| US-09 | As a user, I want to associate activity with a named device profile so that I can distinguish my laptop from other devices.                            | Must-have   |
| US-10 | As a user, I want to record an optional device hostname so that I can identify a device more precisely when I know its hostname.                       | Should-have |
| US-11 | As a user, I want optional public-IP and location metadata so that I can review network context for an event.                                          | Should-have |
| US-12 | As a user, I want to know whether an event was confirmed, inferred, or manually entered so that I can assess its reliability.                          | Must-have   |
| US-13 | As a user, I want to export and restore my records so that I can back up or migrate my data.                                                           | Must-have   |
| US-14 | As a user, I want to delete an account and its related history so that I can remove information I no longer need.                                      | Must-have   |
| US-15 | As a user, I want to disable history recording and external metadata lookups so that I control data collection.                                        | Must-have   |
| US-16 | As a user, I want to view events by date, website, email, device, and event type so that I can investigate a specific period.                          | Should-have |
| US-17 | As a user, I want to edit or remove incorrect historical entries so that my personal records remain accurate.                                          | Must-have   |
| US-18 | As a user, I want to know when a sign-in or sign-out could not be verified so that I am not misled by incomplete website signals.                      | Must-have   |

### Primary use cases

1. Register an email profile and populate the account inventory.
2. Review and search existing accounts.
3. Detect a potential account creation or authentication event on a supported site.
4. Confirm, reject, or edit the suggested event.
5. Review an account's timeline and associated device profile.
6. Export, back up, restore, or delete stored records.

## 8. Functional Requirements

Priorities use the following definitions:

- Must-have (P0): Required for a safe and useful MVP.
- Should-have (P1): Important for v1 but may follow core inventory functionality.
- Nice-to-have (P2): Suitable for a later release or optional integration.

### 8.1 Account inventory

| ID    | Requirement                                   | Priority    | Acceptance criteria                                                                               |
| ----- | --------------------------------------------- | ----------- | ------------------------------------------------------------------------------------------------- |
| FR-01 | Create, edit, and delete email profiles       | Must-have   | A user can manage multiple profiles and their display names                                       |
| FR-02 | Add, edit, and delete website account records | Must-have   | Required fields are validated; edits persist across restarts                                      |
| FR-03 | Associate an account with an email profile    | Must-have   | Each account references a valid profile or an explicitly selected unknown-email state             |
| FR-04 | Store website identity and account metadata   | Must-have   | Record supports website name, domain, email profile, category, notes, and timestamps              |
| FR-05 | Search and filter accounts                    | Must-have   | Search supports website name/domain and email; filters support category and status                |
| FR-06 | Detect likely duplicate records               | Must-have   | The UI warns about a matching normalized domain and email profile before saving                   |
| FR-07 | Open the associated website                   | Must-have   | A user action opens the stored, validated HTTPS URL where available                               |
| FR-08 | Import and export records                     | Must-have   | Supported JSON export/import round-trip preserves documented fields                               |
| FR-09 | Provide categories and optional tags          | Should-have | Users can classify accounts using built-in or custom categories                                   |
| FR-10 | Provide account status labels                 | Should-have | Users can mark records active, inactive, or pending review without claiming provider verification |

### 8.2 Detection and activity history

| ID    | Requirement                                    | Priority    | Acceptance criteria                                                                                  |
| ----- | ---------------------------------------------- | ----------- | ---------------------------------------------------------------------------------------------------- |
| FR-11 | Maintain an account activity timeline          | Must-have   | Each event displays event type, time, source, and verification state                                 |
| FR-12 | Record account creation events                 | Must-have   | Manual entries and supported creation signals are distinguishable                                    |
| FR-13 | Record sign-in events                          | Must-have   | Only observed, inferred, provider-reported, or user-confirmed events are saved with their source     |
| FR-14 | Record sign-out events                         | Should-have | A sign-out event is labeled according to evidence; a click alone is not proof of successful sign-out |
| FR-15 | Allow user confirmation or rejection           | Must-have   | Suggested events can be accepted, rejected, or dismissed                                             |
| FR-16 | Store event time and recording time separately | Must-have   | `occurredAt` and `recordedAt` are both present                                                       |
| FR-17 | Track event source and confidence              | Must-have   | Every event has a source and a verification state                                                    |
| FR-18 | Associate events with device profiles          | Must-have   | Each event can reference a selected or current default device profile                                |
| FR-19 | Provide timeline filters                       | Should-have | Events can be filtered by date range, account, email, event type, source, and device                 |
| FR-20 | Allow event correction and deletion            | Must-have   | Users can edit supported fields and remove erroneous entries                                         |
| FR-21 | Prevent duplicate event creation               | Must-have   | Repeated signals within a configurable deduplication window are not blindly saved as separate events |
| FR-22 | Support manual historical entries              | Must-have   | Users can enter past events with a user-specified event time and a system-recorded creation time     |
| FR-23 | Identify unsupported or ambiguous cases        | Must-have   | The extension explains when a site cannot be reliably monitored                                      |

### 8.3 Device and network metadata

| ID    | Requirement                                            | Priority    | Acceptance criteria                                                                           |
| ----- | ------------------------------------------------------ | ----------- | --------------------------------------------------------------------------------------------- |
| FR-24 | Maintain device profiles                               | Must-have   | Users can create, rename, select, and delete named profiles                                   |
| FR-25 | Capture available browser and OS metadata              | Should-have | Only supported and justified metadata is stored; unavailable values remain null               |
| FR-26 | Allow user-entered hostnames                           | Should-have | A hostname can be stored as a user-supplied value with its source indicated                   |
| FR-27 | Support optional public-IP enrichment                  | Should-have | Disabled by default; lookup behavior and data disclosure are shown before enabling            |
| FR-28 | Support optional approximate geolocation               | Should-have | Disabled by default and performed only with explicit consent and a configured source          |
| FR-29 | Store metadata provenance                              | Must-have   | IP and location fields identify source and observation time                                   |
| FR-30 | Avoid claiming exact location or device identity       | Must-have   | UI identifies approximate or user-supplied information and does not imply verification        |
| FR-31 | Allow metadata collection to be disabled independently | Must-have   | Disabling a lookup prevents future requests; users can also delete previously stored metadata |

### 8.4 Privacy and data management

| ID    | Requirement                          | Priority    | Acceptance criteria                                                                                                              |
| ----- | ------------------------------------ | ----------- | -------------------------------------------------------------------------------------------------------------------------------- |
| FR-32 | Keep core data local by default      | Must-have   | No account or activity data leaves the browser in default mode                                                                   |
| FR-33 | Provide history recording controls   | Must-have   | Users can pause and resume activity recording                                                                                    |
| FR-34 | Provide data retention controls      | Must-have   | Users can choose a retention policy or disable automatic expiry                                                                  |
| FR-35 | Provide complete local-data deletion | Must-have   | A clear action removes supported inventory, history, and relevant stored settings after confirmation                             |
| FR-36 | Define account deletion behavior     | Must-have   | The UI clearly explains whether linked activity events will also be deleted; the recommended default is to delete linked history |
| FR-37 | Validate imported data               | Must-have   | Invalid records, unsupported versions, and oversized imports are rejected safely with actionable errors                          |
| FR-38 | Provide privacy disclosures          | Must-have   | The settings screen describes stored fields, permissions, external lookups, and data deletion                                    |
| FR-39 | Support export of activity history   | Should-have | Users can export documented activity fields to JSON and CSV                                                                      |
| FR-40 | Protect data migration               | Must-have   | Schema migrations are versioned and tested against backup and restore scenarios                                                  |

### 8.5 Detection limitations that must be enforced

The extension must use the following rules:

1. A successful form submission does not necessarily mean account creation or authentication succeeded.
2. Clicking a sign-out button does not prove the website terminated the session.
3. Closing a tab or browser does not count as a sign-out.
4. A browser may observe only some authentication states, particularly when a site uses embedded flows, cross-origin redirects, service workers, or a separate application.
5. No automatic history should be invented for activity that occurred before installation.
6. If an event is inferred, the interface must show the evidence category and avoid presenting it as confirmed.
7. Provider-reported authentication events may be supported in a later release through documented, authorized integrations.

## 9. Non-Functional Requirements

| ID     | Category        | Requirement                                                                                                                                                                   | Priority    |
| ------ | --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------- |
| NFR-01 | Performance     | Core inventory and timeline views should open within 500 ms for 1,000 local accounts and 10,000 activity events on the reference test machine, excluding cold browser startup | Must-have   |
| NFR-02 | Responsiveness  | Search should return results within 300 ms at the 95th percentile for 1,000 records                                                                                           | Must-have   |
| NFR-03 | Reliability     | No silent loss of saved data during normal extension updates, browser restarts, or supported schema migrations                                                                | Must-have   |
| NFR-04 | Data integrity  | Referential integrity must be maintained between profiles, accounts, device profiles, and events                                                                              | Must-have   |
| NFR-05 | Security        | Never store passwords, OTPs, session cookies, authentication tokens, or equivalent secrets                                                                                    | Must-have   |
| NFR-06 | Security        | Use least-privilege extension permissions and avoid broad host permissions unless explicitly justified                                                                        | Must-have   |
| NFR-07 | Security        | Validate and normalize imported data, URLs, domains, and user-controlled text                                                                                                 | Must-have   |
| NFR-08 | Privacy         | No external network calls for account inventory or activity data in default mode                                                                                              | Must-have   |
| NFR-09 | Privacy         | Any external IP/location lookup must be opt-in, documented, and independently disableable                                                                                     | Must-have   |
| NFR-10 | Scalability     | The local data model should support at least 10,000 account records and 100,000 activity events without architectural redesign                                                | Should-have |
| NFR-11 | Accessibility   | Core workflows must support keyboard operation, visible focus, accessible labels, and adequate contrast                                                                       | Must-have   |
| NFR-12 | Compatibility   | Support the current stable desktop version of Google Chrome at release time                                                                                                   | Must-have   |
| NFR-13 | Maintainability | Use modular services and versioned data schemas with automated tests                                                                                                          | Must-have   |
| NFR-14 | Usability       | Distinguish confirmed, inferred, manually entered, and unavailable data in the interface                                                                                      | Must-have   |
| NFR-15 | Compliance      | Document privacy practices, data handling, and applicable Chrome Web Store policies before release                                                                            | Must-have   |
| NFR-16 | Resource usage  | Avoid persistent background polling and unnecessary CPU, memory, or network activity                                                                                          | Must-have   |
| NFR-17 | Recovery        | Offer export/restore procedures and clear errors for failed writes, imports, or migrations                                                                                    | Must-have   |

Scalability note: These are local dataset targets, not a promise of unlimited storage. The implementation must test IndexedDB quotas, large imports, migration time, and event cleanup behavior.

## 10. User Flow & Experience

### 10.1 Information architecture

<img width="482" height="289" alt="image" src="https://github.com/user-attachments/assets/ff2759b4-489b-4179-9920-81f2bfc82b80" />


### 10.2 First-run journey

1. Welcome and privacy explanation. Explain local storage, the purpose of permissions, and the limitations of automatic detection.
2. Create an email profile. The user adds an email address and optional display label.
3. Select a device profile. The user chooses or creates a label such as “Personal laptop.”
4. Add accounts. The user adds existing accounts manually or imports a supported file.
5. Configure optional features. The user can enable supported detection or later opt in to IP/location enrichment.
6. Open the dashboard. Display the account inventory, recent activity, and clear next actions.

### 10.3 Account creation or sign-in journey

1. The user visits a supported website.
2. The extension observes a permitted, relevant signal. It must not read or store credentials.
3. The extension creates a candidate event only when its detection rules are satisfied.
4. A compact prompt may ask the user to confirm the account, email profile, and event type.
5. After confirmation, the extension saves the event with timestamps, source, verification status, and the selected device profile.
6. The user can view or correct the event in the activity timeline.

Important: Detection must be conservative. If a website's state cannot be determined reliably, the extension should offer manual entry rather than repeatedly prompting the user or creating false records.

### 10.4 Activity timeline

Each event should display:

- Website or account name.
- Event type: account created, sign-in, sign-out, or manual entry.
- Event time and recording time when relevant.
- Verification status: confirmed, inferred, or unverified.
- Evidence source.
- Device profile.
- Optional public IP and approximate location, if available and enabled.
- A link to event details, edit, and delete actions.

Do not use color alone to communicate verification state. Provide a text label and accessible icon where appropriate.

### 10.5 Settings and consent journey

Before enabling external enrichment, the user must be told:

- What data will be sent to which provider.
- Whether the provider receives the public IP as part of the request.
- Whether the provider may receive request metadata.
- Whether results are approximate.
- How to disable future requests and delete saved enrichment data.

The user must be able to use the account inventory without enabling external enrichment.

## 11. Technical Considerations

### 11.1 Recommended technology stack

| Layer                | Recommendation                                                         | Rationale                                                        |
| -------------------- | ---------------------------------------------------------------------- | ---------------------------------------------------------------- |
| Extension platform   | Chrome Extension Manifest V3                                           | Native support for Chrome extensions                             |
| UI                   | HTML, CSS, vanilla JavaScript                                          | Lightweight, maintainable, avoids unnecessary framework overhead |
| Structured storage   | IndexedDB                                                              | Suitable for related account and activity records                |
| Preferences          | `chrome.storage.local`                                                 | Appropriate for small settings and extension preferences         |
| Background execution | Service worker                                                         | Event-driven coordination under Manifest V3                      |
| Website detection    | Content scripts on explicitly supported sites or user-approved domains | Limits exposure and reduces unnecessary permissions              |
| Testing              | Unit, integration, browser automation, and manual security tests       | Covers storage, detection, permissions, and UI behavior          |
| Packaging            | Chrome Web Store package and documented development build              | Enables controlled distribution                                  |

A backend is not required for the MVP. It should be considered only if a later requirement introduces optional synchronization, authorized provider integrations, or cross-device data sharing.

### 11.2 High-level architecture

<img width="1796" height="1592" alt="mermaid-diagram" src="https://github.com/user-attachments/assets/e4857664-ff8b-4838-88fb-c254e5ea8d4b" />

Architectural boundaries

- Presentation layer: Popup, dashboard, forms, timeline, settings, and confirmation prompts.
- Application core: Use cases, validation, deduplication, event confidence, and business rules.
- Services layer: Account service, activity service, device registry, import/export, and privacy settings.
- Native/external layer: Chrome APIs, IndexedDB, extension messaging, and optional external enrichment adapters.

Content scripts must not access the complete account inventory or unrelated activity history. They should send only the minimum event signal required by the application core.

### 11.3 Storage model

Use versioned records and stable IDs. Store structured inventory and activity data separately so that account management and event history can evolve independently.

#### `EmailProfile`

| Field       | Type               | Description                |
| ----------- | ------------------ | -------------------------- |
| `id`        | string             | Unique profile identifier  |
| `email`     | string             | User-entered email address |
| `label`     | string             | Optional display label     |
| `createdAt` | ISO 8601 timestamp | Creation time              |
| `updatedAt` | ISO 8601 timestamp | Last update                |

#### `Account`

| Field            | Type               | Description                 |
| ---------------- | ------------------ | --------------------------- |
| `id`             | string             | Unique account identifier   |
| `websiteName`    | string             | Display name                |
| `domain`         | string             | Normalized website domain   |
| `websiteUrl`     | string             | Validated website URL       |
| `emailProfileId` | string or null     | Associated email profile    |
| `category`       | string             | Account category            |
| `status`         | string             | User-maintained status      |
| `notes`          | string             | Optional user-entered notes |
| `createdAt`      | ISO 8601 timestamp | Record creation time        |
| `updatedAt`      | ISO 8601 timestamp | Last update                 |

#### `ActivityEvent`

| Field                   | Type                       | Description                                                                 |
| ----------------------- | -------------------------- | --------------------------------------------------------------------------- |
| `id`                    | string                     | Unique event identifier                                                     |
| `accountId`             | string or null             | Associated account, if known                                                |
| `websiteId`             | string or null             | Optional normalized website reference                                       |
| `emailProfileId`        | string or null             | Email profile associated with the event                                     |
| `eventType`             | enum                       | `account_created`, `signin_detected`, `signout_detected`, or `manual_entry` |
| `occurredAt`            | ISO 8601 timestamp         | Time the event reportedly occurred                                          |
| `recordedAt`            | ISO 8601 timestamp         | Time AccountLedger recorded it                                              |
| `timezoneOffsetMinutes` | integer or null            | Local timezone offset at recording                                          |
| `deviceProfileId`       | string or null             | Associated device profile                                                   |
| `evidenceSource`        | enum                       | `user_confirmed`, `website_signal`, `provider_audit`, or `manual_entry`     |
| `verificationStatus`    | enum                       | `confirmed`, `inferred`, or `unverified`                                    |
| `confidence`            | enum or null               | Optional confidence category, such as `high`, `medium`, or `low`            |
| `publicIp`              | string or null             | Optional public IP, if lawfully and reliably obtained                       |
| `ipSource`              | string or null             | Origin of the IP value                                                      |
| `ipObservedAt`          | ISO 8601 timestamp or null | Time the IP was observed                                                    |
| `location`              | object or null             | Optional approximate location metadata                                      |
| `notes`                 | string                     | Optional user-entered context                                               |

The `location` object may include `countryCode`, `region`, `city`, `source`, `observedAt`, and `precision`. Only store fields returned by the configured provider and allowed by the user's settings.

#### `DeviceProfile`

| Field            | Type               | Description                      |
| ---------------- | ------------------ | -------------------------------- |
| `id`             | string             | Unique device profile identifier |
| `label`          | string             | User-defined device name         |
| `hostname`       | string or null     | Optional user-entered hostname   |
| `hostnameSource` | string or null     | Source of the hostname           |
| `browserFamily`  | string or null     | Browser family, if available     |
| `browserVersion` | string or null     | Browser version, if justified    |
| `osFamily`       | string or null     | OS family, if reliably available |
| `createdAt`      | ISO 8601 timestamp | Profile creation time            |
| `updatedAt`      | ISO 8601 timestamp | Last update                      |

#### `PrivacySettings`

Recommended settings include:

- `historyEnabled`
- `ipEnrichmentEnabled`
- `locationEnrichmentEnabled`
- `retentionDays`
- `defaultDeviceProfileId`
- `detectionEnabled`
- `externalLookupProvider`
- `schemaVersion`

Do not store secrets or authentication credentials in these records.

### 11.4 Data integrity rules

1. Every record has a stable ID and schema version.
2. Every activity event has an event time, recording time, source, and verification status.
3. A deleted account must not leave unexplained dangling references. Linked history is deleted by default unless the user explicitly chooses a different documented behavior.
4. Event deduplication must not merge distinct events solely because they occur close together.
5. Import operations must validate records before modifying the existing inventory.
6. Prefer an atomic import or a staged validation-and-commit process so an invalid file cannot leave a partially corrupted dataset.
7. Failed migrations must produce a clear error and preserve a recoverable backup where feasible.
8. Date/time values should use a consistent machine-readable format and be rendered in the user's local timezone.

### 11.5 IP address, hostname, and location limitations

These limitations are architectural constraints, not merely UI concerns.

| Data                   | Availability                                                            | Required behavior                                                                                                  |
| ---------------------- | ----------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| Public IP              | Not reliably exposed by a standard Chrome extension API                 | Use a documented external service only after explicit consent, or a supported provider audit record                |
| Private/local IP       | Not reliably available through ordinary extension APIs                  | Do not use invasive network discovery; leave unavailable unless supplied through an explicitly supported mechanism |
| Hostname               | Not generally available through standard extension APIs                 | Let the user enter a hostname; do not imply automatic discovery                                                    |
| IP geolocation         | Approximate and provider-dependent                                      | Label as approximate and record source and observation time                                                        |
| Device identity        | A browser profile is not a guaranteed unique physical-device identifier | Use user-defined device profiles rather than fingerprinting                                                        |
| Authentication success | Website-dependent                                                       | Treat website signals as evidence, not universal proof                                                             |
| Historical activity    | Not available by default before installation                            | Support manual import or authorized provider integration if available                                              |

Recommended MVP behavior: Implement named device profiles and timestamps first. Keep public-IP and geolocation lookups disabled and outside the core activity-recording path until the privacy and provider requirements are approved.

### 11.6 Chrome permissions

Request only permissions justified by an implemented feature. The final manifest must be reviewed against the actual implementation.

| Permission / capability | Purpose                                                            | Recommendation                            |
| ----------------------- | ------------------------------------------------------------------ | ----------------------------------------- |
| `storage`               | Store extension preferences and small configuration values         | Required                                  |
| `activeTab`             | Temporary access to the active tab after a user gesture, if needed | Prefer over broad access when appropriate |
| `scripting`             | Inject a detection script on explicit user action, if required     | Conditional                               |
| `alarms`                | Scheduled retention cleanup, if required                           | Conditional                               |
| `notifications`         | Optional user notifications                                        | Defer unless necessary                    |
| Host permissions        | Observe supported authentication signals on selected sites         | Request narrowly and only when justified  |

The extension should not request unrestricted access to all websites merely because it might simplify implementation. Permissions must be explained in plain language and tested for least privilege.

### 11.7 Integration boundaries

Potential later integrations:

- Official provider audit logs or security activity APIs.
- Optional IP lookup provider.
- Optional geolocation provider.
- JSON/CSV import and export.

Provider integrations must use authorized documented interfaces. They must not scrape private dashboards or bypass authentication controls. The MVP must not depend on any external provider.

## 12. Assumptions, Risks & Mitigations

### 12.1 Assumptions

| ID   | Assumption                                                                         | Validation method                        |
| ---- | ---------------------------------------------------------------------------------- | ---------------------------------------- |
| A-01 | Local-first account management is valuable without a cloud account                 | Test prototype with target users         |
| A-02 | Users will manually confirm ambiguous events                                       | Usability testing                        |
| A-03 | The initial audience uses desktop Chrome                                           | Confirm target-platform scope            |
| A-04 | A user-defined device label is sufficient for MVP attribution                      | Test with users who use multiple devices |
| A-05 | Users will accept limited detection coverage if limitations are clearly disclosed  | Test expectations during onboarding      |
| A-06 | Vanilla JavaScript and browser-native storage are sufficient for the first release | Prototype storage and UI performance     |

### 12.2 Risk register

| ID   | Risk                                                              | Impact                                | Mitigation                                                                        |
| ---- | ----------------------------------------------------------------- | ------------------------------------- | --------------------------------------------------------------------------------- |
| R-01 | False-positive authentication events                              | Incorrect personal records            | Evidence-based detection, confirmation prompts, deduplication, confidence labels  |
| R-02 | Missed authentication events                                      | Incomplete history                    | Explain coverage limits and support manual entry                                  |
| R-03 | Overly broad permissions                                          | Privacy and store-review concerns     | Least-privilege manifest and permission audit                                     |
| R-04 | Sensitive inventory exposure                                      | Disclosure of account relationships   | Local-first storage, no credential capture, explicit deletion and export controls |
| R-05 | External IP/location lookup leaks request metadata                | User privacy impact                   | Opt-in, provider disclosure, minimal fields, no background enrichment by default  |
| R-06 | Incorrect device attribution                                      | Misleading activity records           | Use user-selected profiles; allow corrections                                     |
| R-07 | Browser storage corruption or failed migration                    | Data loss                             | Schema versioning, backups, migration tests, clear failure behavior               |
| R-08 | Unsupported website changes break detection                       | Detection degradation                 | Keep site adapters isolated and test them independently                           |
| R-09 | Extension performance degrades with long history                  | Slow UI or storage growth             | IndexedDB, indexed queries, pagination, configurable retention                    |
| R-10 | Product scope expands before MVP is stable                        | Delayed release                       | Release gates and explicit prioritization                                         |
| R-11 | Privacy or Chrome Web Store policy non-compliance                 | Rejection or removal                  | Policy review, accurate disclosures, least privilege, documented data flows       |
| R-12 | User mistakes inferred activity for proof of compromise or access | Unnecessary concern or poor decisions | Neutral wording, source attribution, explicit limitations                         |

## 13. Release Plan & Milestones

Planning assumption: One developer working consistently, with time for testing and review. The schedule is an estimate, not a commitment. Automatic detection may extend the timeline because it requires site-specific validation.

## MVP

Weeks 1–4

Foundation and local account inventory

- Finalize user flows, permissions, and schema.
- Build popup/dashboard and email profiles.
- Implement account CRUD, search, filtering, and duplicate warnings.
- Implement IndexedDB, settings, JSON import/export, and deletion.
- Add basic activity timeline with manual event entry.
- Test persistence, data validation, and keyboard navigation.

Exit gate: users can manage accounts and activity history reliably with no external data transmission by default.

## v1.0

Weeks 5–8

Supported-site detection and release hardening

- Implement conservative detection for a small, documented set of supported sites.
- Add confirmation/rejection workflows and deduplication.
- Implement named device profiles and event attribution.
- Add retention controls and complete privacy disclosures.
- Conduct security, data integrity, performance, and permission reviews.
- Package, document, and prepare the Chrome Web Store release.

Exit gate: detection limitations are visible, inferred events are labeled correctly, and the release passes security and recovery tests.

## v2.0

Post-launch

Optional enrichment and advanced workflows

- Add opt-in public-IP and approximate location enrichment.
- Consider official provider audit-log integrations.
- Add advanced event filters, reports, and enhanced backup/restore.
- Evaluate optional encrypted synchronization only if users request it.
- Add site adapters based on user demand and measured detection reliability.

Exit gate: every new integration has a documented threat model, consent flow, data retention policy, and independent test coverage.

### 13.1 Milestone and deliverable table

| Milestone                      | Deliverables                                                   | Approval gate            |
| ------------------------------ | -------------------------------------------------------------- | ------------------------ |
| M0 - Requirements approval     | PRD, UX flow, data schema, permission plan                     | Product scope approved   |
| M1 - Architecture baseline     | Manifest, storage abstraction, module boundaries, test setup   | Architecture review      |
| M2 - Inventory complete        | Email profiles, accounts, search, filtering, import/export     | Functional acceptance    |
| M3 - Activity history complete | Timeline, event schema, manual entries, correction/deletion    | Data integrity review    |
| M4 - Detection prototype       | Supported-site adapter, event prompts, evidence labels         | Detection quality review |
| M5 - Security hardening        | Permission audit, privacy review, migration and recovery tests | Security approval        |
| M6 - Release candidate         | Documentation, packaged extension, test report                 | Release approval         |
| M7 - v2 evaluation             | User feedback, optional integrations proposal                  | Separate scope approval  |

### 13.2 Release-blocking gates

A release must not proceed if any of the following conditions apply:

- Account or activity data is transmitted externally without explicit user consent.
- The extension captures credentials, OTPs, cookies, session identifiers, or authentication tokens.
- Inferred events are displayed as confirmed events.
- A supported import or migration can silently destroy existing records.
- Website content scripts can read the complete account inventory without a documented need.
- Broad permissions cannot be justified by a shipped feature.
- The product claims that IP location, hostname, or sign-out status is more reliable than the available evidence supports.

## 14. Open Questions

These decisions should be resolved before the relevant implementation phase. Suggested defaults are provided to avoid blocking the MVP.

| ID    | Question                                                                   | Suggested default                                                            | Decision deadline            |
| ----- | -------------------------------------------------------------------------- | ---------------------------------------------------------------------------- | ---------------------------- |
| OQ-01 | Which website types should be supported for automatic detection first?     | Begin with a small, explicitly documented allowlist                          | Before detection work        |
| OQ-02 | Should the extension attempt detection on all websites?                    | No; use narrow, supported-site coverage                                      | Requirements approval        |
| OQ-03 | Should IP and location enrichment be part of the MVP?                      | No; defer to v2 and keep disabled by default                                 | Requirements approval        |
| OQ-04 | Should activity history be deleted when an account is deleted?             | Yes, by default, with clear confirmation                                     | Before data-layer completion |
| OQ-05 | Should users be able to keep an event after deleting its account?          | Optional future behavior with an explicit privacy-preserving choice          | Before v1                    |
| OQ-06 | How long should activity events be retained?                               | User-configurable; include a no-expiry option and clear storage implications | Before v1                    |
| OQ-07 | Should users be able to import existing account inventories from CSV?      | Yes, with a documented template and validation                               | Before MVP completion        |
| OQ-08 | Should the product support multiple Chrome profiles or multiple computers? | Support multiple user-defined device profiles; no sync in MVP                | Requirements approval        |
| OQ-09 | Should users receive notifications for detected events?                    | In-extension prompt first; system notifications deferred                     | Before detection work        |
| OQ-10 | Which IP/location provider, if any, should be used?                        | No provider until privacy, reliability, cost, and retention are reviewed     | Before v2                    |
| OQ-11 | Will official provider audit-log integrations be pursued?                  | Evaluate after MVP based on demand and available authorized APIs             | v2 planning                  |
| OQ-12 | Will the extension be open source?                                         | Undecided; select a license before public source publication                 | Before public release        |
| OQ-13 | Is encrypted cloud synchronization required?                               | No; evaluate only after validating a concrete user need                      | v2 planning                  |
| OQ-14 | Should the UI support import/export encryption?                            | Consider password-protected backup as a later enhancement                    | v1/v2 planning               |

## 15. Appendix

### 15.1 Glossary

| Term                | Definition                                                                                     |
| ------------------- | ---------------------------------------------------------------------------------------------- |
| Account inventory   | The list of website accounts managed by the user                                               |
| Email profile       | An email address and optional label used to group accounts                                     |
| Activity event      | A record of a reported, observed, inferred, or user-confirmed account-related occurrence       |
| Evidence source     | The method by which an event was obtained                                                      |
| Verification status | A label indicating whether an event is confirmed, inferred, or unverified                      |
| Device profile      | A user-defined label representing a device or browser environment                              |
| Public IP           | An IP address visible to an external service; it does not uniquely identify a person or device |
| IP geolocation      | An approximate location estimate derived from IP-related data                                  |
| Manifest V3         | Chrome's extension platform and permission/runtime model                                       |
| Content script      | A script operating within a webpage context under extension rules                              |
| Service worker      | An event-driven background component of a Manifest V3 extension                                |
| IndexedDB           | Browser storage designed for structured client-side data                                       |
| Least privilege     | Giving a component only the permissions it needs to perform its job                            |

### 15.2 Competitor and alternative-product notes

The following are product categories to evaluate during research, not a verified competitive ranking.

| Alternative                     | Primary purpose                                    | AccountLedger's intended distinction                                                                                        |
| ------------------------------- | -------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| Password managers               | Store credentials and assist with authentication   | Track account inventory and activity without storing credentials                                                            |
| Browser password/autofill tools | Remember credentials and identity information      | Focus on email-to-website relationships and a user-controlled event timeline                                                |
| Email search and inbox rules    | Find registration emails and service notifications | Maintain a structured inventory that is not limited to inbox contents                                                       |
| Manual spreadsheets             | Track accounts, email addresses, and notes         | Provide browser-integrated workflows and timestamped activity records                                                       |
| Identity/security dashboards    | Display provider-specific account security events  | Combine a local inventory with multiple manually maintained account records; provider-specific integrations remain optional |

Competitive research requirement: Before finalizing the brand or launch strategy, assess existing Chrome Web Store extensions and account-management tools for overlapping features, permission models, privacy policies, pricing, maintenance activity, and user feedback. Do not assume that a particular capability is unique without verifying the current market.

### 15.3 Reference documentation

Implementation should be based on current official documentation, checked again during development:

- [Chrome Extensions documentation](https://developer.chrome.com/docs/extensions/?utm_source=chatgpt.com)
- [Manifest V3 overview](https://developer.chrome.com/docs/extensions/develop/migrate/what-is-mv3?utm_source=chatgpt.com)
- [Chrome storage API](https://developer.chrome.com/docs/extensions/reference/api/storage?utm_source=chatgpt.com)
- [Chrome scripting API](https://developer.chrome.com/docs/extensions/reference/api/scripting?utm_source=chatgpt.com)
- [Chrome activeTab permission](https://developer.chrome.com/docs/extensions/develop/concepts/activeTab?utm_source=chatgpt.com)
- [IndexedDB API - MDN](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API?utm_source=chatgpt.com)
- [Chrome Web Store program policies](https://developer.chrome.com/docs/webstore/program-policies/?utm_source=chatgpt.com)

These references are starting points. Confirm the current API behavior, permission rules, and store policies before release.

## Recommended next step

Approve the MVP scope and technical architecture before writing production code. The first implementation should focus on the local account inventory, email profiles, manual activity timeline, device profiles, import/export, and reliable deletion. Automatic detection should follow only after the data model and evidence/verification rules are tested. Public-IP and geolocation enrichment should remain outside the MVP until the consent flow and provider data handling are explicitly approved.

The most important engineering constraint is to build a trustworthy account ledger first-not to promise universal sign-in/sign-out monitoring that a browser extension cannot reliably guarantee.
