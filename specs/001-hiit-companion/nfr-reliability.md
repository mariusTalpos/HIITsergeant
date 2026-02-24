# Non-Functional Requirements: HIITsergeant Core Experience

This document details non-functional requirements (NFRs) for the HIITsergeant
core experience. It elaborates the `## Non-Functional Requirements` referenced
from `feature-spec.md`.

## Timer Precision

- For solo workouts, audible and visual cues MUST fire within ±250ms of their
  scheduled time under normal device conditions on supported platforms.

## Group Sync Tolerance

- During instructor-led group workouts, participants’ timers SHOULD remain
  within 1 second of the instructor timeline under normal network conditions.
- **Reconnect/resync behavior**: After any resynchronization event (e.g., brief
  connectivity loss), the system SHOULD return participants to within 1 second
  of the instructor timeline. Users MUST see clear feedback when resync is in
  progress or has completed.

## Group Workout Disconnect/Reconnect Lifecycle

These requirements promote group workout network resilience from an edge case
into explicit, testable behavior. They are critical for trust in group classes
and support load.

### During Network Loss

- If a participant loses network connectivity during an instructor-led group
  workout, the app MUST continue running the timer using the **last known
  instructor timeline** (phase, elapsed time, remaining time) if one was received
  before disconnect.
- If no instructor timeline was ever received (e.g., disconnect at join), the
  app MUST show a clear error state and prompt the user to reconnect.

### Sync State Indication

- The UI MUST clearly indicate **sync state** to the user:
  - **Fully connected**: Participant timer is in sync with instructor.
  - **Sync lost** (disconnected): Participant is on last known timeline; no live
    updates.
- The indication MUST be visible without obscuring the workout content (e.g.,
  subtle icon, banner, or status text).

### On Reconnect

- On reconnect, the app MUST resynchronize the participant timer to the
  instructor’s **current phase and time**.
- **Resync timing**: Resync MUST snap the participant timer to the instructor’s
  current state **immediately** upon successful reconnect (no wait for next
  interval boundary), so participants return to sync as quickly as possible.
- Users MUST see clear feedback when resync is in progress and when it has
  completed.

### Reconnect After Session Ended

- If the participant reconnects **after the instructor has ended the session**:
  - The app MUST fetch the final session state from the server.
  - The participant MUST see the session as completed (if they met the
    completion threshold) and receive appropriate credit.
  - Duplicate completion submissions MUST NOT create duplicate session credit
    (see Duplicate Prevention & Idempotency).

### Duplicate Session Credit

- Repeated completion submissions caused by reconnect/retry MUST NOT grant
  duplicate session credit. The server MUST treat completion uploads as
  idempotent per unique session + participant pair.

### Completion Threshold

- A participant’s group workout session MUST count toward their history and
  statistics only if they completed **at least 50% of the planned session
  duration** (or equivalent interval count, if duration is not the primary
  measure).
- Participants who disconnect before reaching the threshold MUST NOT receive
  session credit, even if they reconnect after the session has ended.

## Offline Behavior

- Once a solo workout has started, the timer and cues MUST continue to function
  without network connectivity.
- Completed solo workouts performed offline MUST be queued for later sync and
  MUST appear in local history immediately after completion.

## Startup Latency

- From selecting “start workout” to the timer screen being ready to begin, the
  95th percentile latency SHOULD be ≤2 seconds on supported devices and network
  conditions.

## Notification Delivery

- Delivery of reminders and achievement notifications is **best-effort and
  platform-dependent**; it is **not guaranteed**. The system MUST schedule them
  appropriately but CANNOT guarantee exact delivery time or delivery at all on
  all devices.

## Timer Cue Accessibility

Timer cues (e.g., phase transitions, countdown alerts) MUST be accessible and
configurable so users can work out effectively under real conditions.

### Audio Cues

- The system MUST offer **audio cue intensity or style options**, where supported:
  - Minimal (e.g., short beep).
  - Beep (e.g., distinct tone).
  - Voice (e.g., spoken countdown or phase name), if supported by the platform.
- Users MUST be able to select their preferred audio style in settings.

### Haptic / Vibration Cues

- The system SHOULD offer **haptic or vibration cue options** where the platform
  supports them.
- Users MUST be able to enable or disable haptic cues independently of audio.

### Non-Color Alternatives

- Timer cues MUST NOT rely on color alone. The system MUST provide at least one
  of: text labels, icons, or haptic feedback as alternatives to convey phase or
  state changes.
- Users who cannot distinguish colors MUST be able to understand timer state
  without relying on color.

### Test Cue Option

- Settings MUST include a **"Test cue"** option that allows users to trigger
  audio and/or haptic cues on demand for validation before a workout.

## Availability Targets

- After initial launch, core APIs required for sign-in, entitlement checks, and
  workout sync SHOULD target at least 99.9% monthly availability, excluding
  scheduled maintenance windows.

## Data Durability

- No completed workout that has reached the local “finished” state SHOULD be
  lost. The system MUST ensure durable storage locally and reliable retry for
  syncing to the server when connectivity is restored.
- Completed sessions MUST be persisted locally before any remote sync attempt.

## Duplicate Prevention & Idempotency

- Retried session uploads (e.g., after network recovery) MUST be idempotent: the
  server MUST accept a session upload only once per unique session identifier.
- Duplicate uploads for the same session MUST NOT create duplicate workout
  records in the user’s history or in statistics.

