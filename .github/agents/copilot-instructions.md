# fm-file-cloud Development Guidelines

Auto-generated from all feature plans. Last updated: 2026-03-24

## Active Technologies

- Go (backend + web UI handlers), SQL (PostgreSQL), HTMX for server-driven UI interactions + Go standard library first; PostgreSQL driver, OIDC client integration for Authelia, S3-compatible client, HTMX (frontend library) (001-multi-tenant-file-storage)

## Project Structure

```text
backend/
web/
```

## Commands

```bash
# Build
go build ./...

# Test
go test ./...

# Vet / lint
go vet ./...
```

## Code Style

Go (backend + web UI handlers), SQL (PostgreSQL), HTMX for server-driven UI interactions: Follow standard conventions

## Recent Changes

- 001-multi-tenant-file-storage: Added Go (backend + web UI handlers), SQL (PostgreSQL), HTMX for server-driven UI interactions + Go standard library first; PostgreSQL driver, OIDC client integration for Authelia, S3-compatible client, HTMX (frontend library)

<!-- MANUAL ADDITIONS START -->
<!-- MANUAL ADDITIONS END -->
