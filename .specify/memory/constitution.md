<!--
Sync Impact Report
Version change: N/A → 1.0.0
Modified principles:
- [new] I. Code Quality & Maintainability
- [new] II. Testing Discipline & Coverage (NON-NEGOTIABLE)
- [new] III. User Experience Consistency
- [new] IV. Performance & Responsiveness
- [new] V. Operational Observability
Added sections:
- Non-Functional Standards
- Development Workflow & Quality Gates
Removed sections:
- None (template placeholders replaced with concrete content)
Templates requiring updates:
- .specify/templates/plan-template.md ✅ updated
- .specify/templates/spec-template.md ✅ aligned (no changes needed)
- .specify/templates/tasks-template.md ✅ updated
- README.md ⚠ pending (create or update to reference core principles)
- docs/quickstart.md ⚠ pending (create or update to reference core principles)
Follow-up TODOs:
- TODO(RATIFICATION_DATE): Set original ratification date once team agrees; currently unknown.
-->

# HIITsergeant Constitution

## Core Principles

### I. Code Quality & Maintainability

- All production code MUST be readable, modular, and consistently styled using the
  repository-standard tooling.
- Code MUST pass automated linting and formatting checks with no disabled rules
  unless explicitly documented and justified in the implementation plan.
- Public interfaces (APIs, CLI contracts, UI components) MUST be clearly named and
  documented at the same time they are implemented.
- Architectural decisions that introduce additional layers or abstractions MUST be
  recorded in the plan with at least one simpler alternative that was rejected.

Rationale: High-quality, maintainable code keeps HIITsergeant reliable as the
product grows and reduces the long-term cost of change.

### II. Testing Discipline & Coverage (NON-NEGOTIABLE)

- Every change that can break behavior MUST be covered by automated tests (unit,
  integration, end-to-end, or contract tests as appropriate).
- Tests for new behavior MUST be written before or alongside implementation and
  MUST fail before code changes are applied.
- Critical paths (e.g., workout creation, session timing, progress persistence,
  payments if present) MUST have automated regression tests that run in CI.
- Continuous integration MUST block merges when tests fail or when tests are
  missing for new behavior that can reasonably be tested.

Rationale: Strong, enforced testing discipline is the primary guardrail that
prevents regressions in the workout experience and preserves user trust.

### III. User Experience Consistency

- UX changes MUST follow the shared design language for typography, spacing,
  color, and interaction patterns defined for HIITsergeant.
- User flows for core actions (creating a workout, starting a session, pausing,
  resuming, and completing) MUST remain predictable and consistent across
  supported platforms.
- Copy, error messaging, and visual states (loading, success, error, empty) MUST
  be consistent for equivalent states across the product.
- Any intentional breaking change to user experience MUST be captured in the
  specification with clear "before vs. after" behavior and rationale.

Rationale: Consistent, predictable experiences help users build confidence and
reduce cognitive load during high-intensity workouts.

### IV. Performance & Responsiveness

- Core interactions (screen transitions, starting/pausing workouts, logging
  results) MUST feel instantaneous; target perceived latency of under 200ms on
  modern devices and connections whenever feasible.
- Heavy computations (e.g., complex workout generation, analytics) MUST be
  batched, cached, or offloaded off the main UI thread where the platform
  supports it.
- Network calls MUST be minimized and batched where practical; unnecessary
  polling or chatty APIs are forbidden for performance-critical flows.
- Performance regressions (latency, frame rate, memory usage) MUST be detected
  via profiling or telemetry for high-traffic flows before release.

Rationale: HIITsergeant is used in time-sensitive, physically demanding
contexts; sluggish performance directly harms usability and user safety.

### V. Operational Observability

- Key workflows (workout creation, session execution, data sync, payments if
  present) MUST emit structured logs or metrics sufficient to debug failures and
  performance issues.
- Errors MUST be captured with enough non-PII context (e.g., workout identifier,
  platform, app version) to make issues reproducible.
- Monitoring dashboards or reports SHOULD exist for core KPIs such as workout
  start/completion rates, error rates, and latency for key endpoints or screens.
- Any production incident post-mortem MUST identify missing or misleading
  observability and add or adjust logging, metrics, or alerts accordingly.

Rationale: Without observability, code quality and performance issues remain
invisible until users are impacted; observability turns the principles above
into enforceable, trackable behavior.

## Non-Functional Standards

This section defines baseline expectations for non-functional behavior across the
HIITsergeant codebase.

- **Code quality**: New modules MUST integrate with repository-standard linting,
  formatting, and static analysis. "TODO" or "FIXME" comments in production
  paths MUST be accompanied by tasks in the plan or tasks file.
- **Testing standards**: For behavior that can break, there MUST be at least one
  automated test at an appropriate level. High-risk or business-critical flows
  MUST prefer contract and integration tests in addition to unit tests.
- **User experience**: Feature specifications MUST describe the target user
  journey, including error and edge cases, and MUST reference existing patterns
  where possible instead of inventing new ones.
- **Performance requirements**: Plans MUST capture performance goals (e.g.,
  latency, frame rate, throughput) for relevant features and identify how these
  goals will be validated (profiling, synthetic tests, or runtime telemetry).

## Development Workflow & Quality Gates

- Before Phase 0 research, the implementation plan MUST include an explicit
  "Constitution Check" showing how the design satisfies:
  - Code quality and maintainability (structure, ownership, and tooling)
  - Testing discipline (test types, coverage focus, and CI enforcement)
  - User experience consistency (alignment with existing patterns)
  - Performance requirements (targets and validation strategy)
- During Phase 1 design, the spec and data model MUST be updated if the
  Constitution Check reveals gaps in UX, testing, or performance coverage.
- Every pull request MUST:
  - Link to the relevant plan/spec where the Constitution Check is documented
  - Include or update automated tests for changed behavior
  - Pass linting, tests, and any configured performance or bundle-size checks
  - Call out any intentional deviations from this constitution in the PR
    description with a plan to return to compliance.

## Governance

- This constitution supersedes informal or undocumented practices when there is
  a conflict. Where it is silent, maintainers MAY exercise judgment but SHOULD
  favor consistency with existing patterns.
- Amendments to this constitution MUST be made via a pull request that:
  - Edits this file, including the Sync Impact Report and version line
  - Explains the motivation and impact of the change
  - Identifies any required updates to templates, tooling, or documentation.
- Versioning uses semantic rules:
  - **MAJOR**: Backward-incompatible governance changes or removal/redefinition
    of principles.
  - **MINOR**: Addition of new principles or material expansion of guidance.
  - **PATCH**: Clarifications, wording refinements, or typo fixes that do not
    change expectations.
- Compliance with this constitution MUST be reviewed during:
  - Feature planning (via the Constitution Check in the plan)
  - Code review (via PR checklists and automated gates)
  - Post-incident reviews (to ensure principles and observability remain
    adequate).

**Version**: 1.0.0 | **Ratified**: TODO(RATIFICATION_DATE): original adoption
date unknown; set when team agrees. | **Last Amended**: 2026-02-24
