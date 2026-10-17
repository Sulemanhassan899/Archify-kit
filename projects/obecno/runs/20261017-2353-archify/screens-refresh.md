# Screens refresh spot-check — run 20261017-2353-archify

**Specialist:** auth-screens + employee-manager-screens  
**Workspace (read-only):** `/Users/hassan/Downloads/obecno` @ `77242d1796dab8196266fe0ccdccf4e0a266687b`  
**Existing guides:** `architecture-obecno-screens-20261006-213714/` (pinned `674dbc537595bdbcaba8142674b813cf27ffe7d2`), `architecture-obecno-detailed-20261006-215549/`

---

## Verdict

| Pack | Decision | Rationale |
|------|----------|-----------|
| **auth-screens** | **REUSE** | Cold-start graph (splash → onboarding/login → permissions → role shells, join deep link, device blocked) still matches `app_routes.dart` and auth presentation files. |
| **employee-screens** | **REBUILD (incremental patch)** | Tab shell + attendance sheets still accurate; **entire employee leaves stack and shared leave sheets are absent** from Oct 2026 maps. |
| **manager-screens** + **detailed manager-people** | **REBUILD (incremental patch)** | Five-tab shell, directories, location setup, attendance filters/sheets still match; **Overview action grid, `Manager_leaves`, and leave sheet flows are missing or stale**. |
| **more-screens** | **REUSE** | Unchanged relative to `ProfileSettingsScreen` hub. |

**Overall for this run:** **REUSE auth + more; patch-rebuild employee and manager screen diagrams** (do not regenerate tab shells from scratch—extend candidates with leaves subgraph).

---

## Auth — spot-check vs `auth-screens/candidate.json`

| Area | Guide | Repo today | Delta |
|------|-------|------------|-------|
| Routes | `/`, `/onboarding`, `/login`, `/login/password`, `/enable_permissions`, `/employee_nav`, `/manager_nav`, `/youve_joined`, `/device_blocked`, `/bookdemo` | Same in `lib/core/routes/app_routes.dart` | None material |
| Login flow | Email → password → permissions → role home | `login_email.dart`, `login_pass.dart`, `enable_permission.dart` | None material |
| Join | Deep link → You've joined → login | `youve_joined_screen.dart` | None material |
| OTP | Not in diagram | `lib/features/auth/presentation/login_otp.dart` exists | **Minor:** screen not registered on `GoRouter` (orphan / future path)—optional footnote only |

---

## Employee module — spot-check vs `employee-screens/` + `detailed/employee/`

### Still accurate (REUSE)

- Four tabs: Clock, Attendance, Alerts, More (`employee_nav.dart`).
- Attendance overlays: day details, add/edit punch, holiday, month picker (as documented).
- Attendance day tap can open **`AttendanceDayOverviewSheet`** (`attendence_screen.dart`) — guide lists generic day sheet; name differs but role is “day detail overlay.”

### Missing from guides (must add)

**Pushed screens** (`lib/features/employee_module/leaves/presentation/screens/`):

| Screen | Entry points (code) |
|--------|---------------------|
| `LeavesScreen` | `ClockLeavesCard` on Clock tab; manager Overview “My Leaves” |
| `LeaveHistoryScreen` | From `LeavesScreen` |
| `LeaveTypesScreen` | From `LeavesScreen`; also from manager `TeamLeavesScreen` |
| `HolidaysCalendarScreen` | From `LeavesScreen` / `TeamLeavesScreen` |

**Shared leave sheets** (`lib/shared/bottom_sheets/leave_sheets/`):

- `apply_leave_sheet.dart` — employee apply + manager apply-on-behalf
- `leave_detail_sheet.dart` — employee request detail
- `leave_type_picker_sheet.dart`, `leave_status_filter_sheet.dart`, `leave_history_date_picker_sheet.dart`
- `attendance_day_overview_sheet.dart` — bridges Attendance ↔ apply leave
- `manager_leave_detail_sheet.dart` — manager review/approve (used from team leaves + employee attendance sheet)

**Suggested diagram edges (employee pack):**

- Clock → `ClockLeavesCard` → `LeavesScreen` (solid push)
- Attendance → `AttendanceDayOverviewSheet` → `ApplyLeaveSheet` / `LeaveDetailSheet` (dashed)
- `LeavesScreen` → history / types / holidays (solid); → `ApplyLeaveSheet` (dashed)

---

## Manager module — spot-check vs `manager-screens/` + `detailed/manager-tabs` + `manager-people`

### Still accurate (REUSE)

- Five tabs: Overview, Clock, Team attendance, Alerts, More (`manager_nav.dart` lines 69–75).
- Overview → All employees / All locations (routes + directory card).
- Overview → Add Employee / New Location sheets.
- All locations → location overview → setup → map screens.
- Team attendance → status/location filters, row → attendance details → add/edit punch.
- Employee profile sheet chain (attendance, locations, devices, schedule) as in `manager-people`.

### Missing or incomplete (must add)

**`Manager_leaves` module** — only one screen file, but it is a first-class product surface:

- `lib/features/manager_module/Manager_leaves/presentation/screens/team_leaves_screen.dart`
- **Entry:** Overview action tile “Team Leaves” in `overview_header.dart` (`OverviewActionsGrid`) — **not** in Oct manager-screens candidate (grid documented only Add Location / Add Employee + directories).

**Overview action grid (current code, 2×2):**

1. Add Location → `NewLocationSheet` (in guide)
2. Add Employee → `AddEmployeeSheet` (in guide)
3. **Team Leaves** → `TeamLeavesScreen` (**missing**)
4. **My Leaves** → employee `LeavesScreen` (**missing**)

**`TeamLeavesScreen` behavior (for diagram nodes):**

- Loads team leave requests via `managerOverviewService.loadTeamLeaves` (+ dashboard seed `pendingTeamLeaves` / `teamMonthLeaves`).
- Filters: location (`LocationsFilterSheet`), status (`LeaveStatusFilterSheet`).
- Row tap → `ManagerLeaveDetailSheet` (approve/reject/edit).
- Footer shortcuts → `LeaveTypesScreen`, `HolidaysCalendarScreen`.
- “Leave usage” bars per employee (balances API).

**Leave integration outside `Manager_leaves`:**

- `ManagerEmployeeAttendanceSheet` ↔ `ManagerLeaveDetailSheet` / `AttendanceDayOverviewSheet` (leave days on calendar; `selectLeaveDay` anti-loop).
- `location_overview_screen.dart` — “On Leaves” stat → attendance filter (leave-aware attendance, not team-leaves screen).
- `ManagerAttendanceProvider` leave overlay from overview seed (on-leave rows on team attendance tab).

**Suggested diagram edges (manager pack):**

- Overview → **Team Leaves** → `TeamLeavesScreen` (solid push)
- Overview → **My Leaves** → `LeavesScreen` (solid push, shared employee module)
- `TeamLeavesScreen` → `ManagerLeaveDetailSheet`, `LeaveStatusFilterSheet`, `LocationsFilterSheet` (dashed)
- `TeamLeavesScreen` → `LeaveTypesScreen` / `HolidaysCalendarScreen` (solid)
- Team attendance / employee profile attendance → `ManagerLeaveDetailSheet` (dashed, optional secondary view)

---

## Leave sheets — cross-cutting (both roles)

Document as a **shared “leave overlays” region** linked from employee + manager diagrams (avoid duplicating seven nodes twice if Live Guide supports cross-link):

| File | Primary callers |
|------|-----------------|
| `apply_leave_sheet.dart` | `LeavesScreen`, `attendance_day_overview_sheet`, `manager_leave_detail_sheet` |
| `manager_leave_detail_sheet.dart` | `TeamLeavesScreen`, `manager_employee_attendance_sheet` |
| `leave_detail_sheet.dart` | Employee leave list/history |
| `attendance_day_overview_sheet.dart` | Employee attendance day (leave/punch summary) |

---

## REUSE vs full REBUILD — decision record

- **Not** a full regen of all four HTML packs: tab shells, auth launch, more hub, manager directories, and employee attendance sheet graph remain valid.
- **Yes** to **candidate.json patch + re-finalize** for:
  - `employee-screens` (+ detailed `employee/` optional second view “Leaves stack”)
  - `manager-screens` Overview region (+ detailed `manager-people/` Overview connections)
- Pin new `repository.revision` to `77242d1796dab8196266fe0ccdccf4e0a266687b` when republishing.

---

## Publisher checklist (downstream)

1. Extend `manager-screens/candidate.json`: components `team-leaves`, `my-leaves`, sheets `manager-leave-detail`, `leave-status-filter`; connections from `overview`.
2. Extend `employee-screens/candidate.json`: `leaves-hub` screen cluster + clock card edge + attendance → day overview → apply leave.
3. Mirror same nodes in `architecture-obecno-detailed-20261006-215549/manager-people/candidate.json` (Overview section only).
4. Leave auth-screens and more-screens artifacts as-is; refresh revision metadata only if manifest requires consistency.

---

*Generated: 2026-10-17, Archify run 20261017-2353-archify.*
