# Research: HIITsergeant Implementation

**Branch**: `001-hiit-companion` | **Date**: 2026-02-24

This document consolidates research findings that resolve NEEDS CLARIFICATION items from the implementation plan and inform technology choices.

---

## 1. Mobile Framework: React Native vs. Native (Swift/Kotlin)

**Decision**: React Native (Expo) for MVP mobile apps.

**Rationale**:
- Single codebase for iOS and Android reduces MVP delivery time and maintenance.
- Expo provides OTA updates, managed workflow, and good defaults for auth (Expo AuthSession), push (Expo Notifications), and in-app purchases (expo-in-app-purchases or RevenueCat).
- Touch-first, one-handed UX is achievable with React Native; timer precision can be validated with native modules if needed (e.g., for ±250ms cue timing).
- Web support later can share logic via React/React Native Web or a separate web app; for Phase 2+, web is low-priority and can be a separate React app.

**Decision threshold (timer)**: If timer cue precision cannot meet ±250ms NFR on ≥20% of target devices (iOS 15+, Android 10+) after optimization (native timer module, requestAnimationFrame, high-priority thread), trigger exit-criteria evaluation for React Native.

**React Native/Expo exit criteria** (when to revisit native):
- Timer precision threshold exceeded (see above) and native timer module cannot resolve.
- Critical UX regressions (e.g., jank during timer, one-handed reachability) that cannot be resolved with React Native.
- App-store or platform requirements that block Expo managed workflow (e.g., specific native APIs).
- Team commits to native rewrite; Phase 2+ timeline allows 2–3x effort for Swift/Kotlin.

**Alternatives considered**:
- **Native Swift/Kotlin**: Best platform polish and performance; doubles development effort for MVP. Revisit only if exit criteria met.
- **Flutter**: Strong cross-platform story; smaller ecosystem for payments/auth integrations; team familiarity assumed lower.

---

## 2. Backend Language & Framework

**Decision**: Node.js 20+ with TypeScript; **Fastify** for REST API.

**Rationale**:
- TypeScript aligns with React Native/Expo and enables shared types via `packages/shared`.
- Node.js ecosystem has mature integrations for Stripe, auth, and WebSockets.
- Fastify chosen over Express: built-in schema validation (JSON Schema), better performance, smaller footprint. Express deferred: use only if team has strong preference or existing Express codebase to integrate.

**Alternatives considered**:
- **Express**: More widely known; no built-in validation; larger. Deferred unless team preference overrides.
- **.NET 8**: Strong typing; less natural type sharing with TypeScript clients.
- **Go**: Excellent performance; smaller ecosystem for payment integrations.

---

## 3. Storage & Sync

**Decision**: PostgreSQL for backend; **WatermelonDB** for mobile offline (SQLite-backed); sync via REST + idempotent uploads.

**Rationale**:
- PostgreSQL supports JSONB for flexible workout/plan schemas and robust transactional semantics for entitlement and billing.
- **WatermelonDB** chosen over expo-sqlite: reactive queries, lazy loading, sync primitives (pull/push), and first-class React Native support. expo-sqlite deferred: use only if WatermelonDB bundle size or complexity is prohibitive for MVP.
- Sync strategy: client uploads completed sessions with idempotency keys; server deduplicates per session+user. No real-time sync required for solo workouts.

**Decision thresholds**:
- **Offline storage**: Revisit WatermelonDB if (a) bundle size impact >500KB, (b) sync conflicts exceed 5% of uploads, or (c) team cannot achieve reactive UI patterns. Fallback: expo-sqlite + manual sync queue.

**Alternatives considered**:
- **expo-sqlite**: Simpler; no built-in reactivity or sync primitives. Deferred.
- **Supabase/Firebase**: Faster setup; vendor lock-in; less control.
- **Realm**: Good sync; adds another stack.

---

## 4. Auth (Passwordless)

**Decision**: **Firebase Auth** for MVP passwordless sign-in (Apple, Google, Microsoft).

**Rationale**:
- Firebase Auth supports Apple/Google/Microsoft OAuth, returns JWTs, and has a generous free tier. Simpler setup than Auth0 for MVP.
- Auth0 deferred: use if Phase 2 enterprise requires org-level SSO, custom branding, or advanced MFA. Criteria: enterprise customer requests SSO or compliance requires Auth0 features.
- No passwords stored; tokens used for API auth.

**Identity-linking assumption**:
- **Single provider per account at MVP**: User signs in with one provider (e.g., Apple); that provider's `uid` maps to HIITsergeant `user_id`. No multi-provider linking (e.g., Apple + Google → same account) in MVP.
- **Phase 2+**: If cross-device or "Sign in with Google on web, Apple on mobile" is required, add account-linking (Firebase `linkWithCredential` or Auth0 account linking). Deferred until user research confirms need.

---

## 5. Payments

**Decision**:
- **Web**: Stripe (subscriptions + one-time for lifetime).
- **Mobile**: RevenueCat for subscription/lifetime management; abstracts Apple IAP and Google Play Billing.

**Stripe / RevenueCat entitlement & webhook boundaries**:

| Boundary | Stripe (Web) | RevenueCat (Mobile) |
|----------|--------------|---------------------|
| **Ownership** | HIITsergeant backend owns entitlement computation | HIITsergeant backend owns entitlement computation |
| **Webhook role** | Stripe sends subscription/payment events; backend updates billing source records; backend recomputes entitlement | RevenueCat sends subscriber info events; backend updates billing source records; backend recomputes entitlement |
| **Stored data** | customer_id, subscription_id, status, current_period_end | app_user_id (maps to HIITsergeant user_id), product_id, entitlement_ids, expires_at |
| **Backend** | Stripe webhook → verify signature → upsert web_subscription → recompute entitlement | RevenueCat webhook → verify signature → upsert app_store_purchase → recompute entitlement |
| **No** | Backend does NOT store card details; does NOT rely on Stripe for entitlement gating | Backend does NOT store raw receipts; does NOT rely on RevenueCat for entitlement gating (RevenueCat is source of truth for IAP state only) |

**Rationale**: Single source of truth for effective entitlement is always HIITsergeant backend. Stripe and RevenueCat are **inputs** (billing source records); backend resolves precedence per billing-entitlements.md.

**Alternatives considered**:
- **Direct IAP + custom verification**: More control; more implementation and edge-case handling.
- **Paddle**: Good for global tax; less common for mobile-first.

---

## 6. Realtime (Group Workouts)

**Decision**: WebSockets via **Socket.io** for instructor-led group sync. Server-authoritative timeline; instructor broadcasts; participants consume only.

**Rationale**:
- Group workouts require sub-second sync per nfr-reliability; WebSockets provide low-latency push.
- **Socket.io** chosen over native `ws`: built-in reconnection, room/namespace semantics, fallback to long-polling. Native `ws` deferred if Socket.io overhead is measurable (>50ms) or bundle size is critical.
- Fallback: long-polling or SSE if WebSockets are blocked; document in contracts.

**Server-authoritative timeline model**:
- **Instructor is source of truth**: Instructor device sends phase, elapsed_sec, remaining_sec to server at configurable intervals (e.g., 500ms–1s). Server does NOT derive timeline from participants.
- **Server broadcasts to participants**: Server receives instructor state and broadcasts to all participants in the group room. Participants display received values; they do NOT send timer state.
- **Server stores last known state**: For resync, server persists last instructor broadcast per group session. On participant reconnect, server returns this state immediately (no wait for next broadcast).

**Reconnect / resync model**:
1. **Participant disconnects**: Client continues on last received timeline locally; UI shows "Sync lost."
2. **Participant reconnects**: Client connects to WebSocket; immediately requests current state (REST or WebSocket message).
3. **Server response**: Server returns last instructor state (phase, elapsed_sec, remaining_sec, timestamp). Client snaps local timer to these values.
4. **Resume live updates**: Client resumes receiving instructor broadcasts. No gradual catch-up; immediate snap per nfr-reliability.

**Decision thresholds**:
- **Realtime**: Revisit for Pusher/Ably if (a) self-hosted WebSocket infra exceeds 2 FTE-months, (b) connection scale >10k concurrent groups, or (c) cross-region latency >200ms. Otherwise Socket.io sufficient.

**Alternatives considered**:
- **Native ws**: Lighter; no built-in reconnection. Deferred.
- **Pusher/Ably**: Managed; adds cost. Deferred until scale threshold.
- **Server-Sent Events**: One-way only; instructor needs separate channel. Not suitable for instructor→server→participants flow.

---

## 7. Testing Stack

**Decision**:
- **Backend**: Vitest for unit/integration; contract tests with OpenAPI or JSON Schema.
- **Mobile**: Jest for unit; **Maestro** for E2E.
- **Web (Phase 2+)**: Playwright for E2E.

**Rationale**:
- Vitest is fast and TypeScript-native; aligns with Node backend.
- **Maestro** chosen over Detox: no native build required for E2E, YAML-based flows, cross-framework. Detox deferred: use if Maestro cannot reliably test timer precision or React Native-specific gestures (e.g., swipe-to-dismiss). Criteria: >10% flaky E2E rate or missing coverage for timer flow.
- Contract tests ensure API stability across client updates.

**Alternatives considered**:
- **Detox**: React Native–native; requires native build. Deferred.
- **Appium**: More setup; Maestro sufficient for MVP.

---

## 8. Analytics / Instrumentation

**Decision**: Use existing analytics-instrumentation.md spec; implement via a provider-agnostic event API (e.g., segment-style) so backend can send to PostHog, Mixpanel, or Amplitude without code changes.

**Rationale**:
- Spec defines events and properties; implementation should not lock to a single vendor.
- Privacy-safe boundaries (no medical data, no unnecessary PII) are enforced at the event schema level.

---

## Summary: Resolved Technical Context

| Item | Resolved Choice |
|------|-----------------|
| **Mobile** | React Native (Expo) |
| **Backend** | Node.js 20+ / TypeScript / Fastify |
| **Storage** | PostgreSQL (server); WatermelonDB (mobile offline) |
| **Auth** | Firebase Auth (passwordless); single provider per account at MVP |
| **Web payments** | Stripe |
| **Mobile payments** | RevenueCat (IAP abstraction) |
| **Realtime** | WebSockets via Socket.io; server-authoritative timeline |
| **Testing** | Vitest (backend); Jest + Maestro (mobile E2E); Playwright (web) |
