# Research: Multi-tenant File Storage Application

## Decision 1: Backend and UI technology baseline
- Decision: Use Go for backend and server-rendered web UI with HTMX, with no additional frontend framework.
- Rationale: The feature spec and constitution explicitly require Go + HTMX and emphasize clean code and minimal dependencies.
- Alternatives considered:
  - React/Vue SPA: rejected due to added dependency and build complexity.
  - Full page reload-only HTML: rejected because HTMX supports progressive enhancement and better UX with minimal JavaScript.

## Decision 2: Storage and persistence architecture
- Decision: Use PostgreSQL for metadata/state and S3-compatible object storage for file binaries (RustFS in local/dev), with strict tenant scoping in all data paths.
- Rationale: Required by spec FR-003, FR-031, FR-042 and constitution architecture standards. Separating metadata from binary blobs supports scale and maintainability.
- Alternatives considered:
  - Store file binaries in PostgreSQL: rejected due to operational cost/performance concerns.
  - Local filesystem storage only: rejected due to multi-tenant and deployment portability requirements.

## Decision 3: Tenant isolation enforcement pattern
- Decision: Enforce tenant boundaries at both service and repository layers using explicit tenant ID in method signatures and SQL predicates, plus authorization checks in handlers.
- Rationale: Defense-in-depth aligns with FR-001/FR-016 and constitution security requirements.
- Alternatives considered:
  - Handler-only tenant checks: rejected as too easy to bypass in internal code paths.
  - DB row-level security only: not selected as sole control to avoid configuration coupling and to preserve explicit domain logic.

## Decision 4: Group hierarchy cycle prevention
- Decision: Implement subgroup cycle prevention using a closure table (`group_closure`) updated transactionally on group relation changes.
- Rationale: Explicit requirement FR-051; closure table gives efficient cycle checks and ancestry queries.
- Alternatives considered:
  - Recursive traversal per write: rejected for poorer write-time predictability and higher query complexity.
  - Materialized path: rejected due to update complexity for re-parent operations.

## Decision 5: External one-time share key handling
- Decision: Store only salted hash + key fingerprint/identifier for external keys; enforce single-use and configurable expiry (default 5 days).
- Rationale: Required by entity security notes and FR-052/FR-053; limits impact of datastore/log exposure.
- Alternatives considered:
  - Store raw token in DB: rejected for security risk.
  - Permanent links: rejected due to single-use requirement.

## Decision 6: Upload reliability and retry behavior
- Decision: Use resumable/multipart-aware upload workflow with retry orchestration using exponential backoff for transient S3 failures (e.g., 30s, 60s, 120s), surfacing retry status in UI.
- Rationale: Required by FR-048 and user story 5; improves success rate while preserving user control.
- Alternatives considered:
  - Immediate fail without retry: rejected due to reliability requirements.
  - Infinite retry loop: rejected to avoid unbounded resource usage and poor UX.

## Decision 7: Configuration layering model
- Decision: Implement three-layer configuration exactly as specified: env vars (bootstrap/pre-DB) → database `PlatformConfiguration` (runtime authoritative) → docker-compose seed file (first-boot defaults).
- Rationale: Required by FR-049 and assumptions; keeps runtime updates centralized and auditable.
- Alternatives considered:
  - Env-only runtime config: rejected because operator updates would require restarts.
  - File-only runtime config: rejected because multi-instance consistency and auditability would be weak.

## Decision 8: Authentication and outage continuity
- Decision: Integrate tenant authentication through Authelia OIDC with a circuit breaker around provider health; reject new logins during outage but allow active sessions to continue/extend within TTL constraints.
- Rationale: Required by FR-002 and FR-050.
- Alternatives considered:
  - Hard-fail all sessions on provider outage: rejected due to continuity requirement.
  - Enable local auth for tenant users: rejected because only superuser local auth is allowed (FR-044).

## Decision 9: Testing strategy and thresholds
- Decision: Adopt testing pyramid with unit tests first and explicit integration/e2e coverage for all critical boundaries and journeys; target >=70% unit coverage.
- Rationale: Constitution Principle V and spec testing strategy are explicit.
- Alternatives considered:
  - E2E-heavy strategy: rejected due to speed/maintenance cost.
  - Unit-only strategy: rejected due to cross-service integration risks.

## Decision 10: Dependency policy for v1
- Decision: Keep dependencies minimal by preferring Go standard library and existing ecosystem primitives; any new library must be justified in plan/tasks with security and maintenance review.
- Rationale: Constitution Principle IV and user constraint (“keep dependencies minimal”).
- Alternatives considered:
  - Introduce additional framework abstractions early: rejected as premature complexity.
