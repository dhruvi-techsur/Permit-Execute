---
phase: 01-foundation
plan: "01"
subsystem: infra
tags: [docker, postgres, typeorm, migrations, seed, minio, nestjs, vite]

# Dependency graph
requires: []
provides:
  - Docker Compose multi-service dev stack (postgres:15, minio, backend, frontend)
  - PostgreSQL Phase 1 schema via TypeORM migration (users, password_reset_tokens, refresh_tokens)
  - 4 PostgreSQL enum types (user_role, permit_status, document_status, notification_type)
  - updated_at trigger on users table
  - Idempotent seed with 3 role-based UAT users
  - .env.example documenting all required environment variables
affects:
  - 01-02 (NestJS backend scaffolding — uses docker-compose.yml and DATABASE_URL)
  - 01-03 (Frontend scaffolding — uses docker-compose.yml frontend service)
  - 01-04 (Auth implementation — consumes users, refresh_tokens, password_reset_tokens tables)
  - All subsequent phases that require running stack

# Tech tracking
tech-stack:
  added:
    - postgres:15 (via Docker Compose)
    - minio/minio:RELEASE.2024-01-18T22-51-28Z (via Docker Compose)
    - typeorm (migration interface)
    - bcrypt (password hashing in seed)
  patterns:
    - Docker Compose with healthcheck → depends_on: service_healthy pattern
    - TypeORM migration class with up/down methods
    - Idempotent seed via ON CONFLICT (email) DO UPDATE

key-files:
  created:
    - docker-compose.yml
    - .env.example
    - backend/Dockerfile.dev
    - frontend/Dockerfile.dev
    - backend/src/database/migrations/001_initial_schema.ts
    - backend/src/database/seeds/seed.ts
    - backend/src/database/seeds/run-seed.ts
  modified: []

key-decisions:
  - "postgres:15 pinned (not latest) to ensure deterministic sandbox cache hits"
  - "minio image pinned to RELEASE.2024-01-18T22-51-28Z for same reason"
  - "Seed uses ON CONFLICT DO UPDATE — safe for repeated compose restarts with named volumes"
  - "Seed does NOT overwrite password_hash in upsert — preserves manually set passwords"
  - "backend depends_on postgres with condition: service_healthy — prevents premature migration runs"
  - "DATABASE_URL references service name 'postgres' (not localhost) for compose network routing"

patterns-established:
  - "Compose pattern: DB healthcheck → app depends_on service_healthy → migrate → seed → serve"
  - "Migration pattern: TypeORM MigrationInterface with queryRunner.query() for raw DDL"
  - "Seed pattern: upsert-safe via ON CONFLICT, never overwrites sensitive fields"

# Metrics
duration: 2min
completed: 2026-08-06
---

# Phase 1 Plan 01: Dev Stack Bootstrap Summary

**PostgreSQL 15 + MinIO dev stack via Docker Compose with full Phase 1 schema (users, password_reset_tokens, refresh_tokens, 4 enums, 8 indexes, updated_at trigger) and idempotent 3-role seed**

## Performance

- **Duration:** 2 min
- **Started:** 2026-08-06T00:34:56Z
- **Completed:** 2026-08-06T00:36:35Z
- **Tasks:** 2 completed
- **Files modified:** 7

## Accomplishments

- Multi-service Docker Compose stack: postgres:15 (pinned, healthcheck), minio (pinned), backend (depends_on service_healthy), frontend — all validated with `docker compose config`
- Complete Phase 1 TypeORM migration: users table with UUID PK, user_role enum, all required columns; password_reset_tokens and refresh_tokens with FK cascade; 8 performance indexes; updated_at trigger
- Idempotent seed script: 3 UAT users (applicant, reviewer, admin) using `ON CONFLICT (email) DO UPDATE` — safe for repeated compose restarts
- .env.example documents all 14 required environment variables from TechArch §7.4 with dev defaults

## Task Commits

Each task was committed atomically:

1. **Task 1: Docker Compose dev stack with health checks** - `c0ca76d` (chore)
2. **Task 2: TypeORM migration and idempotent seed** - `3cdbb71` (feat)

**Plan metadata:** (docs commit — see below)

## Files Created/Modified

- `docker-compose.yml` — 4-service dev stack: postgres:15, minio, backend, frontend with healthchecks and volume mounts
- `.env.example` — All required env vars documented with dev-safe defaults
- `backend/Dockerfile.dev` — Minimal node:20-alpine dev image for backend
- `frontend/Dockerfile.dev` — Minimal node:20-alpine dev image with --host flag for sandbox preview proxy
- `backend/src/database/migrations/001_initial_schema.ts` — TypeORM migration: 3 tables, 4 enums, 8 indexes, updated_at trigger
- `backend/src/database/seeds/seed.ts` — Idempotent upsert of 3 role-based UAT users
- `backend/src/database/seeds/run-seed.ts` — CLI entry point for seed invocation

## Decisions Made

- **postgres:15 pinned** (not `:latest`) — sandbox pre-seeds common pinned images, reduces registry pull latency and rate limit risk
- **minio image pinned** to specific release date tag — same reason
- **Seed upsert excludes password_hash** — `ON CONFLICT DO UPDATE` only updates `full_name`, `role`, `updated_at`; never overwrites passwords (security: T-01-02 mitigation)
- **backend depends_on postgres with condition: service_healthy** — eliminates race condition where app starts before DB is ready to accept connections
- **DATABASE_URL uses service name `postgres`** not `localhost` — required for Docker Compose internal networking; localhost would fail inside the container

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered

None.

## User Setup Required

None - no external service configuration required. All services are self-hosted via Docker Compose with dev credentials.

## Next Phase Readiness

- Stack contract established: all subsequent plans can reference `docker-compose.yml` and the schema
- Ready for Plan 01-02: NestJS backend scaffolding (will add package.json, tsconfig, main.ts, app module)
- Ready for Plan 01-03: React frontend scaffolding (will add Vite config, React Router, Tailwind)
- Plan 01-04 (auth implementation) has all required DB tables: users, refresh_tokens, password_reset_tokens

---
*Phase: 01-foundation*
*Completed: 2026-08-06*

## Self-Check: PASSED

- [x] `docker-compose.yml` exists on disk
- [x] `.env.example` exists on disk
- [x] `backend/Dockerfile.dev` exists on disk
- [x] `frontend/Dockerfile.dev` exists on disk
- [x] `backend/src/database/migrations/001_initial_schema.ts` exists on disk
- [x] `backend/src/database/seeds/seed.ts` exists on disk
- [x] `backend/src/database/seeds/run-seed.ts` exists on disk
- [x] Task 1 commit `c0ca76d` exists in git log
- [x] Task 2 commit `3cdbb71` exists in git log
- [x] `docker compose config --quiet` → exit 0 (COMPOSE CONFIG VALID)
- [x] Migration contains all 3 tables (22 matches for users|password_reset_tokens|refresh_tokens)
- [x] Seed has ON CONFLICT idempotency (1 match)
- [x] No stubs or TODOs in created files

## Known Stubs

None found. All files are complete implementations. Note: `backend/Dockerfile.dev` and `frontend/Dockerfile.dev` are intentionally minimal placeholders that will be replaced by proper Dockerfiles in Plans 01-02 and 01-03 respectively — this is per-plan design, not a stub.
