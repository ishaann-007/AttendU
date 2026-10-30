# Architecture Decision Log

This document records the architectural and technical decisions made during the design and implementation of the project.

Each decision should include its context, the chosen approach, the rationale behind it, alternatives considered, and the resulting trade-offs. Decisions are recorded only after the relevant questions have been discussed and resolved.

Unresolved questions and assumptions must remain explicit until they are validated.

## DEC-001: Problem Definition

**Status:** Accepted

### Context
Traditional classroom attendance systems can be manipulated through remote check-ins and proxy attendance, where someone marks attendance on behalf of another student. The project aims to improve attendance integrity without making legitimate check-ins unnecessarily difficult.

### Decision
The system will address both:
- **Remote attendance:** Attempting to mark attendance without being physically present in the designated classroom.
- **Proxy attendance:** Attempting to mark attendance on behalf of another student.

The system will balance fraud resistance, student usability, and implementation feasibility within the hackathon scope.

Verification failures will not automatically be treated as confirmed fraud. The system will support manual review of legitimate but unsuccessful or suspicious check-ins.

### Rationale
- Addressing both remote and proxy attendance provides a meaningful attendance-integrity objective.
- Balanced security and usability keep the solution practical for real classroom use and hackathon implementation.
- Manual review reduces the risk of unfairly rejecting legitimate students because of GPS inaccuracies, connectivity issues, or other technical failures.

### Alternatives Considered
- Preventing only remote attendance.
- Preventing only proxy attendance.
- Prioritizing maximum security regardless of user friction or implementation complexity.
- Automatically rejecting every failed verification without review.

These alternatives were not selected because they either leave a major attendance-fraud scenario unaddressed or create unnecessary usability and fairness risks.

### Consequences
- Subsequent architecture decisions must account for both remote and proxy attendance threats.
- Verification mechanisms must be evaluated for effectiveness, reliability, and feasibility.
- The system must distinguish successful verification, suspicious activity, and technical or inconclusive failures.
- A manual review mechanism must be included in the planned workflow.
- The system must not claim to eliminate attendance fraud; it aims to make common fraud attempts harder and more detectable.


## DEC-002: Product Scope

**Status:** Accepted

### Context
The project must demonstrate a credible, end-to-end classroom attendance workflow within the constraints of a hackathon. Implementing every possible anti-fraud mechanism or administrative feature would increase complexity and threaten delivery of a reliable working product.

### Decision
The project will prioritize a focused minimum viable product (MVP) with a complete attendance workflow.

The MVP will include:

1. Professor session creation and session lifecycle management.
2. Dynamic QR generation and server-side token validation.
3. Student identification and a responsive mobile web check-in interface.
4. Core attendance verification, including location checks where supported.
5. A live attendance dashboard with clear verification outcomes.
6. Duplicate-submission prevention and manual review of flagged or inconclusive check-ins.
7. Persistent attendance records and basic role-based access control.

Advanced analytics, institution-wide administration, dedicated native mobile applications, and production-grade Bluetooth beacon infrastructure are outside the initial MVP scope.

Network and proximity verification will be implemented only after feasibility is established. Simulated integrations may be used for demonstration purposes when real integration is impractical, but simulations must be explicitly identified and must not be represented as genuine security verification.

### Rationale
- A complete working workflow is more valuable than numerous partially implemented features.
- Core attendance functionality can be tested and demonstrated independently.
- Deferring advanced integrations reduces technical and deployment risk.
- Clearly separating real and simulated verification prevents misleading security claims.

### Alternatives Considered
- Implementing every proposed feature in the initial release.
- Building a demonstration-only prototype with most backend behavior mocked.
- Prioritizing advanced proximity mechanisms before completing the core attendance workflow.

These alternatives were rejected because they either introduce excessive implementation risk or fail to establish a reliable end-to-end system.

### Consequences
- Implementation must prioritize the core workflow and its acceptance tests.
- Real verification mechanisms must be distinguished from simulated checks.
- Optional features may be reconsidered after the core workflow is functional.
- The MVP must demonstrate session creation, student check-in, server-side verification, attendance persistence, dashboard updates, and manual review.


## DEC-003: User Roles

**Status:** Accepted

### Context
The MVP requires distinct interfaces and permissions for instructors managing attendance sessions and students submitting attendance. Introducing additional administrative roles would increase implementation complexity without being essential to the initial workflow.

### Decision
The MVP will support two user roles:

1. **Professor:** Can create and manage their lecture sessions, display session QR codes, monitor attendance, review flagged or inconclusive check-ins, and perform authorized attendance corrections.
2. **Student:** Can access eligible sessions, submit attendance, view their own attendance results, and retry or request review when a legitimate check-in fails.

An institution-wide administrator role will not be implemented in the initial MVP.

The backend must enforce role-based permissions. Professors may manage only sessions and attendance records they are authorized to access. Students may not access or modify other students' private attendance records.

### Rationale
- Two roles cover the complete MVP workflow.
- Separate permissions reduce the risk of unauthorized attendance modifications.
- Excluding institution-wide administration keeps the implementation focused.
- Backend-enforced permissions provide protection beyond frontend navigation or UI restrictions.

### Alternatives Considered
- Adding a third, institution-wide administrator role.
- Using one shared role with identical permissions for all users.
- Enforcing role restrictions only in the frontend.

These alternatives were not selected because they add unnecessary scope or fail to provide adequate access control.

### Consequences
- The application must distinguish professor and student identities.
- Backend routes and operations must enforce role-specific permissions.
- Lecture sessions must be associated with their authorized professor.
- Students must be restricted to their own attendance information.
- The authentication mechanism and trusted assignment of roles will be defined in subsequent decisions.


## DEC-004: Student Identity

**Status:** Accepted

### Context
The system must identify students before accepting attendance submissions. Roll-number-only login is insufficient because anyone who knows another student's roll number could impersonate that student.

### Decision
Each student will have an account identified by a unique student ID and authenticated using a password.

The backend will associate authenticated users with their student identities and enforce authorization before allowing attendance submissions or access to attendance records.

Passwords must never be stored in plaintext. They must be processed using an appropriate password-hashing algorithm through a trusted authentication library or framework.

Student IDs will serve as identifiers, not authentication secrets.

### Rationale
- Student ID plus password provides stronger account protection than roll-number-only login.
- The approach is practical to implement within the hackathon timeframe.
- It does not require an external OTP delivery service.
- It establishes a foundation for authorization and attendance-record ownership.

### Alternatives Considered
- Roll-number-only login, rejected because it does not authenticate the person using the identity.
- Student ID plus one-time verification, deferred because it introduces additional verification and delivery requirements.
- More advanced identity verification, deferred until the core attendance workflow is established.

### Consequences
- Student accounts and authentication flows must be implemented.
- Password handling must follow secure storage and verification practices.
- The backend must derive student identity from the authenticated session rather than trusting a submitted student ID.
- Shared credentials remain a possible proxy-attendance vector; this decision does not establish that the account owner is physically present.
- Stronger protection against credential sharing may be evaluated in later security decisions.


## DEC-005: Authentication Model

**Status:** Accepted

### Context
The application has separate professor and student interfaces and requires authenticated requests to protected backend operations. The authentication mechanism must be secure, practical to implement, and compatible with mobile browsers.

### Decision
The application will use server-managed sessions with secure cookies for authentication.

After successful login, the backend will establish an authenticated session and issue a session cookie. Subsequent requests will use that cookie, and the backend will validate the session and enforce the user's role and permissions.

The implementation must:
- Set `HttpOnly` on session cookies.
- Set `Secure` in HTTPS environments.
- Configure `SameSite` appropriately for the deployment architecture.
- Enforce session expiration and invalidate sessions on logout.
- Protect state-changing requests against cross-site request forgery (CSRF).
- Use HTTPS in deployed environments.
- Never trust frontend-supplied role or identity information without server-side verification.

### Rationale
- Server-managed sessions are well suited to a browser-based application.
- Secure cookies avoid exposing session credentials to ordinary frontend JavaScript.
- Established framework support reduces implementation effort and security risk.
- The approach works for both professor and student interfaces without requiring a separate token-management system.

### Alternatives Considered
- JWT-based authentication, which introduces additional decisions about token storage, expiration, renewal, and revocation.
- An external authentication provider, which introduces integration and configuration requirements.

These alternatives were not selected for the initial MVP because server-managed sessions provide a simpler fit for the expected architecture.

### Consequences
- The backend must provide login, logout, session validation, and session expiration behavior.
- Protected API operations must verify the authenticated session and enforce role-based authorization.
- CSRF protection and appropriate cookie configuration are required.
- Deployment must support HTTPS and a compatible browser-to-backend cookie configuration.
- The selected backend framework must support secure session management.


## DEC-006: Lecture Session Creation

**Status:** Accepted

### Context
Professors need to create lecture sessions that define when attendance is permitted, which course the session belongs to, and where the lecture takes place. Attendance eligibility must be based on course enrollment rather than unrestricted access.

### Decision
Professors will create lecture sessions by providing the following required fields:

1. **Course:** The course identifier or name associated with the lecture.
2. **Lecture topic:** A description of the lecture.
3. **Duration:** The planned session duration, expressed in minutes.
4. **Scheduled start time:** The intended date and time of the session.
5. **Classroom location:** A configured classroom location or validated coordinates suitable for location verification.

The backend will generate a unique session identifier and manage session-specific security tokens.

Only students enrolled in the associated course will be eligible to check in. Enrollment eligibility must be enforced by the backend.

Input validation must reject invalid durations, malformed or missing required fields, and invalid scheduling or location data.

### Rationale
- The required fields provide sufficient context for attendance sessions.
- Scheduled start time and duration support predictable session lifecycle management.
- Classroom coordinates provide a basis for evaluating GPS-based proximity.
- Course enrollment limits attendance submissions to the intended student population.
- Server-generated identifiers and tokens prevent clients from choosing trusted session security values.

### Alternatives Considered
- Allowing sessions without a scheduled start time or classroom location.
- Allowing any authenticated student to check in.
- Requiring professors to select every eligible student individually for every session.

These alternatives were not selected because they weaken session context, attendance eligibility, or implementation efficiency.

### Consequences
- Course and enrollment records must exist in the data model.
- Sessions must be associated with a course, professor, scheduled start time, duration, and classroom location.
- Location verification depends on valid configured coordinates and browser/device capabilities.
- The session lifecycle and check-in time window must be defined separately.
- The backend must validate session creation and enrollment eligibility.

## DEC-007: Session Lifecycle

**Status:** Accepted

### Context
Lecture sessions must have predictable states so that attendance is accepted only during authorized periods. Professors need explicit control over activation and temporary interruptions, while the backend must prevent submissions against stale or inactive sessions.

### Decision
Each lecture session will use the following lifecycle states:

- **Scheduled:** The session has been created but has not been activated. New attendance submissions are not permitted.
- **Active:** The professor has started the session. Eligible students may submit attendance.
- **Paused:** The professor has temporarily suspended the active session. New attendance submissions are not permitted until the session is resumed.
- **Closed:** The session has ended. New attendance submissions are permanently rejected.
- **Cancelled:** The session will not take place. New attendance submissions are permanently rejected.

Professors must explicitly start sessions. They may pause and resume active sessions or close them early. A session may be cancelled before activation.

The backend will automatically close active or paused sessions when their configured end time is reached.

The backend is the authoritative source of session state and timing. Every attendance submission must validate the current server-side session state and time window.

Closed and cancelled sessions cannot be reactivated. A new session must be created if another attendance period is required.

Existing accepted attendance records must be preserved when a session is paused, closed, or cancelled.

### Rationale
- Manual activation gives professors control over when attendance opens.
- Pausing supports temporary classroom interruptions without requiring a new session.
- Server-side state enforcement prevents stale clients from submitting attendance after a session becomes inactive.
- Automatic closure prevents sessions from remaining active indefinitely.
- Terminal closed and cancelled states simplify lifecycle enforcement and auditability.

### Alternatives Considered
- Activating every session automatically at its scheduled start time.
- Allowing attendance submissions while a session is paused.
- Allowing closed sessions to reopen without creating a new session.

These alternatives were not selected because they reduce instructor control or complicate the enforcement of attendance windows.

### Consequences
- The backend must enforce valid state transitions and authorized professor actions.
- Session state changes must be reflected in connected clients.
- Session expiration must be enforced independently of frontend timers.
- Paused sessions must reject new attendance submissions until resumed.
- Existing attendance records must remain available for review after session closure.
- Session time calculations must use a consistent server-side time source.

## DEC-008: Session Eligibility

**Status:** Accepted

### Context
Course enrollment establishes which students are eligible for a lecture, but the system also needs to control when those students may submit attendance. A configurable check-in window helps limit attendance submissions to the intended period without requiring professors to maintain a separate student allowlist for every lecture.

### Decision
A student may attempt attendance check-in only when all of the following conditions are satisfied:

1. The student is authenticated.
2. The student is enrolled in the course associated with the session.
3. The session is in the `Active` state.
4. The server-side time falls within the configured check-in window.
5. The session has not reached its configured end time.

The check-in window will be configurable for each session and measured from the actual time at which the professor starts the session, rather than its scheduled start time.

The check-in window must not extend beyond the session's configured end time.

Students arriving after the check-in window closes cannot submit ordinary self-service check-ins. They may request review by the professor, who can record legitimate attendance through an authorized manual-review workflow.

Rejected submissions must not create verified attendance records.

### Rationale
- Course enrollment prevents unrelated students from checking in.
- A configurable window provides control over attendance timing.
- Measuring from actual activation accommodates lectures that begin later than scheduled.
- Separating late-arrival review from ordinary check-in preserves flexibility without silently weakening the eligibility rules.
- Server-side time validation prevents clients from bypassing the configured window.

### Alternatives Considered
- Allowing all enrolled students to check in at any point while the session is active.
- Requiring a professor-managed allowlist for every lecture.
- Measuring the check-in window exclusively from the scheduled start time.
- Automatically accepting late attendance without review.

These alternatives were not selected because they either provide insufficient timing control, add unnecessary administration, or create inconsistent attendance rules.

### Consequences
- Session records must store the actual activation timestamp and configured check-in window.
- The backend must evaluate enrollment, session state, session end time, and check-in-window eligibility.
- The default check-in window duration must be established during implementation.
- A manual-review workflow must support legitimate late arrivals.
- Attendance eligibility must be enforced independently of the frontend.


### DEC-009: QR Payload

**Status:** Accepted

**Decision:** The QR code will contain a session identifier and an opaque, cryptographically random token. The token will not encode student identity or sensitive information. The backend will validate the token against the session and its current validity.

**Context:** The system needs a scannable QR code that identifies the lecture session and provides a temporary credential for attendance check-in. The payload should support frequent token rotation, server-side verification, and a feasible hackathon implementation.

**Rationale:**
- An opaque token keeps the payload simple and avoids exposing meaningful session-security data.
- The backend remains the authority for token validity, session status, expiry, and check-in eligibility.
- Unlike a self-contained signed token, an opaque token naturally supports server-side invalidation when a token rotates or a session changes state.
- A QR URL can simplify mobile scanning, but URLs can expose tokens through browser history, logs, or referrer data. The MVP will use the session ID and token as the logical payload; the exact scanning and URL transport format will be finalized during API design.
- Cryptographically secure randomness makes tokens difficult to guess, provided tokens are generated and validated correctly.

**Alternatives considered:**
1. **Session ID + signed token:** Tamper-resistant when signatures and expiry are correctly validated, but introduces signing-key management and does not automatically prevent replay.
2. **URL containing a token:** Convenient for mobile scanning, but a URL is only a transport format and does not independently provide replay protection. Token leakage must also be considered.
3. **Session ID + opaque, cryptographically random token — chosen:** Straightforward server-side validation, rotation, and invalidation with manageable implementation complexity.

**Security requirements:**
- Generate tokens using a cryptographically secure random-number generator.
- Apply a short validity period and invalidate superseded tokens.
- Validate the token, associated session, server-side expiry, session lifecycle, and check-in eligibility on every submission.
- Prevent duplicate attendance records and rate-limit repeated invalid attempts where appropriate.
- Never treat possession of a valid QR token alone as proof of physical presence.
- Recognize that photographing or sharing a currently valid QR may still enable replay; rotation reduces the attack window but does not eliminate this risk.

**Consequences:** The backend must maintain enough token state to validate expiry and invalidate previous tokens. Token generation, storage, rotation interval, and validation details will be specified in subsequent decisions. The scanning interface must transmit the payload securely over HTTPS in deployed environments.


### DEC-010: Token Generation

**Status:** Accepted

**Decision:** The backend will generate QR tokens using a cryptographically secure random-number generator provided by a trusted framework or cryptographic library. The backend will maintain server-side token state, associating each token with its session, validity period, and active or invalidated status. Where practical, the system will store a secure hash of each token rather than its raw value.

**Context:** The attendance system needs unpredictable tokens that cannot feasibly be guessed and can be validated, expired, and invalidated by the backend.

**Rationale:**
- Cryptographically secure randomness makes tokens difficult to predict or guess.
- Server-side state provides direct control over token validity and invalidation.
- The design supports frequent token rotation and immediate invalidation of superseded tokens.
- It is straightforward to implement and test within the hackathon MVP without introducing signing-key management.

**Alternatives considered:**
1. **Signed tokens:** Provide integrity protection but add signing-key management and do not independently prevent replay.
2. **Simple random, sequential, or timestamp-based values:** May be predictable and are unsuitable as security credentials.
3. **Cryptographically random, server-managed tokens — chosen:** Offers strong unpredictability and straightforward lifecycle control.

**Consequences:**
- The backend must maintain token validity state and validate tokens on every attendance submission.
- Token storage must be protected, and raw tokens must not be written to application logs.
- Token length, encoding, expiry, and rotation interval will be specified in subsequent decisions.
- Token possession alone does not prove physical presence; location and other verification checks remain separate.

### DEC-011: Token Rotation Interval

**Status:** Accepted

**Decision:** The backend will rotate the active QR token every **5 seconds**. The immediately previous token may be accepted for a maximum **2-second grace period** after supersession, subject to all other session and attendance validation rules. No older token may be accepted.

**Context:** Frequent token rotation reduces the period during which a copied QR code may be reused. Students must also have sufficient time to scan the code and submit a check-in despite ordinary network delays.

**Rationale:**
- A 5-second rotation interval limits the lifetime of the current QR token.
- A 2-second grace period accommodates some scanning and network delays around rotation boundaries.
- Server-controlled timing prevents client clocks from becoming the source of truth.
- The design balances replay resistance with practical classroom usability.

**Alternatives considered:**
1. **5-second rotation with no grace period:** Tighter rejection of superseded tokens, but potentially more legitimate check-in failures near rotation boundaries.
2. **7-second rotation with a 2-second grace period:** More time to scan, but a longer token exposure window.
3. **5-second rotation with a 1-second grace period:** Tighter grace window, but less tolerance for delays.
4. **5-second rotation with a 2-second grace period — chosen:** Frequent rotation with a small, explicitly bounded transition window.

**Implementation requirements:**
- The backend determines token issuance, rotation, supersession, and validity using server-side time.
- Only the current token and the immediately previous token within its 2-second grace period may be accepted.
- The grace deadline is fixed at 2 seconds after supersession and cannot be extended by repeated submissions.
- Paused, closed, cancelled, or otherwise ineligible sessions must reject check-ins regardless of token validity.
- Duplicate attendance submissions must be prevented independently of token validation.
- Client countdowns and QR refreshes must not independently determine token validity.

**Consequences:** The professor dashboard and backend must synchronize QR updates reliably. The 5-second interval may make scanning more time-sensitive, so the team must test scan success and submission latency under realistic conditions. Rotation and grace-period rules reduce replay opportunities but do not eliminate QR sharing or prove physical presence.

### DEC-012: Token Validation

**Status:** Accepted

**Decision:** The backend will validate each submitted QR token against server-side token records and server-controlled timestamps. A token will be accepted for further attendance verification only if it is recognized, associated with the correct session, and within its permitted validity window.

**Context:** The system must reject forged, unknown, expired, invalidated, and replayed tokens while allowing a narrowly bounded grace period for submissions crossing a rotation boundary.

**Validation requirements:**
1. Look up the submitted token using its secure server-side representation.
2. Verify that the token belongs to the expected session.
3. Check its issuance time, expiry, and active or superseded status using server-side time.
4. Accept the current token while valid.
5. Permit the immediately previous token only until 2 seconds after supersession.
6. Reject unknown tokens, tokens outside their permitted validity window, invalidated tokens, and tokens from other sessions.
7. Verify that the session is active and within its configured check-in window.
8. Verify student authentication and course enrollment before recording attendance.
9. Prevent duplicate attendance records, including when a student retries a submission.
10. Apply appropriate rate limits and log relevant validation outcomes without exposing raw tokens.

**Rationale:**
- Server-side lookup prevents arbitrary or fabricated tokens from being accepted based solely on timestamps.
- Timestamp validation enables precise enforcement of token expiry and the grace deadline.
- Session association prevents a token from one lecture being used for another.
- Centralized backend validation ensures consistent decisions across professor and student interfaces.

**Alternatives considered:**
1. **Strict validation without grace:** Simple, but may reject legitimate submissions crossing a rotation boundary.
2. **Timestamp-only validation:** Insufficient because a timestamp alone does not prove that a token was issued by the backend or remains authorized.
3. **Token lookup plus timestamp validation — chosen:** Combines server-side authorization state with explicit validity-window enforcement.

**Consequences:** The backend must maintain token validity records and use consistent server-side time. Token validation is only one stage of attendance verification; a valid token does not establish physical presence. The 2-second grace period improves tolerance for timing delays but slightly increases the opportunity to reuse a superseded token. Its effectiveness and usability must be tested under realistic conditions.

### DEC-013: GPS Verification

**Status:** Accepted

**Decision:** The system will use GPS-based geofencing with a nominal radius of **25 metres** around the configured classroom coordinates. The browser will request the student's location and provide coordinates and an accuracy estimate. If the reported accuracy is insufficient for a reliable decision, the student will be asked to retry. If the location remains inconclusive, the student may request manual review.

**Context:** GPS verification is intended to provide evidence that a student is near the classroom during attendance check-in. The system must balance location-based fraud prevention with the limitations of GPS in indoor environments.

**Rationale:**
- A 25-metre nominal radius provides a practical starting point for classroom-level location verification.
- Accuracy-aware handling reduces false rejections caused by unreliable indoor GPS readings.
- Retrying gives the device another opportunity to obtain a more reliable location.
- Manual review provides a fallback when automatic verification cannot reach a reliable conclusion.
- GPS verification is feasible in a mobile web application using browser location APIs and backend distance calculations.

**Implementation requirements:**
- Each classroom must have configured geographic coordinates.
- The browser must request location permission and obtain the student's coordinates and reported accuracy.
- The backend must calculate the distance between the submitted location and the configured classroom coordinates.
- The nominal geofence radius must be configurable, with 25 metres as the initial default.
- A configurable GPS accuracy threshold must be defined and tested.
- If location accuracy is insufficient, prompt the student to retry instead of automatically recording a failure or flagging suspected fraud.
- If location remains inconclusive after a bounded retry process, provide a manual-review path.
- A reliable reading outside the geofence may fail the GPS check, but this alone must not be treated as conclusive evidence of fraud.
- GPS verification must be combined with the other required attendance checks. A successful GPS check alone does not prove physical presence.
- Location collection must be limited to what is needed for attendance verification and handled according to the project's privacy requirements.

**Alternatives considered:**
1. **Strict 25-metre geofence without accuracy handling:** Simpler, but risks false rejections from inaccurate indoor readings.
2. **Wider 50–100-metre geofence:** More tolerant of location errors, but may permit check-ins from areas farther from the classroom.
3. **25-metre geofence with accuracy handling and retry — chosen:** Balances verification, usability, and implementation feasibility.

**Limitations:** GPS coordinates can be inaccurate or spoofed. Browser-reported accuracy is an estimate rather than a guarantee. The geofence radius and accuracy threshold must be validated through testing on real devices at the intended venue.

**Consequences:** The backend must implement distance and accuracy checks, and the student interface must handle location permissions, retries, and inconclusive results. The system must distinguish technical uncertainty from suspected abuse and provide a path for legitimate students whose locations cannot be reliably verified.

### DEC-014: Network Verification

**Status:** Accepted

**Decision:** The system will define a network-verification interface intended to support trusted campus or classroom network verification. If suitable infrastructure is available, a real verification mechanism may be implemented. Otherwise, the hackathon demo will use an explicitly labelled simulated result. Simulated verification must never be represented as actual network-based security.

**Context:** Network verification could provide an additional signal that a student is near the classroom. However, the implementation depends on available campus infrastructure and the limitations of browser-based applications.

**Rationale:**
- Designing a verification interface keeps the architecture extensible without making the MVP dependent on unavailable infrastructure.
- A simulated fallback allows the team to demonstrate the end-to-end workflow reliably.
- Explicitly distinguishing simulation from real verification avoids misleading users about the system's security.
- The design leaves room to integrate trusted institutional network infrastructure in a future version.

**Alternatives considered:**
1. **Simulation only:** Simple and predictable for the demo, but does not establish a path to real network verification.
2. **Real network verification only:** Potentially stronger when supported by trusted infrastructure, but may block the MVP if the necessary access or configuration is unavailable.
3. **Real-verification interface with a labelled simulated fallback — chosen:** Balances feasibility, transparency, and future extensibility.

**Implementation requirements:**
- Define a backend interface for reporting network-verification outcomes.
- Use a real verification mechanism only when the network infrastructure and its trust assumptions are established.
- Do not treat a client-reported network name or an unverified claim of Wi-Fi connectivity as proof of presence.
- Do not treat a public IP address alone as definitive proof that a student is in the classroom.
- Clearly label simulated results in the demo interface and test data.
- Keep real and simulated outcomes distinguishable in logs and attendance-review records.
- A simulated success must not be presented as a real security check or used to claim that physical presence has been verified.
- Network verification must remain one signal in the overall attendance-verification process.

**Consequences:** The MVP can proceed without campus network access, while preserving an integration point for future infrastructure. The team must document which checks are real and which are simulated. The availability and design of real network verification remain dependent on the deployment environment.


### DEC-015: Proximity Verification

**Status:** Accepted

**Decision:** The system will define a proximity-verification interface that can support Bluetooth Low Energy (BLE) beacon detection. A real BLE integration may be implemented if suitable hardware, device support, and sufficient development time are available. Otherwise, the hackathon demo will use an explicitly labelled simulated proximity result.

**Context:** Bluetooth proximity could provide an additional signal that a student is near the classroom. However, implementation feasibility depends on beacon availability, device compatibility, browser permissions, and the capabilities of the chosen application platform.

**Rationale:**
- A defined interface preserves a path to real proximity verification without making the MVP dependent on BLE hardware.
- A simulated fallback allows the attendance workflow to be demonstrated consistently.
- Optional integration lets the team evaluate real BLE feasibility before committing implementation time.
- Explicitly distinguishing simulated and real results prevents the demo from overstating its security.

**Alternatives considered:**
1. **Real BLE beacon only:** Potentially useful for proximity checks, but introduces hardware and compatibility risks.
2. **Simulation only:** Easy to implement, but provides no real proximity verification and limits future integration planning.
3. **Optional real BLE integration with a labelled simulated fallback — chosen:** Balances feasibility, transparency, and extensibility.

**Implementation requirements:**
- Define a backend interface for receiving and recording proximity-verification outcomes.
- Investigate BLE support on the target devices and application platform before attempting real integration.
- If real BLE is implemented, use a configured classroom beacon and document how beacon identifiers and observed signals are validated.
- Do not treat a client-reported beacon identifier or simulated success as definitive proof of physical presence.
- Clearly label simulated results in the interface, logs, and demo documentation.
- Keep real and simulated outcomes distinguishable for auditing and manual review.
- Do not make successful BLE verification a mandatory prerequisite for the MVP unless the required hardware and device support are confirmed.
- Treat BLE as a supplementary signal alongside token validation, GPS, and any available real network verification.

**Consequences:** The MVP can proceed without dedicated beacon hardware. Real BLE integration remains conditional on feasibility testing. The team must document which proximity checks are implemented, simulated, or unavailable and must not claim that simulated results establish classroom presence.

### DEC-016: Overall Attendance Validation Policy

**Status:** Accepted

**Decision:** The system will use a risk-based validation policy. Mandatory identity, authorization, session eligibility, token validity, and duplicate-prevention checks must pass before attendance can be confirmed. Additional signals, including GPS, real network verification, and real BLE proximity verification when available, will contribute to the assessment. Uncertain or unavailable checks will be handled according to explicit retry, inconclusive, or manual-review rules.

**Context:** The system combines several verification mechanisms with different reliability levels and implementation constraints. A single rigid policy could reject legitimate students because of technical problems, while an overly permissive policy could allow unauthorized attendance.

**Rationale:**
- Mandatory security checks prevent invalid tokens, unauthorized users, ineligible sessions, and duplicate attendance records from being accepted.
- Risk-based handling distinguishes suspicious activity from technical failures and inconclusive evidence.
- Multiple verification signals provide more context than relying on GPS or QR validation alone.
- Manual review provides a recovery path for legitimate students whose check-ins cannot be verified automatically.
- Explicit treatment of simulated checks prevents the demo from overstating actual security.

**Mandatory checks:**
1. The student must be authenticated and authorized for the requested session.
2. The student must be enrolled in the relevant course.
3. The session must be active and within its permitted check-in window.
4. The submitted QR token must pass DEC-012 validation.
5. Duplicate attendance submissions must be prevented.
6. All confirmed attendance records must be created or updated by the backend.

**Additional verification signals:**
- **GPS:** Evaluate distance and reported accuracy using DEC-013. Uncertain readings may trigger retries or manual review.
- **Network:** Use real verification only when supported by trusted infrastructure. Simulated results must be explicitly labelled.
- **BLE proximity:** Use real verification only when a supported integration is available. Simulated results must remain distinguishable from real checks.

**Outcome handling:**
- **Verified:** All mandatory checks pass and the configured verification policy provides sufficient evidence to confirm attendance.
- **Flagged:** A check produces a suspicious result requiring further examination. Flagging alone does not establish fraud.
- **Failed:** A mandatory requirement fails, such as an invalid token, unauthorized student, ineligible session, or duplicate submission. The backend must not create a new confirmed attendance record.
- **Inconclusive:** Available evidence is insufficient because of technical failure, poor location accuracy, or unavailable verification infrastructure. The student may retry or request manual review.

**Implementation requirements:**
- Enforce mandatory checks on the backend; never trust frontend status or client-reported verification outcomes without validation.
- Define explicit rules for how each available verification signal affects the final outcome.
- Do not allow a simulated network or BLE success to count as genuine physical-presence evidence.
- Do not classify a GPS failure or inconclusive result as confirmed fraud by itself.
- Record verification outcomes and relevant reasons for authorized review.
- Apply authorization rules to manual review and attendance corrections.
- Keep the verification policy configurable so that thresholds and signal requirements can be adjusted after testing.

**Alternatives considered:**
1. **All checks must pass:** Simple, but can create excessive false rejections when optional signals are unavailable or unreliable.
2. **Only QR and identity are mandatory, with other checks treated as supporting evidence:** Usable, but may leave the treatment of suspicious combinations underspecified.
3. **Risk-based validation — chosen:** Enforces essential requirements while distinguishing suspicious, failed, and inconclusive outcomes.

**Consequences:** The backend must implement a clear validation pipeline and outcome classification. The team must document the policy for available real and simulated checks and test both successful and failure scenarios. Risk-based validation improves flexibility but does not guarantee the elimination of proxy or remote attendance.

## DEC-017 — Attendance States

**Status:** Accepted

### Decision
Each attendance attempt will have one of four outcome statuses: `Verified`, `Flagged`, `Failed`, or `Inconclusive`. Manual review will be tracked separately from the attendance outcome.

### Status Definitions
- **Verified:** All mandatory checks pass and the configured validation policy provides sufficient evidence to accept attendance.
- **Flagged:** The attempt raises a suspicious condition that requires examination. A flag is not proof of fraud.
- **Failed:** A mandatory requirement fails, such as an invalid token, an unauthorized student, an ineligible session, or a duplicate check-in. No new confirmed attendance record is created.
- **Inconclusive:** The system cannot reach a reliable decision because evidence is insufficient or a technical issue prevents verification. The student may retry or request manual review where appropriate.

### Manual Review
Manual review is a separate workflow, not a fifth attendance status. It will track whether a case is pending review, approved, or rejected, along with the reviewing professor and decision details where appropriate.

A professor's approval or rejection will update the attendance record through an authorized backend operation. The system must preserve an audit trail of manual changes.

### Rationale
Four distinct statuses make it easier to distinguish accepted attendance, suspicious attempts, definite validation failures, and technical uncertainty. Separating manual review from the outcome avoids mixing system validation results with human decisions.

### Consequences
- The student interface must display clear outcomes and actionable next steps.
- The professor dashboard must support filtering and reviewing flagged and inconclusive attempts.
- Backend logic must enforce valid status transitions and role-based permissions.
- Status changes and manual overrides must be recorded for auditing.
- A technical failure or GPS uncertainty must not automatically be treated as evidence of fraud.

### Alternatives Considered
- **Three statuses:** Simpler, but does not clearly distinguish technical uncertainty from definite failure.
- **Detailed status model:** Separate statuses for pending, manually approved, and manually rejected would mix attendance outcomes with review workflow states.

### Open Questions
- Which flagged conditions require mandatory professor review?
- How long should inconclusive attempts remain eligible for retry or review?
- What audit-log details and retention period are appropriate?

## DEC-018 — Duplicate Check-in Handling

**Status:** Accepted

### Decision
A student may have at most one attendance record per lecture session. The first valid check-in wins. Subsequent check-in attempts for the same student and session will not create another attendance record or overwrite the existing one.

### Rules
- The backend must enforce uniqueness for the combination of student ID and lecture session ID.
- Duplicate prevention must be enforced at the database level where possible, not only in frontend logic.
- If a student submits another check-in after successful verification, the system returns a clear duplicate response and preserves the original attendance record.
- Concurrent submissions must not create multiple attendance records.
- Rejected duplicate attempts may be logged for debugging and abuse monitoring without unnecessarily storing sensitive information.
- If a student believes their attendance record is incorrect, they must use the manual-review process rather than submitting repeated check-ins.
- Authorized professors may correct attendance through an explicit, audited manual-review action. This does not permit ordinary check-in attempts to overwrite records.

### Rationale
First-valid-check-in handling prevents duplicate attendance, avoids conflicting records, and provides predictable behavior during repeated scans, network retries, and concurrent requests.

### Consequences
- The database must support a unique constraint on student ID and session ID for attendance records.
- The backend must handle duplicate-key or concurrent-request outcomes safely.
- The user interface must distinguish a duplicate submission from a new successful check-in.
- Manual corrections must be authorized and auditable.

### Alternatives Considered
- **Latest valid check-in wins:** Rejected because later attempts could unexpectedly overwrite an earlier result.
- **Store all attempts as attendance records:** Rejected because it complicates the attendance model. Attempt logs may still be stored separately for auditing and abuse monitoring.

### Open Questions
- Should a student be allowed to retry automatically after an inconclusive attempt? This must not create duplicate attendance records.
- What details should be retained for duplicate-attempt logs, and for how long?

## DEC-019 — Exceptions and Manual Overrides

**Status:** Accepted

### Decision
Authorized professors may manually approve or reject attendance cases when automated verification fails, remains inconclusive, or produces a flagged result. Every manual decision must include a reason and be recorded in an audit trail.

### Rules
- Only an authenticated professor authorized to manage the relevant lecture session may perform a manual override.
- The professor must select the student, lecture session, decision, and a reason for the action.
- Manual decisions may approve attendance or reject the attendance request.
- A manual approval can create or correct an attendance record when justified, including when the student was physically present but automated checks failed.
- A manual rejection must record the decision without silently deleting previously accepted attendance.
- The system must record the professor's identity, affected student and session, previous and resulting state where applicable, reason, and server-generated timestamp.
- Audit records must be protected from unauthorized modification or deletion through the normal application interface.
- Students should be able to see the resulting attendance outcome and, where appropriate, the reason or next steps without exposing internal security details.
- Manual overrides must not bypass authentication, professor authorization, or audit logging.

### Rationale
Automated checks can fail because of GPS inaccuracies, unsupported devices, connectivity issues, or unavailable verification infrastructure. Controlled overrides provide a fair exception path while discouraging arbitrary attendance changes.

### Consequences
- The backend must enforce professor authorization and validate every override.
- The database must support manual-review state and audit records.
- The professor dashboard must provide a review interface for flagged and inconclusive cases.
- Attendance records must distinguish automated verification from manual approval.
- Manual review outcomes must remain separate from the four attendance statuses defined in DEC-017.

### Alternatives Considered
- **Unrestricted override:** Rejected because changes without justification or auditability create opportunities for misuse.
- **Review requests only:** Rejected because genuine attendance problems sometimes require an actual correction to the attendance record.

### Open Questions
- Should a student be allowed to appeal a professor's manual rejection?
- Should manual overrides require a second approver in a future institutional version?
- What retention period should apply to audit records?

## DEC-020 — Application Structure

**Status:** Accepted

### Decision
The application will use a modular monolith architecture. The frontend and backend may be maintained as separate codebases or directories, but the backend will run as one deployable application with clearly separated functional modules.

### Proposed Modules
- **Authentication and Authorization:** Login, logout, session management, role checks, and access control.
- **Users and Enrollment:** Student and professor profiles, student identity, and course enrollment.
- **Lecture Sessions:** Session creation, scheduling, lifecycle, and classroom configuration.
- **QR Token Engine:** Secure token generation, rotation, expiration, and validation.
- **Attendance:** Check-in processing, attendance statuses, duplicate prevention, and attendance records.
- **Verification:** GPS verification and interfaces for network and BLE proximity checks.
- **Manual Review:** Professor review actions, controlled overrides, and review outcomes.
- **Audit and Monitoring:** Security-relevant events, administrative changes, and diagnostic logs.
- **Real-Time Updates:** Live attendance counts, session status changes, and dashboard updates.

These are logical module boundaries, not a requirement to create a separate deployment or service for every module.

### Rules
- Each module should have a clear responsibility and a documented interface.
- Business rules and security-sensitive validation must be enforced by the backend.
- Modules should avoid unnecessary direct dependencies on one another's internal implementation.
- Database access and transaction boundaries must be designed to preserve attendance integrity.
- Frontend components must not be treated as security boundaries.
- The architecture should allow modules to be refactored or extracted into separate services later if a genuine need arises.

### Rationale
A modular monolith reduces deployment and integration complexity while preserving code organization. It is appropriate for a hackathon MVP and supports future expansion without requiring a distributed system from the beginning.

### Consequences
- The team can develop and test modules independently within one application.
- Deployment and local development remain comparatively simple.
- Module boundaries and shared interfaces must be maintained through code review and testing.
- Real-time communication and verification integrations must fit into the chosen application architecture.

### Alternatives Considered
- **Multiple independently deployed services:** Rejected for the MVP because service-to-service communication, deployment, and debugging add unnecessary complexity.
- **One unstructured application:** Rejected because tightly coupled code would make security-sensitive logic and future changes harder to maintain.

### Open Questions
- Which frontend and backend frameworks will be used?
- Which database will store users, sessions, tokens, attendance, and audit records?
- Which mechanism will deliver real-time dashboard updates?

## DEC-021 — Backend Framework

**Status:** Accepted

### Decision
The backend will use Node.js, Express, and TypeScript.

- **Node.js:** JavaScript runtime for executing backend code.
- **Express:** HTTP framework for defining API routes and middleware.
- **TypeScript:** Typed language layer for improving maintainability and catching errors during development.

### Rules
- Backend modules must follow the modular monolith structure defined in DEC-020.
- API request bodies, query parameters, and path parameters must be validated at runtime using a schema-validation library. TypeScript types alone do not validate incoming data.
- Authentication, authorization, session eligibility, token validation, and duplicate prevention must be enforced server-side.
- Security-sensitive operations must use established, maintained libraries rather than custom cryptographic implementations.
- Errors must be handled consistently, without exposing stack traces, secrets, raw tokens, or sensitive internal details to clients.
- Configuration and secrets must be supplied through environment variables or the chosen deployment platform's secret-management mechanism. Secrets must not be committed to version control.
- The backend must use automated tests for critical attendance and security rules.

### Rationale
Node.js and Express provide a lightweight, flexible foundation suitable for a hackathon MVP. TypeScript improves clarity around API contracts, session states, attendance outcomes, and other security-sensitive data structures. Using TypeScript on both frontend and backend is possible if the frontend stack supports it.

### Consequences
- The project will use a Node.js-compatible package manager and TypeScript build configuration.
- Dependencies must be selected and maintained deliberately.
- Runtime validation, authentication, database integration, logging, and testing require explicit setup.
- Express does not provide a complete authentication or security architecture automatically; these concerns must be implemented and tested separately.

### Alternatives Considered
- **Python + FastAPI:** A strong alternative with type-annotated APIs and automatic API documentation.
- **Java + Spring Boot:** A mature framework with extensive enterprise features, but potentially more setup than needed for this MVP.

### Open Questions
- Which database and data-access library should be used?
- Which frontend framework will consume the API?
- Which real-time communication mechanism will the professor dashboard use?

## DEC-022 — Database

**Status:** Accepted

### Decision
PostgreSQL will be the primary relational database for the application. A TypeScript-compatible ORM or query builder will be selected during implementation.

### Data to Be Stored
- **Users:** Student and professor identities, authentication-related fields, and roles.
- **Courses and Enrollment:** Course information and the relationship between students and courses.
- **Lecture Sessions:** Session details, lifecycle state, scheduled times, actual start and end times, and classroom configuration.
- **QR Tokens:** Session-associated token representations, validity periods, rotation state, and invalidation information.
- **Attendance:** One authoritative attendance record per student per lecture session, including outcome status and verification metadata.
- **Manual Reviews:** Review decisions, reasons, reviewer identity, and timestamps.
- **Audit Logs:** Security-relevant events and authorized changes to attendance or session records.

### Rules
- Use primary keys, foreign keys, appropriate constraints, and indexes to preserve data integrity.
- Enforce at most one attendance record per student per lecture session with a database-level unique constraint.
- Use transactions for operations that must succeed or fail together, particularly attendance creation and manual corrections.
- Token rotation and validation must use authoritative server-side state and time.
- Passwords must never be stored in plaintext. Store only secure password hashes produced by an established password-hashing library.
- Where practical, store a secure hash of QR tokens rather than the raw token. Token comparison and lookup must be designed to support the rotation and grace-period rules in DEC-011 and DEC-012.
- Apply least-privilege database access and keep credentials outside version control.
- Avoid storing precise student location longer than necessary for verification and review.
- Database migrations must be version-controlled and repeatable across development and deployment environments.

### Rationale
PostgreSQL provides relational integrity, transactions, unique constraints, and indexing needed for consistent attendance records. These capabilities reduce the risk of duplicate records and conflicting updates during concurrent check-ins.

### Consequences
- The project must configure a PostgreSQL database for development and deployment.
- The selected data-access library must support migrations, transactions, and the required database constraints.
- Database schema changes must be coordinated with backend validation and API contracts.
- Backups and access controls must be considered before production use.

### Alternatives Considered
- **MongoDB:** Rejected because the application's structured relationships and integrity requirements make a relational database a natural fit.
- **SQLite:** Rejected as the primary target because PostgreSQL is a better fit for a shared multi-user deployment, although SQLite remains suitable for isolated prototypes and tests.

### Open Questions
- Which ORM or query builder should be selected?
- Which PostgreSQL hosting option will be used?
- What backup and retention policies are appropriate for the MVP?

## DEC-023 — Real-Time Communication

**Status:** Accepted

### Decision
The application will use Socket.IO for real-time communication between the backend and connected frontend clients. REST APIs will handle standard operations such as authentication, session management, check-in requests, and attendance retrieval.

### Real-Time Events
The system may publish the following events:
- `session:updated` — lecture session state or relevant session details changed.
- `attendance:updated` — an attendance record was created or its status changed.
- `attendance:summary` — updated aggregate attendance counts for an authorized professor dashboard.
- `review:updated` — a manual-review decision changed.
- `connection:status` — client connection status for user-interface feedback, if needed.

Event names are provisional and may be refined during API design.

### Rules
- The backend remains the authoritative source of session state, attendance records, and validation outcomes.
- Socket.IO events notify clients of changes; they do not replace backend validation or database persistence.
- Clients must authenticate and be authorized before joining session-specific rooms or receiving private attendance information.
- Students may receive only information they are authorized to view. Professor dashboards may receive session-level attendance updates only for sessions they are authorized to manage.
- Attendance changes must be committed successfully before the corresponding confirmed update is broadcast.
- Clients must handle disconnections and reconnections gracefully.
- After reconnecting, clients should retrieve the current authoritative state through an authenticated API rather than assume every event was received.
- The backend must not trust client-supplied events as proof of attendance, token validity, or authorization.
- Real-time events must not expose passwords, raw QR tokens, session secrets, or unnecessary precise location data.

### Rationale
Socket.IO supports event-based updates, room-based communication, and reconnection behavior. This suits a professor dashboard that needs to reflect check-ins and review decisions promptly.

### Consequences
- The backend must configure Socket.IO alongside Express.
- Authentication and authorization rules must cover both HTTP endpoints and socket connections.
- The frontend must handle event subscriptions, reconnection, and state refreshes.
- Event broadcasting must be coordinated with successful database operations to avoid displaying uncommitted attendance changes.

### Alternatives Considered
- **Server-Sent Events:** Rejected because Socket.IO provides a convenient event-based model and room support for the MVP.
- **REST polling:** Rejected because frequent polling introduces unnecessary requests and delays updates between polls.

### Open Questions
- Which events and payload schemas should be part of the API contract?
- How should socket authentication and session-room membership be implemented?
- What fallback behavior is appropriate if real-time updates are temporarily unavailable?

## DEC-024 — API and Data Contracts

**Status:** Accepted

### Decision
The application will use REST APIs for standard client-server operations, OpenAPI for HTTP API documentation, and explicitly defined payload schemas for Socket.IO events.

### Rules
- HTTP endpoints must use consistent naming, request formats, response formats, and error structures.
- Request data must be validated at runtime using a schema-validation library. TypeScript types alone are insufficient.
- OpenAPI documentation must describe endpoint paths, HTTP methods, authentication requirements, request schemas, response schemas, and relevant error responses.
- Socket.IO event names, payload schemas, and authorization requirements must be documented alongside the HTTP API.
- Authentication and authorization must be enforced by the backend for every protected operation.
- Attendance outcomes must use the statuses defined in DEC-017: `Verified`, `Flagged`, `Failed`, and `Inconclusive`.
- Duplicate check-ins must follow DEC-018. Manual overrides must follow DEC-019.
- API responses and events must not expose passwords, raw QR tokens, session secrets, or unnecessary sensitive location information.
- Validation errors must be actionable without revealing internal implementation details.
- API and event contracts must be updated when behavior changes, and critical contracts should be covered by automated tests.

### Initial API Areas
The API is expected to cover:
- Authentication and current-user information.
- Course enrollment and student eligibility.
- Lecture session creation, retrieval, and lifecycle management.
- QR token retrieval and check-in validation.
- Student attendance history and professor attendance dashboards.
- Manual-review requests and authorized professor decisions.
- Relevant audit and operational information restricted to authorized users.

Exact routes and payloads will be finalized during implementation.

### Rationale
Documented contracts reduce ambiguity between frontend and backend development. Runtime validation and explicit authorization requirements are particularly important for QR token validation, attendance integrity, and professor-only operations.

### Consequences
- The project must maintain an OpenAPI specification or generate one from validated route definitions.
- The team must agree on consistent response and error formats.
- Socket.IO events require their own documented contracts.
- Contract changes must be coordinated across frontend, backend, and tests.

### Alternatives Considered
- **Informal documentation:** Rejected because API behavior can drift and security requirements can be overlooked.
- **Shared TypeScript contracts alone:** Rejected as the sole contract because compile-time types do not validate external requests and do not replace API documentation.

### Open Questions
- Which schema-validation library should be used?
- Should the frontend and backend live in a TypeScript monorepo?
- Which standard error format and API versioning approach should be adopted?

## DEC-025 — Deployment Strategy

**Status:** Accepted

### Decision
The MVP will use separate hosted deployments for the frontend and backend, with a managed PostgreSQL database.

The specific hosting providers will be selected during implementation based on availability, free-tier limits, compatibility, and deployment effort.

### Deployment Components
- **Frontend:** Hosts the student portal and professor dashboard.
- **Backend:** Runs the Node.js, Express, and TypeScript API, including Socket.IO.
- **Database:** Managed PostgreSQL for persistent application data.
- **Configuration and Secrets:** Deployment environment variables or the hosting platform's secret-management facility.

### Rules
- The backend must be publicly reachable over HTTPS for the deployed frontend to access it.
- The backend hosting platform must support persistent connections required by Socket.IO, including WebSocket upgrades and any required polling fallback.
- Configure CORS to allow only the intended frontend origins in production.
- Configure environment-specific settings separately for development and production.
- Database credentials, session secrets, and other sensitive configuration must never be committed to the repository or exposed in frontend bundles.
- Production secrets and cookie settings must follow the authentication requirements in DEC-005.
- Database migrations must be applied in a controlled, repeatable manner.
- The application must provide safe error handling and sufficient operational logs for diagnosing deployment issues.
- The team must verify that database connection limits, hosting sleep behavior, deployment restarts, and free-tier restrictions do not undermine the live demonstration.

### Rationale
Separate deployments allow the frontend and backend to be developed and released independently. A managed database reduces operational overhead, while public hosting allows judges and teammates to access the application remotely.

### Consequences
- Frontend and backend URLs must be configured for each environment.
- Cross-origin requests, authentication cookies, and Socket.IO connections require coordinated configuration.
- The backend and database must remain available during the demonstration.
- Hosting-provider selection must account for real-time communication and database compatibility, not just ease of initial deployment.

### Alternatives Considered
- **One combined deployment:** Rejected in favor of independent frontend and backend hosting.
- **Local-only demonstration:** Rejected because remote judges may need to access the application.

### Open Questions
- Which frontend and backend hosting providers will be used?
- Which managed PostgreSQL provider will be used?
- What monitoring, backup, and recovery provisions are necessary for the demo?

## DEC-026 — Authorization Model

**Status:** Accepted

### Decision
The application will use role-based access control (RBAC) combined with resource-level authorization. The backend must verify both the authenticated user's role and their relationship to the requested course, lecture session, student record, or other protected resource.

### Role Permissions

**Student**
- View and update only permitted fields of their own profile.
- View their own attendance records and check-in results.
- Check in only to eligible sessions for courses in which they are enrolled.
- Submit requests for manual review of their own attendance.
- Must not create or manage lecture sessions, access another student's private records, or perform professor-only actions.

**Professor**
- Create and manage lecture sessions for courses they are authorized to teach.
- View attendance information for their authorized sessions.
- Review flagged or inconclusive attendance attempts for those sessions.
- Approve or reject attendance through the controlled manual-review workflow in DEC-019.
- Must not manage unrelated courses or sessions without explicit authorization.

### Rules
- The backend must derive the user's identity and role from the authenticated server-side session, not from client-supplied role or user ID fields.
- Every protected endpoint must enforce authentication, role authorization, and resource-level permission checks as applicable.
- Socket.IO connections and room membership must enforce equivalent authorization rules.
- Student enrollment and professor-to-course authorization must be verified server-side.
- Unauthorized requests must be rejected without disclosing private information about the requested resource.
- Database queries and mutations must be scoped to authorized resources where practical.
- Manual overrides must follow DEC-019 and create the required audit records.
- Tests must cover attempts to access another student's records and another professor's sessions.

### Rationale
Role checks alone are insufficient because users with the same role may have access to different courses, sessions, and student records. Combining RBAC with resource-level checks reduces the risk of unauthorized access and cross-user data exposure.

### Consequences
- The data model must represent course enrollment and professor-course authorization.
- Authorization logic must be consistently applied to HTTP endpoints and real-time events.
- Shared authorization helpers or middleware should be used to reduce inconsistent checks.
- Permission changes must be reflected by the backend rather than relying on stale frontend state.

### Alternatives Considered
- **Role checks only:** Rejected because a role does not establish permission to access every resource.
- **Fine-grained policy engine:** Rejected for the MVP because it adds complexity beyond the current two-role model.

### Open Questions
- How will professor-to-course assignments be created and maintained?
- Which profile fields may students update themselves?
- Should institutional administrators and teaching assistants be supported in a future version?

## DEC-027 — Data Privacy and Retention

**Status:** Accepted

### Decision
The application will follow privacy-by-minimization principles. It will collect and retain only the personal and verification data necessary for attendance management, security, troubleshooting, and authorized manual review.

### Rules
- Collect only the information needed for student identity, course enrollment, session management, attendance verification, and auditability.
- Store password hashes rather than plaintext passwords.
- Avoid retaining precise GPS coordinates after verification unless temporary retention is necessary for a pending review or documented debugging need.
- Where location evidence must be retained, restrict access, document its purpose, and delete it according to a defined retention policy.
- Store the verification outcome and relevant metadata, such as whether GPS verification passed, failed, or was inconclusive, rather than retaining full location histories by default.
- Never store raw QR tokens in application logs. Where practical, store secure token hashes in the database.
- Do not collect unnecessary device identifiers, contact information, or continuous location data.
- Restrict attendance records and verification evidence according to DEC-026.
- Use HTTPS in deployment and secure session cookies as defined in DEC-005.
- Keep secrets and database credentials out of source control and client-side code.
- Establish retention and deletion rules for attendance records, audit logs, token records, and diagnostic logs before production use.
- Provide clear information about what data is collected and why. Any institutional or legal retention requirements must be evaluated before deployment.

### Rationale
Attendance systems handle personal and behavioral information. Minimizing collection and retention reduces privacy exposure, limits the impact of a potential data breach, and makes the system easier to operate responsibly.

### Consequences
- Verification logic must distinguish data needed for an immediate decision from data that needs persistent storage.
- Audit logs must contain enough information to explain important changes without unnecessarily duplicating sensitive data.
- Review workflows must have access to relevant evidence without granting broad access to personal information.
- The team must define retention periods and deletion procedures rather than keeping all records indefinitely.

### Alternatives Considered
- **Detailed evidence retention:** Rejected as the default because persistent GPS readings and extensive device data increase privacy risk.
- **Minimal persistence with almost no diagnostic evidence:** Rejected because it can make disputes, technical failures, and security incidents difficult to investigate.

### Open Questions
- What retention periods should apply to attendance records and audit logs?
- How long should evidence remain available for a pending manual review?
- What privacy notice and deletion process will be appropriate for the intended institution and jurisdiction?


## DEC-028 — Failure Handling and Recovery

**Status:** Accepted

### Decision
The application will fail safely and recover gracefully. Attendance must not be reported as successful until the backend has validated the check-in and confirmed that the attendance record has been persisted. Temporary failures should have a safe retry path, and unresolved legitimate cases may be referred for manual review.

### Rules
- **Database failures:** Do not report a successful check-in unless the attendance transaction commits successfully.
- **Network interruptions:** Show a clear retry option and avoid creating duplicate records when a request is retried.
- **Ambiguous responses:** If the client loses its connection after submitting a check-in, it should query the authoritative attendance result before submitting again.
- **QR token expiration:** Return a clear expired-token response and require a fresh scan, subject to the session eligibility rules in DEC-008 and token rules in DEC-011 and DEC-012.
- **GPS failures:** Allow a reasonable retry when the reading is unavailable or inaccurate. If verification remains inconclusive, offer the configured manual-review path.
- **Socket.IO disconnections:** Preserve server-side records, reconnect where possible, and fetch current state after reconnection.
- **Session closure:** Reject new check-ins after the session's permitted check-in window closes, even if the client still displays an older QR code.
- **Retries:** Use bounded retries for transient failures. Retried operations must be safe against duplicate writes and must not bypass validation.
- **Errors:** Return consistent error responses without exposing stack traces, credentials, raw tokens, or sensitive internal details.
- **Auditability:** Log relevant failures and administrative corrections without unnecessarily storing sensitive information.

### Rationale
Temporary technical failures should not silently create false attendance records or permanently disadvantage legitimate students. Safe retries, authoritative status checks, and controlled manual review provide recovery without weakening the core validation rules.

### Consequences
- Check-in operations must handle concurrent requests and uncertain client outcomes safely.
- The frontend must distinguish successful, failed, and inconclusive results.
- The backend must use database transactions and appropriate constraints for attendance persistence.
- Recovery behavior must be tested for timeouts, lost responses, reconnects, and repeated submissions.

### Alternatives Considered
- **Provisional attendance during outages:** Rejected for the MVP because reconciling provisional records introduces additional complexity and fraud risks.
- **Reject every check-in whenever any dependency fails:** Rejected because recoverable technical issues could unfairly block legitimate students.

### Open Questions
- What retry limits and backoff strategy should be used?
- How long should an inconclusive attempt remain eligible for manual review?
- Which errors should trigger operational alerts during the demonstration?

## DEC-029 — Abuse Prevention and Rate Limiting

**Status:** Accepted

### Decision
The application will implement layered abuse prevention to reduce brute-force attempts, repeated invalid check-ins, unauthorized access, and automated request flooding. Controls will be proportional to the MVP's risk and complexity.

### Rules
- **Login rate limiting:** Limit repeated authentication attempts using appropriate account and IP-based controls.
- **Check-in rate limiting:** Apply configurable limits to check-in requests, especially repeated invalid or expired token submissions.
- **Token validation:** Use the server-side token validation rules defined in DEC-011 and DEC-012. Reject unknown, expired, invalidated, or wrong-session tokens.
- **Duplicate prevention:** Enforce one attendance record per student per session as specified in DEC-018.
- **Retry limits:** Apply bounded retries to transient failures without allowing retries to extend token validity or bypass session eligibility.
- **Authorization:** Enforce DEC-026 on HTTP endpoints and Socket.IO connections, including access to session-specific rooms.
- **Error responses:** Avoid revealing whether a token, account, or protected resource exists when that detail is unnecessary for the client.
- **Security logging:** Record relevant events such as repeated invalid tokens, rate-limit triggers, unauthorized access attempts, and manual overrides.
- **Sensitive data:** Never log passwords, session secrets, or raw QR tokens. Minimize personal and location data in logs in accordance with DEC-027.
- **Configuration:** Rate limits and thresholds must be configurable and tested against realistic classroom traffic.
- **Deployment awareness:** If the backend runs behind a reverse proxy, configure trusted proxy handling carefully so IP-based rate limits cannot be trivially bypassed through spoofed headers.
- **Availability:** Rate limits should reduce abuse without unnecessarily blocking legitimate students who experience poor connectivity or need an allowed retry.

### Rationale
Layered controls are more effective than relying on a single rotating QR token. They help protect authentication, attendance validation, and real-time resources while keeping the implementation achievable for a hackathon MVP.

### Consequences
- The backend must include rate-limiting middleware or an equivalent control.
- Rate limits must be tuned for both abusive behavior and legitimate classroom usage.
- Security-relevant events must be logged without collecting unnecessary sensitive information.
- Tests must cover repeated invalid submissions, duplicate attempts, unauthorized requests, and rate-limit responses.

### Alternatives Considered
- **Basic rate limiting only:** Rejected because it leaves other sensitive operations less protected.
- **Advanced adaptive anti-abuse platform:** Rejected for the MVP because it adds complexity that is not essential to demonstrate the core system.

### Open Questions
- What initial rate-limit thresholds should be used for login, token validation, and check-in?
- Should repeated suspicious attempts temporarily trigger additional verification or account restrictions?
- Which production monitoring and alerting capabilities will be available from the hosting provider?


## DEC-030 — Testing Strategy

**Status:** Accepted

### Decision
The application will use a layered testing strategy comprising unit tests, integration tests, and end-to-end tests. Testing effort will prioritize security-sensitive business rules and the core classroom attendance workflow.

### Testing Layers

**1. Unit Tests**
Test individual functions and modules in isolation, including:
- QR token generation, rotation, expiry, and grace-period handling.
- Session lifecycle and check-in eligibility rules.
- GPS distance and accuracy evaluation.
- Attendance status classification.
- Duplicate-check-in decision logic.
- Authorization helpers and manual-review rules.

**2. Integration Tests**
Test interactions between the Express API, PostgreSQL, authentication, and other backend modules, including:
- Login and protected endpoints.
- Session creation and lifecycle transitions.
- Check-in validation and database persistence.
- Database-level duplicate prevention under concurrent requests.
- Manual overrides and audit-log creation.
- Rate limiting and standardized error handling.
- Socket.IO authentication and session-room authorization.

**3. End-to-End Tests**
Test the primary user journeys:
- A professor creates and activates a lecture session.
- The professor displays a rotating QR code.
- An eligible student authenticates and checks in successfully.
- The professor dashboard receives the attendance update.
- An invalid or expired QR token is rejected.
- A repeated check-in does not create a duplicate record.
- An inconclusive check-in can be retried or sent for manual review.
- An authorized professor approves or rejects a review request.

### Rules
- Tests must verify expected failures as well as successful operations.
- Security-sensitive checks must be enforced and tested on the backend.
- Time-dependent token and session tests should use controllable clocks or equivalent deterministic techniques where practical.
- Integration tests must use an isolated test database or another safe, repeatable test environment.
- Tests must not depend on real student personal data or production credentials.
- Network and BLE simulations must be labelled as simulations and tested separately from genuine verification mechanisms.
- Critical regressions must be fixed before the demonstration.
- The team should document any known limitations that remain untested.

### Rationale
Layered testing provides quick feedback on individual rules, confidence in database and API behavior, and proof that the complete attendance workflow works. It gives the MVP meaningful coverage without requiring a comprehensive enterprise testing program.

### Consequences
- The project must select and configure a test framework and integration-test environment.
- Critical business rules need testable module boundaries.
- Automated tests should run locally and, where feasible, in continuous integration.
- The final demonstration must include a manual smoke test of the deployed application.

### Alternatives Considered
- **Mostly manual testing:** Rejected as the sole strategy because repeated code changes can introduce regressions and manual testing is inconsistent.
- **Extensive automated testing across every layer and risk category:** Deferred because it may consume too much time before the hackathon MVP is complete.

### Open Questions
- Which test framework and end-to-end browser tool should be used?
- Which critical checks must pass before deployment?
- Which tests should run automatically on each commit?


## DEC-031: Demo Acceptance Criteria

**Status:** Accepted

### Context
The hackathon demo must demonstrate that the core attendance system works end to end. The team needs a clear definition of success that prioritizes functional correctness and fraud-resistant attendance over visual polish or features that cannot be completed reliably.

### Decision
The project will target an **end-to-end working MVP**.

The demo is considered successful when the following criteria are met:

1. A professor can authenticate, create a lecture session, and activate it.
2. The system generates a dynamic QR code whose token rotates every 5 seconds, following DEC-011.
3. An authenticated, eligible student can scan the QR code and submit an attendance check-in.
4. The backend validates the token, session state, eligibility, and applicable verification signals.
5. A successful attendance result is persisted in PostgreSQL and returned to the student.
6. The professor's dashboard updates attendance information in real time.
7. Invalid or expired tokens, unauthorized submissions, and duplicate check-ins are handled correctly.
8. Inconclusive verification results can be retried or sent for manual review.
9. An authorized professor can review a case and approve or reject it, with an audit trail.
10. Network or BLE verification that is simulated rather than genuinely implemented is explicitly labelled as simulated in the interface and demo.

### Rationale
An end-to-end MVP provides stronger evidence of the project's value than a polished interface with mocked functionality. It also creates a focused, testable scope suitable for a hackathon while allowing unfinished advanced integrations to remain outside the critical path.

### Alternatives Considered
- **Presentation-first prototype:** Rejected because mocked functionality would not adequately demonstrate the integrity of the attendance workflow.
- **Release-quality target:** Deferred because production-grade readiness and hardware-dependent integrations could jeopardize completion of the core MVP.

### Consequences
- Core authentication, session management, token validation, persistence, attendance processing, and live updates take priority.
- UI polish and advanced integrations are secondary to correctness and reliability.
- Simulations must never be represented as real security evidence.
- The team will validate the acceptance criteria through the testing strategy in DEC-030 before the final demo.

### Open Questions
- Which hosting providers and demo environment will be used?
- What minimum number of concurrent users should the demo support?
- Which network or BLE integrations, if any, can be implemented using available infrastructure?
