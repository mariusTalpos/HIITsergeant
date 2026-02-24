# Enterprise Organizations & Invites Specification

This document describes how gyms/companies (enterprise organizations) interact
with HIITsergeant and how users join those organizations. It expands on
enterprise-related requirements from `feature-spec.md`.

## Organization Model

- **Organization**:
  - Represents a gym/company account.
  - Has non-card billing information (provider/customer references only).
  - Has one or more administrators.
  - Has a configured seat limit.
- **Organization Membership**:
  - Represents a user’s membership in an organization.
  - Tracks whether the user currently occupies a paid seat.
  - Includes role: member vs. admin.

## Admin Capabilities

- Add or remove organization members by assigning/unassigning seats.
- Manage organization-level billing (view plan, status, and seat usage).
- Generate invite links that can be:
  - Shared directly (e.g., via message or email).
  - Rendered as scannable QR codes for display in physical spaces (e.g., gym
    posters, studio TVs).
- Revoke invite links immediately; revoked invites MUST stop accepting new
  redemptions.

## Invite Lifecycle

Invite links MAY be configured with the following constraints:

- **Time-limited**: Admin MAY set an expiration date/time; after expiry, the
  invite MUST NOT accept new redemptions.
- **One-time-use or multi-use**: Admin MAY create invites that accept a single
  redemption (one-time-use) or multiple redemptions (multi-use) until other
  limits apply.
- **Seat-limited**: Invite MAY be constrained to a maximum number of seats it
  can consume; once reached, the invite MUST NOT accept further redemptions.

Defaults (e.g., multi-use, no expiry, org seat limit) MAY be used when admin
does not specify constraints.

## Member Onboarding Flow

1. Admin generates an invite link (and optionally QR code) for the organization.
2. User visits the link or scans the QR code.
3. User is prompted to sign in (passwordless) or confirm existing account.
4. User explicitly accepts the invite:
   - If a seat is available, they are added as a member and consume a seat.
   - If no seats are available, the user is informed and not added, or placed
     into a pending state depending on future decisions.

## Privacy & Data Access

- Enterprise administrators:
  - MAY see high-level aggregated metrics about organization usage (if added in
    future analytics work).
  - MUST NOT see individual user fitness profiles or detailed workout histories
    by default.
  - Do NOT own or inherit personal workout history. See `data-lifecycle.md` for
    data retention, deletion, and ownership rules.

## Membership Removal

- When a user is removed from an organization (seat unassigned or membership
  revoked), enterprise entitlement is revoked. The user’s personal workout
  history is NOT deleted. See `data-lifecycle.md`.

## Expired or Revoked Invite UX

- When a user accesses an expired or revoked invite, the system MUST display a
  **clear reason** (e.g., "This invite has expired" or "This invite has been
  revoked by an administrator").
- The UX MUST provide **next steps** (e.g., "Contact your gym administrator for
  a new invite" or "Ask your organization admin to generate a new invite link").
- No seat MUST be consumed when an invite is expired or revoked.

## Audit Trail

- The system MUST log the following invite events:
  - **Invite creation**: Who created the invite, when, and any constraints (e.g.,
    expiry, one-time-use, seat limit).
  - **Invite redemption**: Who accepted the invite, when, and which invite was used.
  - **Invite revocation**: Who revoked the invite, when, and which invite was revoked.
- These logs MUST be **admin-visible** (organization administrators can view the
  audit trail for their organization).
- Retention of invite logs follows `data-lifecycle.md` (Invite Logs & Audit Events).

## Edge Cases

- **Publicly shared or leaked invite link/QR**:
  - The system MUST allow administrators to revoke or regenerate invite links,
    invalidating old QR codes.
  - Users accessing a revoked or expired invite MUST see a clear message and no
    seat should be consumed.

