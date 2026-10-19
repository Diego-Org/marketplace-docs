# Test Artifacts — Beauty Services Marketplace

**Document ID:** TEST-ART-001  
**Version:** 1.0  
**Status:** Proposed test baseline for TDD

## 1. Purpose

This document defines test layers, artifacts, and domain-level scenarios to use while implementing the marketplace. Tests should be written before or alongside behavior under a Test-Driven Development (TDD) cycle: **Red → Green → Refactor**. A requirement is not considered verified merely because a controller or screen exists.

The first implementation recommendation is four deployable services: Identity, Profiles, Catalog, and a Scheduling & Appointments service with two internal modules. If the team instead deploys Domain 5 separately, keep the authoritative conflict tests in Domain 6 and add explicit contract/reservation tests between them.

## 2. Test strategy

### 2.1 Test layers

| Layer | Purpose | Typical dependencies | Naming suggestion |
|---|---|---|---|
| Unit/domain test | Verify business invariants and state transitions quickly. | No web server, broker, database, or real provider. | `...Test` or `...Tests` according to project convention |
| Application/service test | Verify orchestration, validation, permission checks, and handling of collaborator outcomes. | Fakes/mocks of ports or service interfaces. | `...ServiceTest` |
| Repository/integration test | Verify persistence, constraints, transactions, and mapping. | Test database/container. | `...RepositoryIT` |
| API/controller test | Verify request validation, HTTP status/shape, authentication and authorization integration. | Web test framework, test auth. | `...ControllerTest` |
| Kafka contract test | Verify event schema, headers, compatibility and consumer behavior. | Embedded/container Kafka or contract harness where needed. | `...EventContractTest` / `...ConsumerTest` |
| End-to-end test | Verify complete user journey across the agreed set of services. | Test environment, test doubles or sandbox provider. | `...E2ETest` |
| Non-functional test | Verify load, resilience, security, accessibility, and observability targets. | Staging/performance/security tools. | Separate test plan/report |

The exact Java test framework and package names are implementation decisions. Test names below are illustrative artifacts, not required production class names.

### 2.2 TDD cycle

1. Select one requirement and a concrete failure/edge scenario.
2. Write a failing test that expresses the expected behavior.
3. Implement the smallest change to pass it.
4. Refactor with the test suite still green.
5. Add integration/contract tests for boundaries that unit tests cannot verify.
6. Keep a link from the test back to a requirement ID (for example, `FR-APT-003`).

### 2.3 Test data and environment

- Use synthetic accounts and professional/customer identifiers; never use production personal data in ordinary test suites.
- Freeze or inject clock/time sources for expiry, time-zone, cancellation, and reporting-period tests.
- Use a deterministic test payment adapter for most tests; use provider sandbox tests for webhook and lifecycle integration.
- Test Kafka consumers with duplicate, delayed, out-of-order, malformed, and unsupported-version events.
- Seed working hours, blocks, offers, and appointments explicitly. Avoid tests that depend on the actual current time.
- Ensure each test can clean up its data and run independently where practical.

## 3. Domain 1 — Identity & Access

### Suggested artifacts
- `AccountRegistrationTest`
- `AuthenticationServiceTest`
- `RecoveryChallengeTest`
- `RoleAuthorizationTest`
- `AccountRepositoryIT`
- `IdentityControllerTest`
- `AccountCreatedEventContractTest`
- `AccountEventConsumerIdempotencyTest`

### Scenarios

| ID | Scenario | Expected result |
|---|---|---|
| T-IAM-001 | Register with valid required fields. | Account is created once and a stable account ID is returned. |
| T-IAM-002 | Register a duplicate normalized email/login identifier. | Request is rejected; no second account is created. |
| T-IAM-003 | Register with missing/invalid required fields. | Validation errors are returned; no invalid account is persisted. |
| T-IAM-004 | Verify stored credential data after password registration. | Plaintext password is not persisted or logged. |
| T-IAM-005 | Authenticate with correct credentials for an enabled account. | Authentication result contains only approved identity/role claims. |
| T-IAM-006 | Authenticate with incorrect credentials. | Access is denied without leaking sensitive account details. |
| T-IAM-007 | Authenticate a disabled/suspended account. | Access is denied according to policy. |
| T-IAM-008 | Recover access with expired, reused, or invalid challenge. | Recovery is rejected and no credential changes. |
| T-IAM-009 | Caller attempts an admin operation without ADMIN permission. | Server denies the operation. |
| T-IAM-010 | Publish/consume a duplicate account-created event. | Consumer does not create duplicate downstream profiles or projections. |
| T-IAM-011 | Role change event contains a credential/token field. | Contract validation rejects the event schema; sensitive data is never emitted. |

## 4. Domain 2 — Profiles & Professionals

### Suggested artifacts
- `ClientProfileServiceTest`
- `ProfessionalProfileLifecycleTest`
- `SocialLinkValidationTest`
- `ProfileVisibilityAuthorizationTest`
- `ProfileRepositoryIT`
- `ProfessionalProfileEventContractTest`

### Scenarios

| ID | Scenario | Expected result |
|---|---|---|
| T-PROF-001 | Create client profile linked to an existing account. | Profile is created with the correct account ID. |
| T-PROF-002 | Attempt to create profile against a nonexistent/unauthorized account ID. | Request is rejected. |
| T-PROF-003 | Activate a professional profile with all required fields. | State transition succeeds and activation event is published once. |
| T-PROF-004 | Activate profile missing a required field. | Activation is rejected and the profile remains non-public. |
| T-PROF-005 | Add malformed or unsupported social URL. | Link is rejected or normalized according to policy. |
| T-PROF-006 | Anonymous user requests private client/profile fields. | Private fields are not returned. |
| T-PROF-007 | Professional tries to edit another professional's profile. | Operation is denied. |
| T-PROF-008 | Suspended profile tries to publish/update public status as active. | Invalid transition is denied; only authorized workflow can restore it. |
| T-PROF-009 | Reprocess profile-activated event. | Projection consumer remains idempotent. |
| T-PROF-010 | Identity account is disabled while profile remains active. | Defined integration policy prevents an ineligible profile from remaining newly discoverable after processing; any propagation delay is observable. |

## 5. Domain 3 — Catalog & Offers

### Suggested artifacts
- `CategoryServiceTest`
- `ServiceOfferValidationTest`
- `ProfessionalOfferOwnershipTest`
- `OfferPriceSnapshotTest`
- `CatalogRepositoryIT`
- `OfferEventContractTest`

### Scenarios

| ID | Scenario | Expected result |
|---|---|---|
| T-CAT-001 | Create valid category and service definition. | Definition is stored and available to permitted callers. |
| T-CAT-002 | Deactivate a category/service already referenced historically. | New use is blocked, but historical records remain readable. |
| T-CAT-003 | Create offer with positive price and duration for an eligible professional. | Offer becomes active. |
| T-CAT-004 | Create offer with zero/negative duration or invalid price. | Validation rejects the offer. |
| T-CAT-005 | Professional edits an offer belonging to another professional. | Operation is denied. |
| T-CAT-006 | Two professionals offer the same generic service at different prices/durations. | Both offers coexist with independent commercial terms. |
| T-CAT-007 | Professional gives a more specific name than the taxonomy's generic service. | Accepted if custom offer naming is allowed by the approved rules. |
| T-CAT-008 | Offer is deactivated after a search result is cached. | A new booking attempt is rejected by authoritative validation. |
| T-CAT-009 | Price changes after an appointment is created. | Existing appointment keeps its agreed price/duration snapshot. |
| T-CAT-010 | Duplicate offer-published event is consumed. | Search projection does not duplicate the offer. |

## 6. Domain 4 — Discovery & Search

### Suggested artifacts
- `ProfessionalSearchServiceTest`
- `SearchFilterTest`
- `PublicProfileProjectionTest`
- `SearchControllerTest`
- `SearchEventConsumerTest`
- `SearchProjectionRebuildIT`

### Scenarios

| ID | Scenario | Expected result |
|---|---|---|
| T-DSC-001 | Search for a category/service with matching active offers. | Only eligible matching records are returned. |
| T-DSC-002 | Filter by location, price, rating, or availability with data present. | Results satisfy all supported filters. |
| T-DSC-003 | Send invalid price range or unsupported filter. | Validation message is returned; no misleading results. |
| T-DSC-004 | Search finds no results. | Explicit empty state is returned. |
| T-DSC-005 | Profile has private data or inactive state. | Private fields/ineligible profile are omitted. |
| T-DSC-006 | Offer/profile is deactivated but search projection has not yet consumed event. | Search may be temporarily stale, but booking service rejects the offer; freshness is observable. |
| T-DSC-007 | Same offer event arrives twice. | Search index has one logical record. |
| T-DSC-008 | Events arrive out of order. | Projection uses event versions/timestamps or authoritative refresh strategy to avoid reactivating obsolete data. |
| T-DSC-009 | Search service/index is unavailable. | API fails gracefully and logs/alerts; it does not invent availability. |
| T-DSC-010 | Rebuild projection from a controlled fixture/event history. | Rebuilt results match expected source state. |

## 7. Domain 5 — Agenda & Availability

### Suggested artifacts
- `WorkingHoursValidationTest`
- `AvailabilityCalculationTest`
- `TimeBlockServiceTest`
- `TimeZoneAndBoundaryTest`
- `ScheduleRepositoryIT`
- `AvailabilityEventContractTest`

### Scenarios

| ID | Scenario | Expected result |
|---|---|---|
| T-AVL-001 | Configure a valid recurring schedule and time zone. | Schedule is saved and interpreted in its defined time zone. |
| T-AVL-002 | Configure end time earlier than or equal to start time without an allowed overnight rule. | Schedule is rejected. |
| T-AVL-003 | Define overlapping incompatible work intervals/blocks. | Validation rejects or normalizes them according to an explicit policy. |
| T-AVL-004 | Query slots for a service duration that fits exactly at the schedule boundary. | Slot appears only if the full duration is contained in a valid interval. |
| T-AVL-005 | Query with duration longer than every free interval. | No candidate slot is returned. |
| T-AVL-006 | Add a block that overlaps candidate slots. | Affected candidate slots disappear. |
| T-AVL-007 | Attempt a block that overlaps an existing confirmed appointment. | Operation is rejected or handled by an explicitly approved workflow; no silent booking deletion. |
| T-AVL-008 | Query across a daylight-saving transition or time-zone boundary where relevant. | Intervals are consistent with the selected time-zone rules. |
| T-AVL-009 | Schedule changes while a client has an old candidate list. | Final appointment validation rechecks availability and rejects stale slot if unavailable. |
| T-AVL-010 | Replay a schedule-updated event. | Consumers do not duplicate blocks or schedule records. |

## 8. Domain 6 — Appointments & Attendance

### Suggested artifacts
- `AppointmentCreationTest`
- `AppointmentStateMachineTest`
- `AppointmentCancellationPolicyTest`
- `AppointmentConflictIntegrationTest`
- `AppointmentRepositoryIT`
- `AppointmentEventContractTest`
- `ConcurrentBookingE2ETest`

### Scenarios

| ID | Scenario | Expected result |
|---|---|---|
| T-APT-001 | Create booking for valid client, active profile/offer, available slot. | Appointment stores correct references, interval, state, and agreed commercial snapshot. |
| T-APT-002 | Create booking for inactive profile or deactivated offer. | Booking is rejected. |
| T-APT-003 | Create booking when the slot is no longer available. | Conflict response is returned and no confirmed appointment is stored. |
| T-APT-004 | Two concurrent requests book overlapping intervals for one professional. | At most one conflicting appointment commits as confirmed. |
| T-APT-005 | Two bookings for the same professional have non-overlapping intervals. | Both can succeed if other rules permit. |
| T-APT-006 | Service duration overlaps a neighboring appointment even though start time differs. | Conflicting booking is rejected. |
| T-APT-007 | Client tries to view/cancel another client's appointment. | Access is denied. |
| T-APT-008 | Execute an invalid state transition (for example, cancelled to completed without an approved path). | Transition is rejected and state remains unchanged. |
| T-APT-009 | Cancel appointment under allowed policy. | Cancellation is recorded; downstream payment workflow is triggered separately if applicable. |
| T-APT-010 | Refund is still pending after appointment is cancelled. | Appointment reads cancelled; payment reads pending refund. No state is falsely merged. |
| T-APT-011 | Mark eligible appointment completed. | Completion event is published and review eligibility can be derived. |
| T-APT-012 | Mark a non-eligible appointment no-show or apply penalty before policy is approved. | Operation is rejected or remains disabled until policy is approved. |
| T-APT-013 | Same create command is retried with an idempotency key. | One logical booking is created; retry returns the existing outcome. |
| T-APT-014 | Database save succeeds but event publishing fails. | Outbox/retry strategy eventually publishes the event without duplicating the appointment. |

**Highest-priority invariant:** `T-APT-004` must be a real concurrency/integration test, not merely a mocked unit test.

## 9. Domain 7 — Payments & Payouts

### Suggested artifacts
- `PaymentInitiationServiceTest`
- `ProviderWebhookVerificationTest`
- `PaymentStateMachineTest`
- `RefundPolicyServiceTest`
- `PaymentRepositoryIT`
- `PaymentEventContractTest`
- `PaymentProviderSandboxIT`

### Scenarios

| ID | Scenario | Expected result |
|---|---|---|
| T-PAY-001 | Initiate payment for an eligible checkout with valid amount. | Internal transaction is created and provider request is correlated. |
| T-PAY-002 | Retry payment-initiation request with the same idempotency key. | No duplicate charge is intentionally created. |
| T-PAY-003 | Provider rejects payment. | Payment state becomes rejected/failed according to mapping; no false success is shown. |
| T-PAY-004 | Provider call times out with unknown outcome. | State remains pending/unknown until reconciled; service does not assume success/failure prematurely. |
| T-PAY-005 | Receive valid provider callback. | Callback is authenticated and payment state updated. |
| T-PAY-006 | Receive callback with invalid signature or mismatched amount/currency. | Callback is rejected and financial state is unchanged. |
| T-PAY-007 | Receive the same webhook twice. | One logical financial transition is applied. |
| T-PAY-008 | Receive out-of-order callbacks. | State machine prevents invalid regression and follows provider reconciliation policy. |
| T-PAY-009 | Cancel appointment when payment is approved and refund applies. | Refund operation is initiated and tracked independently. |
| T-PAY-010 | Provider confirms refund only later. | Refund remains pending until trusted confirmation. |
| T-PAY-011 | Query financial record as unauthorized user/professional. | Protected financial information is not exposed. |
| T-PAY-012 | Verify transaction persistence. | Full card credentials are not stored; provider reference and required reconciliation fields are stored. |

## 10. Domain 8 — Reviews & Reputation

### Suggested artifacts
- `ReviewEligibilityTest`
- `ReviewSubmissionServiceTest`
- `RatingAggregationTest`
- `ReviewRepositoryIT`
- `ReviewEventContractTest`

### Scenarios

| ID | Scenario | Expected result |
|---|---|---|
| T-REV-001 | Client reviews an eligible completed appointment. | Review is linked to the correct appointment, client, and professional. |
| T-REV-002 | Client tries to review an appointment owned by another account. | Review is rejected. |
| T-REV-003 | Client tries to review a pending/cancelled/no-show appointment when not eligible. | Review is rejected. |
| T-REV-004 | Client submits a rating outside the approved range. | Validation rejects the rating. |
| T-REV-005 | Client submits a duplicate review for the same appointment. | Duplicate is rejected if one-review-per-appointment is the approved rule. |
| T-REV-006 | Review is hidden by a valid moderation action. | Hidden review no longer contributes to public average under the approved formula. |
| T-REV-007 | Recalculate average after adding/removing a visible review. | Count and average match a known fixture and rounding policy. |
| T-REV-008 | Duplicate review-published event is processed. | Search/analytics projection counts it once. |

## 11. Domain 9 — Reports & Moderation

### Suggested artifacts
- `ReportCreationTest`
- `ModerationCaseStateMachineTest`
- `ModeratorAuthorizationTest`
- `ModerationDecisionAuditTest`
- `ModerationEventContractTest`

### Scenarios

| ID | Scenario | Expected result |
|---|---|---|
| T-MOD-001 | Submit a report with valid target type and reason. | Report receives a reference ID and initial state. |
| T-MOD-002 | Submit report for a nonexistent/unsupported target. | Request is rejected or recorded as an explicitly unverified target according to policy. |
| T-MOD-003 | Unauthenticated/non-moderator opens a moderation case. | Access is denied. |
| T-MOD-004 | Moderator records a decision with required reason. | Decision, moderator, timestamp, and resulting action request are recorded. |
| T-MOD-005 | Moderator attempts an invalid case transition. | Operation is rejected. |
| T-MOD-006 | Owner domain rejects a suspension/hide request. | Case records the rejected/failed action; source record is not falsely shown as modified. |
| T-MOD-007 | Same moderation event is consumed twice. | No duplicate suspension/hide command is applied unintentionally. |
| T-MOD-008 | Inspect logs/events for unnecessary personal data or credentials. | Sensitive data is absent. |

## 12. Domain 10 — Analytics & Indicators

### Suggested artifacts
- `ProfessionalMetricsCalculationTest`
- `PlatformMetricsCalculationTest`
- `MetricsProjectionIdempotencyTest`
- `MetricsAuthorizationTest`
- `MetricsProjectionRebuildIT`

### Scenarios

| ID | Scenario | Expected result |
|---|---|---|
| T-ANL-001 | Aggregate bookings for a known period and fixture. | Count matches documented treatment of states/time zone. |
| T-ANL-002 | Aggregate completed, cancelled, and no-show appointments. | Each is counted according to approved metric definitions; categories are not conflated. |
| T-ANL-003 | Calculate review count/average. | Result matches the Reviews domain's eligible-public-review rules. |
| T-ANL-004 | Calculate payment volume with refunds/cancellations. | Gross/net labels and refund treatment follow documented definitions. |
| T-ANL-005 | Professional queries another professional's indicators. | Access is denied. |
| T-ANL-006 | Process the same appointment/payment event twice. | Aggregates are not double-counted. |
| T-ANL-007 | Projection consumer falls behind. | Lag is observable and freshness is exposed where required. |
| T-ANL-008 | Rebuild projection from an event fixture/history. | Rebuilt results match expected aggregates. |

## 13. Domain 11 — Platform Administration & Audit Operations

### Suggested artifacts
- `AdminPermissionPolicyTest`
- `AdminOwnerServiceDelegationTest`
- `AdministrativeAuditRecordTest`
- `AdminControllerTest`
- `AdminWorkflowE2ETest`

### Scenarios

| ID | Scenario | Expected result |
|---|---|---|
| T-ADM-001 | Authorized admin invokes a permitted owner-domain operation. | Owner service applies its own validation and the result is shown. |
| T-ADM-002 | Non-admin attempts to access administration. | Server rejects access. |
| T-ADM-003 | Admin UI attempts direct database modification or bypass of owner API. | Architecture/code review and integration test demonstrate that no such path exists. |
| T-ADM-004 | Admin command is submitted twice due to retry. | Idempotency or conflict handling prevents unintended duplicate action. |
| T-ADM-005 | Important administrative action succeeds. | Audit record includes actor, action, target, time, result, and required reason/correlation ID. |
| T-ADM-006 | Audit read attempted by unauthorized user. | Access is denied. |
| T-ADM-007 | Admin investigates cancelled appointment with pending refund. | UI shows appointment and payment/refund states separately. |
| T-ADM-008 | Owner service is temporarily unavailable. | Admin UI reports failure/unknown status and does not claim success. |

## 14. Cross-domain, Kafka, and consistency tests

| ID | Scenario | Expected result |
|---|---|---|
| T-INT-001 | Producer emits event after a committed domain change. | Event is eventually available; no event describes a rolled-back state. |
| T-INT-002 | Database commit succeeds while broker is unavailable. | Outbox/retry path eventually publishes the event. |
| T-INT-003 | Consumer receives duplicate event. | Processing is idempotent. |
| T-INT-004 | Consumer receives malformed/unsupported event version. | Event is rejected/quarantined safely; failure is observable. |
| T-INT-005 | Consumer failure is retried repeatedly. | Retry policy is followed; poison message reaches configured dead-letter path without infinite silent loop. |
| T-INT-006 | Trace booking across Catalog, Availability/Appointments, Payment, and notification consumers. | Correlation IDs permit end-to-end investigation. |
| T-INT-007 | Search projection has stale slot data. | Appointment write path still prevents conflicting booking. |
| T-INT-008 | Profile/offer is deactivated while client is checking out. | Final booking validation follows current authoritative state. |
| T-INT-009 | Appointment cancellation and refund processing finish at different times. | Each state is accurately represented and reconciled. |
| T-INT-010 | Event includes a password/token/card credential. | Schema/security test fails and event is not published. |

## 15. End-to-end acceptance scenarios

### E2E-001 — Publish professional and offer
1. Register a professional account.
2. Create and activate a professional profile.
3. Create a service offer with price and duration.
4. Configure valid working hours.
5. Query candidate availability.
6. Verify the profile/offer can be found in the implemented search path.

**Expected:** each step succeeds only when prerequisites are valid; emitted events update projections without creating duplicate records.

### E2E-002 — Create and complete booking
1. Register/authenticate a client.
2. Find an active professional and offer.
3. Select an available slot.
4. Create an appointment.
5. Attempt a competing booking for an overlapping interval.
6. Complete attendance via an authorized professional action.
7. Submit a review from the client.

**Expected:** one conflicting booking at most; transitions valid; review allowed only after eligible completion.

### E2E-003 — Payment failure and retry
1. Initiate payment in the provider test environment.
2. Simulate rejection or timeout.
3. Replay callback/event.
4. Reconcile final payment state and appointment state.

**Expected:** no false success, duplicate charge, or implicit appointment/payment state conflation.

### E2E-004 — Cancellation and refund
1. Create a payable appointment.
2. Cancel under a configured test policy.
3. Simulate delayed refund confirmation.
4. Query appointment and payment separately.

**Expected:** cancelled appointment can coexist with pending refund; final refund state changes only on trusted confirmation.

## 16. Non-functional test artifacts

- **Security:** role-based access matrix; ownership/IDOR tests; credential handling review; dependency and secret scans; webhook signature tests.
- **Concurrency:** parallel appointment creation for overlapping intervals; simultaneous schedule block and booking update; idempotent payment initiation.
- **Performance:** measure common search/profile/availability requests under documented load; initial proposal is p95 ≤ 3 seconds excluding third-party latency, subject to confirmation.
- **Accessibility/responsiveness:** manual and automated checks for registration, search, profile, booking, payment, and admin journeys.
- **Resilience:** broker outage, slow consumer, duplicate/out-of-order event, payment provider timeout, unavailable owner service.
- **Observability:** verify actionable logs/metrics and correlation IDs while excluding secrets and unnecessary personal data.
- **Recovery:** verify database backup/restore and projection rebuild procedures before production.

## 17. MVC-to-hexagonal migration safety net

Before refactoring each service:

1. Freeze external API and Kafka contracts for the change window.
2. Ensure domain and application rules are covered by unit/service tests.
3. Add characterization tests for behavior not yet documented.
4. Add persistence, API, and event contract tests for adapters.
5. Move infrastructure/framework dependencies behind interfaces/adapters in small commits.
6. Run all unit, integration, contract, and E2E tests before and after each step.
7. Compare behavior, not internal class structure; do not rewrite assertions just to make the refactor pass.
8. Deploy gradually if the operating environment supports it; monitor errors, event lag, booking conflicts, and payment reconciliation.

## 18. Definition of Done for a requirement

- Requirement ID is referenced by one or more tests.
- Happy path, invalid input, authorization, and relevant boundary/failure cases are covered.
- Critical domain rules are tested without infrastructure where possible.
- Persistence, API, and Kafka contracts are verified where applicable.
- No known secret or unnecessary personal data appears in logs/events.
- Documentation and decision records are updated.
- CI passes and the behavior is demonstrated in the agreed test environment.
