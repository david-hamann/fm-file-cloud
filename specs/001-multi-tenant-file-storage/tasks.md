# Tasks: Multi-tenant File Storage Application

**Input**: Design documents from `/specs/001-multi-tenant-file-storage/`
**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`, `contracts/http-api.md`

**Tests**: Unit test tasks are REQUIRED for business logic and are prioritized first in each story. Integration and end-to-end tests are explicitly included for each story.

**Organization**: Tasks are grouped by user story to preserve independent implementation, testing, and delivery.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: User story mapping (US1..US26)
- Each task includes concrete file paths
- One task branch + one structured commit + one PR (`gh`) per completed task

## Path Conventions (selected)

- Backend API/service/domain: `backend/cmd/api/`, `backend/internal/{auth,tenant,group,file,share,operator,platform}/`
- Web UI (Go templates + HTMX): `web/templates/`, `web/static/`
- Migrations/config: `migrations/`, `configs/`, `deploy/compose/`
- Tests: `tests/unit/`, `tests/integration/`, `tests/e2e/`

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Initialize repository structure, quality gates, and dependency guardrails.

- [ ] T001 Create base directories in `backend/`, `web/`, `tests/`, `migrations/`, `configs/`, `deploy/compose/`
- [ ] T002 Initialize Go modules and service entrypoint in `backend/go.mod` and `backend/cmd/api/main.go`
- [ ] T003 [P] Configure lint/format/test commands in `Makefile`
- [ ] T004 [P] Add baseline config templates in `configs/app.example.env` and `configs/platform.seed.yaml`
- [ ] T005 [P] Add dependency policy and approval checklist in `docs/dependency-policy.md`
- [ ] T006 Define branch/commit/PR workflow in `docs/delivery-workflow.md`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Build common foundations required by all user stories.

**⚠️ CRITICAL**: No user story work begins until this phase is complete.

- [ ] T007 Create initial schema migrations for core entities in `migrations/0001_initial.sql`
- [ ] T008 [P] Add migration runner and startup checks in `backend/internal/platform/migrate.go`
- [ ] T009 [P] Implement DB access interfaces and repositories in `backend/internal/platform/repository/`
- [ ] T010 [P] Implement shared auth middleware skeleton in `backend/internal/auth/middleware.go`
- [ ] T011 Implement tenant context extraction/propagation in `backend/internal/tenant/context.go`
- [ ] T012 [P] Implement centralized error model and safe user responses in `backend/internal/platform/errors.go`
- [ ] T013 [P] Implement structured logging interfaces and adapters in `backend/internal/platform/logging.go`
- [ ] T014 Implement configuration layering loader (env -> db -> seed) in `backend/internal/platform/config_loader.go`
- [ ] T015 [P] Wire HTTP router and route groups in `backend/internal/platform/http/router.go`
- [ ] T016 [P] Foundational integration test (startup + DB + config seed) in `tests/integration/foundation_startup_test.go`

**Checkpoint**: Foundation complete; user stories can proceed.

---

## Phase 3: User Story 1 - Tenant Created on First Login (Priority: P1) 🎯 MVP

**Goal**: First-time OIDC login creates isolated tenant with admin membership and default group.

**Independent Test**: First login creates tenant + admin + default group; auth outage returns clear unavailable message.

- [ ] T017 [P] [US1] Unit tests for first-login tenant bootstrap logic in `tests/unit/tenant_first_login_test.go`
- [ ] T018 [P] [US1] Integration test for OIDC callback + tenant creation in `tests/integration/tenant_first_login_test.go`
- [ ] T019 [P] [US1] E2E test for first-login tenant creation journey in `tests/e2e/tenant_first_login.spec.ts`
- [ ] T020 [US1] Implement first-login tenant bootstrap service in `backend/internal/tenant/service_first_login.go`
- [ ] T021 [US1] Implement OIDC callback and failure UX path in `backend/internal/auth/handler_oidc.go` and `web/templates/auth/unavailable.html`

---

## Phase 4: User Story 2 - Tenant Invite and Join (Priority: P1)

**Goal**: Admin invites user by email; invited user joins tenant after acceptance.

**Independent Test**: Invite is issued and accepted user gains tenant access.

- [ ] T022 [P] [US2] Unit tests for invite lifecycle in `tests/unit/invite_service_test.go`
- [ ] T023 [P] [US2] Integration test for invite issue/accept flow in `tests/integration/invite_flow_test.go`
- [ ] T024 [P] [US2] E2E test for admin invite + user join in `tests/e2e/invite_join.spec.ts`
- [ ] T025 [US2] Implement invite domain/service in `backend/internal/tenant/service_invite.go`
- [ ] T026 [US2] Implement invite endpoints and HTMX forms in `backend/internal/tenant/handler_invite.go` and `web/templates/tenant/invite_form.html`

---

## Phase 5: User Story 3 - Cross-Tenant Isolation Enforcement (Priority: P1)

**Goal**: Deny access to resources outside requestor tenant unless explicitly shared.

**Independent Test**: Direct access to another tenant resource is rejected.

- [ ] T027 [P] [US3] Unit tests for tenant authorization guard in `tests/unit/tenant_authz_guard_test.go`
- [ ] T028 [P] [US3] Integration test for cross-tenant denial in `tests/integration/cross_tenant_denial_test.go`
- [ ] T029 [P] [US3] E2E test for unauthorized cross-tenant URL access in `tests/e2e/cross_tenant_isolation.spec.ts`
- [ ] T030 [US3] Implement tenant-scoped authorization policy in `backend/internal/auth/policy_tenant_scope.go`

---

## Phase 6: User Story 4 - Basic File Upload (Priority: P1)

**Goal**: Upload files to S3-compatible storage and persist metadata.

**Independent Test**: Uploaded file appears in list with filename/size/uploader/timestamp.

- [ ] T031 [P] [US4] Unit tests for file metadata creation in `tests/unit/file_upload_metadata_test.go`
- [ ] T032 [P] [US4] Integration test for upload + metadata persistence in `tests/integration/file_upload_roundtrip_test.go`
- [ ] T033 [P] [US4] E2E test for upload + list display in `tests/e2e/file_upload_basic.spec.ts`
- [ ] T034 [US4] Implement upload service (storage + metadata) in `backend/internal/file/service_upload.go`
- [ ] T035 [US4] Implement upload/list handlers + templates in `backend/internal/file/handler_upload.go` and `web/templates/files/list.html`

---

## Phase 7: User Story 5 - Upload Reliability Controls (Priority: P1)

**Goal**: Show upload progress, support cancel, and auto-retry on transient storage failures.

**Independent Test**: Large upload shows progress/cancel; transient failure retries with exponential backoff.

- [ ] T036 [P] [US5] Unit tests for retry policy and state transitions in `tests/unit/upload_retry_policy_test.go`
- [ ] T037 [P] [US5] Integration test for transient storage outage and recovery in `tests/integration/upload_retry_storage_outage_test.go`
- [ ] T038 [P] [US5] E2E test for progress/cancel/retry UX in `tests/e2e/upload_reliability.spec.ts`
- [ ] T039 [US5] Implement retry orchestrator and status model in `backend/internal/file/service_upload_retry.go`
- [ ] T040 [US5] Implement HTMX progress and retry status partials in `web/templates/files/upload_status_partial.html`

---

## Phase 8: User Story 6 - Group Creation and Admin Safeguards (Priority: P1)

**Goal**: Groups must always have at least one admin user.

**Independent Test**: Group creation without admin fails; last-admin removal blocked.

- [ ] T041 [P] [US6] Unit tests for group admin invariant in `tests/unit/group_admin_invariant_test.go`
- [ ] T042 [P] [US6] Integration test for create/remove-admin constraints in `tests/integration/group_admin_guardrails_test.go`
- [ ] T043 [P] [US6] E2E test for group creation/admin management in `tests/e2e/group_admin_guardrails.spec.ts`
- [ ] T044 [US6] Implement group admin invariant checks in `backend/internal/group/service_admin_rules.go`

---

## Phase 9: User Story 7 - Group Membership Management (Priority: P1)

**Goal**: Group admins add/remove users and recalculate effective permissions.

**Independent Test**: Membership updates immediately affect effective access.

- [ ] T045 [P] [US7] Unit tests for membership add/remove and permission recalculation in `tests/unit/group_membership_service_test.go`
- [ ] T046 [P] [US7] Integration test for group membership mutation in `tests/integration/group_membership_flow_test.go`
- [ ] T047 [P] [US7] E2E test for group member management UI in `tests/e2e/group_membership.spec.ts`
- [ ] T048 [US7] Implement membership service and handlers in `backend/internal/group/service_membership.go` and `backend/internal/group/handler_membership.go`

---

## Phase 10: User Story 8 - Sub-Group Hierarchy Safeguards (Priority: P1)

**Goal**: Allow nested groups while preventing cycles.

**Independent Test**: Valid subgroup accepted; cyclic relation rejected with clear message.

- [ ] T049 [P] [US8] Unit tests for closure-table cycle checks in `tests/unit/group_closure_cycle_test.go`
- [ ] T050 [P] [US8] Integration test for subgroup create + cycle rejection in `tests/integration/group_hierarchy_cycle_test.go`
- [ ] T051 [P] [US8] E2E test for subgroup UI flow and cycle error message in `tests/e2e/group_hierarchy.spec.ts`
- [ ] T052 [US8] Implement closure-table repository/service in `backend/internal/group/service_hierarchy.go` and `backend/internal/group/repository_closure.go`

---

## Phase 11: User Story 9 - Internal Sharing to Users and Groups (Priority: P1)

**Goal**: Owners share files with users/groups using granular permissions.

**Independent Test**: Group shares propagate to new members; permission updates apply immediately.

- [ ] T053 [P] [US9] Unit tests for effective permission computation in `tests/unit/file_share_permissions_test.go`
- [ ] T054 [P] [US9] Integration test for user/group share propagation in `tests/integration/file_share_propagation_test.go`
- [ ] T055 [P] [US9] E2E test for internal share workflow in `tests/e2e/internal_sharing.spec.ts`
- [ ] T056 [US9] Implement share service and handlers in `backend/internal/share/service_internal.go` and `backend/internal/share/handler_internal.go`

---

## Phase 12: User Story 10 - External One-Time Read-Only Sharing (Priority: P1)

**Goal**: Issue one-time external read-only keys with single-use enforcement and expiry.

**Independent Test**: First key use succeeds; second use denied; trashed/purged target returns 401.

- [ ] T057 [P] [US10] Unit tests for key hashing/single-use/expiry in `tests/unit/external_key_service_test.go`
- [ ] T058 [P] [US10] Integration test for external share issuance + consume in `tests/integration/external_share_flow_test.go`
- [ ] T059 [P] [US10] E2E test for external read-only flow in `tests/e2e/external_one_time_share.spec.ts`
- [ ] T060 [US10] Implement external key service and consume handler in `backend/internal/share/service_external.go` and `backend/internal/share/handler_external.go`
- [ ] T061 [US10] Implement SMTP notification integration for external key delivery in `backend/internal/platform/notify/smtp_sender.go`

**Checkpoint**: MVP (US1-US10) complete and independently demonstrable.

---

## Phase 13: User Story 11 - Folder Organization (Priority: P2)

- [ ] T062 [P] [US11] Unit tests for folder create/move semantics in `tests/unit/folder_service_test.go`
- [ ] T063 [P] [US11] Integration test for folder/file move persistence in `tests/integration/folder_move_test.go`
- [ ] T064 [P] [US11] E2E test for folder navigation and move in `tests/e2e/folders.spec.ts`
- [ ] T065 [US11] Implement folder service and handlers in `backend/internal/file/service_folder.go` and `backend/internal/file/handler_folder.go`

## Phase 14: User Story 12 - Search and Sort (Priority: P2)

- [ ] T066 [P] [US12] Unit tests for query parser and sort validation in `tests/unit/file_search_query_test.go`
- [ ] T067 [P] [US12] Integration test for filename/type/uploader search in `tests/integration/file_search_test.go`
- [ ] T068 [P] [US12] E2E test for search + sort UX in `tests/e2e/file_search_sort.spec.ts`
- [ ] T069 [US12] Implement search repository and handlers in `backend/internal/file/repository_search.go` and `backend/internal/file/handler_search.go`

## Phase 15: User Story 13 - File Tagging (Priority: P2)

- [ ] T070 [P] [US13] Unit tests for tag add/remove rules in `tests/unit/file_tagging_test.go`
- [ ] T071 [P] [US13] Integration test for tag persistence in `tests/integration/file_tagging_test.go`
- [ ] T072 [P] [US13] E2E test for tag management in `tests/e2e/file_tags.spec.ts`
- [ ] T073 [US13] Implement tag service and handlers in `backend/internal/file/service_tags.go` and `backend/internal/file/handler_tags.go`

## Phase 16: User Story 14 - File TTL Configuration (Priority: P2)

- [ ] T074 [P] [US14] Unit tests for TTL scheduling and expiry rules in `tests/unit/file_ttl_rules_test.go`
- [ ] T075 [P] [US14] Integration test for TTL-to-trash transition in `tests/integration/file_ttl_transition_test.go`
- [ ] T076 [P] [US14] E2E test for TTL configuration flow in `tests/e2e/file_ttl.spec.ts`
- [ ] T077 [US14] Implement TTL update and scheduler hooks in `backend/internal/file/service_ttl.go`

## Phase 17: User Story 15 - Trash Recovery and Purge (Priority: P2)

- [ ] T078 [P] [US15] Unit tests for trash retention and restore invariants in `tests/unit/trash_lifecycle_test.go`
- [ ] T079 [P] [US15] Integration test for restore and purge jobs in `tests/integration/trash_restore_purge_test.go`
- [ ] T080 [P] [US15] E2E test for trash list/restore/purge journey in `tests/e2e/trash_lifecycle.spec.ts`
- [ ] T081 [US15] Implement trash service and handlers in `backend/internal/file/service_trash.go` and `backend/internal/file/handler_trash.go`

## Phase 18: User Story 16 - Trash Restore Conflict Handling (Priority: P2)

- [ ] T082 [P] [US16] Unit tests for restore conflict resolution options in `tests/unit/trash_restore_conflict_test.go`
- [ ] T083 [P] [US16] Integration test for overwrite/cancel/rename behavior in `tests/integration/trash_restore_conflict_test.go`
- [ ] T084 [P] [US16] E2E test for restore conflict UX in `tests/e2e/trash_restore_conflict.spec.ts`
- [ ] T085 [US16] Implement conflict resolver in `backend/internal/file/service_restore_conflict.go` and `web/templates/trash/restore_conflict_modal.html`

## Phase 19: User Story 17 - File Version History and Restore (Priority: P2)

- [ ] T086 [P] [US17] Unit tests for version creation/restore semantics in `tests/unit/file_versioning_test.go`
- [ ] T087 [P] [US17] Integration test for version history persistence in `tests/integration/file_versioning_test.go`
- [ ] T088 [P] [US17] E2E test for version list and restore in `tests/e2e/file_version_restore.spec.ts`
- [ ] T089 [US17] Implement version service and handlers in `backend/internal/file/service_versions.go` and `backend/internal/file/handler_versions.go`

## Phase 20: User Story 18 - Tenant User Administration (Priority: P2)

- [ ] T090 [P] [US18] Unit tests for role/suspend/remove rules in `tests/unit/tenant_user_admin_test.go`
- [ ] T091 [P] [US18] Integration test for admin user management in `tests/integration/tenant_user_admin_test.go`
- [ ] T092 [P] [US18] E2E test for admin panel operations in `tests/e2e/tenant_admin_users.spec.ts`
- [ ] T093 [US18] Implement tenant admin handlers in `backend/internal/tenant/handler_admin_users.go` and `web/templates/admin/users.html`

## Phase 21: User Story 19 - Platform Superuser Bootstrap and Access Control (Priority: P2)

- [ ] T094 [P] [US19] Unit tests for bootstrap + must-rotate-password flow in `tests/unit/superuser_bootstrap_test.go`
- [ ] T095 [P] [US19] Integration test for startup bootstrap and operator access control in `tests/integration/superuser_bootstrap_access_test.go`
- [ ] T096 [P] [US19] E2E test for first-login password rotation and operator access denial to non-superusers in `tests/e2e/superuser_bootstrap.spec.ts`
- [ ] T097 [US19] Implement bootstrap and superuser auth services in `backend/internal/operator/service_bootstrap.go` and `backend/internal/operator/handler_auth.go`

## Phase 22: User Story 20 - Authentication Outage Continuity for Active Sessions (Priority: P2)

- [ ] T098 [P] [US20] Unit tests for auth circuit-breaker/session extension policy in `tests/unit/auth_outage_continuity_test.go`
- [ ] T099 [P] [US20] Integration test simulating Authelia outage and recovery in `tests/integration/auth_outage_recovery_test.go`
- [ ] T100 [P] [US20] E2E test for outage status message and continued active session in `tests/e2e/auth_outage_continuity.spec.ts`
- [ ] T101 [US20] Implement auth provider circuit breaker in `backend/internal/auth/service_provider_health.go`

## Phase 23: User Story 21 - System Operator Configuration Management (Priority: P2)

- [ ] T102 [P] [US21] Unit tests for runtime config validation in `tests/unit/platform_config_validation_test.go`
- [ ] T103 [P] [US21] Integration test for config update + audit recording in `tests/integration/platform_config_update_test.go`
- [ ] T104 [P] [US21] E2E test for operator config UI in `tests/e2e/operator_config.spec.ts`
- [ ] T105 [US21] Implement operator config service/handlers in `backend/internal/operator/service_config.go` and `backend/internal/operator/handler_config.go`

## Phase 24: User Story 22 - Docker-Compose Ecosystem Startup (Priority: P2)

- [ ] T106 [P] [US22] Integration smoke test for compose startup health in `tests/integration/compose_startup_health_test.go`
- [ ] T107 [P] [US22] E2E environment test for basic upload/share workflow on compose stack in `tests/e2e/compose_bootstrap_flow.spec.ts`
- [ ] T108 [US22] Implement compose topology and health checks in `deploy/compose/docker-compose.yml` and `deploy/compose/healthcheck.sh`

## Phase 25: User Story 23 - Android Client Access (Priority: P3)

- [ ] T109 [P] [US23] Define mobile-facing API contract refinement in `specs/001-multi-tenant-file-storage/contracts/http-api.md`
- [ ] T110 [P] [US23] Integration test for client-compatible auth and file endpoints in `tests/integration/mobile_api_contract_test.go`
- [ ] T111 [US23] Add Android story implementation placeholder and acceptance checklist in `docs/android-roadmap.md`

## Phase 26: User Story 24 - Per-User Storage Quotas (Priority: P2)

- [ ] T112 [P] [US24] Unit tests for per-user quota enforcement in `tests/unit/user_quota_enforcement_test.go`
- [ ] T113 [P] [US24] Integration test for quota limit block/unblock behavior in `tests/integration/user_quota_enforcement_test.go`
- [ ] T114 [P] [US24] E2E test for quota-exceeded UX in `tests/e2e/user_quota.spec.ts`
- [ ] T115 [US24] Implement quota policy checks in `backend/internal/file/service_quota_user.go`

## Phase 27: User Story 25 - Per-Group Storage Quotas (Priority: P2)

- [ ] T116 [P] [US25] Unit tests for per-group quota accounting in `tests/unit/group_quota_enforcement_test.go`
- [ ] T117 [P] [US25] Integration test for group quota blocking in `tests/integration/group_quota_enforcement_test.go`
- [ ] T118 [P] [US25] E2E test for group quota messaging in `tests/e2e/group_quota.spec.ts`
- [ ] T119 [US25] Implement group quota policy service in `backend/internal/file/service_quota_group.go`

## Phase 28: User Story 26 - Per-Group Maximum File Size (Priority: P2)

- [ ] T120 [P] [US26] Unit tests for group max-size pre-write validation in `tests/unit/group_max_file_size_test.go`
- [ ] T121 [P] [US26] Integration test for oversized upload rejection and audit logging in `tests/integration/group_max_file_size_test.go`
- [ ] T122 [P] [US26] E2E test for file-size-limit UX in `tests/e2e/group_max_file_size.spec.ts`
- [ ] T123 [US26] Implement pre-write size guard and audit event logging in `backend/internal/file/service_size_guard.go`

---

## Final Phase: Polish & Cross-Cutting Concerns

- [ ] T124 [P] Documentation updates for setup and operations in `README.md` and `docs/operations.md`
- [ ] T125 [P] Accessibility validation (WCAG 2.1 AA) for critical web flows in `docs/accessibility-validation.md`
- [ ] T126 [P] Security review checklist completion in `docs/security-review.md`
- [ ] T127 [P] Dependency audit and pruning report in `docs/dependency-audit.md`
- [ ] T128 [P] Performance checks against SC-002/SC-009 in `docs/performance-validation.md`
- [ ] T129 Execute quickstart validation and capture evidence in `specs/001-multi-tenant-file-storage/quickstart-validation.md`

---

## Dependencies & Execution Order

### Phase dependencies
- Setup (Phase 1): start immediately
- Foundational (Phase 2): depends on setup; blocks all stories
- User stories (Phases 3-28): all depend on Phase 2
- Polish: depends on completion of targeted stories

### Priority dependencies
- MVP target: US1-US10 (P1)
- Extended web platform: US11-US22, US24-US26 (P2)
- Future scope: US23 (P3)

### Within each user story
- Unit tests first (must fail before implementation)
- Integration/e2e tests implemented next
- Service/domain implementation
- Handler/template integration
- Story acceptance verification

## Parallel opportunities

- Tasks marked [P] are parallelizable across different files.
- After Phase 2, separate story phases can run in parallel by different contributors.
- Tests in a story can run in parallel once fixtures are in place.

## Implementation strategy

### MVP first
1. Complete Phase 1 + Phase 2
2. Complete US1-US10
3. Validate MVP independently and demo

### Incremental delivery
1. Deliver MVP (P1)
2. Add P2 stories in order of operational risk (auth/config/lifecycle first)
3. Add Android integration scope (P3)

## Notes

- Keep dependencies minimal: prefer Go stdlib and existing project patterns.
- Any new dependency requires explicit security + maintenance justification.
- Preserve interface-driven DI for all external dependencies (DB, S3, OIDC, SMTP).
- Ensure tenant isolation checks on all tenant-scoped operations.
