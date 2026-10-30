# AttendU

**A classroom attendance system designed to improve attendance integrity through rotating QR codes and multi-layer verification.**

## Overview

AttendU is a proposed real-time classroom attendance system designed to reduce proxy attendance and attempts to mark attendance remotely.

The system uses time-sensitive QR tokens, server-side validation, and additional verification signals to assess attendance attempts. Professors can manage lecture sessions and monitor attendance, while students can authenticate and submit attendance through a mobile-friendly web application.

The project prioritizes fraud resistance, usability, and implementation feasibility.

## Problem Statement

Traditional attendance processes can be vulnerable to:

- **Proxy attendance:** Someone marks attendance on behalf of another student.
- **Remote attendance:** A student attempts to mark attendance without being physically present in the classroom.

AttendU is designed to make these attempts more difficult through multiple validation layers and a controlled attendance workflow.

## Planned Features

### 1. Professor Session Management

- Professor authentication.
- Lecture session creation with course, topic, duration, scheduled start, and classroom location.
- Explicit session activation, pause, resume, and closure.
- Automatic session closure at the configured end time.

### 2. Dynamic QR Code Verification

- QR tokens rotate every 5 seconds.
- The immediately previous token can be accepted for a maximum 2-second grace period.
- Tokens are generated using a cryptographically secure random number generator.
- Token validation is performed on the backend.
- Tokens are associated with their lecture session.

### 3. Student Attendance

- Student authentication using a unique student ID and password.
- QR-based attendance check-in.
- Enrollment and session eligibility checks.
- Duplicate attendance prevention.

### 4. Multi-Layer Verification

AttendU is designed around mandatory backend checks and additional verification signals.

**Mandatory checks**
- Student authentication.
- Authorization and course enrollment.
- Session eligibility.
- QR token validity.
- Duplicate attendance prevention.

**Additional verification signals**

- **GPS:** An initial nominal geofence radius of 25 metres around the classroom coordinates. GPS accuracy is considered; insufficient results can be retried or sent for manual review.
- **Network verification:** Intended to use trusted campus-network infrastructure where available. A client-reported SSID or public IP alone is not considered proof.
- **Bluetooth proximity:** An interface is planned for BLE-based proximity verification, subject to hardware and device support.

Network and Bluetooth integrations may be simulated during development or demonstration. Simulated checks must be clearly identified and must not be represented as real proof of physical presence.

### 5. Attendance Outcomes

The system defines four attendance outcomes:

| Outcome | Meaning |
|---|---|
| Verified | Attendance passes the applicable verification policy. |
| Flagged | The attempt requires attention or further assessment. |
| Failed | The attempt does not pass required validation. |
| Inconclusive | Available verification information is insufficient for a definitive result. |

Manual review is a separate workflow, not an additional attendance status.

### 6. Manual Review

Authorized professors can review exceptional attendance cases.

- Approval or rejection requires a reason.
- Manual decisions are recorded in an audit trail.
- The audit record includes the professor, student, session, previous and resulting state, reason, and timestamp.
- Students can see the outcome of the review.

### 7. Real-Time Dashboard

The planned professor dashboard will provide:
- Active session visibility.
- Live attendance updates.
- Attendance outcome monitoring.
- Visibility into cases requiring review.
- Manual review functionality.

Socket.IO is the selected real-time communication technology.

## Technology Decisions

The following technology choices have been finalized in the project decision log.

| Area | Decision |
|---|---|
| Backend runtime | Node.js |
| Backend framework | Express |
| Backend language | TypeScript |
| Database | PostgreSQL |
| Application structure | Modular monolith |
| Standard API communication | REST |
| Real-time communication | Socket.IO |
| API documentation approach | OpenAPI |
| Frontend/backend organization | Separate codebases or directories |

### Planned Backend Modules

The backend is organized conceptually into these modules:

- Authentication
- Users and enrollment
- Sessions
- QR token engine
- Attendance
- Verification
- Manual review
- Audit and monitoring
- Real-time updates

The backend is intended to remain a single deployable application.

## Security and Reliability Principles

The design includes the following requirements:

- Server-side authentication and authorization.
- Secure password hashing.
- Secure session cookies.
- Backend-side token and eligibility validation.
- Role-based and resource-level access control.
- Database-level duplicate attendance prevention.
- Rate limiting for login and check-in attempts.
- Audit trails for manual decisions.
- Data minimization, including avoiding unnecessary persistent precise GPS data.
- Safe handling of retries, expired tokens, session closure, and real-time reconnection.

The backend and database are the authoritative sources for attendance state. A successful attendance result should only be reported after the relevant data has been persisted.

## Planned MVP

The initial minimum viable product is intended to demonstrate:

1. Professor authentication.
2. Lecture session creation and activation.
3. QR token generation and 5-second rotation.
4. Student authentication and QR-based check-in.
5. Backend validation of attendance attempts.
6. PostgreSQL persistence.
7. Live attendance updates on the professor dashboard.
8. Rejection of invalid, expired, unauthorized, and duplicate attempts.
9. Retry or manual review for inconclusive verification.
10. Professor approval or rejection with an audit trail.

## Current Design Decisions and Open Items

The project maintains a decision log in `decisions.md` to document finalized choices, rationale, alternatives, consequences, and unresolved questions.

Some implementation details are not finalized yet, including:

- Frontend framework.
- ORM or database query builder.
- Hosting provider.
- Exact REST API routes.
- Runtime validation library.
- Exact rules for combining additional verification signals.
- GPS accuracy threshold and related edge-case policy.
- Check-in window duration.
- Real campus-network and BLE integration feasibility.
- Testing framework and deployment-specific configuration.

These items should not be treated as finalized technology or implementation decisions.

## Project Status

AttendU is at the design and planning stage. The architecture and product decisions are documented, but this README does not claim that the planned features have already been implemented.

## Repository Structure

The final repository structure has not yet been decided. The project is intended to keep frontend and backend code organized separately, with the backend implemented as a modular monolith.

## License

License not yet specified.