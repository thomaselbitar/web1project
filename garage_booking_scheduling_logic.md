# Garage Booking System

## Scheduling Logic Specification --- Version 2

**Course:** Web Programming 1\
**Project:** Garage Booking System\
**Team:** Thomas El Bitar and Toni Fghali\
**Version:** 2.0\
**Date:** November 2, 2026

------------------------------------------------------------------------

## 1. Purpose

This document defines the scheduling logic for the Garage Booking
System. The goal is to let the garage manager configure scheduling rules
for each service while keeping the booking process appropriate to the
nature of that service.

The system supports two scheduling approaches:

1.  **Automatic time-slot scheduling** for Washing and Detailing.
2.  **Manager-reviewed booking requests** for Painting and Mechanical
    Repair.

The manager can configure the schedule rules for each service. The
system must not impose a fixed daily capacity on services whose workload
is evaluated manually by the manager.

## 2. Service categories

The system supports four main services:

  -----------------------------------------------------------------------
  Service                 Scheduling approach     How a customer books
  ----------------------- ----------------------- -----------------------
  Washing                 Automatic time slots    Selects an available
                                                  slot

  Detailing               Automatic time slots    Selects an available
                                                  slot

  Painting                Manager approval        Requests a preferred
                                                  drop-off date

  Mechanical Repair       Manager approval        Requests a preferred
                                                  drop-off date and, if
                                                  applicable, a preferred
                                                  time
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## 3. Washing and Detailing: automatic scheduling

Washing and Detailing use the same scheduling mechanism. Each service
has its own independent configuration, so the manager can set different
durations, gaps, capacities, and working hours for each one.

### 3.1 Manager-configurable settings

  -----------------------------------------------------------------------
  Setting                             Description
  ----------------------------------- -----------------------------------
  Working days                        Days on which the service accepts
                                      appointments

  Start time                          Beginning of the service's working
                                      period

  End time                            End of the service's working period

  Service duration                    Time allocated to perform one
                                      appointment

  Time gap                            Buffer added after a service
                                      appointment before the next slot

  Maximum cars per slot               Number of cars that can be accepted
                                      in the same time slot

  Active / Inactive                   Whether customers can book this
                                      service
  -----------------------------------------------------------------------

**There is no separate daily-capacity setting for Washing or
Detailing.** The number of appointments that can be offered in a day is
automatically determined by the working hours, service duration, time
gap, and maximum cars per slot.

### 3.2 Example configuration

Example settings for Washing:

  Setting                   Example value
  ----------------------- ---------------
  Working hours              09:00--17:00
  Service duration             30 minutes
  Time gap                     15 minutes
  Maximum cars per slot                 2

The scheduling interval is:

`Service duration + Time gap = 30 + 15 = 45 minutes`

The system generates appointment start times at 09:00, 09:45, 10:30, and
so on, while ensuring that the service duration fits within the working
period.

Each generated slot can accept up to two cars. If both places in a slot
are booked, that slot is no longer available.

The daily number of appointments is therefore an outcome of the
configuration, not a value the manager needs to enter separately.

### 3.3 Slot-generation diagram

``` mermaid
flowchart TD
    A[Manager configures service schedule] --> B[Read working days and hours]
    B --> C[Read service duration]
    C --> D[Read time gap]
    D --> E[Read maximum cars per slot]
    E --> F[Generate appointment start times]
    F --> G[Check each slot against existing bookings]
    G --> H[Show available slots to customer]
    H --> I[Customer selects a slot]
    I --> J{Slot still has capacity?}
    J -- Yes --> K[Create confirmed booking]
    J -- No --> L[Ask customer to select another slot]
```

### 3.4 Booking and capacity rules

-   Washing and Detailing have separate configurations and separate slot
    availability.
-   A slot can accept bookings up to its configured maximum cars per
    slot.
-   When the slot reaches its capacity, it must not be offered as
    available.
-   The system must recheck capacity when the customer submits the
    booking, not only when the available slots are displayed. This
    prevents two customers from taking the final place at the same time.
-   The time gap is part of the generated schedule interval. It is a
    buffer between consecutive slots; it is not an additional customer
    appointment.
-   The final appointment must fit within the configured working hours.
-   If the manager changes a schedule rule, the system should apply the
    new rule to future availability. Existing bookings should not be
    silently moved or cancelled.

------------------------------------------------------------------------

## 4. Painting: manager-reviewed booking requests

Painting does not use automatically generated appointment slots. The
customer submits a request with a preferred car drop-off date. The
manager reviews the request and the current Painting workload before
making a decision.

The system intentionally does not model detailed painting stages such as
preparation, spraying, drying, or polishing. A car is considered **In
Progress** while the Painting service is being performed, until the
service is marked as completed.

### 4.1 Customer and manager workflow

1.  The customer selects Painting.
2.  The customer chooses a preferred drop-off date and submits a
    request.
3.  The request receives the `Pending` status.
4.  The manager reviews the request and the Painting workload overview.
5.  The manager chooses one of the following actions:
    -   **Approve:** accept the customer's requested date.
    -   **Propose another date:** send an alternative date to the
        customer.
    -   **Reject:** reject the request.
6.  If the manager proposes another date, the booking status becomes
    `Alternative Proposed` and the customer is notified.
7.  The customer can accept or reject the proposed date.
8.  If the customer accepts, the booking becomes `Approved`.
9.  If the customer rejects the proposed date, the request becomes
    `Rejected` (or is closed as rejected).

### 4.2 Painting workload overview

The manager should see the following indicators for Painting:

-   **Cars currently being painted:** accepted cars whose Painting
    service is in progress.
-   **Pending painting requests:** requests awaiting the manager's
    decision.
-   **Approved upcoming bookings:** accepted Painting bookings scheduled
    for a future drop-off.
-   **Accepted cars waiting to enter the garage:** accepted cars that
    have not yet started the Painting service.

These indicators are informational. They help the manager make a
decision, but they are **not automatic capacity limits**. The system
does not enforce a fixed maximum number of Painting cars per day.

### 4.3 Painting request workflow diagram

``` mermaid
flowchart TD
    A[Customer selects Painting] --> B[Chooses preferred drop-off date]
    B --> C[Submits booking request]
    C --> D[Status: Pending]
    D --> E[Manager reviews Painting workload]
    E --> F{Manager decision}
    F -- Approve requested date --> G[Status: Approved]
    F -- Propose another date --> H[Status: Alternative Proposed]
    F -- Reject --> I[Status: Rejected]
    H --> J[Notify customer of proposed date]
    J --> K{Customer response}
    K -- Accept --> G
    K -- Reject --> I
```

------------------------------------------------------------------------

## 5. Mechanical Repair: manager-reviewed booking requests

Mechanical Repair uses the same request-and-approval approach as
Painting. The customer requests a preferred drop-off date. The manager
reviews the request and the Mechanical Repair workload before deciding
what to do.

This approach is suitable because mechanical repairs can have very
different durations. One repair may take an hour, while another may take
several days. The system therefore does not try to calculate a fixed
appointment duration or generate automatic slots for Mechanical Repair.

### 5.1 Customer and manager workflow

1.  The customer selects Mechanical Repair.
2.  The customer chooses a preferred drop-off date and may provide a
    preferred time if the garage requires one.
3.  The customer submits a booking request.
4.  The request receives the `Pending` status.
5.  The manager reviews the request and the Mechanical Repair workload
    overview.
6.  The manager can approve the requested date, propose another date, or
    reject the request.
7.  If another date is proposed, the customer must accept or reject it.
8.  The booking becomes `Approved` only after the manager approves the
    original request or the customer accepts the proposed date.

### 5.2 Mechanical Repair workload overview

The manager should see:

-   **Cars currently being repaired:** accepted cars whose Mechanical
    Repair service is in progress.
-   **Pending repair requests:** requests awaiting the manager's
    decision.
-   **Approved upcoming bookings:** accepted Mechanical Repair bookings
    scheduled for a future drop-off.
-   **Accepted cars waiting to enter the garage:** accepted cars that
    have not yet started their repair.

These indicators are informational, not hard capacity restrictions. The
manager uses the actual workload to decide whether to approve a request,
propose another date, or reject it.

### 5.3 Mechanical Repair request workflow diagram

``` mermaid
flowchart TD
    A[Customer selects Mechanical Repair] --> B[Chooses preferred drop-off date]
    B --> C[Submits booking request]
    C --> D[Status: Pending]
    D --> E[Manager reviews Mechanical Repair workload]
    E --> F{Manager decision}
    F -- Approve requested date --> G[Status: Approved]
    F -- Propose another date --> H[Status: Alternative Proposed]
    F -- Reject --> I[Status: Rejected]
    H --> J[Notify customer of proposed date]
    J --> K{Customer response}
    K -- Accept --> G
    K -- Reject --> I
```

------------------------------------------------------------------------

## 6. Workload definitions

The manager sees the same four workload indicators for both Painting and
Mechanical Repair. The indicators are calculated independently for each
service.

  -----------------------------------------------------------------------
  Indicator                           Definition
  ----------------------------------- -----------------------------------
  Currently in progress               Cars accepted for the service whose
                                      work has started and has not been
                                      marked completed

  Pending requests                    Customer requests awaiting a
                                      manager decision

  Approved upcoming bookings          Approved bookings with a future
                                      drop-off date

  Accepted and waiting to enter       Approved cars whose
                                      drop-off/acceptance has happened,
                                      but whose service work has not
                                      started
  -----------------------------------------------------------------------

### 6.1 Avoiding double counting

The four indicators describe different stages of the booking lifecycle.
A booking should not be counted in two workload indicators at the same
time.

For example: - Before the manager decides: `Pending requests`. - After
approval but before the drop-off date: `Approved upcoming bookings`. -
After the car arrives but before work starts:
`Accepted and waiting to enter`. - After work starts:
`Currently in progress`. - After work is completed: no longer counted in
active workload indicators.

A proposed alternative date is not an approved booking. It remains an
unresolved request until the customer responds.

------------------------------------------------------------------------

## 7. Booking statuses

The following statuses are recommended:

  -----------------------------------------------------------------------
  Status                              Meaning
  ----------------------------------- -----------------------------------
  Pending                             Customer submitted a request;
                                      manager has not decided yet

  Alternative Proposed                Manager proposed a different date;
                                      customer response is pending

  Approved                            Manager approved the requested
                                      date, or customer accepted the
                                      alternative

  Rejected                            Manager rejected the request, or
                                      customer rejected the proposed
                                      alternative

  In Progress                         The car's service work has started

  Completed                           The service has been completed

  Cancelled                           The booking was cancelled, where
                                      cancellation is supported
  -----------------------------------------------------------------------

### 7.1 Status transition diagram

``` mermaid
stateDiagram-v2
    [*] --> Pending
    Pending --> Approved: Manager approves requested date
    Pending --> AlternativeProposed: Manager proposes another date
    Pending --> Rejected: Manager rejects
    AlternativeProposed --> Approved: Customer accepts
    AlternativeProposed --> Rejected: Customer rejects
    Approved --> InProgress: Car's service starts
    InProgress --> Completed: Service completed
    Pending --> Cancelled: Customer/manager cancels
    AlternativeProposed --> Cancelled: Customer/manager cancels
    Approved --> Cancelled: Customer/manager cancels
```

**Note:** `In Progress` and `Completed` apply to the operational
lifecycle after approval. Automatic Washing and Detailing bookings can
also move through these statuses when the car arrives and the service
begins.

------------------------------------------------------------------------

## 8. Manager schedule configuration: conceptual data schema

The following is a conceptual schema for the scheduling configuration.
It describes the information the database should store; it is not yet
the final database UML.

### 8.1 ServiceScheduleSettings

One configuration record belongs to one service. Washing and Detailing
use the automatic-slot fields. Painting and Mechanical Repair use
manager approval and do not require duration, gap, or slot-capacity
settings.

  ----------------------------------------------------------------------------
  Attribute         Suggested type               Nullable? Description
  ----------------- ---------------- --------------------- -------------------
  Id                INT                                 No Primary key

  ServiceId         INT                                 No Foreign key to
                                                           Service

  SchedulingType    VARCHAR / ENUM                      No `AutomaticSlots` or
                                                           `ManagerApproval`

  StartTime         TIME                               Yes Start of working
                                                           hours

  EndTime           TIME                               Yes End of working
                                                           hours

  DurationMinutes   INT                                Yes Duration of one
                                                           automatic
                                                           appointment

  GapMinutes        INT                                Yes Buffer between
                                                           consecutive slots

  MaxCarsPerSlot    INT                                Yes Maximum cars
                                                           accepted in one
                                                           generated slot

  IsActive          BOOLEAN                             No Whether booking is
                                                           enabled for this
                                                           service
  ----------------------------------------------------------------------------

**Validation rules:** - `ServiceId` must identify an existing service. -
`StartTime` must be earlier than `EndTime` when working hours are
configured. - `DurationMinutes` must be greater than zero for
automatic-slot services. - `GapMinutes` must be zero or greater for
automatic-slot services. - `MaxCarsPerSlot` must be greater than zero
for automatic-slot services. - `DurationMinutes`, `GapMinutes`, and
`MaxCarsPerSlot` are not required for manager-approval services. - The
system must not store a separate daily-capacity setting for Washing,
Detailing, Painting, or Mechanical Repair in this version.

### 8.2 Working days

Working days can be represented in a separate table so the manager can
enable or disable individual days for each service.

  -----------------------------------------------------------------------------
  Attribute                   Suggested type          Description
  --------------------------- ----------------------- -------------------------
  Id                          INT                     Primary key

  ServiceScheduleSettingsId   INT                     Foreign key to
                                                      ServiceScheduleSettings

  DayOfWeek                   VARCHAR / ENUM          Monday through Sunday

  IsWorkingDay                BOOLEAN                 Whether the service
                                                      operates on this day
  -----------------------------------------------------------------------------

This allows, for example, Washing to operate six days a week while
Painting operates on a different set of days.

### 8.3 Booking

The booking record should contain the customer's request, the manager's
proposed alternative (if any), and the booking's current status.

  ------------------------------------------------------------------------
  Attribute        Suggested type               Nullable? Description
  ---------------- ---------------- --------------------- ----------------
  Id               INT                                 No Primary key

  CustomerId       INT                                 No Foreign key to
                                                          Customer

  ServiceId        INT                                 No Foreign key to
                                                          Service

  RequestedDate    DATE                                No Date initially
                                                          requested by the
                                                          customer

  RequestedTime    TIME                               Yes Selected
                                                          automatic slot
                                                          or optional
                                                          preferred time

  ProposedDate     DATE                               Yes Alternative date
                                                          proposed by the
                                                          manager

  ProposedTime     TIME                               Yes Alternative
                                                          time, if
                                                          applicable

  ScheduledDate    DATE                               Yes Final approved
                                                          drop-off date

  ScheduledTime    TIME                               Yes Final approved
                                                          time, if
                                                          applicable

  Status           VARCHAR / ENUM                      No Current booking
                                                          status

  CreatedAt        DATETIME                            No Request creation
                                                          timestamp

  UpdatedAt        DATETIME                            No Last update
                                                          timestamp
  ------------------------------------------------------------------------

The exact fields and relationships should be finalized during the
database UML design. A booking may also require additional fields such
as vehicle, notes, and cancellation information based on the wider
project requirements.

### 8.4 Conceptual entity relationship diagram

``` mermaid
erDiagram
    SERVICE ||--o| SERVICE_SCHEDULE_SETTINGS : has
    SERVICE_SCHEDULE_SETTINGS ||--o{ WORKING_DAY : defines
    SERVICE ||--o{ BOOKING : receives
    CUSTOMER ||--o{ BOOKING : submits

    SERVICE {
        int Id PK
        string Name
        string Description
    }

    SERVICE_SCHEDULE_SETTINGS {
        int Id PK
        int ServiceId FK
        string SchedulingType
        time StartTime
        time EndTime
        int DurationMinutes
        int GapMinutes
        int MaxCarsPerSlot
        boolean IsActive
    }

    WORKING_DAY {
        int Id PK
        int ServiceScheduleSettingsId FK
        string DayOfWeek
        boolean IsWorkingDay
    }

    BOOKING {
        int Id PK
        int CustomerId FK
        int ServiceId FK
        date RequestedDate
        time RequestedTime
        date ProposedDate
        time ProposedTime
        date ScheduledDate
        time ScheduledTime
        string Status
        datetime CreatedAt
        datetime UpdatedAt
    }

    CUSTOMER {
        int Id PK
        string Name
        string Phone
    }
```

------------------------------------------------------------------------

## 9. Important implementation rules

1.  **Separate configuration per service:** Each service has its own
    scheduling settings. Changing Washing rules must not affect
    Detailing, Painting, or Mechanical Repair.
2.  **No unnecessary daily capacity:** The system does not ask the
    manager to enter a daily capacity in this version.
3.  **Automatic slot capacity:** For Washing and Detailing, capacity is
    enforced per generated slot.
4.  **Manager judgment:** For Painting and Mechanical Repair, workload
    indicators help the manager decide. They do not automatically
    approve or reject requests.
5.  **Customer confirmation:** An alternative date proposed by the
    manager must be accepted by the customer before the booking becomes
    approved.
6.  **Preserve existing bookings:** Changing future schedule settings
    must not silently modify existing bookings. Any conflict with an
    existing booking should be surfaced to the manager.
7.  **Workload is service-specific:** Painting workload counts only
    Painting bookings; Mechanical Repair workload counts only Mechanical
    Repair bookings.
8.  **No detailed production stages:** Painting and Mechanical Repair
    only need the general `In Progress` state for this scheduling scope.
    Detailed work stages are outside the current scope.
9.  **Capacity checks must be reliable:** Automatic-slot capacity must
    be checked again when a booking is saved to avoid overbooking.
10. **Working-day validation:** Requests and generated slots must
    respect the working days configured for the relevant service.

------------------------------------------------------------------------

## 10. Final Scheduling Summary

| Rule | Washing | Detailing | Painting | Mechanical Repair |
|:---|:---:|:---:|:---:|:---:|
| **Scheduling method** | Automatic slots | Automatic slots | Manager approval | Manager approval |
| **Configurable working days** | Yes | Yes | Yes | Yes |
| **Configurable working hours** | Yes | Yes | Optional | Optional |
| **Configurable duration** | Yes | Yes | No | No |
| **Configurable time gap** | Yes | Yes | No | No |
| **Configurable cars per slot** | Yes | Yes | No | No |
| **Separate daily capacity** | No | No | No | No |
| **Customer selects preferred date** | Selects a slot | Selects a slot | Yes | Yes |
| **Manager reviews workload** | Not required | Not required | Yes | Yes |
| **Manager can propose another date** | No | No | Yes | Yes |
| **Customer must accept proposed date** | N/A | N/A | Yes | Yes |
| **Workload overview** | Not required | Not required | Yes | Yes |

### Final Principle

- **Washing and Detailing:** Use configurable scheduling rules to automatically generate available time slots. Capacity is controlled per slot, without a separate daily capacity setting.
- **Painting and Mechanical Repair:** Customers submit booking requests. The manager reviews the current workload and can approve the requested date, propose an alternative date, or reject the request. The customer must accept any proposed alternative before the booking becomes approved.

The manager retains control over the operational capacity of Painting and Mechanical Repair.