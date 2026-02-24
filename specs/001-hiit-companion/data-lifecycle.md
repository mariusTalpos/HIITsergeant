# Data Retention, Deletion & Ownership Rules

This document defines lifecycle rules for user data so account deletion,
membership changes, and history ownership are unambiguous. It complements the
privacy posture in `feature-spec.md` and `enterprise-orgs-invites.md`.

## Workout History

- Users MUST be able to delete individual workout sessions from their history.
- Deleted sessions MUST be removed from the user’s view and from statistics
  (e.g., streaks, totals) in a consistent way.
- The system MAY retain deleted sessions for a limited period for operational
  recovery (e.g., undo) or legal requirements; such retention MUST be
  documented and MUST NOT expose data to the user or third parties after
  deletion.

## Account Deletion

- Users MUST be able to delete their account and associated personal data.
- Account deletion is subject to legal and operational retention constraints
  (e.g., tax records, dispute resolution, fraud prevention). Where retention is
  required, data MUST be anonymized or minimized to the extent permitted by law.
- After account deletion, the user MUST no longer have access to their data, and
  the system MUST NOT use their data for active features.

## Enterprise Membership Changes

- Removing a user from an organization (admin unassigns seat or membership is
  revoked) revokes enterprise entitlement but does **not** delete their personal
  workout history.
- Personal workout history (solo and group sessions) is owned by the user, not
  the organization.
- Organization admins do **not** own or inherit personal workout history by
  default. They MUST NOT gain access to a user’s workout data when the user is
  removed from the organization.

## Invite Logs & Audit Events

- The system MUST log invite creation, redemption, and revocation per
  `enterprise-orgs-invites.md` (Audit Trail).
- Retention for invite logs: retain for a period sufficient for support and
  debugging
  (e.g., 90 days to 1 year), then delete or anonymize, unless longer retention
  is required by law or security policy.
- Audit logs for entitlement changes (see `billing-entitlements.md`) follow the
  same retention principles.

## Shared Achievement Artifacts

- When a user shares an achievement to an external platform (e.g., social
  media), the shared artifact (e.g., image, formatted text) may be cached or
  stored by that platform; HIITsergeant does not control that.
- HIITsergeant MUST NOT retain copies of shared achievement artifacts beyond
  what is needed to generate the share (e.g., transient generation only).
- After account deletion, any achievement data used for shares MUST be deleted
  or anonymized. Shared artifacts already posted to external platforms are
  outside HIITsergeant’s control and cannot be recalled.
