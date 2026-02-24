# Billing & Entitlements Specification

This document details billing flows and entitlements for HIITsergeant. It
expands on functional requirements **FR-005 – FR-005e** and the `Entitlement`
entity from `feature-spec.md`.

## Tiers & Products

- **Free tier**: Basic access to personal workouts and limited features.
- **Personal premium**:
  - Subscription (recurring) and/or lifetime purchase options.
  - Unlocks premium features such as leading group workouts (if applicable) and
    advanced analytics.
- **Enterprise plans**:
  - Sold to organizations (gyms, companies) with multiple seats.
  - Provide premium entitlements to members while seats are active.

## Payments (Web)

- All web payments MUST use a third-party payment provider.
- HIITsergeant MUST NOT store raw credit card details.
- HIITsergeant MAY store:
  - Provider customer reference/ID
  - Subscription or transaction identifiers
  - Derived entitlement status (active/inactive, expiry date)

## Payments (Mobile)

- All in-app purchases on mobile MUST use the platform app store billing
  mechanism.
- HIITsergeant MUST NOT store raw credit card details.
- HIITsergeant MAY store:
  - App-store receipt or verification reference
  - Derived entitlement status (active/inactive, expiry date)

## Entitlement Model

- Entitlements represent the effective access level for a user:
  - Free
  - Personal premium subscription
  - Personal lifetime premium
  - Enterprise premium (via organization membership)
- Entitlements are derived from:
  - Web payment provider data
  - Mobile app-store receipts
  - Enterprise organization seat assignments

## Entitlement Resolution Rules

These rules prevent inconsistent premium access across web/mobile and reduce
support load from entitlement bugs.

### Entitlement Sources

- **Web subscription**: Recurring subscription purchased via web payment provider.
- **App-store subscription**: Recurring subscription purchased via platform app store.
- **Lifetime purchase**: One-time purchase (web or app store) granting permanent premium.
- **Enterprise seat**: Active membership in an organization with a paid seat assigned.

### Server-Side Computation

- The **effective entitlement** for a user MUST be computed **server-side** as a
  single, authoritative value.
- Clients MUST NOT determine premium access locally; they MUST rely on the
  server’s entitlement response for gating premium features.

### Precedence & Conflict Resolution

When multiple entitlements are active for a user, the effective entitlement
MUST be resolved as follows:

1. **Lifetime premium** takes precedence over all other sources (user has
   permanent access regardless of subscription or enterprise status).
2. **Active personal subscription** (web or app store) grants premium; if both
   exist, either is sufficient.
3. **Active enterprise seat** grants premium while the seat is assigned.
4. If no source is active, the user is **free**.

### Enterprise Seat Removed, Personal Subscription Exists

- When an enterprise seat is removed (admin unassigns or seat limit reduced):
  - If the user has an active personal subscription or lifetime purchase, they
    MUST retain premium access.
  - The effective entitlement MUST switch to the personal source without
    requiring user action.

### Subscription Expires, Other Valid Entitlement Exists

- When a subscription expires (web or app store):
  - If the user has another valid entitlement (e.g., app-store subscription
    when web expired, or enterprise seat, or lifetime), they MUST retain
    premium access.
  - The effective entitlement MUST switch to the remaining valid source.

### Refresh Timing & Cache Invalidation

- Entitlement MUST be re-evaluated:
  - On each sign-in or session start.
  - When the user explicitly triggers a refresh (e.g., “Restore purchases”).
  - When the server receives a webhook or event indicating a subscription or
    seat change.
- Clients MAY cache the entitlement for a short period (e.g., up to 5 minutes)
  for performance; cache MUST be invalidated on the above events.
- Stale entitlement MUST NOT block access for longer than the cache duration;
  after that, a fresh server check is required.

### Temporary Provider / Receipt Verification Outages

- When the payment provider or app-store receipt verification service is
  temporarily unavailable:
  - The system SHOULD allow **graceful degradation**: if the user had premium
    within the last N hours (e.g., 24–72), treat them as premium until
    verification can be retried.
  - After the grace period or when verification succeeds, the effective
    entitlement MUST be updated to the true state.
  - Users MUST NOT be charged twice; verification failures MUST NOT trigger
    duplicate billing.

### Audit Logs

- Entitlement changes (e.g., subscription activated, seat assigned, subscription
  expired, seat removed) MUST be logged for support and debugging.
- Logs MUST include: user identifier, timestamp, previous entitlement, new
  entitlement, and source of change (e.g., webhook, admin action, manual refresh).
- Logs MUST NOT include raw payment data or card details.

## Enterprise Billing

- Each `Organization` has:
  - Billing information (provider customer or app-store contract references;
    never raw card data)
  - A configured number of paid seats
  - One or more administrators who can:
    - Manage seats (assign/unassign members)
    - View organization-level billing status
    - Generate member invite links / QR codes

## Privacy Guardrails

- Enterprise administrators MUST NOT see individual user fitness profiles,
  detailed workout histories, or health-related profile fields by default.
- Organization-level reporting (if added later) MUST aggregate data or
  anonymize individuals in line with privacy posture and compliance goals.
