# Obecno API refresh — 20261017-2353

**Specialist:** api-detail  
**Run:** `20261017-2353-archify`  
**Workspace:** `/Users/hassan/Downloads/obecno` (read-only)  
**Archify root:** `/Users/hassan/Documents/Archify-kit/projects/obecno`  
**Decision:** **REUSE** existing guide — no rebuild  
**Reuse target:** `architecture-obecno-api-guide-20261006-222000/obecno-api-full-guide.html`  
(`manifest.json` → `architecture_entry`)  
**Rebuild folder:** `architecture-obecno-api-20261017-2353/` — **not created**

---

## Verdict

**REUSE.** The Oct 6 API full guide’s shape is unchanged and still matches the mobile client: single-scroll HTML with runtime + screen-flow iframes, detailed auth/employee/manager/more maps, and passport grids driven by `screen-catalog.js`. `EmployeeApiEndpoints`, `ManagerEmployeeApiEndpoints` (+ `ManagerApiRoute`), and `ApiClient` remain the path/method source of truth; catalog entries are screen-scoped summaries, not an exhaustive route registry. Newer manager routes (collection location status, permission field POST, richer device review verbs) and employee modules (leaves, company view, tickets, reference data) are **catalog gaps only**, not guide-structure drift.

---

## Guide shape vs source (match check)

| Guide layer | Artifact | Source alignment |
|---|---|---|
| Section nav + iframes | `obecno-api-full-guide.html` (runtime, 4 screen flows, 5 detailed maps) | Unchanged; 50 screen keys referenced via `data-catalog-keys` |
| Passport / scan grids | `screen-catalog.js` → `OBECNO_SCREEN_CATALOG` | All HTML-referenced keys resolve; paths use `/api/v1/…` matching `AppConstants.apiVersion` + endpoint constants |
| HTTP transport story | `enhanced/runtime.html` (`ApiClient` node) | Matches `lib/core/api/api_client.dart` (verbs, retry, 401/419, cancel, multipart) |
| Employee paths | Catalog + employee maps | `EmployeeApiEndpoints` — string constants under `/auth`, `/employee`, `/attendance`, CMS, reference |
| Manager paths | Catalog + manager maps | `ManagerEmployeeApiEndpoints` — **121** `ManagerApiRoute` operations + legacy `/manager/employee/*`, `/manager/location/*` string aliases |

**Naming note:** Dart uses `ManagerEmployeeApiEndpoints` (not a separate `manager_api_endpoints.dart` class name export); guide documents REST paths, not Dart symbol names — still aligned.

---

## Client stack (verified)

| Piece | Location | Notes |
|---|---|---|
| HTTP client | `lib/core/api/api_client.dart` | `package:http`; GET/POST/PUT/PATCH/DELETE + `postMultipart` |
| Base URL | `lib/core/api/constants.dart` | `OBECNO_BASE_URL` default `https://app.obecno.com/` |
| API prefix | same | `OBECNO_API_VERSION` default `/api/v1` prepended in `_resolve` when absent on path |
| Auth header | `ApiClient._headers` ← `TokenService.authorizationHeader` | JSON `Content-Type` / `Accept`; Bearer when session active |
| Session / 401 | `interceptor.dart` + optional `SessionManager` | 401/419 retry path in `_guard`; 403 session handling |
| Retry / timeout | `retry_interceptor.dart`, `AppConstants` | 5 retries, 500ms base; receive 20s on calls |
| Cancel | `api_cancel_token.dart` | `_runCancellable` races body vs cancel token |
| Response shape | `_normalizeResponse` | Unwraps / JSON-decodes string `data` payloads |

Effective URL: `{baseUrl trimmed}{/api/v1}{path}` → e.g. `https://app.obecno.com/api/v1/auth/login`

---

## Endpoint surface summary

### `EmployeeApiEndpoints` (~38 path templates)

- **Auth:** `/auth/login`, `/logout`, `/me`, `/forgot-password`, `/change-password`
- **Attendance / clock:** `/employee/attendance`, `/details`, `/edit`, `/calendar`; helpers `/attendance/monthly/{employeeId}/{yearMonth}`, `/attendance/summary/{employeeId}`
- **Profile / HR:** `/employee/profile`, `/profile/photo`, `/dashboard`, `/salary`
- **Leaves:** `/employee/leaves`, `/balances`, `/types`, `/apply`; manager-review mirror `/employee/team-leaves`, `/team-leaves/review`
- **Company (employee view):** `/employee/company-profile`, `/company-employees`, `/company-calendar`
- **CMS:** `/terms-and-conditions`, `/privacy-policy`
- **Reference:** `/countries`, `/cities`, `/timezones`
- **Tickets:** `/employee/tickets`, `/show`, `/meta`, `/reply`
- **Devices / policy:** `/employee/devices` (+ `/{id}`, `/{id}/delete`, `/delete`); `/employee/permissions`

### `ManagerEmployeeApiEndpoints` (~98 distinct path templates, 121 typed routes)

- **Org / tabs:** `/manager/dashboard`, `/departments`, `/team-members`, `/team-leaves` (+ `POST …/review`), `/team-calendar`, `/timezones`
- **Team attendance:** `/manager/team-attendance`, `/filters`, `/details`, `/edit`, `POST …/edit/save`
- **Locations:** `GET|POST /manager/locations`; per-id GET/PUT/PATCH/DELETE; schedule, permissions (plural + singular aliases), status (per-id + **collection** `/manager/locations/status`), members, active/inactive, delete fallbacks; legacy `/manager/location/*`
- **Employees:** list/create/invite; per-id profile, edit, status, attendance (+ details), salary, appraisals, leaves (+ POST assign), leave-balances/quota, holidays, calendar, schedule (+ working-days), locations, permissions (+ field POST collection/nested), devices (+ review/status/delete/action variants), photo; legacy `/manager/employee/*`
- **Billing (defined, thin UI in catalog):** `/manager/subscriptions`, `/manager/payments`

Repositories continue to prefer `ManagerApiRoute.method` + `.path` for verb-safe calls; string aliases retained for older repository code.

---

## Catalog gaps (non-blocking; patch `screen-catalog.js` later)

Screen catalog REST paths **cover all primary flows** in the guide maps. Omissions vs Dart catalogs (intentionally screen-level):

1. **Employee modules not on passport cards:** leaves suite, salary, dashboard, company-profile/employees/calendar, team-leaves (employee-side review), ticket show/reply, reference lookups, monthly/summary attendance helpers.
2. **Manager admin / depth:** departments, team-members, team-leaves/calendar, subscriptions/payments, `GET …/employees/create`, `GET …/locations/create`, location permission/status collection routes, employee schedule/working-days (catalog `emp-timing` still summarizes as generic `PUT/PATCH /manager/employees/{id}`), legacy fallback paths, granular device review verbs.
3. **Wording:** book-demo catalog path is placeholder; app uses `POST /employee/tickets` (`BookDemoRepository`).
4. **Auth refresh:** no client route for `/auth/refresh` yet (token refresh noted in services only).

None of these invalidate iframe diagrams or section structure.

---

## Comparison to prior refresh (20261009-2122)

| Delta | Impact on guide |
|---|---|
| `ManagerApiRoute` + more location/employee/device/status routes in Dart | Catalog unchanged; REUSE still valid |
| `ApiClient` cancel + 401 retry behavior | Already reflected in runtime narrative |
| Employee endpoint set | Same groups as Oct 9 refresh; no new guide sections required |

---

## Actions

- [x] Read `employee_api_endpoints.dart`, `manager_api_endpoints.dart`, `api_client.dart`, `constants.dart`
- [x] Cross-check paths against `architecture-obecno-api-guide-20261006-222000/screen-catalog.js` + HTML section keys
- [x] Confirmed guide shape (runtime → screen flows → detailed maps + catalog grids)
- [x] Wrote this report
- [ ] Rebuild API guide tree — **skipped (REUSE)**

**Report path:** `/Users/hassan/Documents/Archify-kit/projects/obecno/runs/20261017-2353-archify/api-refresh.md`
