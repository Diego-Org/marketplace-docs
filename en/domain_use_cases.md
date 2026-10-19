# Domain Use Cases — Beauty Services Marketplace

**Document ID:** UC-DOM-001  
**Version:** 1.0  
**Status:** Proposed baseline

## 1. Conventions

- **Primary actor** initiates the use case.
- **Preconditions** are true before the flow starts.
- **Postconditions** describe the result after success.
- **Events** are proposed asynchronous integration events, not a substitute for synchronous validation when an immediate answer is required.
- Names and exact states are provisional where a business decision remains open.

The first implementation recommendation is four deployable services: Identity, Profiles, Catalog, and a combined Scheduling & Appointments service containing use cases for both Domains 5 and 6.

---

## Domain 1 — Identity & Access

### UC-IAM-01 — Register account
**Primary actor:** Client or Professional.  
**Preconditions:** The user is not already registered with the normalized login identifier.

**Main flow**
1. The user selects the account type and submits required registration data.
2. The service validates format, required fields, and identifier uniqueness.
3. The service stores the account and securely processed credential, if password-based registration is selected.
4. The service assigns the initial role according to the signup flow.
5. The service returns the account identifier and the next required step.
6. The service publishes `identity.account-created.v1` after the account change is committed.

**Alternatives/errors:** Invalid fields; identifier already used; external identity provider failure; temporary persistence failure. No duplicate account must be created on retry.

**Postcondition:** An account exists in the policy-defined initial state. Profile creation may occur in a separate step/service.

### UC-IAM-02 — Authenticate user
**Primary actor:** Registered user.
**Preconditions:** Account exists and is not blocked from login.

**Main flow:** User submits credentials or approved social identity; service verifies identity; service checks account state; service issues the configured authentication result and role/claims.

**Alternatives/errors:** Invalid credentials; disabled/suspended account; rate limit triggered; identity provider unavailable. Responses must not expose whether sensitive account details are valid beyond the security policy.

**Postcondition:** Valid session/token result is issued or authentication is rejected.

### UC-IAM-03 — Recover access
**Primary actor:** Account holder.
**Main flow:** User requests recovery; service responds without revealing unnecessary account existence information; sends or initiates a time-limited recovery challenge; validates the challenge; accepts a new credential; invalidates applicable recovery challenge/session state.

**Postcondition:** Access is safely restored or the request is rejected. Recovery tokens must not be logged or published in events.

### UC-IAM-04 — Change access status/role
**Primary actor:** Authorized administrator or approved internal workflow.
**Main flow:** Caller requests a permitted role/status change; service verifies authorization; validates the transition; records change; emits `identity.account-status-changed.v1` or `identity.role-assigned.v1`.

**Alternatives:** Caller lacks permission; transition invalid; account not found. Every action must be auditable.

---

## Domain 2 — Profiles & Professionals

### UC-PROF-01 — Create client profile
**Primary actor:** Client.
**Preconditions:** A valid account exists.

**Main flow:** Client submits profile fields; service validates fields and visibility rules; profile is linked to `accountId`; service returns the profile state and allowed next actions.

**Alternatives:** Profile already exists; invalid data; unauthorized attempt to attach another account ID.

**Postcondition:** Client profile exists and private fields are accessible only to authorized parties.

### UC-PROF-02 — Create or update professional profile
**Primary actor:** Professional.
**Preconditions:** Account has the Professional role.

**Main flow:** Professional submits name/display details, service area, description, and optional social links; service validates URL format and minimum required fields; saves as draft or active according to publication rules; publishes `profiles.professional-profile-activated.v1` when activation occurs.

**Alternatives:** Missing required data; malformed link; account not eligible; profile is suspended and cannot be activated by the user.

**Postcondition:** Profile is saved in an explicit state. Publishing does not automatically create service offers; Catalog remains the owner of offers.

### UC-PROF-03 — Restrict or suspend professional profile
**Primary actor:** Authorized administrative/moderation workflow.
**Main flow:** An authorized request identifies the profile and reason; Profiles validates the request authority and current state; changes the profile state; records the actor/reason reference; publishes a restriction event.

**Postcondition:** Public discovery and new-offer behavior can react to the profile state; past appointment records remain intact.

---

## Domain 3 — Service Catalog & Offers

### UC-CAT-01 — Maintain category/service taxonomy
**Primary actor:** Authorized administrator.
**Main flow:** Administrator creates or updates a category/service definition; service validates name, status, and hierarchy; change is saved and an event is emitted when needed by projections.

**Alternatives:** Duplicate/invalid taxonomy entry; deletion is prohibited because historical records reference it. Prefer deactivation/archival over destructive deletion.

**Postcondition:** A canonical category/service definition is available for eligible offers.

### UC-CAT-02 — Publish professional offer
**Primary actor:** Professional.
**Preconditions:** Professional profile is eligible for publication.

**Main flow:** Professional selects a category/service definition or supplies an allowed custom offer name; specifies price, duration, and description; Catalog validates price/duration and ownership; offer is published; `catalog.service-offer-published.v1` is emitted.

**Alternatives:** Profile is inactive; price invalid; duration zero/negative; category unavailable; user attempts to manage another professional's offer.

**Postcondition:** Offer is active and eligible for search/booking. The actual booking still revalidates offer state.

### UC-CAT-03 — Change or deactivate offer
**Primary actor:** Professional or authorized administrator.
**Main flow:** Caller requests a change; Catalog validates ownership and fields; stores new terms or deactivates offer; publishes the relevant event.

**Postcondition:** New bookings use current active offer terms. Existing appointments preserve the terms captured when booked.

---

## Domain 4 — Discovery & Search

### UC-DSC-01 — Search professionals/offers
**Primary actor:** Client.
**Preconditions:** Public search is available.

**Main flow:** Client submits category/service/name and optional filters; search validates filters; queries the read model; returns active eligible profile and offer summaries in documented order; provides empty-state guidance if no results match.

**Alternatives:** Unsupported filter; stale projection; dependent index temporarily unavailable. A degraded response must not claim that a candidate slot is guaranteed.

**Postcondition:** Client receives comparable discovery results or an explicit empty/error state.

### UC-DSC-02 — View professional and offer details
**Primary actor:** Client.
**Main flow:** Client opens a result; search/profile reads show permitted public profile fields, active offers, published ratings, and social links; private profile fields are omitted.

**Postcondition:** Client can compare service name, price, duration, location/service area, and available reputation details before booking.

### UC-DSC-03 — Refresh search projection
**Primary actor:** Search consumer/system.
**Trigger:** Profile, offer, review, or other subscribed event.

**Main flow:** Consumer validates event schema/version; applies projection update idempotently; records processing outcome and projection freshness.

**Alternatives:** Duplicate event; unsupported version; malformed event; processing failure. Failed records follow retry/dead-letter policy.

**Postcondition:** Read model reflects the event once processed; source domain remains unchanged.

---

## Domain 5 — Agenda & Availability

### UC-AVL-01 — Configure working hours
**Primary actor:** Professional.
**Preconditions:** Professional account is active and authorized.

**Main flow:** Professional enters recurring days/time intervals and time zone; service validates interval boundaries and overlaps; stores schedule; publishes `availability.schedule-updated.v1`.

**Alternatives:** Invalid end-before-start interval; unsupported or missing time zone; schedule update conflicts with existing appointments. Conflict handling must be explicit.

**Postcondition:** Candidate availability can be calculated from the saved schedule.

### UC-AVL-02 — Block or unblock a period
**Primary actor:** Professional.
**Main flow:** Professional selects date/time and optional reason; service validates interval; checks for conflicts with existing appointments via the same service/module or an authoritative coordination flow; saves block or rejects and explains the conflict.

**Postcondition:** New candidate slots exclude the block; existing appointments are not silently removed.

### UC-AVL-03 — Query candidate availability
**Primary actor:** Client or Appointments module.
**Preconditions:** Professional, offer, date range and duration are valid.

**Main flow:** Service combines recurring hours, exceptions, blocks, time-zone rules, duration, and known booking intervals; returns candidate intervals and relevant constraints.

**Important:** This result is a candidate list. Final booking must atomically verify that the interval is still free.

**Postcondition:** Caller receives candidate intervals, possibly none.

---

## Domain 6 — Appointments & Attendance

### UC-APT-01 — Create appointment
**Primary actor:** Client.
**Supporting actors:** Catalog, Availability, Payment (depending on approved flow).
**Preconditions:** Client is authorized; professional/profile and offer are eligible; candidate time selected.

**Main flow**
1. Client reviews professional, offer, price, duration, date/time, and cancellation conditions.
2. Appointment service revalidates offer eligibility and schedule constraints.
3. Service performs an authoritative conflict check at the write boundary.
4. Service stores the appointment with agreed price/duration snapshot and initial state defined by the approved flow.
5. Service emits `appointments.appointment-created.v1` and subsequent state event as appropriate.
6. Client and professional are informed of the resulting state.

**Alternatives:** Slot has been taken by a concurrent request; offer was deactivated; profile became ineligible; request retried; payment is pending/failed. The appointment/payment states must remain coherent but distinct.

**Postcondition:** At most one incompatible appointment is confirmed for the same professional/time interval.

### UC-APT-02 — Cancel appointment
**Primary actor:** Client or authorized professional, according to policy.
**Main flow:** Caller requests cancellation; service checks ownership, current state, timing, and policy; presents or validates effects; records cancellation; updates availability behavior; emits `appointments.appointment-cancelled.v1`; payment domain independently processes any refund action.

**Alternatives:** Cancellation is not allowed at that stage; another action changed the state concurrently; refund provider fails. Appointment cancellation must not falsely imply that a refund has completed.

**Postcondition:** Appointment status and financial status are individually queryable and auditable.

### UC-APT-03 — Complete attendance or record no-show
**Primary actor:** Professional or authorized operational workflow.
**Main flow:** Actor opens an eligible appointment; selects completion/no-show as allowed by policy; service validates transition and authority; records event and publishes the corresponding lifecycle event.

**Alternatives:** Invalid state transition; actor lacks access; appointment already finalized. No-show penalties are not applied until policy is approved.

**Postcondition:** Review eligibility and analytics can react to the final attendance state.

### UC-APT-04 — Resolve concurrent booking attempts
**Primary actor:** System.
**Trigger:** Two or more simultaneous booking requests target overlapping intervals.

**Main flow:** Each request performs final validation at the authoritative boundary; transaction/locking/constraint rules allow only a valid non-overlapping booking to commit; the losing request gets an explicit conflict response.

**Postcondition:** No two incompatible confirmed appointments exist for the same professional. Kafka eventual consistency alone does not meet this requirement.

---

## Domain 7 — Payments & Payouts

### UC-PAY-01 — Initiate appointment payment
**Primary actor:** Client.
**Supporting actors:** Payment provider, Appointment service.
**Preconditions:** The appointment/checkout state allows payment under the approved policy.

**Main flow:** Payment service creates an internal payment record and idempotency key; requests payment from provider; returns pending/action-required/failed result; stores provider references; informs appointment orchestration of the status.

**Alternatives:** Provider unavailable; duplicate initiation request; rejected payment; timeout with unknown outcome. The service must reconcile unknown outcomes instead of assuming failure or success.

**Postcondition:** Payment state is stored separately and correlated with appointment ID.

### UC-PAY-02 — Process provider callback/webhook
**Primary actor:** Payment provider.
**Main flow:** Provider sends callback; service verifies authenticity; deduplicates event; validates amount/currency/reference; updates payment state; publishes `payments.payment-status-changed.v1`.

**Alternatives:** Invalid signature; unknown transaction; repeated event; out-of-order status. Invalid callbacks do not alter financial state.

**Postcondition:** Transaction history reflects verified provider results.

### UC-PAY-03 — Process refund/payout
**Primary actor:** Approved business workflow or authorized operator.
**Main flow:** Service validates policy eligibility and financial record; requests provider operation; tracks pending/success/failure; publishes status updates; records reconciliation details.

**Postcondition:** Refund/payout is not shown as complete until supported by verified provider status.

---

## Domain 8 — Reviews & Reputation

### UC-REV-01 — Submit review
**Primary actor:** Client.
**Preconditions:** Client owns an eligible completed appointment and has not already reviewed it under the approved policy.

**Main flow:** Client submits a rating and optional comment; service checks appointment eligibility and identity/ownership; validates rating/comment; stores review; makes it visible or pending moderation according to policy; emits a review event.

**Alternatives:** Appointment incomplete/ineligible; duplicate review; invalid rating; content blocked by policy.

**Postcondition:** Review is linked to a specific appointment and contributes to reputation only when visible/eligible.

### UC-REV-02 — Refresh reputation summary
**Primary actor:** Reviews projection/aggregation process.
**Trigger:** Review published/hidden/edited according to policy.

**Main flow:** Service recalculates count and average using eligible visible reviews; stores summary; emits projection update where needed.

**Postcondition:** Public summary follows a documented formula and does not include hidden/ineligible reviews.

---

## Domain 9 — Reports & Moderation

### UC-MOD-01 — Submit report
**Primary actor:** Client or Professional.
**Main flow:** Reporter selects target type and reason; provides optional details; system validates allowed fields and stores the report; returns a reference ID; emits `moderation.report-created.v1`.

**Alternatives:** Invalid target; abusive request rate; duplicate/open report under deduplication policy.

**Postcondition:** A report is available to authorized moderators without exposing it publicly.

### UC-MOD-02 — Review and decide moderation case
**Primary actor:** Authorized Moderator/Administrator.
**Preconditions:** Staff member has moderation permission.

**Main flow:** Moderator opens case and permitted evidence references; reviews relevant context from owner domains; records decision, reason, and action requested; case history is preserved; event is published.

**Alternatives:** Insufficient evidence; conflict of access; target no longer exists; action request rejected by owner domain. Case state records the resulting status.

**Postcondition:** A decision is traceable; the owner domain is responsible for the effective target-state change.

---

## Domain 10 — Analytics & Indicators

### UC-ANL-01 — View professional indicators
**Primary actor:** Professional.
**Preconditions:** Professional is authenticated and authorized for that profile's metrics.

**Main flow:** Professional selects a period; service queries projections; presents booking counts, completed/cancelled appointments, rating summary, and schedule indicators supported by available data; displays period and last update where appropriate.

**Alternatives:** Not enough data; projection is delayed; unsupported period. The UI must not imply real-time precision for eventual data.

**Postcondition:** Professional sees authorized indicators and their time scope.

### UC-ANL-02 — Update analytics projection
**Primary actor:** Analytics consumer.
**Trigger:** Appointment, payment, review, catalog, or availability event.

**Main flow:** Validate event; update metrics idempotently; handle correction/duplicate events; record processing lag; support projection rebuild.

**Postcondition:** Analytics is derived from owner-domain events and does not update operational records.

---

## Domain 11 — Platform Administration & Audit Operations

This is treated as an administrative application/capability, not a separate owner of all domain entities.

### UC-ADM-01 — Administer a user/profile/category
**Primary actor:** Authorized Administrator.
**Main flow:** Administrator opens the relevant administrative screen; system checks permission; operation is sent to the owning service; owner validates the business rule and commits the change; response is displayed; administrative action is logged.

**Alternatives:** Insufficient permission; invalid transition; owner service unavailable; duplicate submission. The admin application must not write directly to the owner's database.

**Postcondition:** The owner service remains the source of truth and the action is traceable.

### UC-ADM-02 — Investigate appointment/payment issue
**Primary actor:** Authorized Support Administrator.
**Main flow:** Administrator searches by allowed identifiers; system retrieves role-appropriate appointment and payment views from owner services; relevant states are displayed separately; administrator performs only approved compensating/operational actions; action is logged.

**Alternatives:** No record; access to financial details denied; provider status pending; services disagree temporarily. Support tooling must not manually mark a refund/transaction successful without authoritative evidence.

**Postcondition:** Investigation result is recorded without bypassing payment/appointment invariants.

### UC-ADM-03 — Audit administrative operation
**Primary actor:** System.
**Trigger:** Important administrative action.

**Main flow:** Record actor, action, target reference, timestamp, result, correlation ID, and required reason; restrict audit read access; retain according to an approved policy.

**Postcondition:** The action can be investigated without logging credentials, tokens, or unnecessary personal data.

---

## 2. Cross-domain scenarios

### UC-X-01 — End-to-end booking (core MVP)
1. Client authenticates (Domain 1).
2. Professional has an active profile (Domain 2) and active offer (Domain 3).
3. Client discovers the offer (Domain 4 or an initial query module).
4. Availability returns candidate intervals (Domain 5).
5. Appointment write path atomically validates the selected interval and persists booking (Domain 6, or combined service).
6. Payment is initiated and tracked separately if the chosen flow requires it (Domain 7).
7. Appointment/payment notifications and read projections are updated asynchronously.
8. After eligible completion, client may review (Domain 8).

**Critical failure cases:** offer deactivated between search and checkout; slot taken concurrently; payment provider timeout; duplicated webhook; cancellation before refund settles; event consumer down. Every case must end in an explicit state that can be retried/reconciled.

### UC-X-02 — Professional profile becomes searchable
1. Identity creates account and emits account event.
2. Professional completes profile and becomes active.
3. Professional publishes one or more offers.
4. Search projection consumes profile/offer events and indexes the active records.
5. Client search returns the profile only if the profile and offer are eligible according to the projection's rules.

**Important:** asynchronous indexing means the profile may not appear immediately. The UI should provide a clear publishing status; direct booking still validates against current source-of-truth state.

### UC-X-03 — Appointment cancellation with payment/refund
1. Client requests cancellation.
2. Appointment service evaluates cancellation policy and records the allowed state transition.
3. Payment service independently receives an authorized refund request if the policy requires it.
4. Payment service processes provider status and emits refund/payment events.
5. Client sees the appointment cancelled while refund may be pending, completed, or failed.
6. Support/admin may investigate without equating cancellation with refund completion.

---

## 3. Open decisions that affect these use cases

- Whether booking is automatically confirmed or awaits professional acceptance.
- Whether payment occurs before booking confirmation, after confirmation, or through a supported alternative.
- Exact appointment states and allowed transitions.
- Cancellation deadline, penalties, refund behavior, and no-show policy.
- Mandatory professional fields and verification requirements.
- Rating scale, duplicate/edit/reply policy, and moderation rules.
- Search ranking, location granularity, and notification channels.
