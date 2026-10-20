# Requirements Specification — Beauty Services Marketplace

**Document ID:** REQ-SPEC-001  
**Version:** 1.0  
**Status:** Proposed baseline — business decisions remain open  
**Scope:** First release (MVP), organized by 11 logical domains

## 1. Purpose and terminology

This document specifies business requirements, functional requirements, quality requirements, and cross-domain contracts for the marketplace. It describes expected behavior, not a commitment that every domain is already implemented as an independent deployable microservice.

**Actors:** Client, Professional, Administrator, external Identity Provider (optional), Payment Provider, and system integrations.  
**Priorities:** `MUST` = required for the MVP's defined scope; `SHOULD` = important but may be negotiated; `COULD` = desirable; `PENDING` = requires a business decision before implementation.  
**Requirement IDs:** `BR` = business requirement, `FR` = functional requirement, `NFR` = non-functional requirement, `INT` = integration/event requirement.

The authoritative domain boundaries and ownership rules are in `project_domains.md`. The use-case flows are in `domain_use_cases.md`; test scenarios are in `test_artifacts.md`.

## 2. Product objective and MVP journey

The product must connect clients with beauty and personal-care professionals and allow a client to discover an offer, understand its price and duration, select a valid time, book, pay according to the approved flow, receive the service, and provide eligible feedback. Professionals must be able to present their profiles and offers, maintain schedules, and manage appointments. Administrators must be able to handle operational issues without bypassing the domain that owns the affected record.

Primary journey:

1. Account creation/authentication.
2. Professional profile and service offer are published.
3. Client searches or browses available professionals/offers.
4. Client reviews offer details and candidate time.
5. Client creates a booking; the system performs an authoritative conflict check.
6. Payment is processed according to the approved payment policy.
7. Appointment is completed or moved through an allowed terminal state.
8. Client may submit a review if eligible.

## 3. Business requirements

| ID | Requirement | Priority |
|---|---|---|
| BR-001 | A user can register and access the platform according to an assigned role. | MUST |
| BR-002 | Clients can maintain profile information required for marketplace use. | MUST |
| BR-003 | Professionals can create a public profile and become discoverable after meeting publication requirements. | MUST |
| BR-004 | The platform provides categories and generic service definitions while allowing professional-specific offers, prices, and duration. | MUST |
| BR-005 | Clients can search and compare active professionals and offers using available filters. | MUST; simple search may precede a standalone search service |
| BR-006 | Professionals can configure working hours, exceptions, and unavailable periods. | MUST |
| BR-007 | Clients can create, consult, and cancel appointments according to defined rules. | MUST |
| BR-008 | Conflicting appointments for the same professional and interval cannot both be confirmed. | MUST |
| BR-009 | The first release includes online payment functionality using a selected provider and approved financial policy. | MUST; exact flow PENDING |
| BR-010 | Eligible clients can review completed/eligible appointments. | MUST |
| BR-011 | Users can report problems and authorized moderators can review reports. | SHOULD |
| BR-012 | Professionals can see basic operational indicators derived from platform records. | SHOULD; basic projections may be phased |
| BR-013 | Administrators can perform permitted operations through the owning domains' rules. | MUST |
| BR-014 | Personal data, financial records, and administrative actions are protected and traceable. | MUST |

## 4. Functional requirements by domain

### 4.1 Domain 1 — Identity & Access

| ID | Requirement | Priority |
|---|---|---|
| FR-IAM-001 | Register accounts with required fields and normalized login identifiers. | MUST |
| FR-IAM-002 | Authenticate a user with valid credentials and return the configured session/token outcome. | MUST |
| FR-IAM-003 | Provide secure account recovery. | MUST |
| FR-IAM-004 | Support social authentication only if the provider and flow are approved. | SHOULD/PENDING |
| FR-IAM-005 | Maintain the roles Client, Professional, and Administrator and evaluate permissions server-side. | MUST |
| FR-IAM-006 | Enable/disable an account according to authorized policy and revoke or expire access as appropriate. | MUST |
| FR-IAM-007 | Publish minimal account lifecycle events without credentials or tokens. | MUST for event integration |

### 4.2 Domain 2 — Profiles & Professionals

| ID | Requirement | Priority |
|---|---|---|
| FR-PROF-001 | Create and update the client profile linked to an account ID. | MUST |
| FR-PROF-002 | Create and update a professional profile linked to an account ID. | MUST |
| FR-PROF-003 | Maintain profile states (for example, draft, active, suspended) with valid transitions. | MUST |
| FR-PROF-004 | Validate the minimum fields needed before a profile can be published. | MUST; field list to be approved |
| FR-PROF-005 | Store external social links associated with the professional profile. | SHOULD |
| FR-PROF-006 | Present public fields separately from private fields and restrict access to private fields. | MUST |
| FR-PROF-007 | Publish profile changes needed by search and downstream projections. | MUST |
| FR-PROF-008 | Do not require professional identity verification unless the product owner approves a verification policy. | PENDING decision; behavior must remain configurable |

### 4.3 Domain 3 — Service Catalog & Offers

| ID | Requirement | Priority |
|---|---|---|
| FR-CAT-001 | Maintain platform categories and generic service definitions. | MUST |
| FR-CAT-002 | Let an eligible professional create, edit, list, activate, and deactivate their offers. | MUST |
| FR-CAT-003 | Record each offer's professional ID, category/service association, name, price, estimated duration, and active state. | MUST |
| FR-CAT-004 | Permit different professionals to set different prices and durations for similar services. | MUST |
| FR-CAT-005 | Permit professional offers to use a finer level of detail than the generic service taxonomy where appropriate. | MUST |
| FR-CAT-006 | Preserve agreed price/duration snapshots for existing appointments when an offer changes later. | MUST |
| FR-CAT-007 | Prevent inactive offers from being selected for new appointments. | MUST |
| FR-CAT-008 | Publish catalog/offer lifecycle events for dependent projections. | MUST for event integration |

### 4.4 Domain 4 — Discovery & Search

| ID | Requirement | Priority |
|---|---|---|
| FR-DSC-001 | Search by category, service, or professional name. | MUST |
| FR-DSC-002 | Filter by service area/location, price, rating, and availability when reliable data exists. | MUST with clear unavailable-filter behavior |
| FR-DSC-003 | Return summaries of active public profiles and offers without exposing private fields. | MUST |
| FR-DSC-004 | Handle empty results, invalid filters, and stale/unavailable projection data gracefully. | MUST |
| FR-DSC-005 | Apply a documented default ordering; paid placement is not part of the MVP unless approved. | MUST/PENDING ranking details |
| FR-DSC-006 | Treat search availability as indicative; booking must revalidate in the appointment owner. | MUST |
| FR-DSC-007 | Update read models from versioned events and expose projection freshness for operations. | SHOULD |

### 4.5 Domain 5 — Agenda & Availability

| ID | Requirement | Priority |
|---|---|---|
| FR-AVL-001 | Configure recurring working hours with a defined time zone. | MUST |
| FR-AVL-002 | Create, edit, and remove date-specific exceptions and manual time blocks. | MUST |
| FR-AVL-003 | Calculate candidate intervals using configured hours, blocks, and the selected offer's duration. | MUST |
| FR-AVL-004 | Return candidate availability for a professional, offer, and date range. | MUST |
| FR-AVL-005 | Reject invalid schedule intervals and overlapping incompatible blocks according to defined rules. | MUST |
| FR-AVL-006 | Warn/reject schedule changes that conflict with existing appointments; do not silently invalidate bookings. | MUST |
| FR-AVL-007 | Publish schedule and block changes for relevant consumers. | SHOULD |
| FR-AVL-008 | Do not treat an availability response as an atomic reservation. | MUST |

### 4.6 Domain 6 — Appointments & Attendance

| ID | Requirement | Priority |
|---|---|---|
| FR-APT-001 | Create a booking linked to a client, professional, offer, time interval, and agreed commercial snapshot. | MUST |
| FR-APT-002 | Validate current offer eligibility and current availability before confirming a booking. | MUST |
| FR-APT-003 | Prevent concurrent requests from confirming incompatible intervals for one professional. | MUST |
| FR-APT-004 | Maintain a documented state machine for pending, confirmed, cancelled, completed, and no-show states as approved. | MUST; exact states PENDING |
| FR-APT-005 | Enforce valid state transitions and record important changes. | MUST |
| FR-APT-006 | Support client cancellation according to the approved policy and record the reason/effects where applicable. | MUST |
| FR-APT-007 | Let an authorized professional mark attendance completed/no-show according to the approved workflow. | SHOULD; no-show policy PENDING |
| FR-APT-008 | Publish appointment lifecycle events for payment, reviews, notifications, and analytics. | MUST |
| FR-APT-009 | Keep appointment state distinct from payment state. | MUST |

### 4.7 Domain 7 — Payments & Payouts

| ID | Requirement | Priority |
|---|---|---|
| FR-PAY-001 | Start online payment through an approved provider for the relevant appointment. | MUST |
| FR-PAY-002 | Store the provider transaction reference, amount, currency, state, and relevant timestamps. | MUST |
| FR-PAY-003 | Process verified webhooks/callbacks idempotently. | MUST |
| FR-PAY-004 | Distinguish payment status from appointment status. | MUST |
| FR-PAY-005 | Implement refunds, commissions, and payouts according to an approved policy and provider capability. | MUST scope/PENDING business decisions |
| FR-PAY-006 | Provide authorized views of transaction and payout records for support/reconciliation. | MUST |
| FR-PAY-007 | Never represent a payment as approved without a trusted provider result. | MUST |
| FR-PAY-008 | Avoid storing complete payment-card credentials when they can remain with the provider. | MUST |

### 4.8 Domain 8 — Reviews & Reputation

| ID | Requirement | Priority |
|---|---|---|
| FR-REV-001 | Accept a review only from a client associated with an eligible appointment. | MUST |
| FR-REV-002 | Store the rating, optional comment, reviewer, professional, appointment, and timestamps. | MUST |
| FR-REV-003 | Enforce the approved rating scale and one-review-per-appointment policy if adopted. | MUST; scale/policy to confirm |
| FR-REV-004 | Calculate and expose a reputation summary from eligible visible reviews. | SHOULD |
| FR-REV-005 | Apply authorized visibility/moderation changes while preserving relevant audit history. | MUST |
| FR-REV-006 | Publish review lifecycle events for search and analytics projections. | SHOULD |

### 4.9 Domain 9 — Reports & Moderation

| ID | Requirement | Priority |
|---|---|---|
| FR-MOD-001 | Create reports for defined target types such as user, profile, review, or appointment. | SHOULD |
| FR-MOD-002 | Track each report/case through documented statuses and store its history. | SHOULD |
| FR-MOD-003 | Restrict case details and moderation actions to authorized staff. | MUST |
| FR-MOD-004 | Record decision, reason, moderator, timestamp, and any resulting action request. | MUST |
| FR-MOD-005 | Request profile/account/review actions from their owning domain rather than modifying their data directly. | MUST |
| FR-MOD-006 | Notify the appropriate operational parties according to a published moderation policy. | PENDING channel/policy details |

### 4.10 Domain 10 — Analytics & Indicators

| ID | Requirement | Priority |
|---|---|---|
| FR-ANL-001 | Calculate basic appointment counts and statuses for selected periods. | SHOULD |
| FR-ANL-002 | Calculate cancellations and completed appointments using documented definitions. | SHOULD |
| FR-ANL-003 | Show eligible review count and average rating. | SHOULD |
| FR-ANL-004 | Show occupied and available schedule indicators where source data permits reliable calculation. | SHOULD |
| FR-ANL-005 | Present relevant payment/volume indicators without exposing unauthorized financial details. | SHOULD |
| FR-ANL-006 | Show the measurement period, time zone, and last projection update when useful. | SHOULD |
| FR-ANL-007 | Consume source-domain events and rebuild projections without changing source records. | SHOULD |

### 4.11 Domain 11 — Platform Administration & Audit Operations

This is initially an administrative application/BFF capability rather than a mandatory standalone business microservice.

| ID | Requirement | Priority |
|---|---|---|
| FR-ADM-001 | Provide an administrative interface protected by server-side role/permission checks. | MUST |
| FR-ADM-002 | Allow authorized staff to view and manage users through Identity's supported operations. | MUST |
| FR-ADM-003 | Allow authorized staff to review professional profiles through Profiles' supported operations. | MUST |
| FR-ADM-004 | Allow authorized staff to maintain categories through Catalog's supported operations. | MUST |
| FR-ADM-005 | Allow authorized staff to investigate appointment/payment issues using owner-domain APIs and proper permissions. | MUST |
| FR-ADM-006 | Link moderators to the Moderation domain's case workflow instead of duplicating it. | MUST |
| FR-ADM-007 | Record important administrative actions, including actor, operation, timestamp, and result; capture reason when policy requires it. | SHOULD |
| FR-ADM-008 | Avoid direct writes to other domains' databases. | MUST |

## 5. Cross-domain integration requirements

| ID | Requirement | Priority |
|---|---|---|
| INT-001 | Each domain must remain the authoritative owner of its own data; other domains must not write directly to its database. | MUST |
| INT-002 | Kafka event names and payloads must be versioned and documented. | MUST |
| INT-003 | Every event should carry an event ID, event type/version, occurred-at time, entity ID, and correlation ID; causation ID should be provided where relevant. | MUST |
| INT-004 | Consumers must be idempotent because duplicate delivery is possible. | MUST |
| INT-005 | Failed event handling must have documented retry and dead-letter procedures and monitoring. | MUST before production |
| INT-006 | Events must not include passwords, access tokens, payment-card credentials, or unnecessary personal data. | MUST |
| INT-007 | A state-changing database operation and its outbound event must not silently diverge; evaluate transactional outbox or equivalent reliability pattern. | MUST before production |
| INT-008 | Synchronous calls are required where a current authoritative answer is needed; Kafka must not be used as ad-hoc synchronous RPC. | MUST |
| INT-009 | Cross-domain workflows must document partial failure and compensation behavior, especially for booking/payment/refund. | MUST before payment launch |

### Proposed initial event contracts (names are provisional)

- `identity.account-created.v1`
- `identity.account-status-changed.v1`
- `profiles.professional-profile-activated.v1`
- `profiles.professional-profile-restricted.v1`
- `catalog.service-offer-published.v1`
- `catalog.service-offer-updated.v1`
- `availability.schedule-updated.v1`
- `availability.time-block-changed.v1`
- `appointments.appointment-created.v1`
- `appointments.appointment-confirmed.v1`
- `appointments.appointment-cancelled.v1`
- `appointments.appointment-completed.v1`
- `payments.payment-status-changed.v1`
- `reviews.review-published.v1`
- `moderation.case-decided.v1`

These names are a proposed vocabulary, not a final event schema. Each event needs a schema, producer, consumer, version policy, and failure strategy before implementation.

## 6. Non-functional requirements

| ID | Category | Requirement | Validation approach |
|---|---|---|---|
| NFR-SEC-001 | Security | Passwords must never be persisted in plaintext; use a maintained password-hashing mechanism. | Implementation review and authentication tests. |
| NFR-SEC-002 | Authorization | Enforce permissions on server-side endpoints/use cases, not just in the UI. | Negative role and ownership tests. |
| NFR-SEC-003 | Secrets | Never commit passwords, signing keys, provider credentials, or production tokens. | Secret scanning and configuration review. |
| NFR-PRV-001 | Privacy | Collect only data needed for defined purposes and restrict access to private profile/financial data. | Data-flow review and access tests. |
| NFR-PRV-002 | LGPD | Provide transparent privacy information and a documented process for applicable data-subject requests. | Product/legal review before launch. |
| NFR-DATA-001 | Integrity | Booking conflict prevention must be enforced at the authoritative write boundary, including concurrent requests. | Concurrent integration test. |
| NFR-DATA-002 | Integrity | Payment callbacks and Kafka events must be safe to process more than once. | Duplicate/replay tests. |
| NFR-PERF-001 | Performance | Initial target proposal: 95% of common internal application requests complete within 3 seconds under an agreed load, excluding third-party latency. | Load test with documented environment and workload. |
| NFR-UX-001 | Usability | Primary tasks must provide clear loading, success, empty, validation, and error states. | Usability/acceptance tests. |
| NFR-RESP-001 | Responsiveness | Primary journeys must be operable on supported mobile and desktop viewports. | Responsive UI tests. |
| NFR-ACC-001 | Accessibility | Target WCAG 2.2 AA for primary journeys where achievable within the agreed scope. | Automated and manual accessibility checks. |
| NFR-OBS-001 | Observability | Critical failures, failed event processing, and external integration errors must be logged/monitored without unnecessary personal data. | Monitoring and log inspection. |
| NFR-MAINT-001 | Maintainability | Domain rules must be testable without requiring a web server, Kafka broker, or external payment provider. | Unit-test structure/code review. |
| NFR-TEST-001 | Testability | Critical rules for permissions, offers, bookings, payments, and review eligibility must have automated tests. | CI test report. |
| NFR-COMP-001 | Compatibility | Supported browser versions and API compatibility policy must be documented. | Browser and contract test matrix. |
| NFR-AVAIL-001 | Availability | A production availability target and recovery objectives must be set before public launch; do not promise an SLA without supporting operations. | Operations readiness review. |

## 7. MVP exclusions

The first release does not include an internal photo portfolio, AI recommendations/matching, predictive demand, advanced loyalty, complex multi-establishment/franchise administration, automated marketing, or advanced professional financial management. External social profile links are allowed. Items may be reconsidered after usage evidence.

## 8. Unresolved business decisions

| ID | Decision | Affected requirements |
|---|---|---|
| DEC-01 | Payment timing and supported methods | FR-PAY-001, FR-APT-002, booking flow |
| DEC-02 | Payment provider and fees | FR-PAY-001–008 |
| DEC-03 | Commission and payout schedule | FR-PAY-005, FR-ADM-005 |
| DEC-04 | Cancellation deadline, penalties, refund policy, and no-show handling | FR-APT-004–007, payment compensation |
| DEC-05 | Automatic booking confirmation versus professional acceptance | FR-APT-001–005 |
| DEC-06 | Professional identity verification | FR-PROF-008 |
| DEC-07 | Mandatory profile fields and public location precision | FR-PROF-004, FR-DSC-002 |
| DEC-08 | Search ranking and “available now” definition | FR-DSC-002, FR-DSC-005 |
| DEC-09 | Notification channels | cross-domain integrations and UX |
| DEC-10 | Rating scale, one-review policy, editing/replies | FR-REV-001–006 |
| DEC-11 | Deployment strategy: independent microservices versus modular monolith for early MVP | Architecture and operations |

## 9. Approval and change control

A requirement is approved only after the product owner confirms its behavior and relevant pending decisions are resolved. Changes should preserve IDs when possible, document the reason, and update related use cases, event contracts, tests, and traceability. Requirements are not proof of implementation; completion is demonstrated by passing agreed acceptance and automated tests.
