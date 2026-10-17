# Runtime architecture refresh — 20261017-2353-archify

**Compared:** `architecture-obecno-runtime-20261006-213000/obecno-runtime.html` vs current Flutter sources under `lib/` (entry, `AppBindings`, `lib/core/api/`, providers, `GoRouter`).

## Verdict: **REUSE**

The October 2026 runtime diagram still matches the app’s bootstrap and networking spine. No full rebuild is warranted; at most add optional callouts on the existing diagram if you want newer shell wrappers documented.

## Findings

- **Bootstrap unchanged:** `main()` → `WidgetsFlutterBinding` → `AppBindings.init()` → `runApp(MyApp())`, with global `bindings` and `runZonedGuarded` error reporting — same composition-root pattern the diagram labels “AppBindings · DI & services” and “Flutter shell · MyApp + providers”.
- **Provider model still custom ChangeNotifier, not Riverpod:** `MultiProvider` + `ChangeNotifierProvider` in `main.dart` wiring auth, manager, employee, alerts, join, etc. — consistent with “MyApp + providers” (no structural DI framework change).
- **Routing still GoRouter from splash to role nav:** `MaterialApp.router` + `routerConfig: router`; `/` → `SplashScreen`, then `/employee_nav` / `/manager_nav` and auth/onboarding paths in `lib/core/routes/app_routes.dart` — matches “GoRouter · Splash → role nav”.
- **API layer intact:** `TokenService` + `ApiClient` (versioned paths, auth headers, retry, 401 → session handling via `SessionManager`) + repositories taking `ApiClient` — aligns with “Token & session” and “ApiClient · HTTP + retry” → “Obecno REST API”.
- **Session/auth flow still splash-driven:** Splash calls `AuthProvider.checkSession()` (secure storage via `TokenService`); `SessionManager` validates on unauthorized and coordinates logout — same logical “token & session” lane (implementation split into `SessionManager` is an internal detail not shown as its own box).
- **Minor diagram deltas (non-blocking):** `AppGuard` / `LocationFlagMonitorHost` wrap the router child; `JoinDeepLinkService` and `ReminderNotificationService` init in `main()` (diagram already has deep links & OS notifications); extra providers (e.g. alerts, join invite) and a headless `locationFlagBackgroundMain` entry — incremental, not a new runtime tier.

## Recommendation

**REUSE** `obecno-runtime.html` as the canonical runtime view for this Archify run. If refreshing visuals, patch labels only (e.g. note `SessionManager` under token/session, optional `AppGuard` on the Flutter shell edge).

**Evidence paths (workspace):** `/Users/hassan/Downloads/obecno/lib/main.dart`, `/Users/hassan/Downloads/obecno/lib/core/binding/app_binding.dart`, `/Users/hassan/Downloads/obecno/lib/core/api/api_client.dart`, `/Users/hassan/Downloads/obecno/lib/core/api/session_manager.dart`, `/Users/hassan/Downloads/obecno/lib/core/routes/app_routes.dart`, `/Users/hassan/Downloads/obecno/lib/features/launch/splash/splash.dart`.

**Report written:** `/Users/hassan/Documents/Archify-kit/projects/obecno/runs/20261017-2353-archify/runtime-refresh.md`
