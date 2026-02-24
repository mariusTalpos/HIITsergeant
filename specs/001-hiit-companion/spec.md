# Feature Specification: HIITsergeant core companion experience

**Feature Branch**: `001-hiit-companion`  
**Created**: 2026-02-24  
**Status**: Draft  
**Input**: User description summarizing HIITsergeant as a HIIT companion app for
planning, executing, and tracking workouts; supporting free and premium tiers,
group workouts led by instructors, rich progress statistics and charts, audio
and visual workout timers, notifications, and motivational achievements with
social sharing.

## Modular Specification Files

This spec has been split into focused module documents:

- `feature-spec.md` – Core feature spec (user stories, functional requirements,
  entities, success criteria, clarifications).
- `scope-phases.md` – MVP vs post-MVP scope (Phase 1, Phase 2, Phase 2+).
- `nfr-reliability.md` – Non-functional requirements for timer precision, group
  sync, group workout disconnect/reconnect lifecycle (timer continuation, sync
  state UI, resync behavior, completion threshold), offline behavior, startup
  latency, notification delivery (best-effort, not guaranteed), timer cue
  accessibility (audio style, haptic, non-color alternatives, test cue),
  availability, data durability, and duplicate prevention/idempotency for session
  uploads.
- `billing-entitlements.md` – Billing flows, free vs. premium tiers, entitlement
  modeling, and entitlement resolution rules (sources, precedence, conflict
  resolution, refresh timing, outage handling, audit logs).
- `enterprise-orgs-invites.md` – Enterprise organizations, seats, admin
  capabilities, invite/QR onboarding, invite lifecycle (time-limited,
  one-time/multi-use, seat-limited), revocation, expired/revoked UX, and
  admin-visible audit trail.
- `data-lifecycle.md` – Data retention, deletion, and ownership rules (session
  delete, account delete, enterprise membership changes, invite/audit retention,
  shared achievement artifacts).
- `stats-streaks-achievements.md` – Progress statistics, streak logic, calorie
  estimate methodology and confidence rules, achievements, and achievement
  presentation/pacing (popup limits, batching, history view, no auto-share,
  share payload defaults).
- `notifications.md` – Reminder and achievement notification behavior, consent
  (explicit opt-in, platform permissions), configurable categories, quiet hours,
  and UX.
- `analytics-instrumentation.md` – Analytics events and instrumentation to
  validate success criteria and NFRs: core funnel events, event properties,
  session identifiers, mapping to SC-001–SC-005, and privacy-safe boundaries.

For full details, refer to the files above.

