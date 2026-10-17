# Obecno API refresh — 20261009-2122

**Specialist:** api-detail  
**Workspace:** `/Users/hassan/Downloads/obecno` (read-only)  
**Decision:** **REUSE** existing guide — no rebuild  
**Reuse target:** `architecture-obecno-api-guide-20261006-222000/obecno-api-full-guide.html`  
(`manifest.json` → `architecture_entry`)  
**Rebuild folder:** `architecture-obecno-api-20261009-2122/` — **not created** (no major gaps)

---

## Verdict

The Oct 6 API full guide already covers mobile runtime, auth / employee / manager / more screen flows, embedded Archify maps, and a screen-keyed passport catalog (`screen-catalog.js`). Source endpoints and client wiring still match that shape. Incremental catalog omissions (leaves, company view, reference lookups, some manager admin routes) are not architecture-level gaps — prefer a later catalog patch over a new guide tree.

---

## Client stack (verified)

| Piece | Location | Notes |
|---|---|---|
| HTTP client | `lib/core/api/api_client.dart` | `package:http`; GET/POST/PUT/PATCH/DELETE + multipart |
| Base URL | `lib/core/api/constants.dart` | `OBECNO_BASE_URL` default `https://app.obecno.com/` |
| API prefix | same | `OBECNO_API_VERSION` default `/api/v1` prepended in `_resolve` |
| Auth header | `ApiClient._headers` ← `TokenService.authorizationHeader` | `"{tokenType} {accessToken}"`, default type `Bearer` (`TokenModel`) |
| Session / 401 | `lib/core/services/interceptor.dart` | `AuthFailureHandler` → refresh attempt → clear session |
| Retry | `lib/core/services/retry_interceptor.dart` | `RetryPolicy`; constants: 5 retries, 500ms base |
| Timeouts | `AppConstants` | connect 15s / receive 20s / send 20s |
| Cancel | `api_cancel_token.dart` | raced against in-flight requests |
| Repos base | `base_repository.dart` / `base_provider.dart` | shared request helpers + stale-call guard |

Effective URL form: `{baseUrl without trailing slash}{/api/v1}{path}`  
e.g. `https://app.obecno.com/api/v1/auth/login`

---

## Endpoint catalogs (source of truth)

1. **`EmployeeApiEndpoints`** — `lib/core/api/employee_api_endpoints.dart`  
   Auth, employee attendance/clock, profile, dashboard, leaves, salary, company view, CMS, team-leaves, reference data, tickets, devices, permissions.

2. **`ManagerEmployeeApiEndpoints`** — `lib/core/api/manager_api_endpoints.dart`  
   Dashboard, departments, team members/leaves/calendar/attendance, locations (+ schedule/permissions/status), employees CRUD/invite/devices/permissions/photo, legacy `/manager/employee/*` and `/manager/location/*` fallbacks, subscriptions, payments.

---

## Key API surfaces (verified via repositories)

### Auth / session
- `POST /auth/login` — email check + full sign-in (`AuthRepository`)
- `GET /auth/me` — session restore / TTL cache in `AuthProvider`
- `POST /auth/forgot-password`, `POST /auth/change-password`
- `GET /employee/permissions` — policy flags after auth
- Logout path defined: `POST /auth/logout` (endpoint present)
- Refresh: `TokenService` notes full rotation needs backend `/auth/refresh` (not a live client route yet)

### Employee / clock / attendance
- `GET|POST /employee/attendance` — clock + history (`ClockAttendanceRepository`, attendance repos)
- `GET /employee/attendance/details`, edit path `/employee/attendance/edit`
- `GET /employee/calendar`
- Helpers: `/attendance/monthly/{id}/{ym}`, `/attendance/summary/{id}`

### Employee profile / more
- `GET|PATCH /employee/profile`, photo `/employee/profile/photo`
- Devices: `GET|POST /employee/devices`, delete variants (`DeviceRepository`)
- Tickets / help / book-demo: `POST /employee/tickets` (+ show/meta/reply) — book demo reuses tickets, not a separate path
- CMS: `/terms-and-conditions`, `/privacy-policy`

### Employee leaves / company / reference (in source; thin or missing in passport catalog)
- Leaves: `/employee/leaves`, balances, types, apply; `/employee/team-leaves` + review
- Company: `/employee/company-profile`, `company-employees`, `company-calendar`
- Lookups: `/countries`, `/cities`, `/timezones`
- `/employee/dashboard`, `/employee/salary`

### Manager
- Overview: `GET /manager/dashboard`, team attendance for live status
- Team attendance: list, filters, details, edit, edit/save
- Employees: list/create/invite, per-user profile/edit/status/locations/schedule/working-days/devices/permissions/photo/salary/leaves/calendar (+ legacy `/manager/employee/*` fallbacks)
- Locations: CRUD, schedule, permissions, status, delete (+ legacy `/manager/location/*`)
- Also defined: departments, team-members, team-leaves(+review), team-calendar, subscriptions, payments

### Repository map
| Repository | Primary catalog |
|---|---|
| `AuthRepository` | Employee auth + permissions |
| `ClockAttendanceRepository` / attendance repos | Employee attendance |
| `ManagerOverviewRepository` | Manager dashboard / team attendance |
| `ManagerAttendanceRepository` | Team + per-employee attendance |
| `ManagerEmployeesRepository` | Employees (+ countries/cities) |
| `ManagerLocationsRepository` | Locations (+ timezones) |
| `ProfileRepository` / `DeviceRepository` / `HelpFeedbackRepository` | More |
| `BookDemoRepository` / `TicketRepository` | Tickets |

---

## Guide comparison (high level)

| Area | Existing guide | Source today | Gap severity |
|---|---|---|---|
| Runtime + ApiClient story | Runtime iframe + passport base note | Unchanged stack | None |
| Auth / login / me / forgot | Catalog + auth maps | Same paths | None |
| Clock / attendance / devices / profile | Catalog dense | Same | None |
| Manager dashboard / team attendance / employees / locations | Catalog + manager maps | Same core routes | None |
| Book demo | Catalog: vague “book-demo endpoint” | Actually `POST /employee/tickets` | Minor (wording) |
| Leaves / salary / company / reference | Mostly absent from catalog | Present in `EmployeeApiEndpoints` | Minor catalog |
| Manager departments / subscriptions / payments / team-leaves / legacy fallbacks | Partially / not listed | Present in manager endpoints | Minor catalog |
| Ticket reply/show | meta + list only in places | Full ticket suite in source | Minor |

**Major gap requiring rebuild?** No. Screen-flow architecture and primary REST surfaces remain accurate; keep `architecture_entry` on the Oct 6 guide.

---

## Actions taken

- [x] Traced `lib/core/api/*`, repositories, `TokenService` / interceptor  
- [x] Diffed against `screen-catalog.js` + guide section structure  
- [x] Wrote this note  
- [ ] Rebuild under `architecture-obecno-api-20261009-2122/` — **skipped**

## Follow-ups (optional, non-blocking)

1. Passport catalog patch: book-demo → `POST /api/v1/employee/tickets`; add leaves / company / reference / ticket reply-show.  
2. When product ships `/auth/refresh`, document it next to `AuthFailureHandler`.
