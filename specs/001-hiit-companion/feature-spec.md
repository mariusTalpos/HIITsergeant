# Feature Specification: HIITsergeant core companion experience

**Feature Branch**: `001-hiit-companion`  
**Created**: 2026-02-24  
**Status**: Draft  
**Input**: User description summarizing HIITsergeant as a HIIT companion app for
planning, executing, and tracking workouts; supporting free and premium tiers,
group workouts led by instructors, rich progress statistics and charts, audio
and visual workout timers, notifications, and motivational achievements with
social sharing.

## Clarifications

### Session 2026-02-24

- Q: Account creation (friction vs. cross-device sync) → A: Passwordless sign-in
  only (Apple/Google/Microsoft), no passwords stored by HIITsergeant.
- Q: Payments data handling (no card storage) → A: Web uses a payment provider
  (store no card details; keep only customer/token reference and entitlement
  status). Mobile uses platform app-store billing.
- Q: Health data + “avoid HIPAA” posture → A: Store basic fitness profile (age
  range, height, weight) plus workout stats; explicitly not a medical app (no
  diagnoses, no treatment guidance, no provider/medical-record integrations).
- Q: Premium plan split (personal vs enterprise) → A: Enterprise = organization
  accounts (gym/company) with admin-managed seats, billing, and premium access
  for members.
- Q: How do users join an enterprise organization? → A: Admin generates an invite
  link that can be shared directly and represented as a scannable QR code; users
  accept to join and consume a seat.

## Scope: MVP vs Post-MVP

See `scope-phases.md` for the full phased delivery split. Summary:

- **MVP**: Sign-in, plan CRUD, solo timer, history, basic stats, basic
  achievements, core notifications, free/premium gating.
- **Phase 2**: Instructor-led live group sync; enterprise orgs, seats,
  invites/QR.
- **Phase 2+**: Advanced analytics, richer share assets, missed notifications
  inbox.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Plan and run personal HIIT workouts (Priority: P1) **[MVP]**

A regular exerciser wants to use HIITsergeant as their primary companion for
planning and executing HIIT workouts. They open the app, create or select a
training plan, start a workout, and rely on clear audio and visual cues to
guide them through intense intervals and rest periods. After finishing, they
see a summary of the session and their key stats update automatically.

**Why this priority**: This is the core value of HIITsergeant—without a reliable
personal HIIT timer and planner, the rest of the product has little value.

**Independent Test**: A new user, starting from a fresh account, can create a
plan, start a workout, complete at least one interval session using audio and
visual cues only, and see their workout recorded in history with basic stats.

**Acceptance Scenarios**:

1. **Given** a user who is signed in to HIITsergeant, **when** they create a new
   HIIT workout plan with named exercises, work/rest durations, and total
   rounds, **then** the plan is saved and visible in their list of plans.
2. **Given** a user with at least one saved plan, **when** they start a workout
   based on that plan, **then** the timer guides them through each interval
   using clear audio cues and color-coded visuals for work vs. rest.
3. **Given** a user who has completed a workout, **when** they return to their
   history or dashboard, **then** the session appears with date, duration,
   calories estimate (if supported), and is included in their streak/consistency
   statistics.
4. **Given** a user with workout history, **when** they delete an individual
   session, **then** it is removed from their view and from statistics. Users
   can also delete their account and associated personal data per
   `data-lifecycle.md`.

---

### User Story 2 - Track progress and stay motivated (Priority: P2) **[MVP]**

A user who trains regularly wants to understand their workout habits and stay
motivated over time. They open HIITsergeant to see how often they work out, how
long sessions last, calories burned, and current streaks, all presented in clear
charts. As they reach meaningful milestones, they earn achievements that feel
rewarding and can optionally be shared to social media.

**Why this priority**: Progress tracking and achievements keep users engaged
over weeks and months, increasing long-term retention and perceived value.

**Independent Test**: A user who completes several workouts can view intuitive
charts of their training frequency and duration, see badges unlocked as they hit
milestones, and share at least one achievement externally.

**Acceptance Scenarios**:

1. **Given** a user who has completed multiple workouts over several days,
   **when** they open the progress or statistics section, **then** they see
   charts that clearly show workout frequency (e.g., per week) and total time
   spent training.
2. **Given** a user who reaches a defined milestone (such as a multi-day streak
   or total workout count), **when** they complete the qualifying workout,
   **then** they receive an in-app achievement notification tied to that
   milestone.
3. **Given** a user who has unlocked at least one achievement, **when** they
   choose to share it, **then** the app generates a shareable summary suitable
   for posting on common social channels (e.g., image or formatted text),
   without requiring the user to reveal sensitive personal data.

---

### User Story 3 - Join instructor-led group workouts (Priority: P3) **[Phase 2]**

A user wants the added motivation of training with others in real time. They
join a live, instructor-led group workout from within HIITsergeant. Once they
accept that the instructor controls the timer, their workout timing is synced
to the instructor’s intervals. At the end of the session, their stats and
streaks are updated just like a personal workout. Free users can join eligible
instructor classes even if group workouts are a premium feature overall.

**Why this priority**: Group workouts add social accountability and variety,
which can significantly increase engagement for premium users while still
allowing free users to experience instructor-led sessions.

**Independent Test**: A user can discover a group workout, join it, explicitly
accept instructor timer control, complete the session, and see the session
reflected in their personal stats—even if they are on the free tier.

**Acceptance Scenarios**:

1. **Given** an upcoming or active instructor-led group workout, **when** a user
   browses available sessions, **then** they can see session details such as
   start time, duration, difficulty level, and whether free users can join at no
   cost.
2. **Given** a user has joined a group workout, **when** they accept the prompt
   handing timer control to the instructor, **then** their local timer follows
   the instructor’s intervals and signals (audio and visual) throughout the
   session.
3. **Given** a user completes an instructor-led group workout, **when** they
   view their history and stats, **then** the group session appears alongside
   personal workouts and contributes to appropriate metrics (e.g., streaks,
   total workouts, time trained).
4. **Given** a user loses network during a group workout and later reconnects,
   **when** they view the session, **then** the timer continued on the last
   known timeline during disconnect, the UI indicated sync loss, and on
   reconnect the timer resynced immediately. If they completed ≥50% of the
   session, they receive credit once (no duplicates).

---

[Additional user stories may be added for notifications, subscription
management, and instructor tooling as needed, following the same structure.]

### Edge Cases

- **Network loss during group workout** (resolved): See `nfr-reliability.md`
  (Group Workout Disconnect/Reconnect Lifecycle) and FR-006a for timer
  continuation on last known timeline, sync-state UI, immediate resync on
  reconnect, post-session reconnect handling, duplicate prevention, and 50%
  completion threshold.
- **Music playing loudly** (addressed): See `nfr-reliability.md` (Timer Cue
  Accessibility) and FR-002a: configurable audio intensity/style, haptic
  alternatives, and "Test cue" for validation.
- **Missed notifications** **[Phase 2+]**: User misses a training notification or
  achievement prompt: how and where can they see missed items later without
  overwhelming them? (See `scope-phases.md` — missed notifications inbox.)
  Achievements: stored in history/inbox view per FR-009a; users can always
  review unlocked achievements later.
- **Enterprise invite** **[Phase 2]** (addressed): See `enterprise-orgs-invites.md`:
  invite lifecycle (time-limited, one-time/multi-use, seat-limited), admin
  revocation, expired/revoked UX with clear reason and next steps, and
  admin-visible audit trail.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-000** **[MVP]**: The system MUST support passwordless sign-in (e.g., “Sign in with”
  a trusted identity provider) and MUST NOT store user passwords.
- **FR-000a** **[MVP]**: The system MUST support an optional basic fitness profile (age
  range, height, weight) for personalization and estimates, and MUST allow users
  to update or remove this information.
- **FR-000b** **[MVP]**: The system MUST NOT collect or store medical data such as
  diagnoses, medications, clinical notes, or provider identifiers, and MUST NOT
  present the app as medical advice or a diagnostic tool.
- **FR-001** **[MVP]**: The system MUST allow users to create, edit, and delete HIIT
  workout plans consisting of exercises, work/rest durations, number of rounds,
  and optional labels or categories.
- **FR-002** **[MVP]**: The system MUST provide an in-workout timer that delivers clear
  audio cues (e.g., start, prepare, rest, near-complete) and color-coded visual
  states (e.g., work vs. rest) suitable for use while the user is listening to
  music.
- **FR-002a** **[MVP]**: Timer cues MUST follow the accessibility rules in
  `nfr-reliability.md` (Timer Cue Accessibility): configurable audio style,
  optional haptic cues, non-color alternatives (text/icon/haptic), and a "Test
  cue" option in settings.
- **FR-003** **[MVP]**: The system MUST record completed workouts (personal and
  instructor-led) with date, duration, and key stats (e.g., estimated calories
  burned when supported) and make them visible in a history view.
- **FR-003a** **[MVP]**: Calorie values MUST be estimates only (not medical
  guidance). The system MUST follow the methodology and confidence rules in
  `stats-streaks-achievements.md` (Calorie Estimates): derive from duration +
  workout structure + optional profile; hide or label lower-confidence when
  inputs missing; allow profile edit/removal and display disable.
- **FR-004** **[MVP]**: The system MUST calculate and display progress statistics such as
  workout frequency, total time trained, and streaks using chart types that make
  trends easy to understand at a glance.
- **FR-005** **[MVP]**: The system MUST support a free tier and at least one premium tier
  (subscription and/or lifetime purchase) and MUST restrict premium-only
  features (such as leading group workouts or advanced analytics) while allowing
  free users to join designated instructor-led classes at no cost.
- **FR-0050** **[Phase 2]**: Premium tiers MUST include both personal plans (individual users)
  and enterprise plans (organizations such as gyms or companies).
- **FR-005a** **[MVP]**: For web-based purchases, the system MUST use a payment provider
  and MUST NOT store credit card details (only a provider customer reference,
  transaction/subscription identifiers, and entitlement status).
- **FR-005b** **[MVP]**: For mobile purchases, the system MUST use platform app-store
  billing and MUST NOT store credit card details.
- **FR-005f** **[MVP]**: The system MUST compute effective entitlement server-side
  and apply the resolution rules defined in `billing-entitlements.md` (sources,
  precedence, conflict resolution, refresh timing, outage handling, audit logs).
- **FR-005c** **[Phase 2]**: The system MUST support enterprise organizations with at least one
  administrator who can manage membership seats (add/remove members) and manage
  enterprise billing.
- **FR-005d** **[Phase 2]**: Enterprise membership MUST grant premium entitlements to members
  while their seat is active. By default, enterprise administrators MUST NOT be
  able to view individual user fitness profiles or detailed workout histories.
- **FR-005e** **[Phase 2]**: The system MUST allow enterprise administrators to invite members
  via an invite link that can also be presented as a scannable QR code. Users
  MUST explicitly accept the invite to join and consume a seat.
- **FR-005g** **[Phase 2]**: Invite links MUST support lifecycle rules per
  `enterprise-orgs-invites.md`: time-limited, one-time-use or multi-use,
  seat-limited; admin revocation; expired/revoked UX with clear reason and next
  steps; audit trail (creation, redemption, revocation) visible to org admins.
- **FR-006** **[Phase 2]**: The system MUST enable users to join instructor-led group workouts
  and, upon explicit acceptance, have their workout timer controlled by the
  instructor’s schedule for the duration of the session.
- **FR-006a** **[Phase 2]**: During group workouts, the system MUST clearly indicate sync state
  (fully connected vs. sync lost) in the UI. On network loss, the timer MUST
  continue using the last known instructor timeline when available. On
  reconnect, the timer MUST resync immediately to the instructor’s current
  state. Session credit MUST follow the completion threshold and idempotency
  rules defined in `nfr-reliability.md`.
- **FR-007** **[Phase 2]**: The system MUST update user statistics and streaks automatically
  when a group workout is completed, treating it similarly to a personal
  workout, while still distinguishing group vs. solo sessions for reporting.
- **FR-008** **[MVP]**: The system MUST send training reminders and achievement
  notifications at appropriate times (e.g., before planned workouts, when a
  streak is at risk, or when a milestone is achieved) with user-configurable
  settings for frequency and channels.
- **FR-008a** **[MVP]**: Notifications MUST follow the consent and configuration
  rules in `notifications.md`: explicit opt-in for push, respect platform
  permissions, separately configurable categories (training reminders, streak
  warnings, achievements, group class reminders), and optional quiet hours.
- **FR-009** **[MVP]**: The system MUST award achievements based on well-defined progress
  rules (e.g., total workouts, streak length, total time trained) and present
  them incrementally to avoid overwhelming the user.
- **FR-009a** **[MVP]**: Achievement presentation MUST follow the pacing rules in
  `stats-streaks-achievements.md` (Achievement Presentation & Pacing): max one
  immediate popup at a time, batch additional unlocks into post-workout summary,
  store in history/inbox view for later review.
- **FR-010** **[MVP]**: The system MUST allow users to share selected achievements to
  external platforms (e.g., via generic sharing mechanisms) in a way that
  highlights their accomplishment without exposing sensitive health data.
  Sharing MUST be user-initiated only (no auto-share); share payloads MUST omit
  sensitive profile and health data by default.
- **FR-011** **[MVP]**: The system MUST follow the data retention, deletion, and
  ownership rules in `data-lifecycle.md`: users can delete individual workout
  sessions and their account; enterprise membership changes do not delete
  personal history; org admins do not own personal data; retention for invite
  logs and audit events is defined; shared achievement artifacts are handled as
  specified.
- **FR-012** **[MVP]**: The system MUST implement analytics instrumentation per
  `analytics-instrumentation.md`: core funnel events, consistent event
  properties, unique session identifiers for funnel integrity, mapping to
  success criteria (SC-001 through SC-005), and privacy-safe boundaries.

### Key Entities *(include if feature involves data)*

- **User**: Represents an individual using HIITsergeant, including profile
  details, tier (free or premium), passwordless sign-in identity, optional basic
  fitness profile (age range, height, weight), notification preferences, and
  aggregated workout statistics.
- **Workout Plan**: Represents a configurable HIIT plan containing exercises,
  work/rest durations, total rounds, and any labels for goal or difficulty.
- **Workout Session**: Represents a single completed or in-progress workout
  instance, including reference to the plan (if applicable), timestamps,
  duration, type (solo or group), and recorded stats.
- **Group Workout**: Represents an instructor-led session with scheduled time,
  expected duration, capacity (if any), and rules for whether free users can
  join.
- **Achievement**: Represents a milestone definition (e.g., streak days, total
  workouts) and a user-specific unlock record with date and optional share
  metadata.
- **Entitlement**: Represents the user’s active access rights (free, personal
  premium subscription, personal lifetime, enterprise access), derived from web
  payment provider references and/or mobile app-store receipts.
- **Organization**: Represents a gym/company account with billing information
  (non-card), administrators, and seat limits.
- **Organization Membership**: Represents a user’s membership in an organization,
  including whether they occupy an active paid seat and their role (member vs.
  admin).
- **Invite**: Represents an organization invite link (and optional QR code) with
  lifecycle constraints (expiry, one-time vs multi-use, seat limit), status
  (active, revoked, expired), and audit events (creation, redemption,
  revocation).

## Non-Functional Requirements

See `nfr-reliability.md` for detailed non-functional requirements related to
timer precision, group sync tolerance, offline behavior, startup latency,
notification delivery, timer cue accessibility, availability, and data durability.

## Success Criteria *(mandatory)*

Instrumentation requirements in `analytics-instrumentation.md` define the events
and properties needed to validate these outcomes.

### Measurable Outcomes

- **SC-001** **[MVP]**: At least 80% of new users can create and complete their first HIIT
  workout within 10 minutes of first opening the app, without external help.
- **SC-002** **[MVP]**: For users who complete at least three workouts, at least 70% can
  correctly describe their recent training frequency and total time based solely
  on the app’s charts and summaries.
- **SC-003** **[MVP]**: Users who enable notifications and achievements have at least a
  25% higher 4-week retention rate compared to similar users who do not enable
  them.
- **SC-004** **[Phase 2]**: At least 90% of users who attempt to join a group workout are able
  to join, complete the session, and see it counted in their stats without
  support intervention.
- **SC-005** **[MVP]**: Self-reported satisfaction (e.g., via in-app rating prompt) with
  the workout timer’s audio and visual clarity averages at least 4.2/5 among
  users who have completed five or more workouts.

