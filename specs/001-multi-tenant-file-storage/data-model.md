# Data Model: Multi-tenant File Storage

## Entity: Tenant
- Fields:
  - `id` (UUID, PK)
  - `name` (string, required)
  - `residency_region` (string, required)
  - `created_at`, `updated_at` (timestamp)
- Relationships:
  - has many `User`, `Group`, `Folder`, `File`
- Validation:
  - `residency_region` must be from configured allowed regions

## Entity: User (Tenant user)
- Fields:
  - `id` (UUID, PK)
  - `tenant_id` (UUID, FK Tenant)
  - `email` (string, unique per tenant)
  - `oidc_subject` (string, unique global)
  - `role` (enum: Member|Editor|Admin)
  - `status` (enum: active|suspended|removed)
  - `created_at`, `updated_at`
- Relationships:
  - belongs to `Tenant`
  - many-to-many with `Group` via `GroupMembership`
- Validation:
  - must belong to at least one group

## Entity: SuperuserAccount
- Fields:
  - `id` (UUID, PK)
  - `email` (string, unique)
  - `password_hash` (string)
  - `must_rotate_password` (bool)
  - `status` (enum: active|disabled)
  - `created_at`, `updated_at`
- Relationships:
  - belongs to `SuperuserGroup` membership model
- Validation:
  - system must maintain at least one active superuser

## Entity: SuperuserGroup
- Fields:
  - `id` (UUID, PK)
  - `name` (string, fixed logical group)
  - `created_at`
- Relationships:
  - has many `SuperuserAccount`

## Entity: Group
- Fields:
  - `id` (UUID, PK)
  - `tenant_id` (UUID, FK Tenant)
  - `name` (string, required)
  - `created_at`, `updated_at`
- Relationships:
  - belongs to `Tenant`
  - many-to-many with `User` and `Group` via `GroupMembership`
- Validation:
  - at least one admin user always required

## Entity: GroupMembership
- Fields:
  - `id` (UUID, PK)
  - `tenant_id` (UUID, FK Tenant)
  - `parent_group_id` (UUID, FK Group)
  - `member_type` (enum: user|group)
  - `member_user_id` (nullable UUID FK User)
  - `member_group_id` (nullable UUID FK Group)
  - `role` (string/enum)
  - `is_group_admin` (bool, for user members)
  - `created_at`
- Validation:
  - exactly one of `member_user_id` / `member_group_id` must be set
  - cycle prevention via closure table checks before subgroup insertion

## Entity: GroupClosure (for cycle prevention)
- Fields:
  - `ancestor_group_id` (UUID, FK Group)
  - `descendant_group_id` (UUID, FK Group)
  - `depth` (int)
- Constraints:
  - composite PK (`ancestor_group_id`, `descendant_group_id`)
- Validation:
  - insertion rejected if proposed edge would create reverse ancestry

## Entity: Folder
- Fields:
  - `id` (UUID, PK)
  - `tenant_id` (UUID, FK Tenant)
  - `name` (string)
  - `parent_folder_id` (nullable UUID FK Folder)
  - `owner_user_id` (UUID FK User)
  - `created_at`, `updated_at`

## Entity: File
- Fields:
  - `id` (UUID, PK)
  - `tenant_id` (UUID, FK Tenant)
  - `folder_id` (nullable UUID FK Folder)
  - `name` (string)
  - `mime_type` (string)
  - `size_bytes` (bigint)
  - `storage_key` (string)
  - `uploader_user_id` (UUID FK User)
  - `current_version_id` (nullable UUID FK FileVersion)
  - `ttl_at` (nullable timestamp)
  - `status` (enum: active|trash|purged)
  - `created_at`, `updated_at`
- Relationships:
  - has many `FileVersion`, `FileTag`, `FileShare`, `TrashItem`

## Entity: FileVersion
- Fields:
  - `id` (UUID, PK)
  - `file_id` (UUID, FK File)
  - `version_number` (int)
  - `storage_key` (string)
  - `size_bytes` (bigint)
  - `uploader_user_id` (UUID FK User)
  - `created_at`
  - `change_description` (nullable string)

## Entity: Tag and FileTag
- `Tag` fields:
  - `id` (UUID, PK)
  - `tenant_id` (UUID, FK Tenant)
  - `name` (string)
- `FileTag` fields:
  - `file_id` (UUID FK File)
  - `tag_id` (UUID FK Tag)
- Constraints:
  - unique (`tenant_id`, `name`) for tag labels
  - unique (`file_id`, `tag_id`)

## Entity: TrashItem
- Fields:
  - `id` (UUID, PK)
  - `tenant_id` (UUID, FK Tenant)
  - `file_id` (UUID, FK File)
  - `deletion_reason` (enum: deleted|ttl_expired|overwrite_conflict)
  - `deleted_at` (timestamp)
  - `recoverable_until` (timestamp)
  - `status` (enum: recoverable|restored|purged)

## Entity: FileShare
- Fields:
  - `id` (UUID, PK)
  - `tenant_id` (UUID, FK Tenant)
  - `file_id` (UUID, FK File)
  - `recipient_type` (enum: user|group)
  - `recipient_user_id` (nullable UUID FK User)
  - `recipient_group_id` (nullable UUID FK Group)
  - `permission_level` (enum: view|comment|edit|admin)
  - `shared_by_user_id` (UUID FK User)
  - `created_at`, `updated_at`

## Entity: ExternalFileAccessKey
- Fields:
  - `id` (UUID, PK)
  - `file_id` (UUID, FK File)
  - `recipient_email` (string)
  - `token_hash` (string)
  - `token_fingerprint` (string)
  - `expires_at` (timestamp)
  - `used_at` (nullable timestamp)
  - `status` (enum: active|used|expired|revoked)
  - `created_at`
- Validation:
  - raw token never persisted
  - deny if file is trash/purged

## Entity: PlatformConfiguration
- Fields:
  - `id` (UUID, PK)
  - `s3_endpoint`, `s3_bucket`, `s3_region` (string)
  - `db_dsn_runtime` (string, encrypted/secured storage)
  - `smtp_host`, `smtp_port`, `smtp_username`, `smtp_secret_ref`
  - `trash_retention_days` (int)
  - `external_key_validity_days` (int, default 5)
  - `updated_by_superuser_id` (UUID FK SuperuserAccount)
  - `last_validated_at`, `updated_at`

## Entity: AuditLog
- Fields:
  - `id` (UUID, PK)
  - `tenant_id` (nullable UUID)
  - `actor_type` (enum: tenant_user|superuser|system|external)
  - `actor_id` (nullable UUID/string)
  - `event_type` (string)
  - `resource_type` (string)
  - `resource_id` (nullable string)
  - `metadata_json` (jsonb)
  - `created_at` (timestamp)

## State transitions (selected)
- File lifecycle: `active -> trash -> purged`; `trash -> active` via restore.
- External key lifecycle: `active -> used|expired|revoked` (terminal states).
- Superuser bootstrap: `not_bootstrapped -> bootstrapped(must_rotate_password=true) -> active_rotated`.
