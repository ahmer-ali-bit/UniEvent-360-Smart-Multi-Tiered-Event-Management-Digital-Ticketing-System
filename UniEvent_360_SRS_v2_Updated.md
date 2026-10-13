Software Requirements Specification

UniEvent 360

Smart Multi-Tiered Event Management & Digital Ticketing System

UPDATED VERSION 2.0

Submitted by

Ahmer Ali (S23BINFT1M01017)

Submitted to

Sir Wahaj Ali

Department of Information Technology

Faculty of Computing

The Islamia University of Bahawalpur

Meeting Details

| Sr No | Details | Date | Supervisor Signature |
|---|---|---|---|
| 1 |  |  |  |
| 2 |  |  |  |
| 3 |  |  |  |
| 4 |  |  |  |
| 5 |  |  |  |
| 6 |  |  |  |

# Summary

This document defines the Software Requirements Specification (SRS) — Updated Version 2.0 — for UniEvent 360, a smart multi-tiered event management and digital ticketing system designed to digitize and centralize university event operations.

The system enables students to discover events, book tickets online or receive walk-in tickets, and enter venues using secure one-time unique QR tickets verified online with real-time multi-gate synchronization. It includes advanced features such as anti-clash scheduling against a venue database, an Admin event-approval workflow, real-time ticket inventory, department-based dual pricing, waitlist management, payment integration with a sandbox (mock) mode, and automated notifications.

The platform follows a hierarchical role-based panel structure: all users — including the Super Admin — log in through the same authentication flow, and after a role check each user is redirected to their respective panel. The Super Admin controls the entire system and promotes normal users to Admins; Admins promote normal users to Event Organizers; Event Organizers make users Volunteers. Every student registers with a university roll number, from which the department is automatically detected and used for department-based insider/outsider pricing.

Rich discovery and engagement features include smart dashboard sections (Today / This Week / Trending), interest tags with a personalized feed, rule-based recommendations, bookmarks, advanced filters, event galleries and speaker profiles, event Q&A, feedback and ratings, a referral reward system, and a “Leave Now” ETA navigation alert. The complete system is engineered to operate within the Firebase free (Spark) plan.

# Table of Contents

1. Introduction

1.1 Purpose

1.2 Scope

1.3 Product Perspective & Role Hierarchy

1.4 User Characteristics

1.5 Similar Apps / Literature Review

1.6 Proposed Technologies

2. Requirements

2.1 Functional Requirements (FR001 – FR042)

2.2 Non-Functional Requirements (NFR001 – NFR015)

3. Use Cases and Flow of Processes (UC001 – UC020)

4. Test Plan & Test Cases

5. References

Appendix A: Change Log (Original SRS → v2.0)

Appendix B: Firebase Free (Spark) Plan Compliance

# 1. Introduction

Traditional university event management relies on manual processes, leading to scheduling conflicts, ticket fraud, and poor communication. UniEvent 360 provides a fully digital, multi-tiered solution: a centralized Flutter + Firebase platform where organizers create events (approved by Admins), students discover and book them with department-based pricing, volunteers verify one-time unique QR tickets online with real-time multi-gate synchronization, and a five-level role hierarchy (Super Admin → Admin → Organizer → Volunteer, with Students as the base) governs the entire system through role-check-based panels, promotions, and tamper-proof audit logs.

## 1.1 Purpose

The purpose of this SRS is to provide a complete and structured description of the UniEvent 360 system, including all functional and non-functional requirements. It serves as the reference for developers, the supervisor, and stakeholders. This document specifically aims to:

- Define all core functionalities: roll-number registration with automatic department detection, role-based panels, event creation with approval, ticket booking (online + walk-in), QR validation, payments, refunds, and notifications
- Define the five-role hierarchy (Student, Volunteer, Organizer, Admin, Super Admin) with search-and-promote workflows and audit logging
- Ensure advanced features: venue-database anti-clash scheduling, real-time inventory, waitlists, optional seat tiers, event lifecycle (postpone/cancel/complete), smart discovery, engagement (Q&A, feedback, referrals), and certificates
- Provide secure operations: one-time unique QR tickets with server-side verification, role-based access control enforced by Firestore security rules, and audit trails
- Guarantee the complete system runs within the Firebase free (Spark) plan using in-app/local notifications, on-device PDF generation, caching, denormalization, and batched writes
- Maintain financial transparency through live and sandbox payment modes, cash (walk-in) records, and refund mechanisms

## 1.2 Scope

System Includes:

- User registration with university roll number and automatic department detection; role-check-based login for all five roles
- Hierarchical role promotion: Super Admin → Admin → Event Organizer → Volunteer, with user search and audit logs
- Event creation and management with gallery, speaker profiles, interest tags, and venue selection from an Admin-managed Venue Database
- Anti-clash scheduling by venue ID; Admin approval workflow before publishing
- Event lifecycle management: Postponed (re-approval + refund opt-out), Cancelled (automatic full refunds), Completed (certificates, feedback, reports)
- Digital ticket booking (online) and walk-in physical ticket issuance by organizers — shared live inventory
- Department-based dual pricing (host-department = student rate; other departments = outsider rate) with an Allow-Outsiders toggle
- Optional seat tiers (VIP / Front / General); equal pricing when no tiers are defined
- One-time unique encrypted QR tickets; online server-match verification; real-time multi-gate synchronization; “Already Scanned” protection
- Payment integration (JazzCash / EasyPaisa / Bank Card) plus Sandbox (mock) mode and Cash records for walk-ins; refund management
- Waitlist system with automatic queue allocation
- In-app notification center + device-scheduled local notifications (reminders, “Leave Now” ETA alerts)
- Volunteer management with check-in/check-out performance tracking; live attendance counters
- Discovery and personalization: Today / This Week / Coming Soon / Trending sections, interest-tag feeds, rule-based recommendations, bookmarks, advanced combined filters
- Engagement: event Q&A with moderation, post-event feedback and ratings, referral reward points, PDF certificates
- Event location, map integration, and navigation; Admin and Super Admin dashboards with analytics, reports, and system settings
Out of Scope:

- Offline ticket scanning (an active internet connection is mandatory at scanning gates)
- Non-digital / manual ticketing outside the app's walk-in issuance
- Native iOS release in the first phase (Android first)
- Machine-learning-based recommendations (rule-based only)
- Live video streaming production of events

## 1.3 Product Perspective & Role Hierarchy

UniEvent 360 is a standalone mobile application built on a client-server architecture: Flutter (presentation), Firebase Auth + Firestore + Storage (application & data layers, free Spark plan), and external services (Google Maps, payment gateways). All business rules that formerly required paid Cloud Functions are implemented client-side with Firestore security rules, on-device scheduling, and on-device PDF generation.

Role Hierarchy (promotion chain):

```text
SUPER ADMIN  (pre-seeded account; logs in via same role check; full system control)
   └── promotes →  ADMIN  (Admin Panel becomes visible to that user)
                      └── promotes →  EVENT ORGANIZER  (Organizer Panel)
                                         └── promotes →  VOLUNTEER  (Scanner Panel)
STUDENT / USER  = default role for every new registration → Student Panel
```
Rules: a user may only promote one level below their own role; promotions are enforced server-side (Firestore rules), recorded in tamper-proof audit logs, and notified to the promoted user. Demotion follows the same authority.

## 1.4 User Characteristics

Five types of users interact with this system. Each panel opens automatically through the role check performed at login:

| User Role | Technical Level | Primary Interaction | How the Role is Granted |
|---|---|---|---|
| Student / User | Basic | Discover & book events, QR tickets, waitlist, feedback, certificates, navigation | Default at registration (roll number required) |
| Volunteer | Basic | Scan QR tickets at gates (internet required), attendance, duty check-in/out | Promoted by an Event Organizer |
| Organizer | Intermediate | Create & manage events, issue walk-in tickets, promote Volunteers, reports | Promoted by an Admin |
| Admin | Advanced | Approve/reject events, promote Organizers, manage venues, monitor, reports | Promoted by the Super Admin |
| Super Admin | Advanced | Full system control, promote Admins, audit logs, analytics, settings | Pre-seeded account (role-check login) |

## 1.5 Similar Apps and Systems / Literature Review

| System / Study | Key Feature | Shortcoming Addressed by This Project |
|---|---|---|
| Eventbrite | Ticketing | Not university-specific; no department-based pricing or approval hierarchy |
| Facebook Events | Discovery | No secure QR entry, no attendance verification, no roles |
| Manual university systems | Simple registers | No automation, ticket fraud, scheduling clashes, no analytics |

## 1.6 Proposed Technologies

| Layer | Technology | Role in the System |
|---|---|---|
| Frontend | Flutter (Dart), Material 3 | All mobile UI screens, role-based routing, state management |
| Authentication | Firebase Authentication | Email/password auth for all five roles; role check via Firestore user document |
| Database | Cloud Firestore (Spark plan) | Users, events, venues, bookings, tickets, payments, attendance, waitlist, comments, feedback, referrals, audit logs, notifications |
| Storage | Firebase Storage | Compressed event/gallery/speaker images, profile photos, certificates |
| Notifications | In-app center + flutter_local_notifications | All alerts and device-scheduled reminders (24h/1h, Leave Now ETA) — free plan; FCM push is a future optional upgrade |
| Payments | JazzCash / EasyPaisa / Bank APIs + Sandbox mode | Live payments and a mock gateway for testing/demos (Super Admin toggle); Cash records for walk-ins |
| Maps & ETA | Google Maps API + Geolocator | Venue pins, directions, on-device ETA computation for “Leave Now” alerts |
| QR & Security | UUID QR + server-side signature (qr_flutter, mobile_scanner) | One-time unique encrypted QR per ticket; online server-match verification; atomic one-time use |
| Crash Reporting | Firebase Crashlytics | Automatic crash/error detection and reporting (NFR013) |
| PDF Generation | Flutter pdf/printing packages | On-device ticket receipts and participation certificates (no server cost) |
| Analytics | Firebase Analytics | Usage insights within free quota |
| Tools & VCS | Android Studio / VS Code, Git & GitHub | Development, testing, version control |

# 2. Requirements

The system provides a centralized, fully digital platform for university events supporting five roles with role-check-based panels. It combines secure ticketing (one-time unique QR, online verification, real-time multi-gate sync), governance (approval workflow, promotion hierarchy, audit logs), and engagement (discovery feeds, Q&A, feedback, referrals) — all engineered for the Firebase free (Spark) plan.

## 2.1 Functional Requirements

Each requirement defines a specific functionality with its purpose, users, input, and expected output.

### 2.1.1 FR001 – User Authentication & Role-Based Panel Access

| Name | FR001 |
|---|---|
| Purpose | To allow all users — including the Super Admin — to securely register and log in through a single authentication flow. After login, the system checks the user's role and opens the corresponding panel (Student, Volunteer, Organizer, Admin, or Super Admin). |
| User(s) | User (Student), Volunteer, Organizer, Admin, Super Admin |
| Input | Full Name, Email Address, Phone, Password, University Roll Number (registration); Email + Password (login) |
| Output | Account created with default role “Student” and department auto-detected; on login, role check redirects the user to their assigned panel |

### 2.1.2 FR002 – Event Creation & Management

| Name | FR002 |
|---|---|
| Purpose | To allow organizers to create, update, and manage events with rich content and all control settings. |
| User(s) | Organizer |
| Input | Title, description, date/time, venue (dropdown from Venue Database), category, level (University/Faculty/Department), gallery images (max 6, compressed), speaker profiles (photo, bio, topic), interest tags, ticket mode (Simple/Tiers), capacity, booking mode (Online/Physical/Both), allowOutsiders toggle, refundAllowed toggle |
| Output | Event saved with status “Pending Approval”; visible to users only after Admin approval |

### 2.1.3 FR003 – Anti-Clash Scheduling

| Name | FR003 |
|---|---|
| Purpose | To prevent scheduling conflicts by matching the selected venue ID (from the Venue Database) and overlapping date/time ranges against existing events. |
| User(s) | Organizer, System |
| Input | venueId, start date/time, end date/time |
| Output | Conflict validation result; creation blocked with the conflicting event details if a clash exists |

### 2.1.4 FR004 – Event Categorization

| Name | FR004 |
|---|---|
| Purpose | To classify events by category and multi-tier level. |
| User(s) | Organizer |
| Input | Event category and level (University / Faculty / Department) |
| Output | Categorized event; level used in filters and dashboards |

### 2.1.5 FR005 – Ticket Booking System

| Name | FR005 |
|---|---|
| Purpose | To allow users to book tickets for published events with real-time availability. |
| User(s) | User |
| Input | Selected event, ticket tier (if tiers enabled), quantity |
| Output | Tickets reserved; booking record created; proceeds to payment |

### 2.1.6 FR006 – Department-Based Dual Pricing

| Name | FR006 |
|---|---|
| Purpose | To automatically apply different ticket prices based on department: host-department students pay the Student (insider) rate; students of other departments pay the Outsider rate. Applies within every tier. |
| User(s) | System |
| Input | User's department (detected from roll number at registration) and the event's host department |
| Output | Correct price auto-applied at booking; if allowOutsiders is OFF, other-department students cannot book |

### 2.1.7 FR007 – Real-Time Ticket Inventory

| Name | FR007 |
|---|---|
| Purpose | To track available tickets per event and per tier in real time across all devices and both booking channels (online + walk-in share one inventory). |
| User(s) | User, Organizer |
| Input | Booking, cancellation, and walk-in issuance events |
| Output | Updated live ticket counts everywhere (transactions + listeners) |

### 2.1.8 FR008 – Waitlist System

| Name | FR008 |
|---|---|
| Purpose | To queue users when tickets are sold out and allocate released tickets automatically. |
| User(s) | User |
| Input | Waitlist join/leave request |
| Output | User added with queue position; on cancellation or capacity increase, the first queued user is notified with a time-bound booking window |

### 2.1.9 FR009 – Payment Integration

| Name | FR009 |
|---|---|
| Purpose | To process secure ticket payments in LIVE mode (JazzCash / EasyPaisa / Bank Card) or SANDBOX mode (mock gateway), with PDF receipts generated on-device. |
| User(s) | User |
| Input | Payment method and details; referral points discount (optional) |
| Output | Payment confirmation; booking confirmed; QR ticket issued; transaction recorded |

### 2.1.10 FR010 – Refund Management

| Name | FR010 |
|---|---|
| Purpose | To process refunds: user-initiated cancellations follow the event's refundAllowed policy; event cancellation triggers automatic FULL refunds for all booked users; postponement offers a refund opt-out window. |
| User(s) | Organizer, Admin, System |
| Input | Refund request / lifecycle trigger |
| Output | Refund processed; payment status “Refunded”; receipt and notification to user; audit log entry |

### 2.1.11 FR011 – QR-Based Ticket Validation

| Name | FR011 |
|---|---|
| Purpose | To verify tickets securely using one-time unique QR codes matched online against server ticket records. |
| User(s) | Volunteer |
| Input | Scanned QR payload (unique ticket ID + signature) |
| Output | Verification result: Valid Entry / Fake Ticket / Tampered QR / Wrong Event / Already Scanned (with gate and time details) |

### 2.1.12 FR012 – Real-Time Multi-Gate Synchronization

| Name | FR012 |
|---|---|
| Purpose | To ensure that when a ticket is scanned at any gate, its “Used” status is instantly updated server-side and reflected on all other volunteer devices, preventing double entry. An active internet connection is mandatory for scanning. |
| User(s) | Volunteer, System |
| Input | QR scan data sent to the server over an active connection |
| Output | Ticket marked “Used” via atomic transaction; live status update on all gates within seconds; simultaneous scans resolved — only the first succeeds |

### 2.1.13 FR013 – Notification System

| Name | FR013 |
|---|---|
| Purpose | To deliver all alerts through an in-app notification center and device-scheduled local notifications (booking confirmation, reminders 24h/1h before, Leave Now ETA, bookmark, waitlist, approval, promotion, refund, certificate, feedback requests) within the free plan. |
| User(s) | System |
| Input | Event/booking/role triggers |
| Output | Notifications delivered in-app (badge count) and as local device alerts; user category preferences respected |

### 2.1.14 FR014 – Volunteer Management

| Name | FR014 |
|---|---|
| Purpose | To assign volunteers to events and track their duty performance. |
| User(s) | Organizer |
| Input | Volunteer assignment; check-in/check-out times; scan counts |
| Output | Assignments stored; performance data available in organizer reports |

### 2.1.15 FR015 – Certificate Generation

| Name | FR015 |
|---|---|
| Purpose | To generate participation certificates as PDFs on-device for users marked Present through QR attendance after event completion. |
| User(s) | System |
| Input | Finalized attendance data |
| Output | PDF certificate (event name, user name, date, unique ID) delivered via notification and My Certificates screen |

### 2.1.16 FR016 – Social Sharing & Engagement

| Name | FR016 |
|---|---|
| Purpose | To allow sharing of events and tickets through device share sheets (WhatsApp, social media). |
| User(s) | User |
| Input | Event / ticket / referral share action |
| Output | Shareable content generated (deep link, ticket card, invite message) |

### 2.1.17 FR017 – Super Admin Panel & Role Hierarchy

| Name | FR017 |
|---|---|
| Purpose | To provide the Super Admin full control over the entire system, including searching normal users and promoting them to Admins. Once promoted, the Admin Panel becomes visible to that user. |
| User(s) | Super Admin |
| Input | System data, user search query, Admin promotion request |
| Output | System controlled; selected user promoted to Admin; Admin Panel enabled; audit log recorded |

### 2.1.18 FR018 – Event Location & Navigation

| Name | FR018 |
|---|---|
| Purpose | To display event venues on a map (pin from the Venue Database) and provide directions from the user's current location. |
| User(s) | User |
| Input | Venue map pin; user GPS location |
| Output | Map view with marker; navigation route displayed; opens external maps app on tap |

### 2.1.19 FR019 – Admin Panel & Organizer Promotion

| Name | FR019 |
|---|---|
| Purpose | To allow Admins to manage system operations through the Admin Panel and search normal users to promote any user to an Event Organizer. |
| User(s) | Admin |
| Input | User search query, Organizer promotion request |
| Output | Selected user becomes Event Organizer; Organizer Panel enabled; audit log recorded |

### 2.1.20 FR020 – Volunteer Promotion by Organizer

| Name | FR020 |
|---|---|
| Purpose | To allow Event Organizers to search users and make any user a Volunteer. |
| User(s) | Organizer |
| Input | User search query, Volunteer promotion request |
| Output | Selected user becomes Volunteer; Volunteer (scanner) Panel enabled; audit log recorded |

### 2.1.21 FR021 – Event Approval Workflow

| Name | FR021 |
|---|---|
| Purpose | To ensure no event is published without Admin approval. Events remain “Pending Approval” and become visible to users only after approval; rejections include a reason and allow resubmission. |
| User(s) | Organizer, Admin |
| Input | Event data; approval decision (Approve / Reject with reason) |
| Output | Approved event published and notified; rejection reason sent to organizer; audit log recorded |

### 2.1.22 FR022 – Unique QR Generation & Verification

| Name | FR022 |
|---|---|
| Purpose | To generate a one-time unique encrypted QR code per ticket at issuance (static — no rotation), embedding a unique ticket ID and signature whose secret lives only server-side. On scanning, the ID is matched online against server records: match = valid, no match = fake, used = already scanned. |
| User(s) | System, Volunteer |
| Input | Scanned QR payload (unique ticket ID + signature) |
| Output | Verification result and attendance record; screenshot reuse blocked by one-time-use + live multi-gate sync |

### 2.1.23 FR023 – Audit Logs

| Name | FR023 |
|---|---|
| Purpose | To automatically record all sensitive actions — role promotions/demotions, event approvals/rejections, postponements, cancellations, refunds, deactivations — with actor, target, old value, new value, and timestamp. Logs are tamper-proof and viewable by the Super Admin only. |
| User(s) | System, Super Admin |
| Input | Sensitive system actions |
| Output | Immutable log entries displayed in the Super Admin audit trail |

### 2.1.24 FR024 – Roll Number Registration & Department Detection

| Name | FR024 |
|---|---|
| Purpose | To register every student with their university roll number and automatically detect their department from its code, which drives department-based dual pricing. Invalid formats and duplicate roll numbers are rejected; the department is locked after registration. |
| User(s) | User, System |
| Input | University roll number (e.g., S23BINFT1M01017) |
| Output | Department stored in user profile (e.g., BINFT → BS Information Technology); “Verified Student” status |

### 2.1.25 FR025 – Live Attendance Counter

| Name | FR025 |
|---|---|
| Purpose | To display real-time entry statistics during events (scanned vs total, gate-wise counts, progress graph) on Organizer, Admin, and Super Admin dashboards. |
| User(s) | Organizer, Admin, Super Admin |
| Input | Live QR scan/attendance data |
| Output | Real-time counters and graphs; peak-entry insights in final reports |

### 2.1.26 FR026 – Feedback & Rating System

| Name | FR026 |
|---|---|
| Purpose | To allow attended users to rate (1–5 stars) and review an event after completion; averages are shown on event and organizer profiles. |
| User(s) | User, Organizer |
| Input | Star rating and review text (feedback window after completion) |
| Output | Feedback stored; average rating displayed; organizer reply/view available |

### 2.1.27 FR027 – Sandbox (Mock) Payment Mode

| Name | FR027 |
|---|---|
| Purpose | To provide a simulated payment gateway for testing and demonstration when live merchant services are unavailable. Sandbox transactions behave like real payments but are clearly marked “Sandbox”. |
| User(s) | User; Super Admin (mode toggle) |
| Input | Mock payment credentials |
| Output | Simulated success/failure responses; transactions marked sandbox in all records and reports |

### 2.1.28 FR028 – Event Lifecycle Management (Postpone / Cancel / Complete)

| Name | FR028 |
|---|---|
| Purpose | To manage event status transitions: Postponed (reason, anti-clash re-check, Admin re-approval, notifications, tickets remain valid with a 48-hour full-refund opt-out window), Cancelled (automatic FULL refunds to all booked users, QR invalidation, waitlist clearing), and Completed (automatic at end time or manual; finalizes attendance and triggers certificates, feedback window, and final reports). |
| User(s) | Organizer, Admin, System |
| Input | Status-change request with reason (and new schedule for postponement) |
| Output | Status updated; all affected users notified; refunds/certificates/feedback/reports triggered; audit log recorded |

### 2.1.29 FR029 – Event Media Gallery

| Name | FR029 |
|---|---|
| Purpose | To allow organizers to upload up to six images per event, displayed as a swipeable gallery. Images are compressed on-device before upload to stay within free storage limits. |
| User(s) | Organizer |
| Input | Multiple event images (auto-compressed ≤ 200KB each) |
| Output | Gallery displayed on the event details screen with full-screen viewer |

### 2.1.30 FR030 – Speaker & Guest Profiles

| Name | FR030 |
|---|---|
| Purpose | To display a “Who's Coming?” section on event pages showing guest speakers with photo, name, designation, bio, and talk topic. |
| User(s) | Organizer, User |
| Input | Speaker name, photo, designation, bio, topic |
| Output | Speaker cards rendered on the event details screen |

### 2.1.31 FR031 – Smart Discovery Sections

| Name | FR031 |
|---|---|
| Purpose | To organize dashboards into time-based and popularity-based sections: Happening Today, This Week, Coming Soon, and Trending (highest bookingsCount, 🔥 badge) using denormalized counters and indexed queries. |
| User(s) | User, System |
| Input | Published events with dates and live booking counters |
| Output | Categorized event sections on the student dashboard |

### 2.1.32 FR032 – Interest Tags & Personalized Feed

| Name | FR032 |
|---|---|
| Purpose | To let users follow interest tags (selected at onboarding, editable in profile), generating a personalized “For You” feed via array-contains-any queries on event tags. |
| User(s) | User, Organizer (tags events) |
| Input | Selected interest tags; event tags |
| Output | “For You” feed of matching events; new-match notifications via pull model on app open |

### 2.1.33 FR033 – Rule-Based Recommendations

| Name | FR033 |
|---|---|
| Purpose | To recommend events using simple rules (no machine learning): events of the user's own department and events similar (by tags) to those the user attended or bookmarked. |
| User(s) | System |
| Input | User department, attendance history, bookmarks |
| Output | “Recommended for You” and “Because you attended X” sections |

### 2.1.34 FR034 – Bookmarks & Interested Events

| Name | FR034 |
|---|---|
| Purpose | To allow users to mark events as “Interested”, saving them in a bookmark list with automatic device-scheduled local reminders. |
| User(s) | User |
| Input | Bookmark action on an event |
| Output | Event saved to My Interests; local reminder scheduled before the event |

### 2.1.35 FR035 – Advanced Combined Filters

| Name | FR035 |
|---|---|
| Purpose | To filter events by combining criteria: Free/Paid, date range, host department, level (University/Faculty/Department), and venue. |
| User(s) | User |
| Input | Selected filter combinations |
| Output | Filtered event list matching all criteria; “No results” state with reset |

### 2.1.36 FR036 – Referral & Invite System

| Name | FR036 |
|---|---|
| Purpose | To give every user a unique referral code; when a friend registers with it, both users earn reward points redeemable as ticket discounts. Self-referral is blocked by rules. |
| User(s) | User, System |
| Input | Referral code at registration; redemption at booking |
| Output | Points credited to both users; discount applied at payment; referral history stored |

### 2.1.37 FR037 – Event Chat & Comments (Q&A)

| Name | FR037 |
|---|---|
| Purpose | To provide a moderated question-answer section on each event page; organizers can reply and pin answers; organizers/admins can hide inappropriate comments. |
| User(s) | User, Organizer, Admin |
| Input | Comment/question text |
| Output | Thread displayed on the event page with pinned organizer answers; moderation controls |

### 2.1.38 FR038 – Venue Database & ID-Based Anti-Clash

| Name | FR038 |
|---|---|
| Purpose | To maintain an Admin-managed venue registry (name, campus, capacity, facilities, photos, map pin). Organizers select venues from a dropdown; clash checks match by venueId, making conflict detection exact and typo-proof. |
| User(s) | Admin (manages), Organizer (selects), System (validates) |
| Input | Venue records; selected venueId with date/time |
| Output | Venue dropdown at event creation; exact ID-based clash validation; map pins for navigation |

### 2.1.39 FR039 – Optional Seat Tiers

| Name | FR039 |
|---|---|
| Purpose | To let organizers optionally define seat tiers (e.g., VIP, Front, General) with separate student/outsider prices and quantities. If no tiers are defined, the system treats all seats as a single “General” tier with equal pricing. Department-based pricing applies within every tier. |
| User(s) | Organizer, System |
| Input | Tier-mode selection; tier names, prices, quantities (optional) |
| Output | Tier-based booking screen with per-tier live availability, OR equal pricing for all seats in simple mode |

### 2.1.40 FR040 – “Leave Now” Smart ETA Alert

| Name | FR040 |
|---|---|
| Purpose | To notify users holding tickets for today's events of the right departure time, computed on-device from GPS location and the venue pin: “Event starts in 1 hour — leave now, travel time ~40 min”. Tapping opens map navigation. |
| User(s) | System, User |
| Input | User live location, venue coordinates, event start time, user preference toggle |
| Output | Device-scheduled local notification with ETA and one-tap navigation |

### 2.1.41 FR041 – Booking Mode Control (Online / Physical / Hybrid)

| Name | FR041 |
|---|---|
| Purpose | To allow the organizer to choose how tickets are booked: Online (app only), Physical (walk-in tickets manually issued by the organizer), or Both. Both channels share the same live inventory; walk-in tickets receive the same unique QR and entry verification. Online booking can be switched off at any time (existing tickets stay valid). |
| User(s) | Organizer, User |
| Input | Booking-mode selection; walk-in details (roll number, name, quantity, tier, Cash/Free payment) |
| Output | Online booking enabled/disabled accordingly; walk-in tickets issued with QR (delivered to the user's app if registered, or as a shareable guest ticket); cash payments recorded in reports |

### 2.1.42 FR042 – Department Access Control (Allow Outsiders Toggle)

| Name | FR042 |
|---|---|
| Purpose | To let the organizer enable or disable participation of students from other departments. When OFF, only host-department students can book or receive walk-in tickets; when ON, other-department students participate at the outsider rate. |
| User(s) | Organizer, System |
| Input | allowOutsiders toggle (ON/OFF) at event creation or update |
| Output | Booking and walk-in issuance automatically allowed or blocked based on the user's department |

## 2.2 Non-Functional Requirements

| ID | Quality Attribute | Requirement Description |
|---|---|---|
| NFR001 | Performance | The system responds to user actions within 2–3 seconds under normal conditions, including event loading, booking, and QR validation. |
| NFR002 | Usability | The interface is simple and intuitive; first-time users navigate without external guidance; onboarding introduces core flows. |
| NFR003 | Security | User data, payments, and QR tickets are stored and transmitted securely (Firebase Auth, HTTPS, encrypted QR signatures whose secrets remain server-side). |
| NFR004 | Reliability | The system operates continuously without crashes and handles errors gracefully without data loss. |
| NFR005 | Scalability | The system supports large numbers of users, events, and transactions via Firestore real-time infrastructure and optimized queries. |
| NFR006 | Availability | The application is available 24/7 with minimal downtime. |
| NFR007 | Maintainability | Modular feature-based architecture (13 modules) allows easy updates, fixes, and extensions. |
| NFR008 | Portability | Runs smoothly on Android devices with varying screen sizes and hardware. |
| NFR009 | Data Consistency | Tickets, payments, attendance, and inventory remain accurate and synchronized across all users, gates, and devices (atomic transactions). |
| NFR010 | QR Integrity | Each QR code is generated once per ticket, contains a unique encrypted ID verified online against server records, and is usable only once. Unmatched IDs are rejected as fake; any re-scan at any gate immediately displays “Already Scanned”. |
| NFR011 | Location Accuracy | Accurate venue pins and navigation routes; on-device ETA computation with optional Directions API. |
| NFR012 | Role Integrity | Role promotions may only be performed by the authorized higher role (Super Admin → Admin → Organizer → Volunteer), are enforced server-side by Firestore security rules, and are recorded in tamper-proof audit logs. |
| NFR013 | Crash Reporting & Monitoring | Firebase Crashlytics automatically detects, reports, and logs crashes and errors in real time for continuous reliability improvement. |
| NFR014 | Online Verification Constraint | Volunteer scanning devices must have an active internet connection during entry verification. The app displays live connectivity status and disables the scanner with a clear warning when offline. Event venues must provide Wi-Fi/cellular coverage with a backup hotspot arrangement. |
| NFR015 | Free-Tier Efficiency | The complete system operates within Firebase Spark (free) limits — 50,000 reads/day, 20,000 writes/day, 1 GiB storage — using offline persistence caching, pagination, denormalized counters, batched writes, on-device image compression, local notifications, and client-side scheduling instead of paid server functions. |

# 3. Use Cases and Flow of Processes

Actors: User (Student), Volunteer, Organizer, Admin, Super Admin, and the System (automated processes). The table below maps all use cases to functional requirements.

| Use Case | Name | Actor(s) | Requirement(s) |
|---|---|---|---|
| UC001 | User Registration and Login | All Users | FR001, FR024 |
| UC002 | Create Event | Organizer | FR002, FR003, FR029, FR030, FR038–FR042 |
| UC003 | Schedule Validation | Organizer, System | FR003, FR038 |
| UC004 | Book Ticket | User | FR005–FR007, FR039, FR041, FR042 |
| UC005 | Join Waitlist | User | FR008 |
| UC006 | Process Payment | User | FR009, FR027, FR036 |
| UC007 | Scan Ticket | Volunteer | FR011, FR012, FR022, NFR014 |
| UC008 | Receive Notifications | System, User | FR013 |
| UC009 | Manage Volunteers | Organizer | FR014, FR020 |
| UC010 | Generate Certificates | System | FR015 |
| UC011 | Super Admin & Role Management | Super Admin | FR017, FR023, FR027 |
| UC012 | Event Location & Navigation | User | FR018, FR040 |
| UC013 | Role Promotion Hierarchy | Super Admin, Admin, Organizer | FR017, FR019, FR020, FR023 |
| UC014 | Event Approval | Organizer, Admin | FR021 |
| UC015 | Submit Feedback & Rating | User | FR026 |
| UC016 | Event Lifecycle Management | Organizer, Admin, System | FR028, FR010, FR003 |
| UC017 | Discover & Browse Events | User | FR031–FR035 |
| UC018 | Invite Friend (Referral) | User | FR036 |
| UC019 | Event Q&A | User, Organizer | FR037 |
| UC020 | Issue Walk-in Ticket | Organizer | FR041, FR042, FR006, FR007 |

## 3.1 Use Case 1 — User Registration and Login

| ID | UC001 |
|---|---|
| Name | User Registration and Login |
| Description | Users create an account with their university roll number (department auto-detected) or log in; after a role check, the panel matching their role opens — including the Super Admin Panel for the pre-seeded Super Admin account. |
| Requirement(s) | FR001, FR024 |
| Actor(s) | User, Organizer, Volunteer, Admin, Super Admin |
| Precondition | User has access to the application |
| Postcondition | User logged in and viewing the panel assigned to their role |
| Basic Flow | 1. User opens the application<br>2. New user registers: name, email, phone, password, roll number; existing user enters email + password<br>3. System validates input (email format, password ≥ 8 characters, valid & unique roll number)<br>4. System auto-detects department from roll number; new users receive default role “Student”<br>5. Firebase Authentication verifies credentials<br>6. System reads the user's role from Firestore (role check)<br>7. Respective panel opens: Student / Volunteer / Organizer / Admin / Super Admin |
| Alternative Flow | • First-time registration continues to Interest Tags selection → personalized “For You” feed |
| Exceptions | • Invalid input → error message below the field<br>• Duplicate roll number → registration rejected<br>• Deactivated account → access denied<br>• Network issue → retry option |

Diagram: The activity diagram for this use case is provided in the diagrams section / SDD.

## 3.2 Use Case 2 — Create Event

| ID | UC002 |
|---|---|
| Name | Create Event |
| Description | An organizer creates an event with rich content (gallery, speakers, tags), venue from the Venue Database, ticket setup (simple equal pricing or optional tiers), booking mode (Online/Physical/Both), allowOutsiders and refund policies. The system validates input, performs an ID-based anti-clash check, and stores the event as Pending Approval. |
| Requirement(s) | FR002, FR003, FR029, FR030, FR038–FR042 |
| Actor(s) | Organizer |
| Precondition | Organizer is logged in |
| Postcondition | Event stored with status “Pending Approval” awaiting Admin decision |
| Basic Flow | 1. Organizer selects “Create Event” from the dashboard<br>2. System displays the event creation form<br>3. Organizer enters details: title, description, date/time, venue (dropdown), category, level, tags<br>4. Organizer uploads gallery images (≤6, auto-compressed) and speaker profiles (photo, bio, topic)<br>5. Organizer configures tickets: Simple mode (equal pricing) or Tier mode (VIP/Front/General with student & outsider prices), capacity<br>6. Organizer sets booking mode (Online/Physical/Both), allowOutsiders, refundAllowed and submits<br>7. System validates data and runs the anti-clash check (venueId + overlapping time)<br>8. If valid, event is saved with status “Pending Approval” and the Admin is notified |
| Alternative Flow | • Organizer saves as draft and resumes later; edits and resubmits after rejection |
| Exceptions | • Required fields empty → validation errors<br>• Venue/time clash → creation prevented with conflict details<br>• Image upload fails → retry or skip<br>• Network error → data not saved |

Diagram: The activity diagram for this use case is provided in the diagrams section / SDD.

## 3.3 Use Case 3 — Schedule Validation

| ID | UC003 |
|---|---|
| Name | Schedule Validation |
| Description | The system prevents an event from being scheduled when the selected venue (by ID) already has an overlapping event on the same date/time range. |
| Requirement(s) | FR003, FR038 |
| Actor(s) | Organizer, System |
| Precondition | Organizer is creating or updating an event |
| Postcondition | Event schedule is approved (no conflict) or rejected (conflict detected) |
| Basic Flow | 1. Organizer selects venue from the Venue Database dropdown and enters date/time<br>2. System queries events with the same venueId for overlapping ranges<br>3. No conflict → validation succeeds; creation continues<br>4. Conflict → system shows the conflicting event (title, date, time) |
| Alternative Flow | • Organizer modifies date/time/venue → system re-validates |
| Exceptions | • Venue already booked → conflict message<br>• Invalid date/time (past date) → validation error |

Diagram: The activity diagram for this use case is provided in the diagrams section / SDD.

## 3.4 Use Case 4 — Book Ticket

| ID | UC004 |
|---|---|
| Name | Book Ticket |
| Description | A user books tickets for a published event: real-time availability, optional tier selection, automatic department-based pricing, allowOutsiders enforcement, and reservation through atomic transactions. |
| Requirement(s) | FR005–FR007, FR039, FR041, FR042, FR006 |
| Actor(s) | User (Student) |
| Precondition | User is logged in; event is Published; booking mode allows online booking |
| Postcondition | Tickets reserved and booking proceeds to payment |
| Basic Flow | 1. User opens the event details screen showing live availability and tiers<br>2. User selects tier (if enabled) and quantity<br>3. System compares user's department with host department → applies student or outsider rate per tier<br>4. If allowOutsiders is OFF and user is from another department → booking blocked with message<br>5. System reserves tickets in an atomic transaction; live inventory updates everywhere<br>6. User proceeds to payment (skipped for free events) |
| Alternative Flow | • User changes quantity/tier → pricing recalculated dynamically<br>• Booking mode = Physical only → screen shows “On-Spot Registration Only” |
| Exceptions | • Sold out → waitlist suggested<br>• Quantity exceeds availability → error<br>• Booking window expires → reservation released<br>• Network issue → booking not processed |

Diagram: The activity diagram for this use case is provided in the diagrams section / SDD.

## 3.5 Use Case 5 — Join Waitlist

| ID | UC005 |
|---|---|
| Name | Join Waitlist |
| Description | When tickets are sold out, users join a queue; released tickets (cancellation or capacity increase) are offered to the first queued user with a time-bound booking window. |
| Requirement(s) | FR008 |
| Actor(s) | User (Student) |
| Precondition | User is logged in; event tickets are sold out |
| Postcondition | User is queued and notified when a ticket becomes available |
| Basic Flow | 1. User selects a sold-out event and taps “Join Waitlist”<br>2. System records the request and assigns a queue position<br>3. System confirms waitlist registration<br>4. A ticket is released → first queued user receives a notification with a time-bound booking window<br>5. User books within the window → status “Converted”; otherwise the offer passes to the next user |
| Alternative Flow | • User leaves the waitlist → removed from queue |
| Exceptions | • Already in waitlist → message shown<br>• Waitlist limit reached → request denied<br>• Booking window expires → next user notified |

Diagram: The activity diagram for this use case is provided in the diagrams section / SDD.

## 3.6 Use Case 6 — Process Payment

| ID | UC006 |
|---|---|
| Name | Process Payment |
| Description | The user completes payment in LIVE mode (JazzCash/EasyPaisa/Bank Card) or SANDBOX mode (mock gateway marked in records), optionally redeeming referral points, and receives an on-device PDF receipt. |
| Requirement(s) | FR009, FR027, FR036 |
| Actor(s) | User (Student) |
| Precondition | Tickets are reserved; payment required (price > 0) |
| Postcondition | Payment processed; booking confirmed; QR ticket issued |
| Basic Flow | 1. User proceeds to the payment screen<br>2. System displays available methods and the active mode (Live/Sandbox) and applies referral-points discount if redeemed<br>3. User selects a method and enters details<br>4. System processes the transaction (gateway or sandbox simulation)<br>5. Success → booking confirmed; one-time unique QR ticket issued; PDF receipt generated on-device<br>6. Transaction record stored |
| Alternative Flow | • User changes payment method → process restarts with new method |
| Exceptions | • Payment failed → error with retry; reservation auto-releases after timeout<br>• Invalid details → retry required<br>• Gateway timeout → transaction marked pending<br>• Network issue → payment not processed |

Diagram: The activity diagram for this use case is provided in the diagrams section / SDD.

## 3.7 Use Case 7 — Scan Ticket

| ID | UC007 |
|---|---|
| Name | Scan Ticket |
| Description | A volunteer with an active internet connection scans a one-time unique QR; the server matches the ticket ID against its records, verifies signature and event match, and atomically marks it Used. All gates receive the update in real time; a re-scan anywhere shows “Already Scanned”. |
| Requirement(s) | FR011, FR012, FR022, NFR014 |
| Actor(s) | Volunteer |
| Precondition | Volunteer is logged in with the assigned event; device has active internet; attendee holds a valid QR ticket |
| Postcondition | Ticket verified and marked “Used”; attendance recorded; entry granted; all gates synchronized |
| Basic Flow | 1. Volunteer opens the scanner for the assigned event; app verifies connectivity (offline → scanner locked with warning)<br>2. Volunteer scans the QR code<br>3. Scan payload is sent to the server<br>4. Server checks: unique ID exists in ticket records (else FAKE), signature valid (else TAMPERED), ticket belongs to this event (else WRONG EVENT), status is Active (else ALREADY SCANNED with gate & time)<br>5. All checks pass → atomic transaction marks the ticket Used; attendance record created (volunteer, gate, time)<br>6. Entry granted; every other gate device shows the ticket as Used within seconds; live attendance counter increments |
| Alternative Flow | • Same ticket scanned simultaneously at two gates → only the first transaction succeeds; the second instantly shows “Already Scanned” |
| Exceptions | • Fake/unmatched QR → access denied<br>• Tampered QR → rejected<br>• Already used → entry rejected with usage details<br>• No internet → scanner disabled with warning<br>• Weak network → automatic retry with “Verifying…” state |

Diagram: The activity diagram for this use case is provided in the diagrams section / SDD.

## 3.8 Use Case 8 — Receive Notifications

| ID | UC008 |
|---|---|
| Name | Receive Notifications |
| Description | The system informs users of every significant event through the in-app notification center and device-scheduled local notifications (free plan; no push service required). |
| Requirement(s) | FR013 |
| Actor(s) | System, User |
| Precondition | User is registered; notification categories enabled in Settings |
| Postcondition | User receives alerts in-app and on-device |
| Basic Flow | 1. A trigger occurs (booking, approval decision, promotion, postponement, cancellation, waitlist slot, refund, certificate, feedback request, reminder, Leave Now ETA)<br>2. System writes an in-app notification document → badge count updates live<br>3. Device-scheduled local notifications fire at their planned times (24h/1h reminders, ETA alerts, bookmark reminders)<br>4. User opens the Notification Center and reads entries |
| Alternative Flow | • User disables a category in Settings → those notifications are skipped |
| Exceptions | • Delivery failure → entry remains in the center<br>• Scheduled notification missed (device off) → shown in-app |

Diagram: The activity diagram for this use case is provided in the diagrams section / SDD.

## 3.9 Use Case 9 — Manage Volunteers

| ID | UC009 |
|---|---|
| Name | Manage Volunteers |
| Description | An organizer searches registered users, promotes them to Volunteers, assigns them to events with gates, and tracks duty performance (check-in/out, scan counts). |
| Requirement(s) | FR014, FR020 |
| Actor(s) | Organizer |
| Precondition | Organizer is logged in; event exists |
| Postcondition | Volunteers promoted, assigned, and tracked |
| Basic Flow | 1. Organizer opens Manage Volunteers for an event<br>2. Searches a registered user and selects “Make Volunteer” → role updated + audit log + notification<br>3. Assigns the volunteer to the event and gate<br>4. During the event, tracks check-in/check-out and live scan counts<br>5. Performance data stored for reports |
| Alternative Flow | • Organizer removes or reassigns a volunteer → access updated |
| Exceptions | • User already a volunteer → message<br>• Duplicate assignment → error<br>• Network issue → data not updated |

Diagram: The activity diagram for this use case is provided in the diagrams section / SDD.

## 3.10 Use Case 10 — Generate Certificates

| ID | UC010 |
|---|---|
| Name | Generate Certificates |
| Description | After completion, the system finalizes attendance and generates PDF certificates on-device for Present users (free-plan friendly), delivering them through notifications and the My Certificates screen. |
| Requirement(s) | FR015 |
| Actor(s) | System |
| Precondition | Event is Completed; attendance finalized via QR scans |
| Postcondition | Certificates generated and delivered to eligible users |
| Basic Flow | 1. Event status becomes Completed<br>2. System finalizes attendance (scanned = Present; others = Absent)<br>3. PDF certificates generated on-device for Present users (event name, user name, date, unique certificate ID)<br>4. Certificates delivered via notification + My Certificates download/share |
| Alternative Flow | • Organizer manually triggers generation |
| Exceptions | • No attendance data → no certificates<br>• PDF generation failure → retry |

Diagram: The activity diagram for this use case is provided in the diagrams section / SDD.

## 3.11 Use Case 11 — Super Admin & Role Management

| ID | UC011 |
|---|---|
| Name | Super Admin & Role Management |
| Description | The Super Admin controls the whole platform: promotes Admins, reviews tamper-proof audit logs and system analytics, deactivates accounts, toggles payment mode (Live ↔ Sandbox), and exports reports. |
| Requirement(s) | FR017, FR023, FR027 |
| Actor(s) | Super Admin |
| Precondition | Super Admin logs in via the standard role check |
| Postcondition | System governed; promotions and settings applied with audit trails |
| Basic Flow | 1. Super Admin opens the Super Admin Panel (role check)<br>2. Views system analytics (users, events, revenue)<br>3. Searches a normal user → “Make Admin” → role updated, Admin Panel enabled for that user, audit log recorded, notification sent<br>4. Reviews the audit trail (promotions, approvals, cancellations, refunds)<br>5. Deactivates accounts when necessary<br>6. Toggles payment mode Live ↔ Sandbox; exports system reports |
| Alternative Flow | • Searches/filters users and logs |
| Exceptions | • Unauthorized action → denied<br>• Network issue → change not saved |

Diagram: The activity diagram for this use case is provided in the diagrams section / SDD.

## 3.12 Use Case 12 — Event Location & Navigation (+ Leave Now)

| ID | UC012 |
|---|---|
| Name | Event Location & Navigation (+ Leave Now) |
| Description | Users view the venue on a map (pin from the Venue Database), get directions, and — for today's booked events — receive an on-device “Leave Now” ETA alert computed from GPS and the venue pin. |
| Requirement(s) | FR018, FR040 |
| Actor(s) | User |
| Precondition | User is logged in; event has a valid venue pin; GPS enabled for navigation/ETA |
| Postcondition | Location displayed; route provided; timely departure alert delivered |
| Basic Flow | 1. User opens event details → “View Location” → map with venue marker<br>2. User taps “Get Directions” → route computed and displayed; external maps app opens on tap<br>3. For today's booked events, the app computes travel ETA on-device from current location<br>4. “Leave Now” local notification fires at the right departure time (if enabled) |
| Alternative Flow | • GPS disabled → venue location shown without navigation/ETA |
| Exceptions | • Location unavailable → error<br>• Map load failure → retry<br>• ETA computation failure → fallback reminder only |

Diagram: The activity diagram for this use case is provided in the diagrams section / SDD.

## 3.13 Use Case 13 — Role Promotion Hierarchy

| ID | UC013 |
|---|---|
| Name | Role Promotion Hierarchy |
| Description | Higher authorities search normal users and promote them one level down: Super Admin → Admin, Admin → Organizer, Organizer → Volunteer. Promoted users see their new panel through the role check; every action is audited. |
| Requirement(s) | FR017, FR019, FR020, FR023, NFR012 |
| Actor(s) | Super Admin, Admin, Organizer |
| Precondition | The promoting user is logged in with the required role |
| Postcondition | Target user's role updated; corresponding panel enabled; audit log recorded |
| Basic Flow | 1. Super Admin searches a normal user → “Make Admin” → Admin Panel becomes visible to that user<br>2. Admin searches a normal user → “Make Organizer” → Organizer Panel enabled<br>3. Organizer searches a user → “Make Volunteer” → Volunteer (scanner) Panel enabled<br>4. Each promotion: Firestore role update + audit log entry + notification to the promoted user |
| Alternative Flow | • Demotion by the same authority → panel access revoked |
| Exceptions | • User already holds an equal/higher role → error<br>• Skipping hierarchy levels → blocked by server rules<br>• Network issue → role not updated |

Diagram: The activity diagram for this use case is provided in the diagrams section / SDD.

## 3.14 Use Case 14 — Event Approval

| ID | UC014 |
|---|---|
| Name | Event Approval |
| Description | Every event is reviewed by an Admin before publishing. Pending events are invisible to users; approval publishes and notifies; rejection returns a reason for resubmission. |
| Requirement(s) | FR021 |
| Actor(s) | Organizer, Admin |
| Precondition | Organizer has submitted a complete event |
| Postcondition | Event Published (approved) or Rejected with reason |
| Basic Flow | 1. Organizer submits event → status “Pending Approval”<br>2. Admin receives an in-app notification and opens the Approval Queue<br>3. Admin reviews details, gallery, speakers, pricing, and the anti-clash result<br>4. Approve → event Published, users notified, organizer informed<br>5. Reject → reason recorded and sent; organizer edits and resubmits |
| Alternative Flow | • Admin requests changes via rejection reason |
| Exceptions | • No action by Admin → pending reminder sent<br>• Network issue → decision not saved |

Diagram: The activity diagram for this use case is provided in the diagrams section / SDD.

## 3.15 Use Case 15 — Submit Feedback & Rating

| ID | UC015 |
|---|---|
| Name | Submit Feedback & Rating |
| Description | After completion, attended users rate (1–5 stars) and review the event; averages appear on event and organizer profiles. |
| Requirement(s) | FR026 |
| Actor(s) | User |
| Precondition | Event Completed; user marked Present via QR scan |
| Postcondition | Feedback stored; average rating updated |
| Basic Flow | 1. Event completes → attendees receive a “Rate the event” notification<br>2. User opens the feedback screen<br>3. Selects stars and writes an optional review<br>4. System stores feedback and updates the average rating (7-day window) |
| Alternative Flow | • User edits their own feedback within the window |
| Exceptions | • User did not attend → no feedback option<br>• Duplicate submission → previous feedback updated |

Diagram: The activity diagram for this use case is provided in the diagrams section / SDD.

## 3.16 Use Case 16 — Event Lifecycle Management

| ID | UC016 |
|---|---|
| Name | Event Lifecycle Management |
| Description | Manages post-publish status transitions: Postponed (re-approval, tickets valid, refund opt-out), Cancelled (automatic full refunds, QR invalidation, waitlist cleared), Completed (auto at end time; triggers certificates, feedback, final reports). |
| Requirement(s) | FR028, FR010, FR003 |
| Actor(s) | Organizer, Admin, System |
| Precondition | Event is Published |
| Postcondition | Event reaches its final state with all users informed and finances settled |
| Basic Flow | 1. Postpone: organizer submits reason + new schedule → anti-clash re-check → Admin re-approval → all booked users notified (tickets valid on new date; 48-hour full-refund opt-out window)<br>2. Cancel: reason submitted → Admin approval (Admin/Super Admin may cancel directly) → automatic FULL refunds to all booked users → QR tickets invalidated (“Event Cancelled” on scan) → waitlist cleared<br>3. Complete: end time passes → status Completed automatically (or organizer marks manually) → attendance finalized → certificates generated → feedback window opens → final reports delivered to Organizer/Admin/Super Admin |
| Alternative Flow | • Refund opt-out during postponement → ticket cancelled and refunded; seat returns to inventory/waitlist |
| Exceptions | • Postponement rejected → event stays Published<br>• Refund processing failure → flagged for Super Admin<br>• Auto-complete fails → organizer marks manually |

Diagram: The activity diagram for this use case is provided in the diagrams section / SDD.

## 3.17 Use Case 17 — Discover & Browse Events

| ID | UC017 |
|---|---|
| Name | Discover & Browse Events |
| Description | The dashboard organizes events into smart sections and a personalized feed with powerful combined filters and bookmarks. |
| Requirement(s) | FR031–FR035 |
| Actor(s) | User |
| Precondition | User is logged in |
| Postcondition | User finds and shortlists relevant events quickly |
| Basic Flow | 1. User opens the dashboard → sections: For You (interest tags), Happening Today, This Week, Coming Soon, Trending 🔥, Your Department, Recommended<br>2. User applies combined filters (Free/Paid, date range, department, level, venue) and search<br>3. User opens an event → gallery, “Who's Coming?” speakers, agenda, rating, Q&A preview<br>4. User taps “Interested” → bookmarked and a device reminder is scheduled |
| Alternative Flow | • Recommendations adapt as attendance/bookmark history grows (rule-based) |
| Exceptions | • No events match filters → “No results” with reset<br>• Images fail to load → cached/placeholder shown |

Diagram: The activity diagram for this use case is provided in the diagrams section / SDD.

## 3.18 Use Case 18 — Invite Friend (Referral)

| ID | UC018 |
|---|---|
| Name | Invite Friend (Referral) |
| Description | Users share a unique referral code; when a friend registers with it, both earn reward points redeemable as ticket discounts. |
| Requirement(s) | FR036 |
| Actor(s) | User |
| Precondition | User is logged in with a referral code |
| Postcondition | Points credited to both users on successful registration |
| Basic Flow | 1. User opens Referral & Points → unique code + share link (WhatsApp, etc.)<br>2. Friend registers entering the code<br>3. System validates (code exists, not self-referral)<br>4. Points credited to both users (e.g., 50 each)<br>5. Points redeemed as a discount during payment |
| Alternative Flow | • User checks referral history and point balance |
| Exceptions | • Invalid/unknown code → error<br>• Self-referral → blocked by rules<br>• Duplicate use by same referee → rejected |

Diagram: The activity diagram for this use case is provided in the diagrams section / SDD.

## 3.19 Use Case 19 — Event Q&A

| ID | UC019 |
|---|---|
| Name | Event Q&A |
| Description | Each event page hosts a moderated question-answer thread; organizers reply and pin key answers; inappropriate comments can be hidden. |
| Requirement(s) | FR037 |
| Actor(s) | User, Organizer |
| Precondition | Event is Published; user logged in |
| Postcondition | Questions answered publicly; important answers pinned |
| Basic Flow | 1. User opens the event → Q&A section<br>2. Posts a question (e.g., “Where is parking?”)<br>3. Organizer replies and/or pins the best answer<br>4. All visitors see the thread (listener active only while the page is open — read-efficient) |
| Alternative Flow | • Organizer/Admin hides inappropriate comments |
| Exceptions | • Offline → comment queued locally, posted when online<br>• Empty text → validation error |

Diagram: The activity diagram for this use case is provided in the diagrams section / SDD.

## 3.20 Use Case 20 — Issue Walk-in Ticket (Physical Booking)

| ID | UC020 |
|---|---|
| Name | Issue Walk-in Ticket (Physical Booking) |
| Description | The organizer manually issues tickets on the spot for students who did not book online, applying the same department, outsider, and inventory checks. The ticket carries the same unique QR; registered users receive it in-app, others get a shareable guest QR. |
| Requirement(s) | FR041, FR042, FR006, FR007 |
| Actor(s) | Organizer, System |
| Precondition | Event booking mode is Physical or Both; organizer logged in; internet available |
| Postcondition | Walk-in ticket issued with QR; inventory reduced; cash payment recorded |
| Basic Flow | 1. Organizer opens “Issue Walk-in Ticket” for the event<br>2. Enters the student's roll number, name, quantity, and tier<br>3. Selects payment: Cash Received or Free<br>4. System detects department from the roll number, applies insider/outsider rules and allowOutsiders, and checks live availability<br>5. Ticket issued → delivered to the user's My Tickets (if registered) or shown as a guest QR (share/print)<br>6. Inventory and cash reports update live |
| Alternative Flow | • Walk-in cancellation → organizer marks cash refund manually; record updated |
| Exceptions | • Sold out → issuance blocked<br>• Other-department student with allowOutsiders OFF → blocked<br>• Invalid roll number → error<br>• No internet → issuance paused (online verification required) |

Diagram: The activity diagram for this use case is provided in the diagrams section / SDD.

# 4. Test Plan & Test Cases

The system is tested against documented test cases covering every functional requirement. The complete test-case document is maintained with the project deliverables; representative cases are listed below.

| Test ID | Requirement | Test Case | Input | Expected Output |
|---|---|---|---|---|
| TC001 | FR001 | Login with valid credentials | Correct email + password | Login successful; role-based panel opens |
| TC002 | FR001 | Login with invalid password | Wrong password | Error message; access denied |
| TC003 | FR024 | Register with valid roll number | S23BINFT1M01017 | Department auto-detected (BS Information Technology); role = Student |
| TC004 | FR024 | Register with duplicate roll number | Already-registered roll no. | Registration rejected with message |
| TC005 | FR017/FR023 | Super Admin promotes user to Admin | Search user → Make Admin | Role updated; Admin Panel visible to that user; audit log entry created |
| TC006 | NFR012 | Organizer attempts to promote to Admin | Cross-level promotion | Blocked by server rules; access denied |
| TC007 | FR021 | Publish without approval | Organizer creates event | Status “Pending”; not visible to users until Admin approval |
| TC008 | FR003 | Create clashing event | Same venueId, overlapping time | Creation blocked; conflicting event shown |
| TC009 | FR006/FR042 | Other-department booking, allowOutsiders OFF | BS CS student books BS IT event | Booking blocked with restriction message |
| TC010 | FR006/FR039 | Department pricing within tiers | Host-dept student selects VIP tier | Student (insider) VIP rate auto-applied |
| TC011 | FR022 | Fake/tampered QR presented | QR not in ticket records | Server rejects: “Fake Ticket”; entry denied |
| TC012 | FR012 | Re-scan a used ticket at a different gate | Same QR scanned at Gate 2 after Gate 1 | Gate 2 instantly shows “Already Scanned” with gate/time details |
| TC013 | FR012 | Simultaneous scan at two gates | Same ticket at Gate 1 and Gate 2 together | Only one gate succeeds (atomic transaction); other shows “Already Scanned” |
| TC014 | NFR014 | Volunteer device without internet | Scanner opened with no connectivity | Scanner disabled; “Internet required” warning displayed |
| TC015 | FR027 | Sandbox payment | Mock gateway transaction | Success response; transaction marked “Sandbox” in records |
| TC016 | FR028 | Event cancellation refunds | Cancel a Published event with bookings | Automatic FULL refunds to all booked users; QR invalidated; waitlist cleared |
| TC017 | FR036 | Referral reward | Friend registers with code | Points credited to both users; self-referral blocked |
| TC018 | FR041 | Walk-in issuance when sold out | Issue ticket at zero inventory | Issuance blocked with “Sold Out” |

# 5. References

[1] Google, Firebase Documentation (Authentication, Firestore, Storage, Crashlytics, Spark Plan Limits), 2025. https://firebase.google.com/docs

[2] Flutter, Flutter Documentation (Material 3, local notifications, pdf, qr_flutter, mobile_scanner), 2025. https://docs.flutter.dev

[3] Google, Google Maps Platform Documentation, 2025. https://developers.google.com/maps

[4] IEEE, IEEE Recommended Practice for Software Requirements Specifications, IEEE Std 830-1998

[5] Pressman, R. S., Software Engineering: A Practitioner's Approach, 8th Edition, McGraw-Hill, 2019

[6] Sommerville, I., Software Engineering, 10th Edition, Pearson, 2016

[7] UniEvent 360 Project Proposal

[8] UniEvent 360 Software Requirements Specification — Original Version 1.0

# Appendix A: Change Log (Original SRS → Updated v2.0)

| Area | Original SRS (v1.0) | Updated SRS (v2.0) |
|---|---|---|
| Roles | 4 roles (Student, Organizer, Volunteer, Admin) | 5 roles: + Super Admin (pre-seeded). Hierarchy: Super Admin → Admin → Organizer → Volunteer; every panel opens via role check at the same login (FR001, FR017, FR019, FR020, UC013) |
| Role governance | Not defined | User search + promotion chain, demotion, tamper-proof Audit Logs (FR023), Role Integrity (NFR012) |
| Registration | Name/email/password | + University Roll Number (mandatory) → department auto-detected & locked (FR024); default role = Student |
| Dual pricing (FR006) | Student vs outsider (user type) | Department-based: host-department students = student rate; other-department students = outsider rate; Allow-Outsiders toggle (FR042) |
| Student verification | — | Removed separate profile verification step; roll number at registration is the verification |
| Event publishing | Organizer publishes directly | Admin Approval Workflow (FR021, UC014): Pending → Approved/Rejected with reason |
| Anti-clash (FR003) | Venue text match | Venue Database with venueId-based exact matching; Admin-managed venues (FR038) |
| QR tickets | Static UUID, offline scanning mode (FR012) | One-time unique encrypted QR (FR022); OFFLINE MODE REMOVED — online-only verification with Real-Time Multi-Gate Synchronization (FR012, NFR014); atomic transactions prevent double entry |
| Event lifecycle | Published/Cancelled only | Full lifecycle (FR028, UC016): Postponed (re-approval + refund opt-out), Cancelled (automatic FULL refunds), Completed (auto; triggers certificates, feedback, reports) |
| Ticket structure | Single price types | Optional Seat Tiers (FR039): VIP/Front/General with per-tier insider/outsider prices; no tiers → single General tier = equal pricing |
| Booking channels | Online only | Booking Mode Control (FR041, UC020): Online / Physical (walk-in issuance with cash records + guest QR) / Both — shared live inventory |
| Payments | Live gateways only | + Sandbox (Mock) Payment Mode (FR027) toggled by Super Admin; on-device PDF receipts |
| Notifications | FCM push | In-app notification center + device-scheduled local notifications (free plan); FCM push documented as optional future upgrade |
| Discovery | Search bar only | Smart sections Today/This Week/Coming Soon/Trending (FR031), interest tags + For You feed (FR032), rule-based recommendations (FR033), bookmarks (FR034), combined filters (FR035) |
| Event content | Title/description/1 image | Media gallery ≤6 images (FR029), Speaker/Guest profiles “Who's Coming?” (FR030) |
| Engagement | Social sharing only | + Event Q&A with moderation (FR037), Feedback & Ratings (FR026, UC015), Referral & Invite points (FR036, UC018) |
| Navigation | Map + directions | + “Leave Now” on-device ETA alert (FR040) |
| Monitoring | Admin dashboard | Live Attendance Counter (FR025); Super Admin analytics + audit trail + system settings |
| Quality | NFR001–011 | + NFR012 Role Integrity, NFR013 Crashlytics, NFR014 Online Verification Constraint, NFR015 Free-Tier Efficiency; Test Plan chapter added (Ch. 4) |
| Platform cost | Not specified | Entire system engineered for Firebase free (Spark) plan — no Cloud Functions (Appendix B) |

# Appendix B: Firebase Free (Spark) Plan Compliance

The complete system is designed to operate within Firebase Spark (free) limits without paid Cloud Functions:

| Resource | Free Limit | Compliance Strategy |
|---|---|---|
| Firestore reads | 50,000 / day | Offline persistence cache ON; pagination (10–20 items); denormalized fields & counters (bookingsCount for Trending); listeners only while needed (chat page, gate sync); projected queries with limits |
| Firestore writes | 20,000 / day | Batched writes (booking = ticket+booking+payment+counter in one batch); pull-model feeds (no per-follower notification writes); logs only for sensitive actions |
| Storage | 1 GiB | On-device image compression (banner ≤300KB, gallery ≤200KB, speaker/profile ≤100KB); gallery capped at 6 images; cleanup for drafts/cancelled events |
| Network egress | 10 GiB / month | Client image caching; compressed assets; paginated media |
| Cloud Functions | Not available on Spark | Replaced by: in-app notifications + local scheduled notifications (reminders, Leave Now), on-device PDF receipts/certificates, client-side auto-complete status, admin-triggered batch refunds |
| Authentication | Unlimited (email/password) | Email/password only; phone OTP avoided (paid SMS) |
| Maps / ETA | Google Maps free monthly quota | On-device ETA (haversine + average speed) by default; Directions API only on explicit user action with 15-minute caching; external maps app via deep link |

Estimated daily usage (200 active students, one 500-attendance event): ≈ 15,000 reads + ≈ 3,000 writes — within limits with ≈ 3× headroom. Usage is monitored monthly via the Firebase console.
