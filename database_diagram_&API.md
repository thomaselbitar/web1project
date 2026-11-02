# Garage Booking System
## Database Design & REST API Endpoints – Version 1

**Course:** Web Programming 1  
**Team:** Thomas El Bitar & Toni Fghali

**Technology Stack**
- Frontend: React + Vite
- Backend: Node.js + Express.js
- Database: PostgreSQL
- API Architecture: REST API
- Authentication: JWT
- API Documentation: Swagger / OpenAPI

---

# Part 1 – Database ER Diagram Description

## 1. Database Overview

The database is designed to manage customers, their cars, garage services, booking requests, technician assignments, booking progress, reviews and notifications.

Each booking is associated with:
- One customer.
- One car.
- One service.
- Optionally, one preferred technician.
- One assigned technician after manager approval.

**Important design decision:** A booking contains only one service. Therefore, there is no need for a `BookingServices` junction table.

## 2. Database Tables

| Table | Description |
|---|---|
| **Users** | Stores all accounts, including customers, technicians and managers. Contains personal information, phone number, password hash, role and technician type. |
| **Cars** | Stores customer vehicles, including plate number, make, model, year, color and notes. Each car belongs to one customer. |
| **Departments** | Stores the garage departments: Repair, Painting, Detailing and Washing. |
| **Services** | Stores individual services offered by each department, including name, description, estimated price, booking type, duration and active status. |
| **Bookings** | Stores customer booking requests, selected car and service, requested date/time, proposed date/time, assigned technician, status, final price and payment information. |
| **StatusHistory** | Records every booking status change, including the previous status, new status, the user who performed the change and the timestamp. |
| **Reviews** | Stores customer ratings for the garage and assigned technician, with an optional comment. Each completed booking can have one review. |
| **Notifications** | Stores in-app notifications for customers, technicians and managers, including notification content and read status. |
| **Settings** | Stores general garage configuration, such as cancellation cutoff and no-show grace period. |
| **ServiceScheduleSettings** | Stores scheduling configurations for services, including working hours, duration, gap between slots and cars allowed per slot. |

## 3. Main Database Relationships

- One **User (Customer)** can own multiple Cars.
- One **User (Customer)** can create multiple Bookings.
- One **Car** can have multiple Bookings.
- One **Department** can contain multiple Services.
- One **Service** can be associated with multiple Bookings.
- One **User (Technician)** can be assigned multiple Bookings.
- One **Booking** can have multiple StatusHistory records.
- One **Booking** can have zero or one Review.
- One **User** can receive multiple Notifications.
- One **Service** can have its own scheduling configuration.

## 4. Booking and Availability Logic

The system uses different scheduling methods depending on the service type.

| Department | Scheduling method | Description |
|---|---|---|
| Washing | Automatic time slots | Customers select an available date and time. |
| Detailing | Automatic time slots | Customers select an available date and time. |
| Repair / Mechanic | Manager approval | Customers submit a requested date. The manager reviews the request and assigns a technician. |
| Painting | Manager approval | Customers submit a requested starting date. The manager reviews the request and assigns a technician. |

### Washing and Detailing

Available slots are generated automatically using:
- Garage working hours.
- Service duration.
- Gap between appointments.
- Maximum cars allowed per slot.

There is no separate daily capacity because availability is calculated from the generated time slots.

### Repair and Painting

These services use a request-and-approval workflow.

The manager can:
- Approve the requested booking.
- Propose an alternative date/time, which requires customer acceptance.
- Reject the request with a reason.

The manager also sees each technician's active workload before assigning a job. A configurable maximum active-job limit generates a warning but can be overridden by the manager.

**Manager approval is always required before a booking becomes confirmed.**

## 5. Booking Lifecycle

The internal booking statuses are:

1. Requested
2. Confirmed
3. Checked In
4. In Progress
5. Awaiting Manager Review
6. Ready
7. Delivered
8. Rejected
9. Cancelled
10. No-Show

Every status change is recorded in `StatusHistory`.

The technician marks the work as finished, but the manager must approve the completion before the customer is notified that the car is ready.

Payment is cash at pickup. The system only records whether the booking has been paid.

---

# Part 2 – REST API Endpoints

## 1. API Configuration

**Base URL:**

`http://localhost:3000/api/v1`

All protected endpoints require JWT authentication and appropriate role authorization.

## 2. Authentication

Base route: `/auth`

| Method | Endpoint | Description | Access |
|---|---|---|---|
| POST | `/auth/register` | Register a customer | Public |
| POST | `/auth/login` | Login | Public |
| POST | `/auth/logout` | Logout | Authenticated |
| GET | `/auth/me` | Get logged-in user information | Authenticated |
| PUT | `/auth/change-password` | Change password | Authenticated |

## 3. Users Management

Base route: `/users`

| Method | Endpoint | Description | Access |
|---|---|---|---|
| GET | `/users/me` | Get own profile | Authenticated |
| PUT | `/users/me` | Update own profile | Authenticated |
| GET | `/users` | Get all users | Manager |
| GET | `/users/:id` | Get user details | Manager |
| POST | `/users` | Create technician account | Manager |
| PUT | `/users/:id` | Update user information | Manager |
| PATCH | `/users/:id/status` | Activate/deactivate account | Manager |

## 4. Cars Management

Base route: `/cars`

| Method | Endpoint | Description | Access |
|---|---|---|---|
| POST | `/cars` | Add a car | Customer |
| GET | `/cars` | Get customer's cars | Customer |
| GET | `/cars/:id` | Get car details | Owner / Manager |
| PUT | `/cars/:id` | Update car information | Owner |
| DELETE | `/cars/:id` | Delete a car if unused | Owner |
| GET | `/cars/:id/bookings` | Get car service history | Owner / Manager |

## 5. Departments and Services

Base routes: `/departments` and `/services`

| Method | Endpoint | Description | Access |
|---|---|---|---|
| GET | `/departments` | Get all departments | Public |
| GET | `/departments/:id` | Get department details | Public |
| GET | `/services` | Get available services | Public |
| GET | `/services/:id` | Get service details | Public |
| POST | `/services` | Create a service | Manager |
| PUT | `/services/:id` | Update a service | Manager |
| PATCH | `/services/:id/status` | Activate/deactivate service | Manager |
| DELETE | `/services/:id` | Delete service if unused | Manager |

## 6. Availability and Scheduling

Base route: `/availability`

| Method | Endpoint | Description | Access |
|---|---|---|---|
| GET | `/availability` | Check service availability for a date | Public |
| GET | `/availability/slots` | Get available time slots for timed services | Public |
| GET | `/availability/workload` | View technician workload | Manager |

Availability is calculated according to the service's booking type and its scheduling configuration.

## 7. Bookings Management

Base route: `/bookings`

### Customer Endpoints

| Method | Endpoint | Description | Access |
|---|---|---|---|
| POST | `/bookings` | Create booking request | Customer |
| GET | `/bookings` | Get customer's bookings | Customer |
| GET | `/bookings/:id` | Get booking details | Owner / Manager |
| PATCH | `/bookings/:id/reschedule` | Request a new date/time | Customer |
| PATCH | `/bookings/:id/cancel` | Cancel a booking | Customer |
| GET | `/bookings/:id/history` | View booking status history | Owner / Manager |
| PATCH | `/bookings/:id/accept-time` | Accept proposed date/time | Customer |

### Manager Endpoints

| Method | Endpoint | Description | Access |
|---|---|---|---|
| GET | `/bookings/pending` | Get pending booking requests | Manager |
| PATCH | `/bookings/:id/approve` | Approve a booking | Manager |
| PATCH | `/bookings/:id/reject` | Reject booking with reason | Manager |
| PATCH | `/bookings/:id/propose-time` | Propose alternative date/time | Manager |
| PATCH | `/bookings/:id/assign-technician` | Assign or change technician | Manager |
| PATCH | `/bookings/:id/check-in` | Confirm car arrival | Manager |
| PATCH | `/bookings/:id/no-show` | Mark customer as no-show | Manager |
| PATCH | `/bookings/:id/approve-completion` | Approve completed work and set final price | Manager |
| PATCH | `/bookings/:id/deliver` | Mark car as delivered | Manager |
| PATCH | `/bookings/:id/payment` | Mark cash payment as paid | Manager |

## 8. Technician Work Management

Base route: `/bookings`

| Method | Endpoint | Description | Access |
|---|---|---|---|
| GET | `/bookings/my-jobs` | Get technician's assigned jobs | Technician |
| PATCH | `/bookings/:id/start-work` | Start working on the car | Technician |
| PATCH | `/bookings/:id/finish-work` | Mark work as finished | Technician |
| POST | `/bookings/:id/notes` | Add internal job notes | Technician |

Technicians can only access and update their assigned bookings.

## 9. Reviews and Ratings

Base route: `/reviews`

| Method | Endpoint | Description | Access |
|---|---|---|---|
| POST | `/reviews` | Submit a review | Customer |
| GET | `/reviews` | Get reviews | Public |
| GET | `/reviews/:id` | Get review details | Public |
| GET | `/reviews/technicians/:id` | Get technician reviews | Public |
| GET | `/reviews/technicians/:id/rating` | Get technician average rating | Public |
| DELETE | `/reviews/:id` | Delete a review if permitted | Manager |

Reviews contain a garage rating, technician rating and optional comment. A customer can submit one review per completed booking.

## 10. Notifications

Base route: `/notifications`

| Method | Endpoint | Description | Access |
|---|---|---|---|
| GET | `/notifications` | Get user's notifications | Authenticated |
| GET | `/notifications/unread` | Get unread notifications | Authenticated |
| PATCH | `/notifications/:id/read` | Mark notification as read | Owner |
| PATCH | `/notifications/read-all` | Mark all notifications as read | Authenticated |
| DELETE | `/notifications/:id` | Delete notification | Owner |

Notifications are generated when booking requests are approved, rejected, rescheduled, checked in or ready for pickup.

## 11. Garage Settings

Base route: `/settings`

| Method | Endpoint | Description | Access |
|---|---|---|---|
| GET | `/settings` | Get general garage settings | Manager |
| PUT | `/settings` | Update general settings | Manager |
| GET | `/settings/service-schedules` | Get service scheduling configurations | Manager |
| PUT | `/settings/service-schedules/:serviceId` | Update service scheduling configuration | Manager |

Settings include working hours, days off, slot duration, slot gaps, cars per slot, cancellation cutoff and no-show grace period.

## 12. Manager Dashboard and Statistics

Base route: `/dashboard`

| Method | Endpoint | Description | Access |
|---|---|---|---|
| GET | `/dashboard/summary` | Get overall garage statistics | Manager |
| GET | `/dashboard/bookings` | Get bookings overview | Manager |
| GET | `/dashboard/bookings/calendar` | Get calendar bookings | Manager |
| GET | `/dashboard/technicians/workload` | Get technician workload | Manager |
| GET | `/dashboard/revenue` | Get payment statistics | Manager |
| GET | `/dashboard/services/popular` | Get most requested services | Manager |

Dashboard statistics are considered optional enhancements for Version 1.

---

# Part 3 – API Implementation Rules

1. Every protected endpoint must validate the JWT token.
2. Role-based authorization must be applied to all protected routes.
3. Customers can only access their own cars, bookings and notifications.
4. Technicians can only access their assigned jobs.
5. Only managers can approve or reject booking requests and assign technicians.
6. A booking cannot be marked as ready before manager approval of the finished work.
7. Booking status changes must be recorded in `StatusHistory`.
8. Booking availability must be checked again by the backend when a request is approved to prevent conflicting bookings.
9. Deleted or deactivated services must remain identifiable in historical bookings.
10. Payment is recorded as cash paid or unpaid; no online payment gateway is implemented.

---

**End of Version 1 – Database Design & API Specification**