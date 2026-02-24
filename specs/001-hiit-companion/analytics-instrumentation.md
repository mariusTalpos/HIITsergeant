# Analytics & Instrumentation Specification

This document outlines analytics and instrumentation expectations for
HIITsergeant. It supports the ability to validate success criteria and
non-functional requirements defined in `feature-spec.md` and
`nfr-reliability.md`.

## Objectives

- Measure usage of core features (personal workouts, group workouts,
  achievements, notifications).
- Validate success criteria (SC-001 through SC-005) without retrofitting
  events later.
- Provide signals for performance and reliability (e.g., timer issues, sync
  failures).

## Core Funnel Events

The system MUST track the following core funnel events to support success
criteria measurement:

- **Plan created**: User creates a new workout plan.
- **Workout started**: User starts a solo or group workout.
- **Workout completed**: User completes a workout (solo or group).
- **Group join attempt**: User attempts to join an instructor-led group workout.
- **Group join success**: User successfully joins a group workout.
- **Achievement unlocked**: User unlocks an achievement.
- **Achievement shared**: User shares an achievement to an external platform.

Additional events (e.g., plan edited/deleted, streak events, notification
scheduled/delivered/tapped) MAY be tracked to support richer analysis.

## Event Properties

Events MUST include consistent properties where applicable to support
segmentation, funnel analysis, and success-criteria validation:

- **Platform**: Web, iOS, Android, or equivalent.
- **Tier**: Free or premium.
- **Entitlement source**: Personal subscription, lifetime, enterprise, or none.
- **Workout type**: Solo or group.
- **Plan source**: User-created, template, or other.
- **App version**: Application or client version identifier.

Properties MUST be included only when relevant to the event (e.g., workout
type applies to workout events, not plan creation).

## Session Identifiers

- The system MUST define **unique session identifiers** for user sessions to
  support deduplication and funnel integrity.
- Session identifiers MUST be used consistently across events within a session
  so that funnel steps (e.g., plan created → workout started → workout
  completed) can be correlated.
- Session boundaries (e.g., app open to app background/close) MUST be defined
  so that funnel metrics are unambiguous.

## Mapping to Success Criteria

Instrumentation MUST directly support validation of:

| Success Criterion | Required Events / Signals |
|-------------------|---------------------------|
| **SC-001** (80% complete first workout in 10 min) | App open/session start, plan created, workout started, workout completed; session timestamps for time-to-complete. |
| **SC-002** (70% describe frequency/time from charts) | Workout completed; optional survey or in-app check; cohort of users with ≥3 workouts. |
| **SC-003** (25% retention uplift from notifications) | Notification/achievement enablement, workout completed over time; 4-week retention by cohort. |
| **SC-004** (90% group join → complete → stats) | Group join attempt, group join success, workout completed (group); funnel completion rate. |
| **SC-005** (4.2/5 timer satisfaction) | In-app rating/satisfaction event; cohort of users with ≥5 workouts. |

## Reliability & Performance Signals

- Timer anomalies (e.g., cue significantly delayed beyond NFR thresholds).
- Group sync resynchronization events and failures.
- Offline workout completions and subsequent sync success/fail.

## Privacy-Safe Analytics Boundaries

- Analytics MUST:
  - **Exclude medical data**: No diagnoses, medications, clinical notes, or
    provider identifiers. No fitness profile fields beyond what is needed for
    aggregate segmentation (e.g., tier, entitlement source).
  - **Exclude unnecessary sensitive fields**: No raw health metrics (e.g.,
    weight, height) in event payloads unless explicitly required for a
    validated use case and approved.
  - Prefer aggregated or pseudonymous identifiers for reporting.
  - Follow the same non-medical and privacy boundaries defined in
    `feature-spec.md` and `data-lifecycle.md`.

