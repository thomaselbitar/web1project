# Car Garage Booking System

## Project Description and Workflow — Version 2

**Course:** Web Programming 1  
**Team:** Thomas El Bitar and Toni Fghali  
**Project type:** Web application  
**Backend:** Node.js  
**Database:** PostgreSQL  
**Frontend:** React with Vite (proposed)  
**Document version:** 2.0  
**Updated:** November 2, 2026

------------------------------------------------------------------------

## 1. Project Overview

The Car Garage Booking System is a web application that allows customers
to book services for their vehicles and follow each vehicle’s progress
until the work is completed and the car is collected.

The system provides one central platform for customers, technicians, and
the garage manager. Customers can manage their vehicles, browse
available services, create bookings, receive updates, and review
completed work. Technicians can view their assigned jobs, update work
progress, and submit completed jobs for manager review. The manager
controls the service catalog, schedule rules, booking decisions,
technician assignments, garage workload, and final job approval.

The system supports four main service categories:

1.  **Washing**
2.  **Detailing**
3.  **Painting**
4.  **Mechanical Repair**

The scheduling method depends on the nature of the service:

- Washing and Detailing use automatically generated appointment slots
  based on rules configured by the manager.
- Painting and Mechanical Repair use customer booking requests reviewed
  by the manager. The manager checks the current workload before
  approving, proposing another date, or rejecting a request.

The system records cash payments at pickup. Online payment processing is
not part of the project scope.

## 2. Problem Statement

Many garages manage appointments through phone calls, messages, or
walk-in customers. This can make it difficult to coordinate
appointments, understand the garage’s current workload, assign
technicians, and keep customers informed.

The system addresses these problems by:

- Providing customers with an online service catalog and booking
  process.
- Giving customers access to their vehicles and service history.
- Generating available time slots for Washing and Detailing
  automatically.
- Giving the manager a clear workload overview for Painting and
  Mechanical Repair requests.
- Centralizing booking approval, technician assignment, and service
  progress.
- Keeping customers informed through booking statuses and notifications.
- Recording completed work, final prices, payments, and customer
  reviews.

## 3. Project Objectives

The main objectives are to:

- Allow customers to register and manage multiple vehicles.
- Allow customers to browse garage services and submit bookings.
- Apply a different scheduling workflow for automatic-slot services and
  manager-reviewed services.
- Allow the manager to configure scheduling rules without changing
  application code.
- Allow the manager to review Painting and Mechanical Repair workload
  before making booking decisions.
- Allow technicians to manage the jobs assigned to them.
- Track each booking from its creation until delivery of the vehicle.
- Maintain a history of booking status changes.
- Allow customers to rate the technician and the garage after
  completion.
- Record cash payment status.

------------------------------------------------------------------------

## 4. Users and Roles

The application has three roles. Customers create their own accounts.
The manager creates internal staff accounts.

| Role       | Description                                                      | Main permissions                                                                                                                                                   |
|:-----------|:-----------------------------------------------------------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Customer   | Vehicle owner using the garage                                   | Manage personal profile and vehicles, browse services, create and manage bookings, follow progress, receive notifications, submit reviews                          |
| Technician | Garage employee such as a mechanic, painter, detailer, or washer | View assigned jobs, access vehicle and customer notes relevant to the job, start work, add internal notes, mark work as finished                                   |
| Manager    | Garage owner or supervisor                                       | Manage customers, technicians, services, schedule settings, bookings, workload, assignments, check-ins, completed-work approval, final prices, and payment records |

Technicians share the same application role. Each technician has a
technician type, such as Mechanic, Painter, Detailer, or Washer. The
manager assigns jobs according to the service being performed.

### 4.1 Access control

- Customers can access only their own profile, vehicles, bookings, and
  reviews.
- Technicians can access only jobs assigned to them.
- The manager can access and manage all garage information.
- Only the manager can create internal staff accounts.
- Only the manager can change service prices and scheduling settings,
  approve manager-reviewed requests, assign technicians, and approve
  completed work.

------------------------------------------------------------------------

## 5. Main Project Features

### 5.1 Accounts and Authentication

- Customers register using their phone number, name, and password.
- Each phone number must be unique.
- Passwords are stored as secure hashes.
- Users can log in, log out, edit their profile, and change their
  password.
- Role-based authorization protects customer, technician, and manager
  operations.
- The manager creates technician accounts; technicians cannot register
  themselves.

### 5.2 Vehicle Management — Customer

Customers can manage multiple vehicles.

Each vehicle stores information such as:

- Plate number
- Make
- Model
- Year
- Color
- Additional notes

Every booking is associated with one specific vehicle. Customers can
view the service history of each vehicle, including previous bookings
and their outcomes.

### 5.3 Service Catalog

The service catalog contains the four main service categories:

| Service category  | Description                                                    | Scheduling method        |
|:------------------|:---------------------------------------------------------------|:-------------------------|
| Washing           | Standard exterior and/or interior car washing                  | Automatic time slots     |
| Detailing         | More extensive interior and/or exterior cleaning and detailing | Automatic time slots     |
| Painting          | Vehicle painting and repainting work                           | Manager-reviewed request |
| Mechanical Repair | Mechanical repairs, troubleshooting, and maintenance           | Manager-reviewed request |

Each service entry can include:

- Name
- Description
- Estimated price
- Service category
- Active or inactive status

The manager can add, edit, activate, and deactivate services. A
deactivated service remains visible in historical bookings but cannot be
selected for new bookings. Customers can browse and filter the service
catalog.

------------------------------------------------------------------------

## 6. Scheduling System

The scheduling system has two distinct approaches. The manager
configures the rules for each service independently.

### 6.1 Washing and Detailing — Automatic Time Slots

Washing and Detailing use automatically generated time slots. Customers
select an available date and time from the schedule.

The manager can configure the following settings separately for Washing
and Detailing:

| Setting               | Description                                     |
|:----------------------|:------------------------------------------------|
| Working days          | Days on which the service is available          |
| Working start time    | Start of the service’s working period           |
| Working end time      | End of the service’s working period             |
| Service duration      | Time allocated to one appointment               |
| Time gap              | Buffer added after each appointment interval    |
| Maximum cars per slot | Number of cars that can book the same time slot |
| Active / Inactive     | Whether customers can book the service          |

There is **no separate daily-capacity setting** for Washing or
Detailing. The number of available appointments per day is determined
automatically by the working hours, duration, time gap, and maximum cars
per slot.

#### Example

Suppose the manager configures Washing as follows:

| Setting               |     Example |
|:----------------------|------------:|
| Working hours         | 09:00–17:00 |
| Service duration      |  30 minutes |
| Time gap              |  15 minutes |
| Maximum cars per slot |           2 |

The next slot starts after the service duration plus the configured gap:

`30 minutes + 15 minutes = 45 minutes`

The generated start times are 09:00, 09:45, 10:30, and so on, provided
the service duration fits within the working hours. Each slot can accept
up to two cars. Once both places are booked, that slot is no longer
available.

The same mechanism applies to Detailing, but its settings are
independent. For example, Detailing can have a longer duration, a
different gap, and only one car per slot.

#### Automatic-slot booking workflow

1.  The customer selects Washing or Detailing.
2.  The customer selects one of their vehicles.
3.  The system reads the manager’s current configuration for that
    service.
4.  The system generates or retrieves available dates and time slots.
5.  The customer selects an available slot and submits the booking.
6.  The backend checks that the slot still has capacity.
7.  If capacity is available, the booking is confirmed automatically.
8.  If the slot has become full, the customer is asked to select another
    available slot.
9.  The manager can view the confirmed booking and assign an appropriate
    technician.
10. When the vehicle arrives, the manager checks it in and the service
    workflow continues.

**Capacity protection:** Availability shown to customers is not
sufficient by itself. The backend must check capacity again when saving
a booking so that two simultaneous requests cannot take the final place
in the same slot.

**Configuration changes:** New scheduling rules apply to future
availability. Existing confirmed bookings must not be silently moved or
cancelled if the manager changes the configuration.

### 6.2 Painting — Manager-Reviewed Requests

Painting does not use automatically generated time slots. The customer
submits a preferred vehicle drop-off date, and the manager makes the
scheduling decision after reviewing the Painting workload.

The system does not model individual painting stages such as
preparation, spraying, drying, or polishing. For scheduling and workload
purposes, a painting job is simply considered **In Progress** from the
moment work starts until the service is completed.

#### Painting workload overview

When reviewing a Painting request, the manager sees:

- **Cars currently being painted:** accepted cars whose Painting work
  has started and has not been completed.
- **Pending painting requests:** customer requests waiting for a manager
  decision.
- **Approved upcoming bookings:** approved Painting bookings with a
  future drop-off date.
- **Accepted cars waiting to enter the garage:** approved cars that have
  arrived or been checked in but whose Painting work has not started.

These indicators are informational. They help the manager understand the
actual workload; they are not fixed capacity limits. The system does not
enforce a maximum number of Painting cars per day.

#### Painting booking workflow

1.  The customer selects Painting.
2.  The customer selects a vehicle and a preferred drop-off date.
3.  The customer submits the request.
4.  The booking receives the `Pending` status.
5.  The manager reviews the request and the Painting workload overview.
6.  The manager chooses one of the following actions:
    - **Approve:** accept the requested date.
    - **Propose another date:** send an alternative date to the
      customer.
    - **Reject:** reject the request, optionally with a reason.
7.  If another date is proposed, the status becomes
    `Alternative Proposed` and the customer is notified.
8.  The customer can accept or reject the proposed date.
9.  If accepted, the booking becomes `Approved` with the proposed date
    as its scheduled date.
10. If rejected, the request becomes `Rejected` or closed as rejected.

``` mermaid
flowchart TD
    A[Customer selects Painting] --> B[Selects vehicle and preferred drop-off date]
    B --> C[Submits request]
    C --> D[Pending]
    D --> E[Manager reviews Painting workload]
    E --> F{Manager decision}
    F -->|Approve requested date| G[Approved]
    F -->|Propose another date| H[Alternative Proposed]
    F -->|Reject| I[Rejected]
    H --> J[Notify customer]
    J --> K{Customer response}
    K -->|Accept| G
    K -->|Reject| I
```

### 6.3 Mechanical Repair — Manager-Reviewed Requests

Mechanical Repair uses the same request-and-approval approach as
Painting.

A mechanical repair may take one hour, several hours, or multiple days.
Its duration is not predictable enough to use fixed appointment slots.
Therefore, the system does not generate automatic repair slots or
enforce a fixed daily capacity.

The customer submits a preferred drop-off date. The manager checks the
Mechanical Repair workload and decides whether the garage can accept the
vehicle on that date.

#### Mechanical Repair workload overview

When reviewing a Mechanical Repair request, the manager sees:

- **Cars currently being repaired:** accepted cars whose Mechanical
  Repair work has started and has not been completed.
- **Pending repair requests:** customer requests waiting for a manager
  decision.
- **Approved upcoming bookings:** approved Mechanical Repair bookings
  with a future drop-off date.
- **Accepted cars waiting to enter the garage:** approved cars that have
  arrived or been checked in but whose repair work has not started.

These indicators are informational, not automatic capacity restrictions.
The manager uses the current workload and garage situation to make the
decision.

#### Mechanical Repair booking workflow

1.  The customer selects Mechanical Repair.
2.  The customer selects a vehicle and a preferred drop-off date.
3.  The customer describes the issue or requested work in the booking
    notes.
4.  The customer submits the request.
5.  The booking receives the `Pending` status.
6.  The manager reviews the request and the Mechanical Repair workload
    overview.
7.  The manager approves the requested date, proposes another date, or
    rejects the request.
8.  If another date is proposed, the customer must accept or reject it.
9.  The booking becomes `Approved` only after the manager approves the
    original date or the customer accepts the proposed date.

``` mermaid
flowchart TD
    A[Customer selects Mechanical Repair] --> B[Selects vehicle and preferred drop-off date]
    B --> C[Describes issue and submits request]
    C --> D[Pending]
    D --> E[Manager reviews Mechanical Repair workload]
    E --> F{Manager decision}
    F -->|Approve requested date| G[Approved]
    F -->|Propose another date| H[Alternative Proposed]
    F -->|Reject| I[Rejected]
    H --> J[Notify customer]
    J --> K{Customer response}
    K -->|Accept| G
    K -->|Reject| I
```

### 6.4 Scheduling comparison

| Rule                               | Washing                  | Detailing                | Painting               | Mechanical Repair      |
|:-----------------------------------|:-------------------------|:-------------------------|:-----------------------|:-----------------------|
| Scheduling method                  | Automatic slots          | Automatic slots          | Manager approval       | Manager approval       |
| Configurable working days          | Yes                      | Yes                      | Yes                    | Yes                    |
| Configurable working hours         | Yes                      | Yes                      | Optional / operational | Optional / operational |
| Configurable duration              | Yes                      | Yes                      | No                     | No                     |
| Configurable time gap              | Yes                      | Yes                      | No                     | No                     |
| Configurable cars per slot         | Yes                      | Yes                      | No                     | No                     |
| Separate daily capacity            | No                       | No                       | No                     | No                     |
| Customer selects preferred date    | Selects a slot           | Selects a slot           | Yes                    | Yes                    |
| Manager reviews workload           | Not required for booking | Not required for booking | Yes                    | Yes                    |
| Manager can propose another date   | No                       | No                       | Yes                    | Yes                    |
| Customer must accept proposed date | Not applicable           | Not applicable           | Yes                    | Yes                    |
| Workload overview                  | Not required             | Not required             | Yes                    | Yes                    |

------------------------------------------------------------------------

## 7. Booking Management and Lifecycle

A booking is associated with one customer and one vehicle. It stores the
requested service, requested date or time slot, final scheduled date or
time, assigned technician where applicable, booking status, and relevant
notes.

The booking lifecycle distinguishes between the booking decision and the
actual work performed on the vehicle.

### 7.1 Booking decision

For Washing and Detailing, a booking is confirmed automatically after
the backend successfully reserves an available slot.

For Painting and Mechanical Repair, a booking starts as a request and
follows the manager-review workflow:

- `Pending` → `Approved`
- `Pending` → `Alternative Proposed` → `Approved` (customer accepts)
- `Pending` → `Alternative Proposed` → `Rejected` (customer rejects)
- `Pending` → `Rejected`

The manager can include a rejection reason. When an alternative date is
proposed, the original requested date is not treated as approved. The
booking remains unresolved until the customer responds.

### 7.2 Vehicle arrival and work lifecycle

After a booking is confirmed or approved:

1.  The customer brings the vehicle to the garage on the scheduled date.
2.  The manager checks the vehicle in.
3.  The manager assigns an appropriate technician if one has not already
    been assigned.
4.  The technician views the assigned job and its notes.
5.  The technician starts the work.
6.  The technician marks the work as finished, sending it to the manager
    for review.
7.  The manager reviews the completed work, records the final price, and
    marks the vehicle as ready for pickup.
8.  The customer collects the vehicle and pays in cash.
9.  The manager records the payment and marks the booking as
    delivered/completed.
10. The customer can submit a review.

The manager may optionally enter an estimated completion date after
inspecting the vehicle. This date is informative and does not reserve
additional appointment slots or impose a scheduling capacity limit.

### 7.3 Booking status definitions

| Internal status         | Meaning                                                                                    | Typical action by                                     |
|:------------------------|:-------------------------------------------------------------------------------------------|:------------------------------------------------------|
| Pending                 | A Painting or Mechanical Repair request is waiting for manager review                      | Customer creates request                              |
| Alternative Proposed    | The manager proposed another date and is waiting for the customer’s response               | Manager                                               |
| Confirmed / Approved    | The automatic slot is reserved, or a manager-reviewed request is approved                  | System or manager/customer, depending on booking type |
| Checked In              | The vehicle has arrived at the garage                                                      | Manager                                               |
| In Progress             | A technician has started the service                                                       | Technician                                            |
| Awaiting Manager Review | Technician marked the work as finished                                                     | Technician                                            |
| Ready                   | Manager approved the finished work; vehicle is ready for pickup                            | Manager                                               |
| Delivered / Completed   | Customer collected the vehicle and the booking is closed                                   | Manager                                               |
| Rejected                | Request was rejected by the manager or a proposed alternative was rejected by the customer | Manager or customer                                   |
| Cancelled               | Booking was cancelled according to the cancellation rules                                  | Customer or manager                                   |
| No-show                 | Customer did not bring the vehicle within the configured grace period                      | Manager                                               |

The exact customer-facing labels can be simpler than the internal
statuses. For example, internal `Checked In` can be shown as “Car
received,” and `Ready` can be shown as “Ready for pickup.”

### 7.4 Status transition diagram

``` mermaid
stateDiagram-v2
    [*] --> Pending: Painting / Mechanical request
    [*] --> Confirmed: Washing / Detailing slot reserved

    Pending --> Confirmed: Manager approves
    Pending --> AlternativeProposed: Manager proposes another date
    Pending --> Rejected: Manager rejects

    AlternativeProposed --> Confirmed: Customer accepts
    AlternativeProposed --> Rejected: Customer rejects

    Confirmed --> CheckedIn: Manager receives vehicle
    CheckedIn --> InProgress: Technician starts work
    InProgress --> AwaitingManagerReview: Technician marks finished
    AwaitingManagerReview --> InProgress: Manager requests more work
    AwaitingManagerReview --> Ready: Manager approves completed work
    Ready --> Delivered: Vehicle collected and payment recorded

    Pending --> Cancelled: Cancellation
    AlternativeProposed --> Cancelled: Cancellation
    Confirmed --> Cancelled: Cancellation
    Confirmed --> NoShow: Vehicle does not arrive within grace period
```

Every status change must be stored in a status history record, including
the previous status, new status, date/time, and the user or system
action responsible for the change.

------------------------------------------------------------------------

## 8. Manager Dashboard and Operations

The manager dashboard is the central control area of the application.

### 8.1 Booking overview

The manager can view bookings in a list or calendar and filter them by:

- Date or date range
- Booking status
- Service category
- Technician
- Customer or vehicle

The manager can open a booking to view customer details, vehicle
details, requested services, notes, booking history, and relevant
workload information.

### 8.2 Painting and Mechanical Repair workload panels

The manager sees a separate workload overview for Painting and
Mechanical Repair.

Each panel contains the same four indicators:

| Indicator                     | Definition                                                                  |
|:------------------------------|:----------------------------------------------------------------------------|
| Currently in progress         | Vehicles whose service work has started and is not yet completed            |
| Pending requests              | Customer requests waiting for the manager’s decision                        |
| Approved upcoming bookings    | Approved bookings with future drop-off dates                                |
| Accepted and waiting to enter | Approved vehicles already checked in but whose service work has not started |

The indicators are informational. They help the manager evaluate
workload before approving new requests. They do not enforce a maximum
daily capacity.

A booking should appear in only one of these workload indicators at a
time. For example, after a vehicle is checked in, it is no longer
counted as an upcoming booking.

### 8.3 Technician assignment

- The manager assigns a technician appropriate to the service category.
- The customer’s preferred technician, if the option is offered, is only
  a preference.
- The manager makes the final assignment.
- The manager can see each technician’s current active job count when
  assigning work.
- A configurable maximum active jobs per technician may show a warning.
  The manager can override the warning.
- The project does not include individual technician calendars or
  time-off scheduling.

### 8.4 Service and schedule settings

The manager can add, edit, activate, or deactivate services and
configure the scheduling rules for each service independently.

For Washing and Detailing, the manager configures:

- Working days
- Working start and end times
- Service duration
- Time gap
- Maximum cars per slot
- Active or inactive status

There is no separate daily capacity for these services. The daily number
of available appointments is derived from the generated slots.

For Painting and Mechanical Repair, the manager uses the request-review
and workload-overview process. The system does not enforce a fixed daily
capacity or a maximum number of cars in progress for these services.

### 8.5 Other manager operations

The manager can also:

- Create and manage technician accounts.
- Manage customer accounts.
- Check vehicles in when they arrive.
- Mark no-shows according to the configured grace period.
- Review completed work.
- Enter the final service price.
- Mark a delivered job as paid by cash.
- View customer no-show history when reviewing requests.

------------------------------------------------------------------------

## 9. Technician Workflow

Each technician has a dashboard containing today’s and upcoming assigned
jobs.

For each job, the technician can view:

- Vehicle information
- Customer information relevant to the job
- Assigned service or services
- Customer-provided notes
- Scheduled drop-off information
- Internal notes, if enabled

The technician workflow is:

1.  Log in to the technician account.
2.  View assigned jobs.
3.  Open a job and review its details.
4.  Start the work, changing the booking to `In Progress`.
5.  Perform the service.
6.  Add internal notes when needed.
7.  Mark the work as finished.
8.  The booking moves to `Awaiting Manager Review`.
9.  The manager reviews the work and either requests additional work or
    approves it as ready for pickup.

Technicians cannot approve their own completed work, change service
prices, or modify scheduling rules.

------------------------------------------------------------------------

## 10. Customer Workflow

The main customer journey is:

1.  Register or log in using a phone number and password.
2.  Add one or more vehicles to the account.
3.  Browse the service catalog.
4.  Select a vehicle and a service.
5.  Follow the relevant booking process:
    - Washing / Detailing: select an available generated slot.
    - Painting / Mechanical Repair: submit a preferred drop-off date and
      wait for the manager’s decision.
6.  Receive confirmation or a notification if the manager proposes
    another date or rejects the request.
7.  If an alternative date is proposed, accept or reject it.
8.  Bring the vehicle to the garage on the confirmed/approved date.
9.  Follow the booking status until the vehicle is ready.
10. Collect the vehicle and pay in cash.
11. Submit a review for the technician and the garage.

Customers can view their upcoming and past bookings. Rescheduling and
cancellation are subject to the configured cancellation rules and
booking status.

------------------------------------------------------------------------

## 11. Notifications

The application includes in-app notifications.

### Customer notifications

Customers are notified when:

- A booking is confirmed or approved.
- The manager proposes an alternative date.
- An alternative date is accepted or rejected.
- A booking is rejected or cancelled.
- The vehicle is checked in / received.
- Work is ready and the vehicle can be collected.

### Manager notifications

The manager is notified when:

- A new Painting or Mechanical Repair request is submitted.
- A new automatic-slot booking is created, if manager alerts are
  enabled.
- A technician marks a job as finished and it is awaiting manager
  review.

Notifications should link to the relevant booking where possible.

------------------------------------------------------------------------

## 12. Reviews and Ratings

Customers can submit a review only after a booking has been completed
and delivered.

- One review is allowed per completed booking.
- The customer can rate the technician from 1 to 5 and add an optional
  comment.
- The customer can provide a separate rating for the garage.
- Technician profiles can display their average rating.
- Reviews are linked to the booking and the technician who performed the
  work.

Manager replies to reviews may be included as an optional extra if time
allows.

------------------------------------------------------------------------

## 13. Payment

Payment is handled in cash when the customer collects the vehicle.

- The manager records the final price after reviewing the completed
  work.
- The manager marks the booking as paid when the customer pays at
  pickup.
- The system stores whether the booking is paid or unpaid.
- No online payment gateway, card processing, or online invoicing is
  included in the current project scope.

------------------------------------------------------------------------

## 14. No-Show and Cancellation Rules

### 14.1 No-shows

For bookings with a scheduled appointment or drop-off date, the manager
can mark a booking as `No-show` if the vehicle does not arrive within
the configured grace period.

For timed Washing and Detailing slots, the grace period can be
configured in the manager settings. Once a booking is marked as no-show,
its reserved slot capacity is released.

The manager can see the customer’s previous no-show count when reviewing
future requests.

### 14.2 Cancellation and rescheduling

Customers can view, reschedule, or cancel eligible upcoming bookings,
subject to the cancellation cutoff configured by the manager.

- A cancellation must be recorded in the booking history.
- A cancelled automatic slot must release its reserved capacity.
- A manager-proposed alternative date must be accepted by the customer
  before it becomes the approved scheduled date.
- The exact cancellation rules for manager-reviewed requests should
  respect their current status; for example, a pending request can be
  withdrawn, while an alternative proposal can be declined.

------------------------------------------------------------------------

## 15. Data Model Overview

The database uses PostgreSQL. The following is a conceptual list of the
main entities; exact columns, constraints, keys, and relationships will
be finalized in the database UML.

| Table / Entity              | Purpose                                                                                                                          | Main relationships                                                                    |
|:----------------------------|:---------------------------------------------------------------------------------------------------------------------------------|:--------------------------------------------------------------------------------------|
| `users`                     | Customer, manager, and technician accounts; includes role and technician type where applicable                                   | Referenced by cars, bookings, assignments, reviews, notifications, and status history |
| `cars`                      | Customer vehicles and their identifying details                                                                                  | Belongs to a customer; referenced by bookings                                         |
| `departments`               | Main service categories: Washing, Detailing, Painting, Mechanical Repair                                                         | Has many services                                                                     |
| `services`                  | Service catalog entries, descriptions, prices, active status, and category                                                       | Belongs to a department; linked to bookings                                           |
| `service_schedule_settings` | Per-service scheduling type and manager-configurable rules                                                                       | One configuration per service                                                         |
| `working_days`              | Working-day configuration for each service schedule                                                                              | Belongs to service schedule settings                                                  |
| `bookings`                  | Customer request or confirmed appointment, vehicle, requested/proposed/scheduled date and time, status, price, and payment state | Belongs to customer and car; linked to services and technician assignments            |
| `booking_services`          | Services included in a booking                                                                                                   | Connects bookings and services                                                        |
| `booking_assignments`       | Technician assignment to a booking or service, if assignments are stored separately                                              | Links bookings and technicians                                                        |
| `status_history`            | Records every booking status change, who changed it, and when                                                                    | Belongs to a booking and references the actor                                         |
| `reviews`                   | Technician rating, garage rating, and optional comment                                                                           | Belongs to a completed booking and customer; references technician                    |
| `notifications`             | In-app notifications and read status                                                                                             | Belongs to a user                                                                     |
| `settings`                  | General garage settings such as cancellation cutoff and no-show grace period                                                     | General application configuration                                                     |

### 15.1 Scheduling configuration data

The `service_schedule_settings` entity should support the automatic-slot
settings used by Washing and Detailing.

Suggested fields:

| Attribute            | Purpose                                     |
|:---------------------|:--------------------------------------------|
| `id`                 | Primary key                                 |
| `service_id`         | Related service                             |
| `scheduling_type`    | `AutomaticSlots` or `ManagerApproval`       |
| `working_start_time` | Start of working hours, when applicable     |
| `working_end_time`   | End of working hours, when applicable       |
| `duration_minutes`   | Appointment duration for automatic slots    |
| `gap_minutes`        | Buffer between generated slots              |
| `max_cars_per_slot`  | Maximum cars accepted in one generated slot |
| `is_active`          | Whether booking is enabled                  |

The `working_days` entity stores the days on which each service is
available.

For Painting and Mechanical Repair, duration, gap, and cars-per-slot
values are not required. Their workload is reviewed by the manager
instead of being controlled by fixed daily capacity values.

**Important:** Do not add a separate daily-capacity field for any of the
four services in this scheduling version.

------------------------------------------------------------------------

## 16. Architecture and Technologies

The application follows a three-tier architecture:

1.  **Presentation layer:** the web interface used by customers,
    technicians, and the manager.
2.  **Application layer:** the Node.js backend, which handles
    authentication, authorization, booking logic, scheduling rules,
    notifications, and business validation.
3.  **Data layer:** PostgreSQL, which stores users, vehicles, services,
    bookings, schedule configurations, assignments, reviews, and other
    application data.

``` mermaid
flowchart TD
    A[Customer Browser] --> D[Frontend Application]
    B[Technician Browser] --> D
    C[Manager Browser] --> D
    D -->|REST API / HTTPS| E[Node.js Backend]
    E --> F[Authentication and Authorization]
    E --> G[Booking and Scheduling Logic]
    E --> H[Service and Work Management]
    E --> I[Notification and Review Logic]
    F --> J[(PostgreSQL Database)]
    G --> J
    H --> J
    I --> J
```

### 16.1 Technology choices

| Layer             | Technology                     | Responsibility                           |
|:------------------|:-------------------------------|:-----------------------------------------|
| Frontend          | React with Vite                | Responsive user interfaces for all roles |
| Backend           | Node.js                        | REST API and application/business logic  |
| Database          | PostgreSQL                     | Persistent relational data               |
| API documentation | OpenAPI / Swagger              | API documentation and testing            |
| Authentication    | JWT or secure sessions/cookies | Login and protected access               |
| Password security | Password hashing               | Secure password storage                  |
| Version control   | Git and GitHub                 | Source control and team collaboration    |

The backend is the only application layer that communicates directly
with PostgreSQL. The frontend communicates with the backend through REST
endpoints.

### 16.2 API modules

REST API endpoints should be organized by functional module:

- `auth`
- `users`
- `cars`
- `departments`
- `services`
- `service-schedules`
- `availability`
- `bookings`
- `assignments`
- `status-history`
- `reviews`
- `notifications`
- `settings`

The API should validate inputs, enforce role and ownership rules, return
consistent error responses, and support pagination, filtering, and
sorting for relevant lists.

------------------------------------------------------------------------

## 17. Security Requirements

- Passwords must be hashed; plain-text passwords must never be stored.
- Authentication must use JWT or secure sessions/cookies.
- Every protected endpoint must enforce role-based access.
- Customer endpoints must verify that the requested vehicle or booking
  belongs to the authenticated customer.
- Technician endpoints must verify that the job is assigned to the
  authenticated technician.
- Only the manager can create staff accounts, edit service prices,
  change schedule settings, assign technicians, and approve completed
  work.
- All inputs must be validated on the server.
- Database queries must be parameterized or safely handled by the chosen
  data-access layer to prevent SQL injection.
- Login attempts should be rate limited.
- Sensitive data must not be exposed to users who do not have permission
  to access it.

------------------------------------------------------------------------

## 18. Project Scope

### Included in the current project

- Customer, manager, and technician roles.
- Phone-number registration and authentication.
- Multiple vehicles per customer.
- Service catalog with four service categories.
- Manager-configurable automatic slots for Washing and Detailing.
- Manager-reviewed date requests for Painting and Mechanical Repair.
- Painting and Mechanical Repair workload overviews.
- Alternative-date proposal and customer acceptance/rejection.
- Booking management and status history.
- Technician assignment and job progress.
- Manager review of completed work.
- In-app notifications.
- Customer reviews and ratings.
- Cash payment recording.
- Responsive web interface.
- Node.js REST API and PostgreSQL database.

### Optional extras, depending on available time

- Mock OTP during registration (without real SMS delivery).
- Technician internal notes.
- Manager statistics such as bookings per day, popular services, and
  average ratings.
- Manager replies to customer reviews.
- Email or SMS notifications.

### Out of scope for the current version

- Online payments and payment gateway integration.
- Online invoicing.
- Real SMS delivery and phone-number verification through an external
  provider.
- Live chat.
- Individual technician calendars and time-off management.
- Detailed painting production stages such as spraying and drying.
- Automatic estimation of mechanical repair duration.
- Automatic workload-based approval or rejection for Painting and
  Mechanical Repair.

------------------------------------------------------------------------

## 19. Assumptions and Items to Finalize

The following points should be confirmed during detailed design:

1.  **Multiple services in one booking:** The original concept allows
    one or more services in a booking. The exact rule for combining
    services from different scheduling categories must be finalized
    because automatic slots and manager-reviewed requests follow
    different workflows.
2.  **Technician assignment timing:** The manager assigns an appropriate
    technician after a booking is confirmed or approved. The assignment
    model should support the services included in the booking.
3.  **Customer rescheduling:** The exact cutoff and permitted status
    transitions for rescheduling should be finalized.
4.  **Working days for manager-reviewed services:** Working days can
    guide which drop-off dates customers may request, while the manager
    retains the final decision.
5.  **Automatic booking confirmation:** Washing and Detailing bookings
    are confirmed when a slot is successfully reserved. They do not wait
    for a separate manager approval.
6.  **Workload indicators:** Painting and Mechanical Repair workload
    counts are calculated separately and are informational, not hard
    capacity limits.

------------------------------------------------------------------------

## 20. Final Project Workflow Summary

``` mermaid
flowchart TD
    A[Customer registers / logs in] --> B[Manages vehicles]
    B --> C[Browses service catalog]
    C --> D{Selected service}

    D -->|Washing or Detailing| E[System displays generated slots]
    E --> F[Customer selects available slot]
    F --> G[Backend validates slot capacity]
    G -->|Available| H[Booking confirmed automatically]
    G -->|Full| E

    D -->|Painting or Mechanical Repair| I[Customer requests preferred drop-off date]
    I --> J[Booking marked Pending]
    J --> K[Manager reviews service workload]
    K --> L{Manager decision}
    L -->|Approve| M[Booking Approved]
    L -->|Propose another date| N[Alternative Proposed]
    L -->|Reject| O[Booking Rejected]
    N --> P{Customer response}
    P -->|Accept| M
    P -->|Reject| O

    H --> Q[Vehicle arrives at garage]
    M --> Q
    Q --> R[Manager checks in vehicle]
    R --> S[Manager assigns technician]
    S --> T[Technician starts service]
    T --> U[Technician marks work finished]
    U --> V[Manager reviews completed work]
    V -->|More work required| T
    V -->|Approved| W[Vehicle Ready for Pickup]
    W --> X[Customer collects vehicle and pays cash]
    X --> Y[Manager records payment and closes booking]
    Y --> Z[Customer submits review]
```

### Final principle

The system uses the scheduling method that best fits each service:

- **Washing and Detailing:** the manager configures working days,
  working hours, service duration, time gap, and maximum cars per slot.
  The system generates available slots and confirms a booking when
  capacity is available. No separate daily capacity is configured.
- **Painting and Mechanical Repair:** the customer requests a preferred
  drop-off date. The manager reviews the relevant workload overview and
  can approve the request, propose another date, or reject it. If
  another date is proposed, the customer must accept it before the
  booking becomes approved. Workload indicators inform the manager’s
  decision but do not impose fixed capacity limits.

The manager controls the garage’s operational decisions, technicians
handle assigned work, and customers can follow their vehicles from
booking through pickup.
