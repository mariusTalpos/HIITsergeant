# API Contract Overview

**Branch**: `001-hiit-companion` | **Date**: 2026-02-24

This document defines the interface contract between HIITsergeant clients (mobile, web) and the shared backend API. Implementations MUST conform to these contracts for interoperability and contract testing.

## Base

- **Base URL**: `https://api.hiitsergeant.com/v1` (or configurable)
- **Auth**: Bearer JWT from Firebase Auth or Auth0
- **Content-Type**: `application/json`
- **Idempotency**: Endpoints that create resources accept `Idempotency-Key` header for deduplication

---

## Endpoint Groups

### Auth & User

| Method | Path | Purpose |
|--------|------|---------|
| GET | /me | Current user profile |
| PATCH | /me | Update profile, fitness profile, notification preferences |
| DELETE | /me | Account deletion (per data-lifecycle) |

### Entitlement

| Method | Path | Purpose |
|--------|------|---------|
| GET | /entitlement | Effective entitlement (tier, source, expires_at) |

**Response**: `{ tier: "free"|"premium", source: "web_subscription"|"app_subscription"|"lifetime"|"enterprise"|null, expires_at: string|null }`

**Cache**: Clients MAY cache up to 5 minutes; invalidate on sign-in, "Restore purchases", or 401/403.

### Plans

| Method | Path | Purpose |
|--------|------|---------|
| GET | /plans | List user's plans |
| POST | /plans | Create plan |
| GET | /plans/:id | Get plan |
| PATCH | /plans/:id | Update plan |
| DELETE | /plans/:id | Soft delete plan |

### Sessions (Workouts)

| Method | Path | Purpose |
|--------|------|---------|
| GET | /sessions | List user's sessions (paginated) |
| POST | /sessions | Create/complete session (idempotent via Idempotency-Key) |
| GET | /sessions/:id | Get session |
| DELETE | /sessions/:id | Delete session (per data-lifecycle) |

**POST /sessions**: Body includes plan_id, type (solo|group), started_at, completed_at, duration_sec, stats. Idempotency-Key = `{user_id}:{client_generated_session_id}`.

### Achievements

| Method | Path | Purpose |
|--------|------|---------|
| GET | /achievements | List user's unlocked achievements |
| POST | /achievements/:id/unlock | Record unlock (idempotent; called when milestone reached) |

### Organizations (Phase 2)

| Method | Path | Purpose |
|--------|------|---------|
| GET | /organizations | List user's orgs (admin sees more) |
| GET | /organizations/:id | Get org |
| POST | /organizations/:id/invites | Create invite (admin) |
| POST | /invites/:token/redeem | Redeem invite (join org) |
| DELETE | /invites/:id | Revoke invite (admin) |

### Group Workouts (Phase 2)

| Method | Path | Purpose |
|--------|------|---------|
| GET | /group-workouts | List available/upcoming |
| POST | /group-workouts/:id/join | Join session |
| WebSocket | /group-workouts/:id/sync | Realtime timer sync (instructor broadcasts, participants receive) |

---

## Error Responses

| Code | Meaning |
|------|---------|
| 400 | Bad request (validation error) |
| 401 | Unauthorized (missing/invalid token) |
| 403 | Forbidden (e.g., premium feature, free user) |
| 404 | Not found |
| 409 | Conflict (e.g., duplicate idempotency key with different payload) |
| 422 | Unprocessable (e.g., invite expired, no seats) |
| 500 | Server error |

**Error body**: `{ error: { code: string, message: string, details?: object } }`

---

## WebSocket Contract (Group Sync)

- **Connect**: `wss://api.hiitsergeant.com/v1/group-workouts/:id/sync` with Bearer token
- **Instructor → Participants**: `{ type: "sync", phase: string, elapsed_sec: number, remaining_sec: number, timestamp: string }`
- **Participants**: Receive only; no client→server sync messages for timer state
- **Reconnect**: Client reconnects on disconnect; server sends current state immediately

---

## Contract Testing

Contract tests MUST validate:
- Request/response schemas for each endpoint
- Idempotency behavior for POST /sessions
- Entitlement response shape and cache headers
- Error response format

See `api-schema.json` (or OpenAPI spec) for machine-readable schemas when implemented.
