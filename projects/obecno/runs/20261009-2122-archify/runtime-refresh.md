# Runtime architecture refresh — 20261009-2122

**Decision: reuse existing**

Existing diagram still matches current `lib/` runtime topology. No rebuild.

## Reused artifact

- Folder: `architecture-obecno-runtime-20261006-213000/`
- Candidate: `architecture-obecno-runtime-20261006-213000/candidate.json`
- HTML: `architecture-obecno-runtime-20261006-213000/obecno-runtime.html`

## Repo checked

- Workspace: `/Users/hassan/Downloads/obecno`
- HEAD (short): `7887870`
- Prior candidate revision note: `674dbc5` (diagram still accurate for layers)

## Verified key files

| Layer (diagram) | Evidence path | Status |
|---|---|---|
| Flutter shell / MyApp + providers | `lib/main.dart` | OK — `main()`, `AppBindings`, `MultiProvider`, `MaterialApp.router` |
| GoRouter Splash → role nav | `lib/core/routes/app_routes.dart` | OK — `/`, `/employee_nav`, `/manager_nav` |
| AppBindings DI | `lib/core/binding/app_binding.dart` | OK — `class AppBindings`, ApiClient/session/repos |
| ApiClient HTTP + retry | `lib/core/api/api_client.dart` | OK — `class ApiClient` |
| Token & session | `lib/core/api/api_client.dart`, `session_manager.dart`, `token_service.dart` | OK (wired via bindings) |
| On-device SQLite queue | `lib/shared/location/service/local_queue_service.dart` | OK — `LocalQueueService` |
| SyncService offline→online | `lib/features/clock/services/sync_service.dart` | OK — `class SyncService` |
| Location layer | `lib/shared/location/service/location_provider.dart` | OK — provided from bindings in `main.dart` |
| Native MethodChannel | `android/app/src/main/kotlin/com/example/obecno/MainActivity.kt` | OK — `MethodChannel` |
| OS notifications / reminders | `lib/features/more/services/reminder_notification_service.dart` | OK — init + tap handler in `main.dart` |
| Invite deep links | `lib/features/join/services/join_deep_link_service.dart` | OK — started from `main()` |

## Runtime layers found (unchanged)

1. **Entry / shell** — `main()` → `AppBindings.init()` → reminder + deep-link bootstrap → `MyApp` MultiProvider tree  
2. **Navigation** — GoRouter: splash/onboarding/auth → `/employee_nav` | `/manager_nav`  
3. **UI feature modules** — employee (clock/attendance), manager (overview/employees/locations/attendance), alerts, more, join, launch  
4. **DI / domain services** — `AppBindings` wires repositories, providers, location, sync  
5. **Network** — `ApiClient` + token/session → Obecno REST API  
6. **Offline** — SQLite local queue + `SyncService` drain when online  
7. **Platform edge** — MethodChannel native reminders, OS notifications, AppLinks join invites, Maps picker  

## Non-blocking deltas (not major gaps)

- Demo-only routes/modules (`lib/demo/location_flags`, `lib/demo/monotonic_clock`, demo route in `app_routes.dart`) sit beside production runtime; they do not change the primary journey shown in the existing diagram.
- Line-number chips in the frozen candidate may drift slightly; paths and component roles remain valid.

## Action

Do **not** create `architecture-obecno-runtime-20261009-2122/`. Keep Live Guide pointing at the 20261006-213000 runtime HTML.
