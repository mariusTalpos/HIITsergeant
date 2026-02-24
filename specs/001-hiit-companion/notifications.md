# Notifications Specification

This document focuses on reminder and achievement notifications for
HIITsergeant. It expands on notification-related requirements from
`feature-spec.md` and complements `nfr-reliability.md`.

## Goals

- Remind users about planned or streak-saving workouts.
- Inform users about newly unlocked achievements.
- Avoid overwhelming users with excessive or poorly timed notifications.
- Ensure behavior is predictable and user-controlled.

## Notification Consent & Permissions

- Push notifications require **explicit opt-in**. The system MUST NOT send push
  notifications until the user has granted permission.
- The system MUST respect platform notification permissions (e.g., if the user
  disables notifications at the OS level, the app MUST NOT attempt to send
  push notifications).

## Notification Categories

- Notification categories MUST be **separately configurable**:
  - Training reminders (e.g., upcoming or overdue workouts).
  - Streak warnings (e.g., streak at risk).
  - Achievements (e.g., newly unlocked milestone).
  - Group class reminders (e.g., upcoming instructor-led session).
- Users MUST be able to enable or disable each category independently.

## Quiet Hours / Do-Not-Disturb

- The system SHOULD support optional **quiet hours** or **do-not-disturb windows**
  during which the app does not send scheduled notifications.
- Users MUST be able to configure a time range (e.g., 10 PM–7 AM) during which
  notifications are suppressed.
- Quiet hours are optional; if not configured, notifications follow the schedule
  as defined by category settings.

## Functional Requirements (Context)

- Notifications must:
  - Support reminders for upcoming or overdue workouts.
  - Notify users when they unlock a new achievement.
  - Respect user-configurable settings for frequency and channels, where
    applicable.
  - Follow the consent, category, and quiet-hours rules above.

## Non-Functional Constraints

- Delivery is **best-effort and platform-dependent**:
  - The app schedules notifications, but exact delivery time is not guaranteed.
  - See `nfr-reliability.md` for non-functional expectations.

## UX Considerations

- Provide clear controls in the app for:
  - Enabling/disabling reminders.
  - Choosing which types of notifications to receive (e.g., reminders vs.
    achievements).
- Ensure content is concise and actionable (e.g., “Time for your next HIIT
  session” with a clear call to start).
