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



---


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
