# Quickstart: Multi-tenant File Storage

## Purpose
Guide implementation and validation for v1 using existing stack patterns (Go backend + Go/HTMX web UI + PostgreSQL + S3-compatible storage) while keeping dependencies minimal.

## Prerequisites
- Go toolchain installed (project-selected version)
- Docker + docker-compose installed
- Access to PostgreSQL and S3-compatible storage (RustFS in local/dev)
- Authelia OIDC provider configured for tenant user auth

## 1) Bootstrap environment
- Create env file with required first-boot values:
  - DB connection (pre-DB boot path)
  - S3 endpoint/bucket/credentials
  - Authelia OIDC settings
  - Superuser bootstrap ID/password
- Start dependencies:
  - `docker compose up -d`
- Verify readiness of PostgreSQL, RustFS, Authelia, SMTP.

## 2) Initialize application runtime config
- On first startup:
  - Seed `PlatformConfiguration` from compose-provided default config file if DB config is absent.
  - Create bootstrap superuser if none exists.
  - Fail startup clearly if required bootstrap env vars are missing/invalid.

## 3) Run tests early
- Unit tests first for:
  - tenant isolation checks
  - group admin invariants
  - subgroup cycle prevention
  - one-time key hash/single-use/expiry logic
  - upload retry policy state transitions
- Then integration tests:
  - S3 upload/download roundtrip
  - PostgreSQL persistence for core entities
  - Authelia login flow and outage behavior

## 4) Validate critical workflows
- First login tenant creation + default admin group assignment
- Invite and join tenant flow
- Upload, list, share, and external one-time read-only access
- Trash restore conflict handling (`overwrite|cancel|rename`)
- Operator config update + validation + audit logging

## 5) Security and compliance checks
- Confirm no raw external share tokens are logged/stored
- Confirm user-facing errors avoid internal details
- Confirm tenant boundary checks on every tenant-scoped endpoint
- Confirm dependency additions (if any) are justified and pinned

## Deliverable checkpoints
- Planning docs complete: `plan.md`, `research.md`, `data-model.md`, `contracts/http-api.md`, `quickstart.md`
- Constitution gates re-checked after design updates
- Ready for `/speckit.tasks`
