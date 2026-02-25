# Data Model: HIITsergeant

**Branch**: `001-hiit-companion` | **Date**: 2026-02-24

This document defines the core entities, fields, relationships, and validation rules derived from the feature specification. Implementation may use ORM mappings (e.g., Prisma, TypeORM) or raw SQL; this model is technology-agnostic.

## Entity Overview

| Entity | Purpose | Key Relationships |
|--------|---------|-------------------|
| User | Individual using HIITsergeant | Plans, Sessions, Achievements, Entitlement, OrgMembership |
| WorkoutPlan | HIIT plan definition | User, WorkoutSession |
| WorkoutSession | Completed or in-progress workout | User, WorkoutPlan, GroupWorkout (optional) |
| Achievement | Milestone definition + unlock record | User |
| Entitlement | Effective access rights (computed) | User |
| Organization | Gym/company account | OrgMembership, Invite |
| OrganizationMembership | User's membership in org | User, Organization |
| Invite | Org invite link with lifecycle | Organization |
| InviteRedemption | Audit record for invite redemption | Invite, User |
| GroupWorkout | Instructor-led session | WorkoutSession (participants) |
| GroupWorkoutParticipant | Phase 2: Join/leave/accept-control/completion% | User, GroupWorkout |
| BillingSource | Persisted billing inputs for entitlement resolution | User |

---

## User

Represents an individual using HIITsergeant.

| Field | Type | Constraints | Notes |
|-------|------|-------------|-------|
| id | UUID | PK, immutable | |
| external_id | string | unique, indexed | From auth provider (Firebase/Auth0) |
| email | string? | nullable | From provider if available |
| created_at | timestamp | | |
| updated_at | timestamp | | |
| fitness_profile | JSON? | | age_range, height, weight; optional |
| notification_preferences | JSON | | Categories, quiet hours per notifications.md |
| deleted_at | timestamp? | nullable | Soft delete for account deletion |

**Validation**: No medical data (diagnoses, medications, provider IDs). Fitness profile optional; editable/removable per FR-000a.

---

## WorkoutPlan

Configurable HIIT plan with exercises, work/rest, rounds.

| Field | Type | Constraints | Notes |
|-------|------|-------------|-------|
| id | UUID | PK | |
| user_id | UUID | FK → User, indexed | |
| name | string | required | |
| exercises | JSON | required | Array of { name, work_sec, rest_sec } |
| rounds | integer | required, ≥1 | |
| labels | JSON? | | goal, difficulty, etc. |
| source | enum | | user_created, template, other |
| created_at | timestamp | | |
| updated_at | timestamp | | |
| deleted_at | timestamp? | | Soft delete |

**Validation**: exercises must have at least one entry; work_sec and rest_sec positive integers.

---

## WorkoutSession

Single completed or in-progress workout instance.

| Field | Type | Constraints | Notes |
|-------|------|-------------|-------|
| id | UUID | PK | |
| user_id | UUID | FK → User, indexed | |
| plan_id | UUID? | FK → WorkoutPlan, nullable | |
| group_workout_id | UUID? | FK → GroupWorkout, nullable | |
| type | enum | solo, group | |
| **session_status** | enum | **in_progress, completed, abandoned** | Business status; distinct from sync |
| started_at | timestamp | | |
| completed_at | timestamp? | | Required when session_status = completed |
| duration_sec | integer? | | Required when session_status = completed |
| stats | JSON? | | calories_estimate, etc. |
| sync_status | enum | pending, synced, conflict | **Client-local** sync metadata; see note below |
| idempotency_key | string? | unique | For deduplication |

**Validation**:
- **session_status** drives business rules: `completed_at` and `duration_sec` MUST be set when `session_status = completed`; MUST be null when `session_status = in_progress` or `abandoned`.
- `sync_status` is **client-local** metadata only: indicates whether the client has successfully uploaded this session to the server. The server does NOT persist `sync_status`; it either has the session record (from a successful upload) or not. Clients use it for offline queue and retry UI.
- idempotency_key format: `{user_id}:{client_session_id}` or equivalent. Duplicate uploads with same key are idempotent per nfr-reliability.

---

## Achievement

Milestone definition and user-specific unlock.

| Field | Type | Constraints | Notes |
|-------|------|-------------|-------|
| id | UUID | PK | For unlock record |
| definition_id | string | | e.g., "streak_7", "workouts_10" |
| user_id | UUID | FK → User, indexed | |
| unlocked_at | timestamp | | |
| share_metadata | JSON? | | Optional share tracking |

**Validation**: One unlock per (user_id, definition_id). Definition rules in stats-streaks-achievements.md.

---

## Entitlement (Computed)

Effective access level; not a stored entity but **derived** from persisted billing source records and org data.

| Logical Field | Type | Source |
|---------------|------|--------|
| tier | enum | free, premium |
| source | enum | web_subscription, app_subscription, lifetime, enterprise |
| expires_at | timestamp? | For subscriptions |

**Inputs to resolution** (persisted billing source records):
- **Web subscription**: Provider customer ID, subscription ID, status, expiry (from Stripe or similar).
- **App-store purchase/receipt**: Receipt or verification reference, product ID, status, expiry (from RevenueCat or native verification).
- **Lifetime grant**: One-time purchase record (web or app store) with no expiry.
- **Enterprise seat**: OrganizationMembership with seat_assigned = true.

**Resolution**: Server computes effective entitlement from these inputs per billing-entitlements.md (precedence, conflict resolution). Clients never compute locally.

---

## Organization

Gym/company account.

| Field | Type | Constraints | Notes |
|-------|------|-------------|-------|
| id | UUID | PK | |
| name | string | required | |
| seat_limit | integer | required, ≥1 | |
| billing_customer_id | string? | | Provider reference |
| created_at | timestamp | | |
| updated_at | timestamp | | |

---

## OrganizationMembership

User's membership in an organization.

| Field | Type | Constraints | Notes |
|-------|------|-------------|-------|
| id | UUID | PK | |
| user_id | UUID | FK → User | |
| organization_id | UUID | FK → Organization | |
| role | enum | member, admin | |
| seat_assigned | boolean | | True if occupying paid seat |
| assigned_at | timestamp? | | |
| removed_at | timestamp? | | Soft remove |

**Validation**: Unique (user_id, organization_id) when removed_at is null. Seat count ≤ org.seat_limit.

---

## Invite

Organization invite link with lifecycle.

| Field | Type | Constraints | Notes |
|-------|------|-------------|-------|
| id | UUID | PK | |
| token | string | unique, indexed | URL-safe token |
| organization_id | UUID | FK → Organization | |
| created_by | UUID | FK → User | Admin |
| expires_at | timestamp? | | Null = no expiry |
| max_redemptions | integer? | | Null = unlimited |
| redemptions_count | integer | default 0 | |
| status | enum | active, revoked, expired | |
| revoked_at | timestamp? | | |
| revoked_by | UUID? | FK → User | |

**Validation**: status = active only when expires_at is null or future, and (max_redemptions is null or redemptions_count < max_redemptions). Audit events (creation, redemption, revocation) logged per enterprise-orgs-invites.md.

**Concurrency**: Seat assignment and redemption MUST be **concurrency-safe and transactional**. Use database transactions with row-level locking (e.g., `SELECT ... FOR UPDATE` on Invite and Organization) so that concurrent redemption attempts do not exceed max_redemptions or seat_limit.

---

## InviteRedemption

Audit record for each successful invite redemption. Supports audit trail per enterprise-orgs-invites.md.

| Field | Type | Constraints | Notes |
|-------|------|-------------|-------|
| id | UUID | PK | |
| invite_id | UUID | FK → Invite | |
| user_id | UUID | FK → User | |
| redeemed_at | timestamp | | |
| organization_id | UUID | FK → Organization | Denormalized for querying |

**Validation**: Created atomically within the same transaction as seat assignment (OrganizationMembership) and Invite.redemptions_count increment. Enables admin-visible audit trail.

---

## GroupWorkout

Instructor-led session.

| Field | Type | Constraints | Notes |
|-------|------|-------------|-------|
| id | UUID | PK | |
| instructor_id | UUID | FK → User | |
| scheduled_at | timestamp | | Planned start time |
| **started_at** | timestamp? | | **Actual** start time when instructor goes live |
| duration_sec | integer | | |
| plan_snapshot | JSON | | Copied plan for session |
| free_users_allowed | boolean | | |
| status | enum | scheduled, live, ended | |
| ended_at | timestamp? | | |

**Validation**: Phase 2 scope. Completion threshold 50% per nfr-reliability. `started_at` set when status transitions to `live`.

---

## GroupWorkoutParticipant (Phase 2)

Tracks a user's participation in an instructor-led group workout: join, leave, accept-control, and completion state.

| Field | Type | Constraints | Notes |
|-------|------|-------------|-------|
| id | UUID | PK | |
| group_workout_id | UUID | FK → GroupWorkout | |
| user_id | UUID | FK → User | |
| joined_at | timestamp | | |
| left_at | timestamp? | | Null if still in session |
| control_accepted_at | timestamp? | | When user accepted instructor timer control |
| completion_pct | decimal? | | % of planned duration completed (0–100) |
| session_credit_granted | boolean | default false | True if ≥50% completion per nfr-reliability |

**Validation**: Unique (group_workout_id, user_id). completion_pct computed from left_at or ended_at vs. planned duration. session_credit_granted set when completion_pct ≥ 50.

---

## BillingSource (Persisted Inputs for Entitlement)

Persisted records that feed into entitlement resolution. Implementation may use separate tables (e.g., `web_subscriptions`, `app_store_purchases`, `lifetime_grants`) or a polymorphic structure.

| Logical Record | Purpose |
|----------------|---------|
| Web subscription | Provider customer ID, subscription ID, status, current_period_end |
| App-store purchase/receipt | Receipt ref, product_id, platform (ios/android), status, expires_at |
| Lifetime grant | user_id, source (web/app_store), granted_at, no expiry |
| Enterprise seat | OrganizationMembership with seat_assigned = true |

**Note**: Entitlement remains **computed**; these are the persisted inputs. Server resolves effective entitlement from these per billing-entitlements.md.

---

## State Transitions

### WorkoutSession
- **session_status**: `in_progress` → `completed` when user finishes workout; `completed_at`, `duration_sec`, and `stats` set. `in_progress` → `abandoned` when user exits without completing.
- **sync_status** (client-local): `pending` → `synced` when server accepts upload; idempotent on duplicate key. `pending` → `conflict` if server rejects (e.g., validation error).

### Invite
- `active` → `expired`: When expires_at passed or max_redemptions reached.
- `active` → `revoked`: When admin revokes; revoked_at, revoked_by set.

### OrganizationMembership
- `seat_assigned = true` → `false`: When admin unassigns; removed_at set. User retains personal history per data-lifecycle.

### GroupWorkout
- `scheduled` → `live`: When instructor starts; `started_at` set.
- `live` → `ended`: When session ends; `ended_at` set.

---

## Indexes (Recommended)

- User: external_id (unique)
- WorkoutPlan: user_id, created_at
- WorkoutSession: user_id, completed_at; session_status; idempotency_key (unique)
- Achievement: user_id, definition_id (unique)
- OrganizationMembership: (user_id, organization_id) where removed_at is null
- Invite: token (unique), organization_id, status
- InviteRedemption: invite_id, user_id; redeemed_at
- GroupWorkoutParticipant: (group_workout_id, user_id) unique
