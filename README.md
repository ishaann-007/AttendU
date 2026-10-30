# AttendU

**A classroom attendance system designed to reduce proxy attendance and remote check-ins using dynamic QR codes and multi-layer verification.**

## About

AttendU is a classroom attendance management system designed to improve the integrity of attendance records.

It uses rotating QR codes and server-side validation, along with additional verification signals such as GPS, network verification, and Bluetooth proximity where supported.

The system provides separate workflows for professors and students, with real-time attendance monitoring for professors.

## Problem Statement

Traditional attendance methods can be vulnerable to:

- **Proxy attendance:** A person marks attendance on behalf of another student.
- **Remote attendance:** A student attempts to mark attendance without being physically present in the classroom.

AttendU aims to make these attempts more difficult through time-sensitive tokens, multiple verification layers, and controlled attendance management.

## How It Works

1. A professor creates and activates a lecture session.
2. The system generates a QR code containing a time-sensitive token.
3. The QR token rotates every 5 seconds.
4. Students authenticate and scan the displayed QR code.
5. The backend validates the student's identity, enrollment, session eligibility, and QR token.
6. Additional verification signals are evaluated where available.
7. The attendance attempt receives an appropriate outcome.
8. Professors monitor attendance through a live dashboard and can review exceptional cases.

## Key Features

- Dynamic QR code generation and validation.
- Student and professor authentication.
- GPS-based location verification.
- Optional network and Bluetooth proximity verification.
- Duplicate attendance prevention.
- Real-time attendance monitoring.
- Manual review and attendance overrides with audit records.
- Secure, server-side validation.

Network and Bluetooth verification may require external infrastructure or simulated integrations during development. Simulated checks will be clearly identified.

## Attendance Outcomes

AttendU defines four attendance outcomes:

| Status | Description |
|---|---|
| Verified | The attempt passes the applicable verification policy. |
| Flagged | The attempt requires further attention. |
| Failed | The attempt does not pass required validation. |
| Inconclusive | Available information is insufficient for a definitive result. |

Manual review is a separate workflow and does not introduce another attendance status.

## Technology Stack

The following technology choices have been finalized:

| Component | Technology |
|---|---|
| Backend runtime | Node.js |
| Backend framework | Express |
| Backend language | TypeScript |
| Database | PostgreSQL |
| Real-time communication | Socket.IO |
| API communication | REST |
| API documentation | OpenAPI |
| Application architecture | Modular monolith |

The frontend framework, ORM/query builder, and hosting provider have not yet been finalized.

## Repository Structure

The project uses a single GitHub repository with separate frontend and backend folders.

```text
AttendU/
├── frontend/
├── backend/
├── docs/
├── README.md
└── .gitignore
```

## Project Status

**Current stage: Planning and initial development preparation.**

The product scope and major architectural decisions have been documented. Implementation is planned in stages.

Features described in this README represent the intended system and should not be interpreted as already implemented.

## Documentation

Project decisions, technical reasoning, alternatives, and unresolved questions are maintained in the project decision log.

## License

Not specified yet.