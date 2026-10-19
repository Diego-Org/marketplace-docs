# Project Domains — Beauty Services Marketplace

**Document ID:** ARCH-DOM-001  
**Version:** 1.0  
**Status:** Proposed baseline for review  
**Product:** Marketplace for beauty and personal-care services

## 1. Purpose

This document defines the project's logical business domains, their ownership boundaries, important entities, dependencies, and recommended implementation sequence. A logical domain is not automatically a good standalone microservice: independent deployment should be justified by cohesive business responsibility, independent data ownership, and a manageable integration cost.

The marketplace's primary journey is:

**Find a professional → compare an offer → choose a time → book → pay → receive the service → review.**

The current list contains 11 logical domains. The review below retains all 11 capabilities but adjusts responsibilities and recommends that not every capability become a separately deployed microservice in the first release.

## 2. Main corrections and decisions

1. **Appointment and availability have a strong consistency relationship.** A displayed slot can become stale, and Kafka is asynchronous. If separate services own schedule availability and confirmed appointments from day one, the system needs a carefully designed reservation/hold protocol to prevent double booking. For the MVP, implement domains 5 and 6 as two modules within one **Scheduling & Appointments Service**. Keep their domain models separate so they can be split later if operational needs justify it.
2. **Domain 11, Platform Administration, is primarily an administrative application/capability, not a natural owner of all the data it manages.** User data remains owned by Identity; profile status remains owned by Profiles; categories and offers remain owned by Catalog; moderation cases remain owned by Moderation. Do not build an administration microservice that duplicates their business rules. Start with an Admin UI/BFF or application layer that calls the owner services. A centralized audit capability can be added if it has a distinct, well-defined need.
3. **Discovery and Analytics are read-side capabilities.** They build searchable/queryable views from data owned by other domains. They must not become alternative sources of truth for profile, catalog, appointment, payment, or review records. They can initially be modules or simple read models and become independent services when scale, latency, or team ownership justifies it.
4. **Payments must not own booking state.** A payment can be pending or refunded after an appointment is cancelled. Their state machines are separate and correlated by IDs.
5. **Moderation does not directly own accounts or reviews.** It owns reports and moderation cases; the responsible owner service executes suspension, profile restriction, or review hiding through an authorized operation.
6. **Kafka is for asynchronous domain events, not synchronous questions.** Login, immediate authorization, final slot validation, and payment initiation usually require a synchronous result. Events support projections, notifications, audit and cross-domain reactions. Consumer idempotency, retries, dead-letter handling, event versioning, and correlation IDs must be planned.
7. **Do not treat “MVC first, hexagonal later” as a reason to mix business rules with controllers or framework classes.** In the MVC implementation, keep domain rules in application/domain services and cover them with unit tests. This makes the subsequent migration a boundary refactor rather than a rewrite.

## 3. Recommended first implementation set

### Recommended: four deployable services spanning five logical domains

| Initial service | Logical domains covered | Reason |
|---|---|---|
| 1. Identity & Access | Domain 1 | Provides account identity, authentication, roles, and authorization foundations. |
| 2. Profiles & Professionals | Domain 2 | Enables client and professional profiles and professional activation state. |
| 3. Catalog & Offers | Domain 3 | Lets professionals publish services, prices, and durations. |
| 4. Scheduling & Appointments | Domains 5 + 6 | Delivers the core booking journey while keeping slot availability and booking conflict rules transactionally close. |

This adjusts the proposed selection of **1, 2, 3, and 5**: include **domain 6 (Appointments)** inside the fourth service instead of treating domain 5 as an isolated service. This is the stronger option because a schedule without a booking owner cannot validate real booking conflicts end to end. The codebase can still maintain separate `availability` and `appointments` modules and separate use cases/aggregates.

### If the team insists on keeping domain 5 as its own microservice

That is possible, but its initial scope should be explicitly limited to working hours, exceptions, and blocks. Domain 5 must not claim that a slot is definitively bookable based only on an asynchronous projection of appointments. When domain 6 is introduced, domain 6 must perform an authoritative, atomic conflict check or implement a robust reservation protocol. The user-visible booking flow will remain incomplete until domain 6 exists.

### Important distinction: architecture practice versus product delivery

If the primary goal is learning microservices, Kafka, TDD, MVC, and later hexagonal architecture, the recommended four-service phase is a reasonable training sequence. If the primary goal is to launch the business quickly, 11 independently deployed microservices are likely unnecessary operational overhead; a modular monolith with these bounded contexts is a lower-risk starting point. The domain map remains useful in either architecture.

## 4. Domain catalogue

### Domain 1 — Identity & Access

**Purpose:** Own account identity, credentials, authentication, roles, and access policy.  
**Core entities:** `Account`, `Credential`, `RoleAssignment`, `ExternalIdentity`, `RecoveryChallenge` (names are illustrative).  
**Owns:** account ID, login identifiers, credential hashes, authentication lifecycle, roles/claims, account-level enabled/disabled status.  
**Does not own:** client profile details, professional public profile, services, appointments, payments, or moderation case content.

**Responsibilities**
- Register and authenticate users.
- Recover access and invalidate sessions/tokens where applicable.
- Support social login only if selected for the release.
- Represent the roles `CLIENT`, `PROFESSIONAL`, and `ADMIN` and enforce authorization policies.
- Publish minimal account lifecycle events for other domains.

**Inputs/dependencies:** identity credentials from the user; optional external identity provider.  
**Outputs/events:** `identity.account-created.v1`, `identity.account-status-changed.v1`, `identity.role-assigned.v1`.  
**Critical rules:** no plaintext passwords; unique normalized login identifier; server-side authorization; account ID is stable and never reused.

### Domain 2 — Profiles & Professionals

**Purpose:** Own client profile data and the professional's public presentation and eligibility to be listed.  
**Core entities:** `ClientProfile`, `ProfessionalProfile`, `SocialLink`, `ProfessionalStatus`.  
**Owns:** display name, profile/contact fields approved for the product, public description, service area/location at the chosen precision, external social links, public profile status.  
**Does not own:** authentication credentials, category definitions, individual service offers, schedule, bookings, or payment payout state.

**Responsibilities**
- Create/update the client profile linked to an `accountId`.
- Create/update a professional profile and its public information.
- Track whether the profile is draft, active, suspended, or otherwise eligible to be listed.
- Validate mandatory profile fields before publishing.
- Store links to external social profiles; no internally hosted work gallery in the MVP.

**Dependencies:** Identity for account identity/role; Catalog and Discovery consume professional identifiers/status.  
**Outputs/events:** `profiles.client-profile-updated.v1`, `profiles.professional-profile-activated.v1`, `profiles.professional-profile-restricted.v1`.  
**Critical rules:** public fields must be distinguished from private fields; identity verification remains a business decision, not an assumed requirement.

### Domain 3 — Service Catalog & Offers

**Purpose:** Own the platform taxonomy and the offers each professional actually sells.  
**Core entities:** `Category`, `ServiceDefinition`, `ServiceOffer`, optional `OfferOption`/`ComboDefinition` in a future version.  
**Owns:** standard category/service definitions, professional-specific offer name/description, price, duration, active state, and relationship to the professional.  
**Does not own:** full professional profile, schedule, bookings, payments, search ranking, or ratings.

**Responsibilities**
- Maintain categories such as hair, beard, nails, makeup, eyebrows, and aesthetics.
- Maintain reusable service definitions while allowing different professional offerings and levels of detail.
- Let professionals create, edit, activate, and deactivate their own offers.
- Store price and estimated duration per offer.
- Preserve historical offer/price snapshots referenced by an existing appointment.

**Important model distinction:** `ServiceDefinition` is the platform's generic concept; `ServiceOffer` is the specific professional's commercial offer. Different professionals can charge different prices and define different durations for related services. The taxonomy must not force every professional to use identical service granularity.

**Dependencies:** professional profile ID/status; appointments consume immutable snapshots or identifiers plus agreed terms.  
**Outputs/events:** `catalog.category-changed.v1`, `catalog.service-offer-published.v1`, `catalog.service-offer-updated.v1`, `catalog.service-offer-deactivated.v1`.  
**Critical rules:** only eligible professionals can publish; active offers require valid price and positive duration; changes must not rewrite historical booked terms.

### Domain 4 — Discovery & Search

**Purpose:** Help clients find and compare active professionals and offers.  
**Core components/data:** search query, filters, result projection, ordering strategy.  
**Owns:** search/read models, filter semantics, and ranking/query behavior—not the source profile, service, schedule, or review records.

**Responsibilities**
- Search by category, service, or professional name.
- Filter by service area/location, price, rating, and availability where supported by reliable data.
- Show public profile and offer summaries from read models.
- Apply transparent default ordering; paid placement is not assumed for the MVP.
- Handle empty results and suggest filter adjustments.

**Dependencies:** Profile, Catalog, Availability/Appointments, Reviews; consumes events to update projections.  
**Outputs:** search results and query telemetry. It may publish `discovery.index-updated.v1` if useful internally.  
**Boundary warning:** results may be eventually consistent. Search results are suggestions; the appointment service must revalidate availability at booking time. For an initial MVP, this can be a module/read model rather than a standalone microservice.

### Domain 5 — Agenda & Availability

**Purpose:** Represent when a professional intends and is able to provide services.  
**Core entities:** `WorkingHours`, `AvailabilityException`, `TimeBlock`, `SchedulePolicy`.  
**Owns:** recurring working hours, time zone, exceptions, breaks, and manual blocks.  
**Does not own:** confirmed appointment lifecycle; that belongs to Domain 6, or to the combined Scheduling & Appointments Service in the initial deployment.

**Responsibilities**
- Configure recurring work schedules and exceptions.
- Add, edit, and remove blocked periods.
- Calculate candidate intervals based on service duration and scheduling rules.
- Expose availability queries for a given professional, offer, and date range.
- Reject schedule changes that would silently violate existing confirmed bookings, with the final interaction coordinated with appointments.

**Dependencies:** service duration from Catalog; confirmed bookings from Domain 6 or its same-service module.  
**Outputs/events:** `availability.schedule-updated.v1`, `availability.time-block-changed.v1`.  
**Critical rule:** availability is not a booking. A slot shown as free must be validated atomically at confirmation; an asynchronous Kafka projection alone is insufficient to prevent double booking.

### Domain 6 — Appointments & Attendance

**Purpose:** Own the concrete reservation and its lifecycle.  
**Core entities:** `Appointment`, `AppointmentStatusHistory`, `AttendanceRecord` (illustrative names).  
**Owns:** customer/professional/offer identifiers, date/time interval, price/duration snapshot, appointment state, cancellation metadata, attendance completion/no-show.  
**Does not own:** payment transaction state, generic service catalog definitions, or schedule templates.

**Responsibilities**
- Create an appointment after validating eligibility and current availability.
- Prevent conflicting reservations for the same professional and time interval.
- Preserve the contracted service, price, and duration at booking time.
- Support permitted confirmation, cancellation, completion, and no-show transitions.
- Keep appointment state independent from payment state.
- Publish lifecycle events for payment workflows, notifications, reviews, and analytics.

**Dependencies:** Identity/Profile, Catalog, Availability; Payment integrates by appointment ID.  
**Outputs/events:** `appointments.appointment-created.v1`, `appointments.appointment-confirmed.v1`, `appointments.appointment-cancelled.v1`, `appointments.appointment-completed.v1`, `appointments.appointment-no-show.v1`.  
**Critical rule:** final conflict prevention is an authoritative invariant, not a best-effort search query. If this domain is separated from availability, use a reservation protocol and document its timeout/release behavior.

### Domain 7 — Payments & Payouts

**Purpose:** Own financial transactions, refunds, commissions, and professional payouts.  
**Core entities:** `Payment`, `Transaction`, `Refund`, `Payout`, `PlatformFee`.  
**Owns:** financial status, provider reference, amount/currency, commission calculation inputs, refund/payout records, reconciliation identifiers.  
**Does not own:** appointment status or appointment schedule.

**Responsibilities**
- Initiate payment through an approved provider.
- Process verified provider callbacks/webhooks idempotently.
- Track pending, approved, rejected, cancelled, refunded, and other provider-supported states.
- Apply the approved commission and payout model.
- Reconcile transactions and surface discrepancies to authorized operators.

**Dependencies:** Appointment lifecycle and payment provider.  
**Outputs/events:** `payments.payment-status-changed.v1`, `payments.refund-status-changed.v1`, `payments.payout-status-changed.v1`.  
**Pending decisions:** provider, payment timing, supported methods, platform commission, payout timing, refund/cancellation policy. No option in this list should be silently assumed.

### Domain 8 — Reviews & Reputation

**Purpose:** Own customer feedback tied to eligible completed appointments.  
**Core entities:** `Review`, `ReviewReply` (if approved), `ReputationSummary`.  
**Owns:** rating, optional comment, association to appointment/customer/professional, review visibility state, aggregated public rating.  
**Does not own:** appointment records, moderation cases, or customer account lifecycle.

**Responsibilities**
- Validate that a customer can review an eligible appointment.
- Enforce duplicate-review policy.
- Store the rating and optional comment.
- Calculate/refresh the professional's reputation summary.
- Apply visibility changes received from authorized moderation decisions.

**Dependencies:** Appointment completion event; Moderation.  
**Outputs/events:** `reviews.review-submitted.v1`, `reviews.review-published.v1`, `reviews.review-hidden.v1`.  
**Critical rule:** an evaluation must be linked to a real eligible appointment, not merely to a public professional profile.

### Domain 9 — Reports & Moderation

**Purpose:** Receive complaints and manage moderation cases across profiles, reviews, and marketplace activity.  
**Core entities:** `Report`, `ModerationCase`, `ModerationDecision`, `EvidenceReference`.  
**Owns:** report submission, case status, moderator decision, reason, timestamps, and case history.  
**Does not own:** the underlying user account, professional profile, review, appointment, or payment.

**Responsibilities**
- Record reports concerning a user, professional profile, review, or appointment.
- Deduplicate/triage reports where practical.
- Allow authorized staff to investigate and record a decision.
- Request or trigger actions in owner domains, such as hiding a review or restricting a profile.
- Keep a record of the decision and resulting command/event correlation.

**Dependencies:** Identity, Profiles, Reviews, Appointments, and Admin operations.  
**Outputs/events:** `moderation.report-created.v1`, `moderation.case-decided.v1`, `moderation.action-requested.v1`.  
**Boundary warning:** the moderation service records the decision; the owning service executes the actual account/profile/review state change.

### Domain 10 — Analytics & Indicators

**Purpose:** Provide operational/business indicators without becoming the source of operational truth.  
**Core components:** `ProfessionalMetricsProjection`, `PlatformMetricsProjection`, reporting queries.  
**Owns:** projections, aggregations, metric definitions, reporting query behavior.  
**Does not own:** raw appointments, payment transactions, reviews, profiles, or offers.

**Responsibilities**
- Calculate booking counts, completed visits, cancellations, review averages, occupied/available time, and platform-generated transaction volumes where data permits.
- Offer basic professional and platform dashboards.
- Document each metric's definition, time zone, period, and handling of cancellations/refunds.
- Consume events and rebuild projections if required.

**Dependencies:** Appointment, Payment, Review, Catalog, Availability events.  
**Outputs:** reports and metrics; no changes to source records.  
**Boundary warning:** analytics data may be eventually consistent and must display an appropriate “updated at” time. In the early MVP this may be a module/read model rather than a standalone microservice.

### Domain 11 — Platform Administration & Audit Operations

**Purpose:** Provide an authorized administrative experience and cross-domain operational workflows without taking ownership of other domains' entities.  
**Recommended form:** Admin web application/BFF plus audited administrative use cases. Do not duplicate user, category, appointment, payment, or moderation databases.

**Responsibilities**
- Present authorized workflows to manage users, professionals, categories, reports, appointments, and transaction investigations.
- Call each owning domain's API or publish an approved command through its defined contract.
- Enforce admin access through Identity and server-side permissions.
- Record who performed important administrative actions, when, why, and with what result. Audit storage can be part of a shared platform capability or a separate service only when justified.
- Avoid creating alternative CRUD paths that bypass domain rules.

**Owns:** admin UI/BFF orchestration and, if intentionally isolated, administrative audit records.  
**Does not own:** canonical entities managed by domains 1–10.  
**Dependencies:** all domains that expose administrative operations; Identity for authorization; Moderation for case decisions.  
**Recommendation:** treat this as an application capability, not an eleventh business microservice in the first release. If a dedicated audit service is eventually created, define it as “Audit Trail” with a clear event-retention and access-control responsibility.

## 5. Domain ownership matrix

| Data/capability | Authoritative owner | Other domains may |
|---|---|---|
| Account ID, credential, roles | 1. Identity & Access | Consume minimal identity events or call authorized APIs |
| Client/professional profile | 2. Profiles & Professionals | Build public/read projections |
| Categories, service definitions, offers, offer price/duration | 3. Catalog & Offers | Read/query projections; appointments keep booked snapshots |
| Search index and result ordering | 4. Discovery & Search | Read source-domain events; never update source entities directly |
| Work hours, exceptions, manual blocks | 5. Agenda & Availability | Query candidate availability |
| Reservation, appointment lifecycle, attendance | 6. Appointments | Consume lifecycle events |
| Payment, refund, commission, payout | 7. Payments & Payouts | Consume payment status; no ownership of appointment status |
| Ratings, comments, rating summary | 8. Reviews & Reputation | Moderation may request visibility change |
| Reports and moderation decisions | 9. Reports & Moderation | Send authorized action requests to owning domains |
| Aggregated metrics/projections | 10. Analytics & Indicators | Read event-derived data; no source updates |
| Administrative workflows/audit | 11. Admin application/operations | Invoke owner-domain contracts; no duplicate domain DB |

## 6. Communication rules

### Use synchronous APIs when
- The caller needs an immediate success/failure result (login, current permission check, final booking confirmation, payment initiation).
- The caller needs an authoritative value and stale data would be unsafe.
- A user is waiting for a command to complete.

### Use Kafka events when
- A domain has committed a state change and other domains need to react.
- Search/analytics projections need to be updated.
- Notifications, audit, or downstream workflows can happen asynchronously.

### Minimum event conventions
- Use explicit versioned names, such as `appointments.appointment-confirmed.v1`.
- Include event ID, event type/version, occurred-at timestamp, aggregate/entity ID, correlation ID, and causation ID where applicable.
- Include only the minimum data required; do not publish passwords, tokens, full payment credentials, or unnecessary personal data.
- Consumers must be idempotent. Assume at-least-once delivery.
- Define retry policy and dead-letter handling; monitor lag and failed messages.
- Consider the transactional outbox pattern to avoid committing a database change without publishing its event.
- Do not use Kafka as an improvised synchronous RPC mechanism.

## 7. Recommended build sequence

### Phase A — first four deployable services
1. Identity & Access.
2. Profiles & Professionals.
3. Catalog & Offers.
4. Scheduling & Appointments (domains 5 and 6 as separate modules inside the service).

Build a thin end-to-end flow: register/login → professional profile → offer → work schedule → appointment creation with conflict prevention. Payment can be represented by a test double only in local tests; it must not be represented as a completed payment in the product.

### Phase B — complete the MVP core
5. Payments & Payouts.
6. Reviews & Reputation.
7. Reports & Moderation.
8. Discovery & Search, if the basic query module is no longer sufficient.

### Phase C — operations and scale
9. Analytics & Indicators.
10. Admin workflows/audit refinements.
11. Split search, availability, or other modules into standalone services only when independent scaling, ownership, or reliability requirements justify the added distributed-systems cost.

The phase numbers are an implementation sequence, not a claim that the logical domains have changed their IDs.

## 8. MVC, TDD, Kafka, and hexagonal migration

- It is acceptable to expose controllers in MVC, but keep business decisions out of controllers.
- Unit-test domain rules and application services before infrastructure integration tests.
- Define explicit service interfaces for cross-module and cross-service calls; do not make domain code depend on Kafka classes or web controllers.
- Add Kafka contract/consumer tests for event-producing and event-consuming flows.
- Before migration to hexagonal architecture, establish characterization tests and a green regression suite.
- Move external dependencies behind ports/adapters gradually; keep public API/event contracts stable during the refactor.
- Do not combine the architecture migration with a large business-rule redesign. Separate refactoring changes from behavior changes.

## 9. Decisions still required

- Payment provider, allowed methods, charge timing, commission, payout, refund and cancellation policy.
- Whether professional identity verification is required and at what stage.
- Booking confirmation strategy: automatic or professional acceptance.
- Exact appointment state machine and no-show policy.
- Location precision and ranking behavior in discovery.
- Notification channels for the MVP.
- Rating scale, edit/reply policy, and moderation policy.
- Deployment model: learning-oriented microservices versus a lower-complexity modular monolith.

No pending item should be inferred as an approved business rule until the product owner confirms it.
