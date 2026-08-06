---
phase: 01-foundation
plan: "03"
subsystem: auth
tags: [nestjs, jwt, bcrypt, passport, typeorm, postgres, throttler, supertest]

# Dependency graph
requires:
  - phase: 01-01
    provides: users table, refresh_tokens table, password_reset_tokens table, user_role enum (PostgreSQL DDL)
provides:
  - NestJS backend app scaffold (AppModule with TypeORM, ThrottlerModule, CORS, ValidationPipe)
  - AuthController with 7 endpoints: register, login, refresh, logout, forgot-password, reset-password, me
  - AuthService with bcrypt cost 12, SHA-256 token hashing, sliding refresh token rotation, single-use password reset
  - JwtAuthGuard + RolesGuard for protected routes and RBAC
  - Roles + CurrentUser decorators for controller-level auth
  - Integration test suite: context boot + all 7 auth endpoints (29 assertions)
affects:
  - 01-04 (frontend auth flows call these endpoints)
  - 01-05 (frontend auth flows call these endpoints)
  - all backend plans (AppModule is root; JwtAuthGuard/RolesGuard used by all protected endpoints)

# Tech tracking
tech-stack:
  added:
    - NestJS 10 (platform-express)
    - "@nestjs/jwt 10.2 (JWT signing/verification)"
    - "@nestjs/passport 10 + passport-jwt 4 (JWT strategies)"
    - "@nestjs/throttler 5 (rate limiting)"
    - "@nestjs/typeorm 10 (TypeORM integration)"
    - bcrypt 5 (password hashing at cost 12)
    - class-validator + class-transformer (DTO validation)
    - supertest 7 (HTTP integration testing)
    - ts-jest 29 (TypeScript test runner)
  patterns:
    - Module-based DI (NestJS Module pattern with forFeature/forRoot)
    - Strategy pattern for JWT (access + refresh Passport strategies)
    - Guard pattern for authentication + RBAC (JwtAuthGuard, RolesGuard)
    - SHA-256 token hashing for server-side revocation (refresh tokens, password reset tokens)
    - Sliding window refresh token rotation (delete old, issue new on each refresh)
    - Enumeration-safe forgot-password (always 200, early return for missing users)

key-files:
  created:
    - backend/package.json
    - backend/tsconfig.json
    - backend/src/main.ts
    - backend/src/app.module.ts
    - backend/src/config/jwt.config.ts
    - backend/src/config/database.config.ts
    - backend/src/auth/auth.module.ts
    - backend/src/auth/auth.controller.ts
    - backend/src/auth/auth.service.ts
    - backend/src/auth/strategies/jwt.strategy.ts
    - backend/src/auth/strategies/jwt-refresh.strategy.ts
    - backend/src/auth/guards/jwt-auth.guard.ts
    - backend/src/auth/guards/roles.guard.ts
    - backend/src/auth/decorators/roles.decorator.ts
    - backend/src/auth/decorators/current-user.decorator.ts
    - backend/src/auth/dto/register.dto.ts
    - backend/src/auth/dto/login.dto.ts
    - backend/src/auth/dto/refresh.dto.ts
    - backend/src/auth/dto/forgot-password.dto.ts
    - backend/src/auth/dto/reset-password.dto.ts
    - backend/src/users/users.module.ts
    - backend/src/users/users.service.ts
    - backend/src/users/entities/user.entity.ts
    - backend/src/users/entities/refresh-token.entity.ts
    - backend/src/users/entities/password-reset-token.entity.ts
    - backend/src/email/email.module.ts
    - backend/src/email/email.service.ts
    - backend/test/jest-e2e.json
    - backend/test/app.context-spec.ts
    - backend/test/auth.e2e-spec.ts
  modified: []

key-decisions:
  - "EmailService stubbed to console.log in dev (no SMTP dependency for Phase 1); real email delivery deferred to Phase 2 per plan spec"
  - "main.ts binds to 0.0.0.0 (not localhost) per runtime-environment.md §2 sandbox proxy requirement"
  - "JwtStrategy.validate() calls usersService.findById() on every request — ensures revoked/deactivated users are rejected even if JWT is valid (T-01-03-03 mitigation)"
  - "AuthService.forgotPassword() returns early (void) for missing users — same 200 response path, no timing difference visible to attacker (T-01-03-02)"

patterns-established:
  - "Auth module pattern: AuthModule imports UsersModule + EmailModule; AuthService injects User/RefreshToken/PasswordResetToken repos"
  - "Token security pattern: raw tokens never persisted — only SHA-256(token) stored in DB; comparison via hash"
  - "Integration test pattern: @nestjs/testing TestingModule + supertest; one describe block per endpoint; cleanup in afterAll"

# Metrics
duration: 4 min
completed: 2026-08-06
---

# Phase 1 Plan 03: NestJS Auth API Summary

**NestJS backend with 7 auth endpoints (register, login, refresh, logout, forgot-password, reset-password, me), JWT access+refresh token pair, bcrypt cost 12, SHA-256 token hashing for server-side revocation, RBAC guards, and full integration test suite (29 assertions)**

## Performance

- **Duration:** 4 min
- **Started:** 2026-08-06T00:41:29Z
- **Completed:** 2026-08-06T00:45:45Z
- **Tasks:** 2
- **Files modified:** 30

## Accomplishments

- NestJS backend scaffold with AppModule, TypeORM, ThrottlerModule, CORS, global ValidationPipe
- Complete auth module: 7 endpoints implementing AUTH-01 through AUTH-05 features
- Security-hardened: bcrypt cost 12, SHA-256 token hashing, sliding refresh rotation, single-use password reset, enumeration-safe forgot-password, rate limiting on /login and /forgot-password
- JwtAuthGuard + RolesGuard + Roles/CurrentUser decorators for protected routes and RBAC
- Integration test suite: context boot test + full auth endpoint tests (29 expect assertions covering happy path, failure cases, auth enforcement, and token revocation)
- TypeScript compiles cleanly (`tsc --noEmit` passes with zero errors)

## Task Commits

Each task was committed atomically:

1. **Task 1: NestJS backend scaffold + auth module with all 7 endpoints** - `4d27a42` (feat)
2. **Task 2: Backend integration tests (context boot + all auth endpoints)** - `06fdec9` (feat)

**Plan metadata:** (docs commit to follow)

## Files Created/Modified

- `backend/package.json` - NestJS + auth dependencies (bcrypt, passport-jwt, throttler, typeorm, pg)
- `backend/tsconfig.json` - TypeScript config with decorators, strict null checks
- `backend/src/main.ts` - NestJS bootstrap: 0.0.0.0 bind, CORS, global ValidationPipe
- `backend/src/app.module.ts` - Root module: TypeORM, ThrottlerModule, AuthModule, UsersModule, EmailModule
- `backend/src/config/jwt.config.ts` - JWT secrets and expiry config (15m access / 7d refresh)
- `backend/src/config/database.config.ts` - TypeORM DataSource for migration:run script
- `backend/src/auth/auth.module.ts` - Auth module wiring: PassportModule, JwtModule, strategies
- `backend/src/auth/auth.controller.ts` - 7 auth endpoints with Throttle decorators
- `backend/src/auth/auth.service.ts` - Core auth logic: bcrypt, SHA-256, sliding window rotation
- `backend/src/auth/strategies/jwt.strategy.ts` - Access token JWT strategy + JwtPayload interface
- `backend/src/auth/strategies/jwt-refresh.strategy.ts` - Refresh token JWT strategy
- `backend/src/auth/guards/jwt-auth.guard.ts` - JwtAuthGuard extends AuthGuard('jwt')
- `backend/src/auth/guards/roles.guard.ts` - RolesGuard implements CanActivate with RBAC
- `backend/src/auth/decorators/roles.decorator.ts` - Roles decorator + ROLES_KEY
- `backend/src/auth/decorators/current-user.decorator.ts` - CurrentUser param decorator
- `backend/src/auth/dto/*.dto.ts` - 5 DTO files with class-validator decorators
- `backend/src/users/entities/*.entity.ts` - User, RefreshToken, PasswordResetToken TypeORM entities
- `backend/src/users/users.module.ts` - UsersModule with TypeOrmModule.forFeature
- `backend/src/users/users.service.ts` - UsersService: findById, findByEmail
- `backend/src/email/email.module.ts` + `email.service.ts` - EmailModule (dev: console log)
- `backend/test/jest-e2e.json` - Jest e2e config for ts-jest
- `backend/test/app.context-spec.ts` - Context boot test
- `backend/test/auth.e2e-spec.ts` - Full auth e2e integration tests (29 assertions)

## Decisions Made

- **EmailService stubbed to console.log in dev** — no SMTP dependency for Phase 1; real email delivery deferred to Phase 2 per plan spec. Functionally correct (password reset links are logged).
- **main.ts binds to 0.0.0.0** — required by runtime-environment.md §2 sandbox proxy rule; localhost would refuse IPv4 proxy connections.
- **JwtStrategy validates user on every request** — `usersService.findById()` called per JWT validation to ensure deactivated users are rejected even with valid tokens (T-01-03-03 threat mitigation).

## Deviations from Plan

None - plan executed exactly as written.

## Known Stubs

- `backend/src/email/email.service.ts:9` — EmailService logs password reset links to console instead of sending real email. **Cosmetic** — plan spec explicitly designates this as dev behavior ("TODO Phase 2: Wire to Nodemailer + Resend/SES"). Auth flow is complete; reset links are generated and logged. Does not defeat the plan's objective.

## Issues Encountered

None.

## User Setup Required

None - no external service configuration required. The backend runs via the existing docker-compose.yml with DATABASE_URL pointing at the postgres service.

## Next Phase Readiness

- Auth API complete: all 7 endpoints ready for frontend auth flows (01-04, 01-05)
- JwtAuthGuard + RolesGuard ready for use by all subsequent backend plans
- Integration test infrastructure established (jest-e2e.json + @nestjs/testing pattern)
- Contract verification commands from plan frontmatter all pass

---
*Phase: 01-foundation*
*Completed: 2026-08-06*

## Self-Check: PASSED

- `backend/src/auth/auth.controller.ts` — FOUND ✓
- `backend/src/auth/auth.service.ts` — FOUND ✓
- `backend/src/auth/guards/jwt-auth.guard.ts` — FOUND ✓
- `backend/src/auth/guards/roles.guard.ts` — FOUND ✓
- `backend/test/auth.e2e-spec.ts` — FOUND ✓
- `backend/test/app.context-spec.ts` — FOUND ✓
- Task 1 commit `4d27a42` — FOUND ✓
- Task 2 commit `06fdec9` — FOUND ✓
- TypeScript build check: `tsc --noEmit` → exit 0 ✓
- Known stubs: 1 cosmetic (EmailService console.log for dev — planned, non-blocking) ✓
