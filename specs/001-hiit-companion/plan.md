# Implementation Plan: HIITsergeant Core Companion

**Branch**: `001-hiit-companion` | **Date**: 2026-02-24 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `specs/001-hiit-companion/spec.md`

## Summary

HIITsergeant is a mobile-first HIIT companion app with web support. **Mobile workout execution is the explicit source-of-truth UX**—the canonical experience for planning, running, and completing workouts. Web is a companion surface for account, admin, billing, and stats; workout execution and group participation on web are out of scope until Phase 2+ and may remain mobile-only. The architecture uses a shared backend/API with platform-specific clients; entitlement syncing is server-side and authoritative across all platforms. Phased rollout: MVP mobile-first, then web enhancements.

## Technical Context

**Language/Version**: TypeScript (Node.js 20+ backend; React Native/Expo mobile)  
**Primary Dependencies**: Fastify or Express; Firebase Auth or Auth0; Stripe (web); RevenueCat (mobile IAP); Socket.io or ws (realtime); PostgreSQL; WatermelonDB or expo-sqlite (mobile offline)  
**Storage**: PostgreSQL (backend); SQLite via WatermelonDB or expo-sqlite (mobile offline); sync via REST + idempotent uploads  
**Testing**: Vitest (backend); Jest + Detox or Maestro (mobile E2E); Playwright (web, Phase 2+)  
**Target Platform**: iOS 15+, Android 10+ (API 29+), Web (Chrome/Firefox/Safari recent); tablets as secondary  
**Project Type**: Mobile app (iOS + Android) + Web service (shared API) + Web app (Phase 2+, low priority)  
**Performance Goals**: <200ms perceived latency for core interactions; timer cues ±250ms; 95th percentile workout start ≤2s; 60 fps during timer  
**Constraints**: Offline-capable solo workouts; no medical data; no card storage; server-side entitlement; cross-platform sync  
**Scale/Scope**: 10k+ users; ~50 screens across mobile; MVP ~20 screens; web ~15 screens (Phase 2+)

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

**Post–Phase 1**: Design satisfies all principles. Data model, API contracts, and quickstart align with testing discipline (contract tests), UX consistency (spec-driven flows), and observability (instrumentation per analytics-instrumentation.md).

| Principle | Status | How Design Satisfies |
|-----------|--------|----------------------|
| **I. Code Quality & Maintainability** | ✓ | Shared API contracts; modular mobile feature modules; repository-standard linting (ESLint/Prettier or platform equivalents); ADRs for architecture decisions with rejected alternatives. |
| **II. Testing Discipline (NON-NEGOTIABLE)** | ✓ | Contract tests for API; integration tests for workout sync, entitlement, payments; E2E for critical paths (plan → start → complete); CI blocks on test failure. |
| **III. User Experience Consistency** | ✓ | Shared design system; consistent flows across platforms per spec; copy/error states defined in feature spec; timer UX (audio, haptic, non-color) per nfr-reliability. |
| **IV. Performance & Responsiveness** | ✓ | <200ms target for core interactions; timer precision ±250ms; batched network calls; profiling/telemetry for high-traffic flows. |
| **V. Operational Observability** | ✓ | Instrumentation per analytics-instrumentation.md; structured logs; entitlement/workout/sync metrics; incident post-mortem process. |

## Architecture Overview

### Platform Priority & Responsive Strategy

1. **Primary**: iPhone and Android phones — **source-of-truth UX** for workout execution; full experience, touch-optimized, one-handed use, high-contrast timer.
2. **Secondary**: Tablets — same flows as phones; responsive layout; timer visibility scaled for larger screens.
3. **Tertiary**: Web desktop/laptop — account management, enterprise admin, billing, stats viewing; **no workout execution or group participation** until Phase 2+ (see Web Scope by Phase).

### Shared Backend / API Design

- **Single backend** serves all clients (mobile, web).
- **REST + WebSockets** (or equivalent): REST for CRUD, WebSockets for group workout sync and real-time updates.
- **Server-side entitlement** computed once; clients never gate premium locally without server check.
- **Offline-first for solo workouts**: Local storage + sync queue; conflict resolution and idempotency per nfr-reliability.

### Platform-Specific Considerations

| Area | Mobile | Web |
|------|--------|-----|
| **Billing** | App-store billing (Apple IAP, Google Play Billing); receipt verification | Payment provider (Stripe or similar); no card storage |
| **Auth** | Sign in with Apple/Google; platform OAuth | Sign in with Apple/Google/Microsoft; OAuth/OIDC |
| **Notifications** | Push via FCM/APNs; explicit opt-in; platform permissions | Web push (optional); email fallback |
| **Timer cues** | Audio, haptic, high-contrast visuals; test cue in settings | Audio, visual; no haptic |
| **Offline** | Full offline solo workouts; sync on reconnect | Limited; stats/history require network |

### Entitlement Syncing Across Platforms

- Entitlement computed **server-side** from: web subscription, app-store subscription, lifetime purchase, enterprise seat.
- Clients call `/entitlement` or equivalent on session start and on refresh events.
- Cache up to 5 minutes; invalidate on sign-in, "Restore purchases", webhook.
- Graceful degradation during provider outages (24–72h grace per billing-entitlements).

### Phased Rollout

| Phase | Focus | Platforms |
|-------|-------|-----------|
| **MVP** | Solo workouts, plans, history, stats, achievements, notifications, free/premium | Mobile (iOS + Android) only |
| **Phase 2** | Group workouts, enterprise orgs, invites | Mobile (full) + Web (account/admin/billing/stats only) |
| **Phase 2+** | Advanced analytics, richer share, missed notifications; optional web workout | Mobile + Web enhancements |

### Web Scope by Phase

| Phase | Web Scope | Workout Execution | Group Participation |
|-------|-----------|-------------------|---------------------|
| MVP | No web app | N/A | N/A |
| Phase 2 | Account, enterprise admin, billing, stats | No | No |
| Phase 2+ | Optional web workout, group join | Maybe (low priority) | Maybe (low priority) |

Web is **not** the primary workout surface. Mobile is source-of-truth. Web workout (Phase 2+) is convenience only.

## API Versioning & Backward Compatibility

- **URL versioning**: Base path includes version (e.g., `/v1/plans`). Breaking changes use new version.
- **Backward compat**: Within major version, additive changes only. No removal/renaming without new version.
- **Deprecation**: Deprecated versions supported ≥6 months; `Deprecation` and `Sunset` headers.
- **Client version**: Clients send `X-API-Version`; server may vary behavior for old clients when safe.

## Feature Flags & Rollout Controls

- **Flags** for: group workouts, enterprise, web workout, new payment flows.
- **Rollout**: Per-environment; per-segment (beta, enterprise); percentage rollout.
- **Implementation**: Backend flag service (LaunchDarkly, Unleashed, or env for MVP); clients receive flags on session start.
- **Gating**: Premium + Phase 2+ features require entitlement and flag.

## Realtime Group Sync: Authority & Resync Model

- **Instructor authority**: Instructor device is **single source of truth** for timing. Broadcasts phase, elapsed, remaining at intervals (e.g., 500ms–1s).
- **Participant sync**: Participants receive via WebSocket; display instructor values. Participants do not send timer state.
- **Resync on reconnect**: Client requests current instructor state immediately; server returns phase, elapsed_sec, remaining_sec; client snaps and resumes.
- **Offline during group**: Client continues on last known timeline; UI shows "Sync lost." On reconnect, immediate resync.
- **Completion**: Instructor ends; server broadcasts `ended`; participants upload completion_pct for credit.

## Offline-Capable vs Online-Only Actions

| Action | Offline | Online |
|--------|---------|--------|
| Create/edit/delete plan | Yes | Yes (sync when online) |
| Start/complete solo workout | Yes | Yes (queue upload) |
| View local history | Yes | Yes (cached) |
| View stats | Partial (cached) | Yes |
| Achievements | Partial (cached) | Yes |
| Entitlement | Cached (≤5 min) | Yes |
| Join group workout | No | Yes |
| Group sync | No | Yes (WebSocket) |
| Invite redeem | No | Yes |
| Billing/payment | No | Yes |

## Security & Privacy Implementation Workstreams

| Workstream | Deliverables |
|------------|--------------|
| Webhooks | Stripe/RevenueCat endpoints; signature verification; idempotent handling; entitlement refresh |
| Invites | Transactional redemption; row-level locking; InviteRedemption audit; concurrency-safe seats |
| Audit | Invite/entitlement logs; admin-visible trail; retention per data-lifecycle; no PII beyond user_id |
| Auth | JWT validation; token refresh; secure cookie (web) |

## Testing & Observability by Phase

| Phase | Testing | Observability |
|-------|---------|---------------|
| MVP | Unit, contract, E2E (plan→start→complete); CI blocks | Funnel events; session IDs; error logs; latency |
| Phase 2 | Group sync integration; invite concurrency; webhook tests | Sync metrics; invite audit; webhook retry |
| Phase 2+ | Web E2E; load tests | SC-001–SC-005 dashboards; alerting |

## Key Risks & Tradeoffs

| Risk | Mitigation | Tradeoff |
|------|------------|----------|
| **Cross-platform vs. native** | React Native/Expo vs. Swift/Kotlin: shared code vs. platform polish | Research in Phase 0 to decide. |
| **Offline sync complexity** | Idempotent uploads; last-write-wins or explicit conflict rules | Simpler rules may drop rare edits. |
| **Dual billing (web + app store)** | Server-side entitlement; no local gating | Slight latency on entitlement check. |
| **Web as second-class** | Phase 2 web: account/admin/billing/stats only; no workout execution or group participation | Faster MVP; web workout optional in Phase 2+. |
| **Timer precision on low-end devices** | Profile on target devices; NFR ±250ms | May need to relax on very old hardware. |

## Project Structure

### Documentation (this feature)

```text
specs/001-hiit-companion/
├── plan.md              # This file
├── research.md          # Phase 0 output
├── data-model.md        # Phase 1 output
├── quickstart.md        # Phase 1 output
├── contracts/           # Phase 1 output (API contracts)
└── tasks.md             # Phase 2 output (/speckit.tasks)
```

### Source Code (repository root)

```text
api/                          # Shared backend
├── src/
│   ├── models/
│   ├── services/
│   │   ├── auth/
│   │   ├── entitlement/
│   │   ├── workout/
│   │   ├── billing/
│   │   └── analytics/
│   ├── api/
│   │   ├── rest/
│   │   └── websocket/
│   └── jobs/                 # Sync, webhooks, etc.
└── tests/
    ├── contract/
    ├── integration/
    └── unit/

apps/
├── mobile/                   # React Native or native (iOS + Android)
│   ├── src/
│   │   ├── features/
│   │   │   ├── plans/
│   │   │   ├── timer/
│   │   │   ├── history/
│   │   │   ├── stats/
│   │   │   ├── achievements/
│   │   │   └── auth/
│   │   ├── components/
│   │   ├── services/
│   │   └── design/
│   └── tests/
│
└── web/                      # Phase 2+ (account, admin, billing, stats)
    ├── src/
    │   ├── pages/
    │   ├── components/
    │   └── services/
    └── tests/

packages/                     # Shared code (optional)
└── shared/                   # Types, validation, constants
```

**Structure Decision**: Monorepo with `api/` (backend), `apps/mobile/` (primary client), `apps/web/` (Phase 2+). Shared types and validation in `packages/shared/` to keep contracts consistent. Mobile-first; web added when Phase 2 scope is in flight.

## Complexity Tracking

> No constitution violations requiring justification at this time.
