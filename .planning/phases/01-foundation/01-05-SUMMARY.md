---
phase: 01-foundation
plan: "05"
subsystem: auth
tags: [zustand, axios, react-router, jwt, rbac, playwright]

# Dependency graph
requires:
  - phase: 01-03
    provides: "NestJS backend auth endpoints (login/register/refresh/logout/forgot-password/reset-password)"
  - phase: 01-04
    provides: "Auth page UI components (LoginPage, RegisterPage, ForgotPasswordPage, ResetPasswordPage)"
  - phase: 01-02
    provides: "Design system tokens and UI component library"
provides:
  - "Zustand auth store (useAuthStore) with full auth actions and token management"
  - "Axios apiClient with request/response interceptors and auto-refresh on 401"
  - "ProtectedRoute and RoleRoute guards for RBAC"
  - "Complete React Router routing tree with nested protected routes"
  - "AppShell layout (Sidebar + Header) with role-appropriate navigation"
  - "Playwright e2e test suite for full auth flow (AUTH-01 through AUTH-05)"
affects: ["02-application-submission", "03-review-workflow", "04-dashboard", "05-admin"]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "Zustand store with in-memory accessToken + localStorage refreshToken (TechArch §5.1)"
    - "Axios interceptor pattern: request attaches Bearer, response auto-refreshes on 401 with request queue"
    - "ProtectedRoute + RoleRoute as React Router layout route wrappers"
    - "AppShell as nested layout with Sidebar + Header + Outlet"
    - "Lazy circular-dep resolution: axios.ts uses require() for auth.store to avoid circular imports"

key-files:
  created:
    - frontend/src/store/auth.store.ts
    - frontend/src/lib/api.ts
    - frontend/src/lib/axios.ts
    - frontend/src/router/index.tsx
    - frontend/src/router/ProtectedRoute.tsx
    - frontend/src/router/RoleRoute.tsx
    - frontend/src/layout/AppShell.tsx
    - frontend/src/layout/Sidebar.tsx
    - frontend/src/layout/Header.tsx
    - frontend/src/pages/DashboardPage.tsx
    - frontend/e2e/auth-flow.spec.ts
  modified:
    - frontend/src/App.tsx
    - frontend/src/auth/LoginPage.tsx
    - frontend/src/auth/RegisterPage.tsx
    - frontend/src/auth/ForgotPasswordPage.tsx
    - frontend/src/auth/ResetPasswordPage.tsx

key-decisions:
  - "Circular dependency resolved via lazy require() in axios.ts interceptors (avoids ESM circular import issue)"
  - "refreshAccessToken is called through Zustand store from axios interceptor, not direct api.ts call — cleaner coupling"
  - "DashboardPage left as placeholder with 'Phase 4' note; all protected routes point to it until Phase 4"

patterns-established:
  - "Auth store pattern: useAuthStore for global auth state, localStorage for refreshToken only"
  - "Route guard pattern: ProtectedRoute (auth) + RoleRoute (RBAC) as layout routes in React Router v6"
  - "AppShell pattern: Sidebar (fixed-width) + Header (sticky) + scrollable main content Outlet"

# Metrics
duration: 4 min
completed: 2026-08-06
---

# Phase 1 Plan 05: Auth State, Routing, and App Shell Summary

**Zustand auth store with JWT token management, Axios interceptors for auto-refresh on 401, React Router protected routes (ProtectedRoute + RoleRoute), AppShell layout with role-aware Sidebar + Header, and Playwright e2e test suite for the complete auth flow.**

## Performance

- **Duration:** 4 min
- **Started:** 2026-08-06T00:49:37Z
- **Completed:** 2026-08-06T00:54:04Z
- **Tasks:** 2
- **Files modified:** 16

## Accomplishments

- Zustand auth store implements all auth actions (login/register/logout/forgotPassword/resetPassword/refreshAccessToken/clearAuth/initialize) with accessToken in memory and refreshToken in localStorage
- Axios apiClient with request interceptor (Bearer token attachment) and response interceptor (401 auto-refresh with concurrent request queue to prevent multiple refresh calls)
- ProtectedRoute and RoleRoute as React Router v6 layout route wrappers enforcing authentication and RBAC
- AppShell layout with Sidebar (role-appropriate nav, user info, Logout) and Header (title, notification bell, avatar dropdown)
- All 4 auth pages wired to useAuthStore actions replacing TODO stubs, with navigate on success
- Playwright e2e suite with 8 tests covering: login→dashboard, sidebar nav per role, RBAC redirect, logout+back-button, registration, unauthenticated redirect, forgot password, session persistence

## Task Commits

Each task was committed atomically:

1. **Task 1: Zustand auth store + Axios interceptors + React Router tree** - `a922e39` (feat)
2. **Task 2: App shell (Sidebar + Header) + Playwright e2e auth tests** - `f60c5cb` (feat)

**Plan metadata:** (docs commit follows)

## Files Created/Modified

- `frontend/src/store/auth.store.ts` - Zustand store with all auth actions, token management
- `frontend/src/lib/api.ts` - Typed API functions for all auth endpoints
- `frontend/src/lib/axios.ts` - Axios instance with Bearer request interceptor + 401 auto-refresh response interceptor
- `frontend/src/router/index.tsx` - Complete React Router tree with all routes from UX-Mockup Navigation Map
- `frontend/src/router/ProtectedRoute.tsx` - Auth guard redirecting unauthenticated users to /login
- `frontend/src/router/RoleRoute.tsx` - RBAC guard redirecting wrong-role users to /dashboard
- `frontend/src/layout/AppShell.tsx` - Full viewport layout: Sidebar + Header + scrollable main content
- `frontend/src/layout/Sidebar.tsx` - Role-aware nav, user avatar/name/role, Settings link, Logout button
- `frontend/src/layout/Header.tsx` - Page title, notification bell placeholder, user avatar dropdown
- `frontend/src/pages/DashboardPage.tsx` - Role-appropriate welcome with skeleton while loading
- `frontend/e2e/auth-flow.spec.ts` - 8 Playwright e2e tests for full auth flow (AUTH-01 through AUTH-05)
- `frontend/src/App.tsx` - Updated to mount Router component
- `frontend/src/auth/LoginPage.tsx` - Wired to useAuthStore().login() with navigate to /dashboard
- `frontend/src/auth/RegisterPage.tsx` - Wired to useAuthStore().register() with navigate to /dashboard
- `frontend/src/auth/ForgotPasswordPage.tsx` - Wired to useAuthStore().forgotPassword() (always shows success)
- `frontend/src/auth/ResetPasswordPage.tsx` - Wired to useAuthStore().resetPassword() with redirect to /login

## Decisions Made

- **Circular dependency resolution:** axios.ts uses `require('../store/auth.store')` (lazy CommonJS-style require) inside interceptor callbacks to avoid ESM circular import — Vite/browser handles this at runtime correctly
- **refreshAccessToken via store, not api.ts directly:** The axios interceptor calls `useAuthStore.getState().refreshAccessToken()` rather than calling `api.refresh()` directly, keeping token state management centralized in the store
- **DashboardPage as universal placeholder:** All placeholder protected routes (/review/queue, /admin/*, etc.) render DashboardPage until their respective phases implement them

## Deviations from Plan

None - plan executed exactly as written.

## Known Stubs

- `frontend/src/pages/DashboardPage.tsx` — "Dashboard content coming in Phase 4" — **Cosmetic** (by design per plan spec; placeholder intentional)
- `frontend/src/layout/Header.tsx` — Notification bell with no handler — **Cosmetic** (Phase 3 feature placeholder per plan)
- `frontend/src/layout/Sidebar.tsx` — Settings link navigates to /settings (no settings page yet) — **Cosmetic** (Phase 5 feature)

## Issues Encountered

None - build succeeded on first attempt. TypeScript type check clean. Vite build produces 378KB JS bundle with only informational notice about dynamic import pattern (expected, by design for circular dep avoidance).

## User Setup Required

None - no external service configuration required for this plan.

## Next Phase Readiness

- Complete auth feature loop: users can register, login, and logout with working token refresh
- Protected routing enforces authentication; RBAC route guards enforce role access
- AppShell layout ready for all future phase feature pages to plug into via React Router Outlet
- Phase 1 (Foundation) is complete — all 5 plans executed
- Ready for Phase 2: Application Submission

---
*Phase: 01-foundation*
*Completed: 2026-08-06*

## Self-Check: PASSED

- ✅ `frontend/src/store/auth.store.ts` — exists
- ✅ `frontend/src/lib/axios.ts` — exists
- ✅ `frontend/src/lib/api.ts` — exists
- ✅ `frontend/src/router/index.tsx` — exists
- ✅ `frontend/src/router/ProtectedRoute.tsx` — exists
- ✅ `frontend/src/router/RoleRoute.tsx` — exists
- ✅ `frontend/src/layout/AppShell.tsx` — exists
- ✅ `frontend/src/layout/Sidebar.tsx` — exists
- ✅ `frontend/src/layout/Header.tsx` — exists
- ✅ `frontend/src/pages/DashboardPage.tsx` — exists
- ✅ `frontend/e2e/auth-flow.spec.ts` — exists
- ✅ Task commits a922e39 and f60c5cb present in git log
- ✅ Build check: `npm run build` → exit 0 (tsc + vite build succeeded)
- ✅ Known Stubs section present — no blocking stubs
