# MVP vs Post-MVP Scope

This document defines the phased delivery scope for HIITsergeant. It reduces
delivery risk by keeping the MVP focused while preserving long-term product
direction.

## MVP (Phase 1)

**Goal**: Ship a usable solo HIIT companion with basic monetization and
motivation features.

| Area | Scope |
|------|-------|
| **Sign-in** | Passwordless sign-in (Apple/Google/Microsoft); no passwords stored |
| **Plan CRUD** | Create, edit, delete HIIT workout plans (exercises, work/rest, rounds) |
| **Solo timer** | In-workout timer with audio cues and color-coded visuals |
| **History** | Completed workout history with date, duration, basic stats |
| **Basic stats** | Workout frequency, total time trained, streaks; clear charts |
| **Basic achievements** | Milestone-based achievements (e.g., streak, total workouts); incremental delivery |
| **Core notifications** | Training reminders, achievement notifications; user-configurable; best-effort delivery |
| **Free/premium gating** | Free tier + personal premium (subscription and/or lifetime); no card storage (web: provider; mobile: app store) |

**Out of MVP scope**: Group workouts, enterprise orgs, advanced analytics,
richer share assets, missed-notifications inbox.

## Post-MVP / Phase 2

**Goal**: Add social and organizational features.

| Area | Scope |
|------|-------|
| **Instructor-led group sync** | Live group workouts; instructor controls timer; participant sync; disconnect/reconnect lifecycle; completion threshold |
| **Enterprise orgs** | Organization accounts (gyms, companies); admin-managed seats; billing |
| **Enterprise invites/QR** | Admin-generated invite links; scannable QR codes; user accepts to consume seat |

## Post-MVP / Phase 2+

**Goal**: Enhance engagement and operational visibility.

| Area | Scope |
|------|-------|
| **Advanced analytics** | Richer instrumentation; dashboards; validation of success criteria and NFRs |
| **Richer share assets** | Enhanced achievement share cards; branded or customizable share images |
| **Missed notifications inbox** | In-app view of missed reminders and achievement prompts; catch-up without overwhelming the user |

## Mapping to User Stories & Requirements

- **User Story 1** (Plan and run personal HIIT workouts) → **MVP**
- **User Story 2** (Track progress and stay motivated) → **MVP** (basic stats, basic achievements, basic share)
- **User Story 3** (Join instructor-led group workouts) → **Phase 2**
- **Enterprise orgs, seats, invites** → **Phase 2**
- **Advanced analytics, richer share, missed notifications inbox** → **Phase 2+**

See `feature-spec.md` for phase labels on individual functional requirements.
