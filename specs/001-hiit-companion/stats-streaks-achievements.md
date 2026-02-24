# Stats, Streaks & Achievements Specification

This document details how HIITsergeant tracks user progress and uses statistics
and achievements to drive motivation. It elaborates on User Story 2, related
functional requirements, and the `Achievement` entity from `feature-spec.md`.

## Goals

- Help users understand their workout habits over time.
- Provide clear, visual feedback on frequency, duration, and consistency.
- Reward meaningful milestones with achievements that can be shared.

## Core Metrics

- Workout frequency (e.g., workouts per week).
- Total time trained over selectable periods.
- Streaks (consecutive days/weeks with at least one workout).
- Optional estimated calories burned (when inputs are available; see Calorie
  Estimates below).

## Calorie Estimates

Calorie values are **estimates only**, not medical guidance. This section
defines methodology and confidence rules to avoid false precision and stay
aligned with the non-medical posture.

### Methodology

- Estimates are derived from:
  - Workout duration and structure (intervals, work/rest ratio, intensity).
  - Optional fitness profile inputs (age range, height, weight) when present.
- The system MUST NOT imply medical-grade accuracy or present estimates as
  clinical data.

### Confidence & Display

- If required inputs for a reasonable estimate are missing (e.g., no fitness
  profile), the system MUST either:
  - Hide calorie values, or
  - Label the estimate as lower-confidence (e.g., “approximate” or “rough
    estimate”).
- Users MUST be able to edit or remove profile inputs; calorie estimates MUST
  update accordingly when profile data changes.
- Users MAY disable calorie display entirely if they prefer not to see it.

### Avoid False Precision

- Estimates MUST NOT be presented with unwarranted precision (e.g., avoid
  “347 kcal”; prefer “~350 kcal” or rounded values).
- When estimates vary between similar workouts, the system MUST NOT imply that
  small differences are meaningful.

## Visualizations

- Charts MUST make trends easy to understand at a glance, such as:
  - Bar or line charts for weekly/monthly workout counts.
  - Cumulative time charts for total minutes trained.
  - Streak indicators (e.g., “current streak”, “longest streak”).

## Achievements

- Achievements are unlocked based on well-defined rules, for example:
  - Number of workouts completed.
  - Length of current or longest streak.
  - Total time trained.
- Achievements MUST be:
  - Delivered incrementally over time (not all at once at onboarding).
  - Visible in an in-app achievements view.
  - Optionally shareable to external platforms without exposing sensitive
    health data.

## Achievement Presentation & Pacing

To keep the motivation loop strong without overwhelming users:

- **Immediate popups during/after workout**: The system MUST limit immediate
  achievement popups (e.g., toasts or modals) to at most **one at a time**
  during or immediately after a workout.
- **Batching**: Additional unlocks beyond the first MUST be batched into a
  **post-workout summary** (e.g., "You unlocked 3 achievements!" with a list),
  rather than shown as multiple sequential popups.
- **History/inbox view**: Unlocked achievements MUST be stored and visible in a
  **history or inbox view** so users can review them later. This view MUST
  persist across sessions.
- **No auto-share**: Sharing is **always user-initiated**. The system MUST NOT
  auto-share achievements to external platforms.
- **Share payloads**: Share payloads (e.g., images, formatted text) MUST omit
  sensitive profile and health data by default (e.g., no weight, no calorie
  values unless the user explicitly includes them).

## Functional Links

- Aligns with:
  - **FR-003**, **FR-003a**: Workout history and calorie estimate methodology.
  - **FR-004**: Progress statistics (frequency, time, streaks).
  - **FR-009**, **FR-009a**: Achievement rules, incremental delivery, and
    presentation pacing.
  - **FR-010**: Shareable achievements; user-initiated only; no sensitive data
    in share payloads.
  - **SC-002** and **SC-003**: Understanding of training patterns and impact on
    motivation/retention.

