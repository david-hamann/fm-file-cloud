# Feature Specification: Multi-tenant File Storage Application

**Feature Branch**: `001-multi-tenant-file-storage`  
**Created**: March 24, 2026  
**Status**: Draft  
**Input**: User description: "build a multi-tenant capable shared file storage application. it will have a backend written in Go, and a web UI written in Go and HTMX. For file storage, we will use AWS S3 compatible storage, like RustFS. PostgreSQL will be used to store user and file state. In the future we will also create an Android application"

**Dependency**: Requires completion of feature `000-foundation-dev-environment` for local environment bootstrap, startup diagnostics, and onboarding baseline.

## Clarifications

### Session 2026-03-24

- Q: What data residency model should v1 use? → A: Per-tenant residency configuration, with launch deployments possibly operating in a single configured region.
- Q: Which file attributes should be searchable in v1? → A: Filename, file type, and uploader.

## User Scenarios & Testing *(mandatory)*

<!--
  IMPORTANT: User stories should be PRIORITIZED as user journeys ordered by importance.
  Each user story/journey must be INDEPENDENTLY TESTABLE - meaning if you implement just ONE of them,
  you should still have a viable MVP (Minimum Viable Product) that delivers value.
  
  Assign priorities (P1, P2, P3, etc.) to each story, where P1 is the most critical.
  Think of each story as a standalone slice of functionality that can be:
  - Developed independently
  - Tested independently
  - Deployed independently
  - Demonstrated to users independently
-->

### User Story 1 - Tenant Created on First Login (Priority: P1)

A new organization admin authenticates through OpenID and creates the first tenant workspace.

**Why this priority**: Tenant creation is the root prerequisite for all tenant-scoped behavior.

**Independent Test**: Authenticate as a first-time admin and verify a tenant is created with admin access and that the admin belongs to at least one tenant group.

**Acceptance Scenarios**:

1. **Given** a new organization admin authenticates via OpenID through Authelia, **When** they complete first login, **Then** a new isolated tenant is created and linked to that admin, and a default tenant group (for example, "Tenant Admins") is created with that admin added as a member.
2. **Given** Authelia is unavailable, **When** a new user attempts to log in, **Then** they see a clear message that authentication is temporarily unavailable and are not offered a retry.
3. **Given** a new tenant has just been created on first login, **When** the system inspects the first admin’s membership, **Then** that admin belongs to at least one group within that tenant.

---

### User Story 2 - Tenant Invite and Join (Priority: P1)

A tenant admin invites a user and the user joins the tenant.

**Why this priority**: Team collaboration requires a reliable onboarding path.

**Independent Test**: Send invite, accept invite, and confirm new user appears in the tenant.

**Acceptance Scenarios**:

1. **Given** a tenant admin is logged in, **When** they invite a user by email, **Then** the user receives an invitation.
2. **Given** an invited user accepts the invitation, **When** they log in, **Then** they can access that tenant.

---

### User Story 3 - Cross-Tenant Isolation Enforcement (Priority: P1)

Users can only access resources in their own tenant unless explicitly shared.

**Why this priority**: Isolation is a core security boundary.

**Independent Test**: Attempt direct access to another tenant's resource and verify denial.

**Acceptance Scenarios**:

1. **Given** a user in tenant A, **When** they attempt access to tenant B resources, **Then** access is denied with an authorization error.

---

### User Story 4 - Basic File Upload (Priority: P1)

Users upload files and see persisted metadata.

**Why this priority**: Upload is the first core value delivery action.

**Independent Test**: Upload a file and verify it appears in the file list with metadata.

**Acceptance Scenarios**:

1. **Given** a logged-in user, **When** they upload a file, **Then** the file is stored and confirmation is shown.
2. **Given** an uploaded file, **When** the user views files, **Then** filename, size, uploader, and timestamp are displayed.

---

### User Story 5 - Upload Reliability Controls (Priority: P1)

Users can monitor long uploads and recover from failures.

**Why this priority**: Reliable uploads are critical for production use.

**Independent Test**: Trigger large upload and failure path to confirm progress/cancel/retry behavior.

**Acceptance Scenarios**:

1. **Given** a large upload in progress, **When** the user watches status, **Then** progress is visible and cancellation is available.
2. **Given** an upload fails, **When** the error is returned, **Then** the user sees a clear message and can retry.
3. **Given** an upload fails due to storage being temporarily unavailable, **When** the failure is detected, **Then** the upload is automatically retried with exponential back-off (e.g. 30 s → 60 s → 2 min) and a visible retry status is shown to the user.
4. **Given** automatic retries are exhausted without success, **When** the final attempt fails, **Then** the upload is abandoned, the user sees a clear error message, and can manually retry when ready.

---

### User Story 6 - Group Creation and Admin Safeguards (Priority: P1)

Admins create groups and preserve at least one group admin.

**Why this priority**: Group structure and admin invariants enable safe permission management.

**Independent Test**: Create group with admin and verify last-admin removal is blocked.

**Acceptance Scenarios**:

1. **Given** a tenant admin creates a group, **When** saving it, **Then** at least one user must be assigned as group admin.
2. **Given** a group has one admin, **When** someone attempts to remove that admin role, **Then** the operation is blocked until another admin exists.

---

### User Story 7 - Group Membership Management (Priority: P1)

Group admins manage user membership within groups.

**Why this priority**: Managing group membership is a core administrative operation used daily.

**Independent Test**: Add and remove users from a group and verify membership updates correctly.

**Acceptance Scenarios**:

1. **Given** a group admin, **When** they add users to a group, **Then** users are added with assigned roles.
2. **Given** a group admin, **When** they remove users from a group, **Then** membership updates immediately and effective permissions are recalculated.

---

### User Story 8 - Sub-Group Hierarchy Safeguards (Priority: P1)

Group admins create sub-groups while preserving an acyclic hierarchy.

**Why this priority**: Nested groups are required for organizational modeling, and cycle prevention is a critical integrity guardrail.

**Independent Test**: Create a valid subgroup and then attempt a cyclic subgroup assignment to confirm rejection.

**Acceptance Scenarios**:

1. **Given** a group admin, **When** they create a subgroup, **Then** subgroup membership is maintained separately with at least one admin.
2. **Given** a group admin attempts to add a subgroup that would create a cycle (e.g., adding a parent group as a child), **When** the operation is submitted, **Then** the system detects the cycle and rejects the operation with a clear message explaining that the group hierarchy cannot include cycles.

---

### User Story 9 - Internal Sharing to Users and Groups (Priority: P1)

Owners share files with users/groups with permission levels.

**Why this priority**: Internal collaboration is the main product workflow.

**Independent Test**: Share to group and verify recipients gain/lose access as permissions change.

**Acceptance Scenarios**:

1. **Given** a file owner, **When** they share to users/groups, **Then** recipients get assigned permissions.
2. **Given** a new member added to a shared group, **When** membership updates, **Then** the member inherits group file access.
3. **Given** sharing permissions change, **When** update is saved, **Then** effective access updates immediately.

---

### User Story 10 - External One-Time Read-Only Sharing (Priority: P1)

Owners share read-only access externally using one-time keys.

**Why this priority**: External collaboration is a required business capability.

**Independent Test**: Send external share, consume key once, verify reuse is denied.

**Acceptance Scenarios**:

1. **Given** a file owner provides an external email, **When** read-only external share is created, **Then** a one-time key is sent.
2. **Given** recipient has valid key, **When** they access the shared link, **Then** read-only access is granted without account creation.
3. **Given** key was already used, **When** recipient retries, **Then** access is denied.
4. **Given** a recipient presents a valid one-time key but the shared file has moved to trash or been purged, **When** access is requested, **Then** the system returns HTTP 401 and logs the attempt.

---

### User Story 11 - Folder Organization (Priority: P2)

Users organize files with folders and move operations.

**Why this priority**: Folder structure improves manageability for growing datasets.

**Independent Test**: Create folders, move file, and verify new location.

**Acceptance Scenarios**:

1. **Given** a user in file browser, **When** they create a folder, **Then** it appears and is navigable.
2. **Given** a file in one folder, **When** user moves it, **Then** file appears in destination folder.

---

### User Story 12 - Search and Sort (Priority: P2)

Users discover files by searchable attributes and sorting.

**Why this priority**: Discovery efficiency drives day-to-day productivity.

**Independent Test**: Search by filename/file type/uploader and apply sorting.

**Acceptance Scenarios**:

1. **Given** many files exist, **When** user searches by filename, file type, or uploader, **Then** relevant results are returned.
2. **Given** results list, **When** user sorts by name, size, or date, **Then** ordering updates correctly.

---

### User Story 13 - File Tagging (Priority: P2)

Users apply and manage multiple tags per file.

**Why this priority**: Tags provide flexible classification beyond folder hierarchies.

**Independent Test**: Add, edit, and remove tags and verify metadata persistence.

**Acceptance Scenarios**:

1. **Given** a file owner, **When** they add one or more tags, **Then** tags are stored and visible.
2. **Given** tagged file, **When** owner removes a tag, **Then** metadata updates immediately.

---

### User Story 14 - File TTL Configuration (Priority: P2)

Users configure optional TTL for files.

**Why this priority**: TTL supports lifecycle automation and storage governance.

**Independent Test**: Set TTL and verify file transitions to trash at expiry.

**Acceptance Scenarios**:

1. **Given** a file, **When** owner sets a TTL, **Then** expiry is stored with file metadata.
2. **Given** TTL expires, **When** lifecycle processing runs, **Then** file is moved to trash.

---

### User Story 15 - Trash Recovery and Purge (Priority: P2)

Deleted/expired files remain recoverable before permanent purge.

**Why this priority**: Baseline trash lifecycle behavior protects against accidental deletion.

**Independent Test**: Delete file, restore within retention, and verify purge after retention.

**Acceptance Scenarios**:

1. **Given** file is in trash and retention is active, **When** user restores, **Then** file returns with metadata and sharing intact.
2. **Given** trash item retention expires, **When** purge runs, **Then** file is permanently removed.

---

### User Story 16 - Trash Restore Conflict Handling (Priority: P2)

Users resolve restore conflicts when a target path already contains a file.

**Why this priority**: Restore conflict handling is an independent recovery concern that can be delivered separately from basic trash lifecycle.

**Independent Test**: Attempt restore with same path/name conflict and verify overwrite, cancel, and rename outcomes.

**Acceptance Scenarios**:

1. **Given** a user initiates a trash restore and a file already exists at the same path and name, **When** the conflict is detected, **Then** the user is shown a warning and offered three options: overwrite the existing file, cancel the restore, or restore under the system-generated name `filename-restored.ext`.
2. **Given** the user selects overwrite during a restore conflict, **When** the restore is confirmed, **Then** the existing file is moved to trash before the restored file is placed at that path, leaving the displaced file recoverable within its own retention window.

---

### User Story 17 - File Version History and Restore (Priority: P2)

Users track and restore file versions.

**Why this priority**: Versioning reduces risk of data loss and supports auditability.

**Independent Test**: Upload multiple versions and restore prior version.

**Acceptance Scenarios**:

1. **Given** a file with multiple versions, **When** user opens history, **Then** they see version metadata.
2. **Given** prior version selected, **When** restore is confirmed, **Then** prior version becomes current and history is preserved.

---

### User Story 18 - Tenant User Administration (Priority: P2)

Tenant admins manage members and roles.

**Why this priority**: Operational control is required for tenant governance.

**Independent Test**: Change role, suspend/remove user, verify access updates.

**Acceptance Scenarios**:

1. **Given** tenant admin in admin panel, **When** they change user role, **Then** effective permissions update.
2. **Given** tenant user is removed, **When** they attempt access, **Then** access is denied.

---

### User Story 19 - Platform Superuser Bootstrap and Access Control (Priority: P2)

Platform superuser is bootstrapped via environment variables and granted exclusive operator access.

**Why this priority**: Platform operations require controlled bootstrap and strict operator-area access boundaries.

**Independent Test**: Bootstrap with env vars, require first-login password change, and verify non-superusers cannot access operator interfaces.

**Acceptance Scenarios**:

1. **Given** first-time startup and no superuser exists, **When** bootstrap env vars are valid, **Then** initial superuser is created.
2. **Given** bootstrap-created superuser logs in first time, **When** session starts, **Then** password change is mandatory.
3. **Given** non-superuser attempts operator access, **When** requesting operator interface, **Then** access is denied.

---

### User Story 20 - Authentication Outage Continuity for Active Sessions (Priority: P2)

Active operator sessions remain usable when the external authentication provider is temporarily unavailable.

**Why this priority**: Outage continuity can be delivered independently of bootstrap/access control and reduces operational disruption.

**Independent Test**: Simulate Authelia outage during token refresh and verify session continuity and status messaging.

**Acceptance Scenarios**:

1. **Given** Authelia is unavailable and a superuser's token is about to refresh, **When** the refresh is attempted, **Then** the existing token's TTL is extended to allow continued access, and the superuser sees a system status message indicating authentication service is temporarily offline.

---

### User Story 21 - System Operator Configuration Management (Priority: P2)

Superusers maintain infrastructure settings through the operator interface.

**Why this priority**: Runtime reliability depends on manageable platform integrations.

**Independent Test**: Update S3/database/SMTP configuration and verify validation + audit.

**Acceptance Scenarios**:

1. **Given** superuser in operator interface, **When** updating S3/database/SMTP settings, **Then** each change is validated before apply.
2. **Given** config change is applied, **When** audit log is viewed, **Then** actor and timestamp are recorded.

---

### User Story 23 - Android Client Access (Priority: P3)

Android users access the same tenant-scoped content as web users.

**Why this priority**: Mobile extends platform reach after core web workflows stabilize.

**Independent Test**: Log in on Android and execute browse/upload flow.

**Acceptance Scenarios**:

1. **Given** Android user logs in, **When** app loads tenant workspace, **Then** accessible files match web permissions.
2. **Given** Android user uploads a file, **When** upload succeeds, **Then** file is visible across platforms.

---

### User Story 24 - Per-User Storage Quotas (Priority: P2)

Tenant admins configure and enforce storage quotas for individual users.

**Why this priority**: Per-user quotas provide fair storage allocation and prevent single-user overconsumption.

**Independent Test**: Set a user quota, upload until the threshold is reached, and confirm additional uploads are blocked with clear messaging.

**Acceptance Scenarios**:

1. **Given** a tenant admin sets a per-user storage quota, **When** the user reaches that quota, **Then** additional uploads by that user are rejected with a clear quota-exceeded message.
2. **Given** a per-user quota is increased, **When** the previously blocked user retries an upload within the new limit, **Then** the upload succeeds.

---

### User Story 25 - Per-Group Storage Quotas (Priority: P2)

Tenant admins configure and enforce storage quotas for groups.

**Why this priority**: Per-group quotas keep team-level storage usage predictable and bounded.

**Independent Test**: Set a group quota, upload files in that group context until the threshold is reached, and confirm further uploads are blocked.

**Acceptance Scenarios**:

1. **Given** a tenant admin sets a per-group storage quota, **When** the group reaches that quota, **Then** uploads in that group's storage context are rejected with a clear quota-exceeded message.
2. **Given** a per-group quota is increased, **When** a previously blocked upload is retried within the new limit, **Then** the upload succeeds.

---

### User Story 26 - Per-Group Maximum File Size (Priority: P2)

Tenant admins configure a maximum upload size per group.

**Why this priority**: Per-group size limits enforce policy and prevent oversized uploads before storage costs are incurred.

**Independent Test**: Set a per-group max file size, attempt uploads above and below the limit, and verify correct allow/deny behavior.

**Acceptance Scenarios**:

1. **Given** a tenant admin sets a per-group maximum file size, **When** a user uploads a file larger than that limit to the group, **Then** the upload is rejected before storage write with a clear file-size-limit message.
2. **Given** an upload is rejected due to per-group size limit, **When** the event is processed, **Then** the system logs the rejection with actor, group scope, attempted size, configured limit, and timestamp.

---

### Edge Cases

- What happens when a user is removed from a tenant while actively viewing/editing a file?
- **[RESOLVED]** When S3-compatible storage is temporarily unavailable, the upload client retries automatically with exponential back-off; after the retry window is exhausted the upload fails with a clear error and the user can retry manually.
- What is the maximum file size that can be uploaded, and how is this limit communicated to users?
- What happens if two users simultaneously edit and upload versions of the same file?
- How does the system handle deleted files - permanent deletion after a grace period or soft delete?
- What happens when an update would leave a group with zero admin users?
- What happens when an update would leave the superuser group with zero superusers?
- What happens when an external one-time access key is intercepted, expires, or already used?
- What happens when a file TTL expires while a user is actively viewing or downloading the file?
- What happens when a file in trash reaches retention expiry during an attempted restore?
- How does the system enforce tenant-specific residency policy when multiple regions are configured?
- **[RESOLVED]** When Authelia is unavailable, new login attempts are rejected immediately with a clear system status message; existing user sessions remain valid and may refresh or extend their authentication tokens within the normal TTL window; once Authelia is restored, normal login and token refresh operations resume immediately.
- What happens if system clock skew causes one-time key or TTL expiration checks to disagree between services?
- What happens when a file TTL expires during multipart upload, download streaming, or version restore?
- **[RESOLVED]** When restoring a trash file would conflict with an existing file at the same path/name, the system presents a warning and three options: (1) overwrite — the existing file is moved to trash before the restore completes, ensuring it remains recoverable; (2) do nothing — the restore is cancelled and the trash item remains; (3) rename — the file is restored under a system-generated name (`filename-restored.ext`). If overwrite is chosen and the displaced file itself has a restoration conflict, the same resolution applies recursively.
- What happens when a user's permission is revoked while they have an active download or open file session?
- **[RESOLVED]** The system prevents cyclic subgroup relationships by maintaining a closure table of all ancestor-descendant pairs; when adding a subgroup, the system checks if the operation would create a reverse relationship that already exists; if so, the operation is rejected with a clear error message.
- **[RESOLVED]** If an external one-time key is otherwise valid but the target file has moved to trash or been purged, the system returns HTTP 401 and logs the access attempt.
- What happens when tenant data residency configuration is changed after files already exist in a different region?

## Testing Strategy *(mandatory)*

- **Unit Tests**: Core business logic MUST be covered:
  - Tenant isolation and authorization enforcement
  - Permission/role evaluation for file access
  - Superuser authorization and platform-settings access enforcement
  - File metadata operations (create, update, delete)
  - Tag assignment/removal validation and metadata consistency
  - TTL expiry transitions and trash retention enforcement
  - User and tenant lifecycle management
  - Version management and restore logic
  
- **Integration Tests**: Required for:
  - File upload/download with S3-compatible storage backend
  - Database persistence and retrieval of user/file/sharing metadata
  - Authentication and session management via Authelia OpenID integration for tenant users and group admins
  - Superuser configuration management for S3, database, and SMTP with connectivity validation
  - Initial superuser bootstrap from environment variables with validation and first-login password-change enforcement
  - Tenant boundary enforcement (data isolation)
  - Trash lifecycle transitions (active → trash → purge) with configurable retention
  - External email share key issuance, validation, single-use enforcement, and expiry handling
  - Multi-user concurrent file access scenarios
  
- **End-to-End Tests**: Required for primary user flows:
  - User and group-admin login via Authelia OpenID → tenant access with role-based permissions
  - Complete account creation → file upload → share → recipient access flow
  - File tagging and TTL setup → automatic move to trash on expiry → successful restore within retention
  - External read-only share via email → one-time key entry → single-use access denial on second attempt
  - First boot with superuser env vars → initial superuser creation → first login requires password rotation
  - Superuser login → system operator interface access → update and validate configuration changes
  - Superuser login → update platform settings (S3/database/SMTP) → validation success
  - File versioning and restore flow
  - Tenant admin managing users and permissions
  - Mobile app integration (after Android app development)
  
- **Responsive Validation (if UI)**: Web UI must function on:
  - Desktop: 1920x1080, 1366x768
  - Tablet: iPad Pro (1024x1366), iPad (768x1024)
  - Mobile testing reserved for dedicated Android app
  - Touch-friendly layouts for tablet viewport

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST support multiple isolated tenants with complete data separation - users from Tenant A cannot access resources from Tenant B
- **FR-002**: System MUST authenticate tenant users and group admins via OpenID provided by Authelia
- **FR-003**: System MUST support file upload with size validation and store files in S3-compatible storage (initially rustfs, future AWS S3 compatibility)
- **FR-004**: System MUST track file metadata including: filename, file size, MIME type, upload timestamp, uploader identity, storage location
- **FR-005**: System MUST support file download with streaming to users who have "view" or higher permissions
- **FR-006**: System MUST enforce tenant-user group membership - every tenant user MUST belong to at least one group
- **FR-007**: System MUST support hierarchical group structure where groups can contain sub-groups and individual users
- **FR-008**: System MUST require each group to have at least one user assigned as a group admin at all times
- **FR-009**: System MUST grant group admins authority to manage group membership (add/remove users, assign member roles, assign/revoke group admin role)
- **FR-010**: System MUST support file sharing with granular permissions: "view" (read-only), "comment" (read + add comments), "edit" (read + modify versions), "admin" (full control + permission management)
- **FR-011**: System MUST support sharing files with both individual users and groups; group-based shares apply to all current and future group members
- **FR-012**: System MUST grant group admins admin access to all content shared with their group
- **FR-013**: System MUST allow a file owner to share a file as read-only with an external email recipient who does not have an account
- **FR-014**: System MUST issue a one-time access key for external read-only shares and deliver it to the recipient via email
- **FR-015**: System MUST enforce single-use behavior for external one-time access keys and deny reuse after first successful access
- **FR-016**: System MUST prevent users from accessing files outside their tenant unless explicitly shared with proper permissions
- **FR-017**: System MUST track file versions automatically with upload timestamp, uploader, and version metadata
- **FR-018**: System MUST allow users to restore previous file versions while keeping restore history
- **FR-019**: System MUST support file organization via folders/directories with create, delete, and move operations
- **FR-020**: System MUST provide search functionality across files by filename, file type, and uploader
- **FR-021**: System MUST allow files to have one or more tags that can be created, updated, and removed by authorized users
- **FR-022**: System MUST allow an optional TTL to be set on a file, after which the file is automatically moved to trash
- **FR-023**: System MUST provide a trash section for deleted or TTL-expired files where they remain recoverable for a system-configurable retention period
- **FR-024**: System MUST allow authorized users to restore files from trash during the retention window with metadata and sharing preserved
- **FR-025**: System MUST permanently purge files from trash after retention expires
- **FR-026**: System MUST support user invitations via email with acceptance workflow for joining tenants
- **FR-027**: System MUST provide Admin dashboard for tenant administrators to: manage users, assign roles, manage groups and group membership, view activity logs, manage permissions
- **FR-028**: System MUST enforce role-based access control with roles: Member (read-only), Editor (read + upload versions), Admin (full control)
- **FR-029**: System MUST log all administrative actions (user creation/removal/role changes, group management, permission changes) with timestamp and responsible user
- **FR-030**: System MUST handle concurrent access to shared files with conflict resolution or user notification
- **FR-031**: System MUST support persistence of all user, file, group, and permission data in PostgreSQL with proper relationships and indexing
- **FR-032**: System MUST implement audit logging for compliance including: user login, file access, sharing changes, group membership changes, admin actions
- **FR-033**: System MUST maintain a dedicated superuser group for platform operators; superusers are outside tenant group membership hierarchy
- **FR-034**: System MUST ensure the superuser group always has at least one active superuser and block operations that would leave it empty
- **FR-035**: System MUST allow only superusers to manage global infrastructure connection settings (S3-compatible storage, database, SMTP)
- **FR-036**: System MUST require connection validation before saving infrastructure configuration changes and record all changes in audit logs
- **FR-037**: System MUST bootstrap the initial superuser account using environment variables for superuser ID and password at application startup when no superuser exists
- **FR-038**: System MUST fail startup with a clear error if required superuser bootstrap environment variables are missing or invalid during first-time bootstrap
- **FR-039**: System MUST enforce password change on first login for bootstrap-created superuser credentials
- **FR-040**: System MUST provide a dedicated system operator interface for superusers to maintain and update global system configuration settings
- **FR-044**: System MUST support local authentication only for the bootstrap-created superuser account used for platform-operator access
- **FR-046**: System MUST support per-tenant data residency configuration for file and metadata storage placement, while allowing deployments to operate with a single configured region when only one region is available
- **FR-047**: When a trash restore operation would conflict with an existing file at the same path and name, the system MUST present the user with a clear warning and three resolution options: (1) Overwrite — the existing file is moved to trash before the restore completes, preserving its metadata and sharing so it remains recoverable within its own retention window; (2) Cancel — the restore is abandoned and the trash item is unchanged; (3) Rename — the file is restored under the system-generated name `filename-restored.ext`, leaving the existing file in place unmodified
- **FR-048**: When a file upload fails due to S3-compatible storage being unavailable, the system MUST automatically retry the upload using exponential back-off intervals; the user MUST see a visible retry status throughout; after retries are exhausted without success the upload MUST be abandoned and the user MUST be shown a clear error message with the option to retry manually
- **FR-049**: System MUST implement a three-layer configuration model: (1) environment variables supply bootstrap credentials and pre-database service connection strings required before the database is reachable; (2) database-stored `PlatformConfiguration` holds runtime-adjustable settings manageable by superusers via the operator interface without requiring a restart, and is the authoritative source for all running instances; (3) baseline default values from feature `000-foundation-dev-environment` seed initial runtime configuration on first startup when no stored configuration exists
- **FR-050**: System MUST monitor Authelia service availability using a circuit breaker; when Authelia becomes unreachable, the system MUST immediately set a system status flag to "authentication unavailable"; new login attempts MUST be rejected with a clear message indicating authentication is temporarily offline; existing user sessions with active tokens MUST remain valid and may extend their TTL if refresh is attempted before token expiry; when Authelia recovers and responds, the status flag MUST be cleared and login/refresh operations MUST resume normally
- **FR-051**: System MUST prevent cyclic subgroup relationships by maintaining a closure table of all ancestor-descendant group relationships in the database; when a subgroup assignment is attempted, the system MUST check if the operation would create a reverse relationship (making a group a descendant of one of its descendants); if a cycle would be created, the operation MUST be rejected with a clear message explaining that the group hierarchy cannot include cycles
- **FR-052**: If an external one-time access key is presented for a file that has moved to trash or been purged, the system MUST deny access with HTTP 401 and MUST log the attempt with timestamp, presented key identifier, and outcome
- **FR-053**: System MUST enforce a configurable validity period for external one-time access keys, sourced from system configuration, with a default value of 5 days

### Key Entities

- **Tenant**: Represents an isolated organization/company. Contains users, groups, files, and folders. Each tenant has independent data and a residency configuration indicating where tenant file and metadata storage must reside.
- **User**: Individual account belonging to a tenant. Has email, role within tenant, creation/modification timestamps, status (active/suspended/removed), and references to identity information received from Authelia OpenID Connect (e.g., subject identifier). Tenant users authenticate via Authelia OIDC and DO NOT have local passwords stored in this system. Tenant users MUST be a member of at least one group.
- **SuperuserGroup**: Dedicated platform-level operator group that is not part of tenant group hierarchies. Members can manage global infrastructure settings and system operations.
- **SuperuserAccount**: Platform-level operator account authenticated via local credentials (email and hashed password, and optionally second-factor configuration). Superuser accounts are NOT associated with any tenant **User** records and exist outside tenant scopes. Membership in **SuperuserGroup** (or association to it via configuration) grants these accounts their elevated permissions.
- **Group**: Container for organizing users and sub-groups within a tenant. Supports hierarchical structure with parent group reference. Used for permission management and organizational structure. MUST always include at least one admin user.
- **GroupMembership**: Relationship between a user/sub-group and a parent group. Includes member reference (user or group), parent group reference, membership role, admin flag for users, and membership timestamp.
- **File**: A stored file with metadata. Includes name, size, MIME type, storage path in S3, uploader user reference, tenant reference, creation/modification timestamps, current version reference, one or more tags, and optional TTL.
- **FileVersion**: Represents a specific version of a file. Includes version number, file content reference, uploader user, timestamp, file size, change description (optional).
- **Folder**: Container for organizing files within a tenant. Includes name, parent folder reference, tenant reference, creation timestamp, ownership.
- **TrashItem**: Recoverable record for deleted or TTL-expired files. Includes file reference, deletion reason, deletion timestamp, recoverable-until timestamp, and restore/purge status.
- **FileShare**: Permission record for sharing files/folders. Includes file reference, recipient type (user or group), recipient reference, permission level (view/comment/edit/admin), share timestamp, shared-by user reference. When shared with a group, all group members inherit the permission.
- **ExternalFileAccessKey**: One-time access credential for external read-only sharing. Includes file reference, recipient email, issued key/token, creation timestamp, expiration timestamp (derived from system-configured validity period), used timestamp, and status (active/used/expired/revoked). **Security requirement**: Raw one-time tokens MUST NOT be stored or logged; implementations MUST store only a salted hash of the issued token and MUST log only a non-sensitive key identifier or fingerprint (never the raw token value), to limit impact if databases or logs are exposed.
- **PlatformConfiguration**: Global system configuration for infrastructure integrations, including S3-compatible storage connection details, database connection details, SMTP server settings, trash retention policy, external one-time key validity period, last validated timestamp, and last updated by superuser.
- **BootstrapCredentialSource**: Startup configuration source for initial system credentials, including superuser ID environment variable, superuser password environment variable, bootstrap timestamp, and bootstrap completion status.
- **Role**: Tenant-level role definition. Predefined roles: Member (limited read access), Editor (read + upload versions), Admin (full control).

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Users typically upload, organize, and share their first file within 5 minutes of account creation
- **SC-002**: System supports 1000 concurrent users per tenant without performance degradation (for file sizes up to 25 MB and client network bandwidth ≥ 20 Mbps down / 10 Mbps up, 95th-percentile time-to-first-byte or UI acknowledgement for upload/download operations remains under 3 seconds)
- **SC-003**: File uploads and downloads maintain 99.5% success rate with automatic retry on transient failures
- **SC-004**: Shared files are accessible to recipients within 1 second of permission grant completion
- **SC-005**: Tenant data remains isolated with zero unauthorized cross-tenant data access incidents
- **SC-006**: 95% of file shares complete successfully without permission/visibility issues
- **SC-007**: Tenant administrators can manage user permissions and view audit logs in under 30 seconds per operation
- **SC-008**: System maintains data integrity with zero unintended file version loss during storage operations
- **SC-009**: File search returns relevant results in under 1 second for typical tenant file sizes up to 10,000 files
- **SC-010**: Android app (when released) provides equivalent file access experience as web UI with 95% feature parity
- **SC-012**: 99% of deleted or TTL-expired files remain recoverable from trash throughout the configured retention window
- **SC-013**: 99% of eligible trash restores complete successfully in under 30 seconds

## Assumptions

- **Developer Experience**: Developers are familiar with Go, HTMX, and PostgreSQL; these are standard tools for the team
- **Storage Backend**: S3-compatible storage (RustFS initially) is available and configured before application deployment; scaling storage is not a v1 concern
- **Authentication Scope**: Tenant users and group admins authenticate through Authelia OpenID; local credentials are limited to the bootstrap-created superuser account
- **File Retention**: Files are retained indefinitely unless an organization explicitly deletes them; archival/retention policies are post-MVP features
- **Security Baseline**: HTTPS/TLS is enforced at infrastructure level; application focuses on application-layer security and authorization
- **User Base**: Initial users are within organizations/teams; public file sharing or anonymous access is out of scope for v1
- **Group Scope**: Users must always belong to at least one group; groups reflect organizational structure (departments, teams, projects); sub-groups are optional and support multi-level hierarchies
- **Group Administration**: Every group always has at least one admin user; removing or demoting the final admin is blocked until another admin is assigned; group admins manage group users and have admin access to all content shared with the group
- **Superuser Scope**: Superusers are platform operators in a dedicated superuser group outside tenant group hierarchy; they can manage global S3/database/SMTP settings but are not implicitly granted tenant content access unless explicitly assigned
- **Operator Interface Scope**: System configuration maintenance is performed through a dedicated operator interface accessible only to superusers
- **Bootstrap Security**: Initial superuser ID and password are provided only through environment variables at first boot; bootstrap credentials are rotated immediately after first login
- **Foundational Environment Dependency**: Local environment bootstrap, startup diagnostics, and onboarding workflows are provided by feature `000-foundation-dev-environment`; this feature assumes that baseline is already available
- **Configuration Layering**: Application configuration follows a three-layer hierarchy, with each layer authoritative for its own concerns and not substitutable by another: (1) environment variables supply bootstrap credentials and service connection strings that must exist before the database is reachable (e.g., DB URL, superuser bootstrap credentials); (2) the database (`PlatformConfiguration`) stores runtime-adjustable settings managed by superusers via the operator interface, effective immediately without restart and consistent across all application instances; (3) baseline defaults from feature `000-foundation-dev-environment` seed first-start configuration when no stored configuration exists
- **Data Residency Scope**: Tenant data residency is configured per tenant; launch environments may run in a single configured region while retaining the per-tenant configuration model
- **Supporting Infrastructure**: Email delivery system (for invitations) exists or will be provided by platform; application assumes reliable email
- **External Share Security**: One-time external access keys are read-only, single-use, and expire after a system-configurable duration (default 5 days)
- **File Lifecycle Controls**: Files may include one or more tags and an optional TTL; deleted/expired files move to trash and remain recoverable for a superuser-configured retention period
- **Scalability Approach**: Database indexing and S3 optimization will handle file growth; advanced caching/CDN are future optimizations
- **Mobile Scope**: Android app development is explicitly scoped as future work; v1 focuses on web platform
- **Concurrency Model**: Simultaneous edits to files are rare in typical usage; optimistic locking or version conflicts are acceptable for MVP
- **Data Privacy**: No additional privacy/GDPR requirements beyond basic data isolation; privacy policies are handled separately
