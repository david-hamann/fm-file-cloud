# HTTP/API Contract (Draft): Multi-tenant File Storage

## Scope
This contract defines initial v1 interface boundaries for the Go backend + HTMX web UI and future Android client integration.

## Conventions
- Auth:
  - Tenant user sessions: Authelia OIDC session/cookie.
  - Superuser sessions: local credential flow.
- Content types:
  - HTMX/HTML responses for web UI interactions.
  - JSON for API endpoints intended for Android and programmatic clients.
- Multi-tenant:
  - Tenant-scoped endpoints require tenant context from session and explicit tenant checks.
- Errors:
  - No internal implementation details in user-facing responses.
  - Standard JSON error shape for API endpoints: `{ "error": { "code": "...", "message": "..." } }`.

## Core endpoint groups

### Authentication & Session
- `GET /auth/login` - Start tenant OIDC login.
- `GET /auth/callback` - Complete OIDC login.
- `POST /auth/superuser/login` - Local superuser login.
- `POST /auth/logout` - End session.

### Tenant and Membership
- `POST /tenants` - Create tenant on first login (idempotent for already-associated users).
- `POST /tenants/{tenantId}/invites` - Create invite (admin only).
- `POST /invites/accept` - Accept invitation (token provided in JSON body, e.g. `{ "token": "..." }`).

### Groups
- `POST /tenants/{tenantId}/groups` - Create group (requires >=1 admin user).
- `POST /tenants/{tenantId}/groups/{groupId}/members` - Add user/sub-group member.
- `DELETE /tenants/{tenantId}/groups/{groupId}/members/{memberId}` - Remove member.
- `POST /tenants/{tenantId}/groups/{groupId}/subgroups` - Attach subgroup with cycle prevention checks.

### Files and Versions
- `POST /tenants/{tenantId}/files` - Upload file and metadata.
- `GET /tenants/{tenantId}/files` - List/search/sort files.
- `GET /tenants/{tenantId}/files/{fileId}` - File metadata details.
- `GET /tenants/{tenantId}/files/{fileId}/download` - Stream file content.
- `GET /tenants/{tenantId}/files/{fileId}/versions` - Version history.
- `POST /tenants/{tenantId}/files/{fileId}/versions/{versionId}/restore` - Restore version.

### Sharing
- `POST /tenants/{tenantId}/files/{fileId}/shares` - Share with users/groups.
- `DELETE /tenants/{tenantId}/files/{fileId}/shares/{shareId}` - Revoke share.
- `POST /tenants/{tenantId}/files/{fileId}/external-shares` - Issue external one-time key.
- `POST /external-access/consume` - Single-use external read-only access (key provided in JSON body, e.g. `{ "key": "..." }`).

### Trash lifecycle
- `GET /tenants/{tenantId}/trash` - List recoverable items.
- `POST /tenants/{tenantId}/trash/{trashItemId}/restore` - Restore with conflict options (`overwrite|cancel|rename`).
- `DELETE /tenants/{tenantId}/trash/{trashItemId}` - Immediate purge (privileged/retention-policy governed).

### Platform operator (superuser only)
- `GET /operator/config` - Read effective platform configuration.
- `PUT /operator/config` - Validate and update runtime config.
- `GET /operator/health` - Aggregated ecosystem/service readiness.

## Contract-level validation rules
- File upload requests enforce configured maximum size and quota constraints before write.
- Group updates that would produce zero admins are rejected.
- Subgroup links that would create cycles are rejected.
- External access key validation checks: hash match, not used, not expired, file not trashed/purged.
- Cross-tenant resource access is denied by default unless explicit share grants permission.

## Contract open items for task planning
- Final path naming for HTMX vs JSON routes (`/ui/...` split vs content negotiation).
- Pagination/filter parameter schema for large file lists.
- Precise audit event schema and retention strategy.
