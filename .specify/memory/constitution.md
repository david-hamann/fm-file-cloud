<!--
Sync Impact Report
- Version change: 2.0.1 → 2.0.2
- Modified principles:
  - II. Accessibility and Usability by Default → II. Accessibility and Usability by Default (normalized SHOULD/MUST wording and removed duplicate mobile-first phrasing)
  - V. Comprehensive Testing Strategy → V. Comprehensive Testing Strategy (normalized SHOULD wording for testing pyramid guidance)
  - VII. Interface-Driven Design and Dependency Injection in Go → VII. Interface-Driven Design and Dependency Injection in Go (corrected HTTP client interface guidance and standard-library error wrapping guidance)
- Added sections:
  - Technical Standards heading restored before architecture standards
- Removed sections:
  - None
- Templates requiring updates:
  - ✅ none required (constitution-only consistency clarification)
- Follow-up TODOs:
  - Initialize `.specify/memory/exceptions.md` for tracking governance exceptions
  - Document Go interface examples and anti-patterns in a guide
  - Confirm license audit tool (e.g., `go install github.com/mitchellh/golicense`)
-->

# FM File Cloud Constitution

## Core Principles

### I. Clean Code Is Mandatory
Code MUST be readable, small in scope, and structured for maintainability. Functions and methods MUST
have a single clear responsibility (prefer single-responsibility principle), naming MUST clearly describe
intent and behavior, and duplicated logic MUST be eliminated or consolidated. Acceptable complexity
MUST be justified in comments or planning artifacts; complex algorithms MUST include explanation of
approach.
Rationale: maintainable code reduces defects, shortens review cycles, improves team onboarding, and lowers long-term cost.

### II. Accessibility and Usability by Default
User-facing features MUST prioritize clarity, inclusive design, and accessibility. Workflows MUST minimize
steps and copy MUST be clear; design decisions SHOULD be validated against documented user needs. All web
UI work MUST meet WCAG 2.1 Level AA accessibility standards (keyboard navigation, screen reader support,
color contrast). Features SHOULD be tested against representative user journeys before merge. Default
behavior MUST favor clarity and predictable interaction over feature density.
Rationale: accessible, simple interfaces improve task completion, reduce support burden, and reach all user
capabilities.

### III. Responsive and Mobile-First Design
Web UI work MUST follow mobile-first design: implement core functionality for mobile viewports first,
then enhance for tablet and desktop. Layouts MUST adapt across mobile (320px+), tablet (768px+), and
desktop (1024px+) breakpoints without loss of core functionality. All interactive elements MUST remain
accessible at all viewport sizes. Testing MUST include device simulation and real device validation for
critical user tasks.
Rationale: mobile-first approach ensures efficiency; responsive coverage is essential for reliable
usability across global device distribution; real device testing catches layout bugs that simulators miss.

### IV. Minimal and Auditable Dependencies
Teams MUST prefer standard library or existing project capabilities before adding external dependencies.
Every new dependency MUST include: (1) business justification, (2) maintenance status and update cadence,
(3) security review (known CVEs, maintainer trustworthiness), and (4) simpler alternative analysis.
Dependencies MUST be pinned to specific versions in lock files. Unused dependencies MUST be removed
quarterly. All Go modules MUST have license compatibility audit completed before use.
Rationale: minimizing dependencies reduces attack surface, improves build times, lowers operational risk,
and reduces future maintenance burden.

### V. Comprehensive Testing Strategy
Unit tests MUST cover all business logic and critical paths; target minimum 70% code coverage. Tests MUST
pass before merge. Integration tests and end-to-end tests MUST be explicitly evaluated for cross-service,
database, API-contract, and critical-journey behavior; if omitted, rationale MUST be documented.
Test categorization SHOULD follow the testing pyramid: majority unit tests, fewer integration tests, minimal
e2e tests. Code review MUST verify test adequacy and that tests fail without the implementation.
Rationale: unit tests provide fast correctness feedback; pyramid strategy scales testing efficiency;
integration/e2e coverage protects system boundaries and user-critical flows.

### VI. Atomic Task Branches, Commits, Reviews, and Pull Requests
Every implementation task (T001, T002, etc.) MUST be executed in its own task branch. A task branch
MUST contain changes for exactly one task and MUST not bundle unrelated task or story work. Task branch
names MUST follow `[task-id]-[short-description]`.

Each completed task MUST produce one task-specific commit or a minimal series of task-specific commits
that together cover only that task. Batch commits spanning unrelated tasks are not allowed. Commit
messages MUST follow `.specify/templates/commit-message-template.md`.

Every completed task branch MUST result in a pull request using GitHub CLI `gh`. Each PR MUST map to
exactly one task and MUST include test results, related issue/task IDs, and a summary of what changed
and why. Code review MUST verify compliance with principles before approval.

If a task is abandoned or substantially re-scoped, the branch MUST include a final explanatory commit
documenting the reasoning before closure.

Rationale: atomic task branches and task-specific PRs improve traceability, enable efficient review,
support safe rollbacks, and keep planning, implementation, and review aligned.

### VII. Interface-Driven Design and Dependency Injection in Go
In Go backend and web UI code, external dependencies (storage, HTTP clients, service calls) MUST be defined
and injected as interfaces at the point of use, never as concrete types. Functions and methods MUST accept
interface parameters rather than concrete struct pointers. Prefer small, focused interfaces (e.g., `Reader`)
over large umbrella interfaces.

Common patterns that MUST use interfaces:
- Database access: inject `db.Querier` interface, not `*sql.DB`
- HTTP clients: inject a narrow locally defined interface (for example, an `HTTPDoer` with `Do(*http.Request)`), not `*http.Client` directly
- External services: define narrow service interfaces for each dependency
- Repository/data access: define repository interfaces to enable mock implementations
- Error handling: return `error` values with wrapped context and compare typed or sentinel errors only where needed

Every unit test MUST be able to mock all external dependencies via interface substitution without
modifying non-test code. Integration tests MAY use real implementations. Use structured logging (e.g.,
zap, logrus) to enable testable logging with dependency injection.

Rationale: interface-driven design enables isolated testing, simplifies business logic validation, reduces
coupling, and supports the testing pyramid. This principle directly enables Principle V (Comprehensive
Testing).

### VIII. Security and Error Handling by Default
All services MUST validate all input (user input, API requests, file uploads) and sanitize output to
prevent injection attacks. Error messages MUST never expose internal details (stack traces, database
schemas, system paths) to end users; log details server-side only. All secrets (API keys, database
credentials, tokens) MUST be injected via environment variables or secure vaults, never hardcoded or
committed. Communication with PostgreSQL and external services MUST use encrypted connections (TLS).

Go code MUST use explicit error handling; avoid ignore-the-error patterns. Return meaningful errors with
context (wrap errors with additional context using `fmt.Errorf` with `%w` or equivalent standard-library patterns). Unit tests MUST verify
error behavior for critical paths.

All code changes MUST be reviewed for security implications before approval. Libraries with known CVEs
MUST not be used; outdated dependencies MUST be updated promptly.

Rationale: security-first practices prevent common vulnerabilities; proper error handling improves debugging
and reduces downtime; explicit error management supports testability and reliability.

## Technical Standards

Architecture MUST follow these language-specific practices and principle constraints:

### Go Backend
- MUST use Go with interface-driven dependency injection (Principle VII).
- MUST secure all inputs, log safely, and handle errors explicitly (Principle VIII).
- Persistent state for user, group, and sharing models MUST use PostgreSQL with encrypted connections.
- MUST target 70%+ unit test coverage; use table-driven tests where appropriate.
- MUST use structured logging (zap, logrus) injected as interfaces.
- Configuration and secrets MUST be environment-based; use 12-factor app principles.

### Web UI (Go with HTMX)
- MUST follow mobile-first, responsive design with WCAG 2.1 Level AA accessibility (Principle II+III).
- MUST use interface-driven service layers (Principle VII) injected from handlers.
- MUST validate and sanitize all form inputs server-side (Principle VIII).
- MUST include progressive enhancement to work without JavaScript where reasonable.
- Error pages and user-facing messages MUST not expose system details (Principle VIII).
- MUST be tested with real devices and screen readers for critical flows.

### Android Application
- MUST be implemented natively using platform conventions and best practices.
- MUST meet platform accessibility guidelines (Android Accessibility Suite).
- MUST follow secure coding practices for mobile (input validation, secure storage, permissions).
- MUST use dependency injection for testability where applicable.
- MUST include offline-first capabilities or clear user communication about connectivity requirements.

### PostgreSQL
- Schemas MUST be versioned and managed via migrations tooling.
- MUST use prepared statements and parameterized queries exclusively (no string concatenation).
- MUST implement connection pooling and monitor query performance.
- MUST use role-based access control; application connectors MUST have minimal required permissions.
- MUST have automated backups and disaster recovery procedures documented.

All architecture and implementation plans MUST reflect these standards and principle enforcement before execution begins.

## Development Workflow & Review

- Every feature specification MUST include independent user stories, explicit test strategy, and success
  criteria.
- Every implementation plan MUST pass the Constitution Check gates before Phase 0 research is
  considered complete.
- Task breakdown MUST prioritize unit tests and document integration/e2e inclusion or omission.
- Task planning and execution MUST map one task to one branch named `[task-id]-[short-description]`, one
  task-specific commit set, and one task-specific PR created via `gh`.
- Code review MUST verify compliance with all core principles and technical standards prior to approval.
- Any justified exception MUST be recorded in the relevant plan and referenced in pull request review.

## Governance

### Authority & Precedence
This constitution is the authoritative source for engineering and delivery standards in this repository.
When conflicts occur, this constitution takes precedence over local conventions.

### Amendment Process
All amendments MUST be proposed in writing with impact analysis. Amendments follow semantic versioning:

- **PATCH** (wording, typos, clarifications): Can be approved and merged by any maintainer.
- **MINOR** (new principle, expanded guidance, new technical standards): MUST be reviewed by team leads and proposed 5 days before merge to allow feedback.
- **MAJOR** (backward-incompatible principle changes): MUST go to full team consensus before adoption.

All amendments MUST update dependent templates (plan, spec, tasks) in the same change set.

### Exceptions & Justified Violations
If a MUST requirement cannot be met, the exception MUST be:
1. **Documented in the plan/spec** with specific rationale (e.g., "Unit tests omitted because [reason]; risk/cost analysis: [details]").
2. **Flagged in code review** with explicit PR comment linking to the documented exception.
3. **Recorded in a project exceptions log** (e.g., `.specify/memory/exceptions.md`) for periodic review.

Exceptions are temporary and MUST include a proposed remediation plan or sunset date.

### Compliance & Automation
- Planning artifacts MUST include constitution gate verification as part of the specification template.
- Code reviewers MUST block merges that violate mandatory MUST requirements unless an exception is documented and approved.
- Constitution Check gates SHOULD be automated (via linting, CI checks, or code reviews) where feasible to reduce manual burden.
- Quarterly reviews MUST inspect the exceptions log and evaluate whether new principles or refinements are needed.
- Major releases (e.g., launch or significant feature completion) SHOULD trigger a constitution review to ensure ongoing relevance.

**Version**: 2.0.2 | **Ratified**: 2026-03-24 | **Last Amended**: 2026-03-24
