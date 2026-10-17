# Screens refresh — 20261009-2122-archify

**Decision: REUSE** `architecture-obecno-screens-20261006-213714/`  
No rebuild under `architecture-obecno-screens-20261009-2122/` — route shells, tab trees, and manager directories still match current code. Gaps are minor (orphaned OTP, collapsed location sub-screens already noted as “map + schedule”).

**Verified against:** `/Users/hassan/Downloads/obecno`  
**Primary sources:** `lib/core/routes/app_routes.dart`, `lib/widgets/bottom_nav_bars/employee_nav.dart`, `lib/widgets/bottom_nav_bars/manager_nav.dart`, `lib/features/{auth,employee_module,manager_module,clock,alerts,more,launch,join}`

---

## Counts (verified)

| Area | Screens found | Notes |
|------|---------------|--------|
| **Auth (+ launch / join / gate)** | **11** | 5 under `auth/presentation` + splash, onboarding, book demo, demo request, you've joined, device blocked |
| **Employee (tab shell)** | **4** | Clock, Attendance, Alerts, More — hosted by `/employee_nav` |
| **Manager (module + shell)** | **10** | 5 tabs + All Employees, All Locations, Location Overview, Location Setup, Setup Location Map (`ManagerAlertsScreen` is an unused thin wrapper over shared `AlertsScreen`) |

---

## GoRouter inventory (`lib/core/routes/app_routes.dart`)

| Path | Screen / shell | Module |
|------|----------------|--------|
| `/` | `SplashScreen` | launch |
| `/onboarding` | `OnBoardingScreen` | launch |
| `/login` | `LoginEmailScreen` | auth |
| `/login/password` | `LoginPasswordScreen` (extra: email) | auth |
| `/youve_joined` | `YouveJoinedScreen` (extra: company) | join |
| `/enable_permissions` | `EnablePermissionsScreen` | auth |
| `/device_blocked` | `DeviceBlockedScreen` | more (gate) |
| `/employee_nav` | `EmployeeBottomNavBar` | employee shell |
| `/manager_nav` | `ManagerBottomNavBar` | manager shell |
| `/manager/employees` | `AllEmployeesScreen` | manager |
| `/manager/locations` | `AllLocationsScreen` | manager |
| `/bookdemo` | `BookDemoScreen` | launch |
| `MonotonicClockDemoScreen.routePath` | demo only | demo (out of product scope) |

---

## Auth / launch / join screens (11)

| # | Widget | File | How reached |
|---|--------|------|-------------|
| 1 | `SplashScreen` | `launch/splash/splash.dart` | `/` |
| 2 | `OnBoardingScreen` | `launch/onboarding/onboarding.dart` | `/onboarding` |
| 3 | `BookDemoScreen` | `launch/book_demo/.../book_demo.dart` | `/bookdemo` |
| 4 | `DemoRequestScreen` | `launch/book_demo/.../request_demo.dart` | `Navigator.push` from Book Demo |
| 5 | `LoginEmailScreen` | `auth/presentation/login_email.dart` | `/login` |
| 6 | `LoginPasswordScreen` | `auth/presentation/login_pass.dart` | `/login/password` |
| 7 | `ForgotPasswordScreen` | `auth/presentation/forgot_password.dart` | `Navigator.push` from password |
| 8 | `LoginOtpScreen` | `auth/presentation/login_otp.dart` | **orphaned** — no route / no callers |
| 9 | `EnablePermissionsScreen` | `auth/presentation/enable_permission.dart` | `/enable_permissions` |
| 10 | `YouveJoinedScreen` | `join/.../youve_joined_screen.dart` | `/youve_joined` |
| 11 | `DeviceBlockedScreen` | `more/.../device_blocked_screen.dart` | `/device_blocked` |

**Existing auth diagram coverage:** splash → onboarding → login → password → forgot → permissions → role homes; join deep link; device blocked; book demo. Still accurate.  
**Delta only:** OTP unused; `DemoRequestScreen` is a child of Book Demo (not a separate GoRoute).

---

## Employee screens (4 tabs)

Host: `EmployeeBottomNavBar` → `/employee_nav`

| Tab | Widget | File |
|-----|--------|------|
| 0 Clock | `ClockScreen` | `clock/presentation/screens/clock_screen.dart` |
| 1 Attendance | `EmployeeAttendanceScreen` | `employee_module/attendance/.../attendence_screen.dart` |
| 2 Alerts | `AlertsScreen` | `alerts/presentation/screens/alerts_screen.dart` |
| 3 More | `ProfileSettingsScreen` | `more/.../profile_settings_screen.dart` |

**Overlays (still match employee diagram):** location sheet, attendance details / add / holiday sheets, month picker.  
`employee_module` has no additional routed screens beyond Attendance.

---

## Manager screens (5 tabs + 5 pushed location/people flows)

Host: `ManagerBottomNavBar` → `/manager_nav`

| Tab | Widget | File |
|-----|--------|------|
| 0 Overview | `OverviewScreen` | `Manager_overview/.../overview_screen.dart` |
| 1 Clock | `ClockScreen` | shared clock |
| 2 Attendance | `ManagerAttendanceScreen` | `Manager_attendance/.../manager_attendence_screen.dart` |
| 3 Alerts | `AlertsScreen` | shared alerts (`ManagerAlertsScreen` wraps same UI; **not** used in nav) |
| 4 More | `ProfileSettingsScreen` | shared more hub |

| Pushed / routed | Widget | Entry |
|-----------------|--------|-------|
| All employees | `AllEmployeesScreen` | `/manager/employees` + Overview / location overview |
| All locations | `AllLocationsScreen` | `/manager/locations` + Overview |
| Location overview | `LocationOverviewScreen` | from All Locations / new location sheet |
| Location setup | `LocationSetupScreen` | from Location Overview |
| Setup location map | `SetupLocationMapScreen` | from Location Setup |

**Sheets (still match manager diagram):** New Location, Add Employee, manager attendance details, employee profile.  
**Existing manager diagram:** collapses overview → locations → “Location setup (map + schedule)” — still correct; no new modules.

---

## More / settings (shared; reuse more-screens diagram)

Reached from both shells’ More tab: Account, Offices, Reminders, Linked Devices, Change Password, Terms, Policy, Help & Feedback → Sent. Covered by `more-screens/` in the same architecture folder.

---

## Reuse vs rebuild

| Artifact | Status |
|----------|--------|
| `architecture-obecno-screens-20261006-213714/auth-screens/` | **reuse** |
| `architecture-obecno-screens-20261006-213714/employee-screens/` | **reuse** |
| `architecture-obecno-screens-20261006-213714/manager-screens/` | **reuse** |
| `architecture-obecno-screens-20261006-213714/more-screens/` | **reuse** (shared hub) |
| `architecture-obecno-screens-20261009-2122/` | **not created** — no major missing screens/modules |

Related API guide remains: `architecture-obecno-api-guide-20261006-222000/`.
