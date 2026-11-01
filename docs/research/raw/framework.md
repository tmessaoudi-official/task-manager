# Dive: cross-platform framework decision (neutral)

Research date: 2026-11-01. Target: a task/todo/reminder app, Android plus Ubuntu GNOME Wayland first, iOS/Windows/macOS later from the same codebase. One solo developer working with Claude Code, public GitHub repo.

Evidence grades used throughout:
- **[V]** verified: I fetched or searched the source, URL given.
- **[I]** inferred: reasoned from verified facts, which are named.
- **[U]** unverified: could not be checked, reason given.
- **[S]** speculative: a judgement call with no checkable fact behind it.

pub.dev only shows relative dates ("N days ago"), so the dates below are converted to approximate calendar dates relative to 2026-11-01 and marked "~".

---

## 0. TL;DR

| | |
|---|---|
| **Recommendation** | **Flutter (3.47.x stable now, 3.50 due Nov 2026)**. Add a thin native Kotlin layer for Android home-screen widgets (Jetpack Glance) and, optionally, for exact-alarm scheduling. Add a resident reminder scheduler for Linux, as a tray process. |
| **Runner-up** | **Kotlin Multiplatform + Compose Multiplatform**. It has the best Android depth (Glance, AlarmManager and Room are native Kotlin) but the weakest of the serious options on GNOME Wayland desktop (AWT tray and notification problems, no portal hotkeys). |
| **Not recommended** | React Native/Expo: no official Linux desktop. Tauri 2: no widget, alarm or background plugins on mobile, and its global shortcuts were historically X11-only. .NET MAUI: no Microsoft Linux target, only an Avalonia preview. Capacitor: web-view mobile, no first-party Linux. |
| **Why Flutter wins** | It is the only candidate with all of: (a) a first-party, actively developed **Linux desktop embedder**, now with Impeller as the default renderer and with Canonical as lead desktop maintainer; (b) a mature Android plugin set for notifications, exact alarms, share sheet, quick actions and widgets; (c) the drift and PowerSync DBs on every target **including Linux**; (d) the user's own toolchain, since `/stack` `.env`/`.env.local` pin Flutter 3.47.9 (plus a local 3.41.9 image) with FVM. |

---

## 1. Platform facts every candidate inherits

These are OS facts, not framework strengths or weaknesses. They are listed so that no framework gets scored down for something the platform imposes on all of them.

| Fact | Grade / source |
|---|---|
| Android 14+ denies `SCHEDULE_EXACT_ALARM` by default. The alternative is `USE_EXACT_ALARM`, which needs no runtime prompt but is subject to store approval and auditing. | [V] flutter_local_notifications docs https://pub.dev/packages/flutter_local_notifications ; android_alarm_manager_plus docs https://pub.dev/packages/android_alarm_manager_plus |
| AlarmManager alarms are cleared on reboot, so apps must reschedule on `RECEIVE_BOOT_COMPLETED`. Alarms do not fire for force-stopped apps. OEM battery killers can suppress them (dontkillmyapp.com). | [V] android_alarm_manager_plus FAQ (URL above) |
| Samsung may cap scheduled alarms at 500 per app. | [V] flutter_local_notifications docs (URL above) |
| Android home-screen widgets are **always native** (RemoteViews or Jetpack Glance), whatever the framework. Frameworks differ only in how they bridge data to them. | [I] home_widget says "widgets must be written with native code" https://pub.dev/packages/home_widget ; Glance is an androidx Kotlin library https://developer.android.com/jetpack/androidx/releases/glance |
| Glance stable is 1.2.0 (2026-08-26), minSdk 23. 1.3.0-alpha02 (2026-07-01) needs AGP 9.2+. | [V] https://developer.android.com/jetpack/androidx/releases/glance |
| GNOME shows tray icons only with the AppIndicator/StatusNotifierItem extension. **Ubuntu ships it enabled.** Legacy XEmbed/GtkStatusIcon trays are hidden or removed on GNOME 3.26+. | [V] tray_manager docs https://pub.dev/packages/tray_manager ; https://git.dorkbox.com/dorkbox/SystemTray/raw/commit/0b53d5388fee855bcb3f436e184f63efccee38fa/README.md |
| On Wayland, a global hotkey must go through `org.freedesktop.portal.GlobalShortcuts`. GNOME implemented the portal around GNOME 48, and it shows a one-time permission dialog. GNOME 50 / portal 1.21 requires the app ID to be derivable (a `.desktop` file matching the app ID). | [V, secondary] Chromium bug https://issues.chromium.org/issues/404298968 ; https://github.com/aaddrick/claude-desktop-debian/blob/main/docs/learnings/wayland-global-shortcuts-portal.md . [U] the user's exact GNOME/Ubuntu version was not checked. |
| On Wayland the desktop has the final say on the key combination: the app *requests* a shortcut and the user or compositor may reassign it. The UX must show the assigned trigger. | [V] hotkey_manager docs https://pub.dev/packages/hotkey_manager |

---

## 2. Candidate by candidate

### 2.1 Flutter

**Release state**
- Latest stable line is **3.47**. The official release-notes index lists 3.47.0 as newest and mentions a 3.47.8 security patch (eMudhra CA keys). [V] https://docs.flutter.dev/release/release-notes
- Shorebird lists 3.47.7 (2026-10-08, Dart 3.13.5) as latest stable, with 3.44.9 (May 2026 line) and 3.41.9 (Feb 2026 line, Dart 3.11.5) before it. [V, third-party] https://docs.shorebird.dev/getting-started/flutter-version.md
- **The `/stack` config pins two Flutters.** `GLOBAL_STACK_FLUTTER3_VERSION=3.47.9` in both the master `.env` (line 1384) and the active `.env.local` (line 852), plus `GLOBAL_STACK_LOCAL_FLUTTER3_41_9_VERSION=3.41.9` in `.env.local:929` (the local all-in-one image the user mentioned), and `FVM_VERSION=4.3.2`. [V] I read the files with grep. [U] which containers are actually running was not checked. **A new project should target 3.47.x now and 3.50 when it ships.**
- 3.50 is due November 2026. [V, secondary] search result summary of the Flutter release cadence. In that release the in-SDK Material/Cupertino libraries are "scheduled for formal deprecation"; `material_ui`/`cupertino_ui` 1.0 are now standalone packages, migrated with `dart fix --apply --code=migrate_design_widgets`. [V] https://flutter.dev/blog/whats-new-in-flutter-3-47 . **A greenfield app should start on `material_ui` directly to avoid a migration.** [I]
- 3.47 dependency matrix: Java 17, KGP 2.4.0, AGP 9.1.0, Gradle 9.3.1, compileSdk/targetSdk 36, minSdk 24. [V] same blog. The user's stack has Java 17/21/27 (zulu), Gradle 9.8.2 and Kotlin 2.4.21 pinned. [V] `/stack/.env:471-498`. That toolchain meets the matrix. [I]

**Linux desktop (GTK embedder)**
- 3.47 "Turn linux impeller on by default" (#187573), SDF text rendering, Linux `--flavor` builds, popup windows, glyph gamma correction, stylus improvements, and a resize crash fix. [V] https://docs.flutter.dev/release/release-notes/release-notes-3.47.0
- Impeller on Linux targets Vulkan. A Skia opt-out (`fl_dart_project_set_enable_impeller(project, FALSE)`) exists for now, but "Fallback options will be removed in a future release". [V] https://flutter.dev/blog/whats-new-in-flutter-3-47
- Multi-window is still **experimental**: popup windows, `windowHandle` (GtkWindow), sized-to-content. [V] same blog. A todo app needs a **quick-add popup window**, so this matters. The workaround is a second process or a single window that shows and hides. [I]
- **Canonical became lead maintainer and "strategic steward" of Flutter desktop** (Linux, Windows, macOS embedders) at Google I/O 2026. [V, secondary] https://www.omgubuntu.co.uk/2026/05/flutter-desktop-canonical-maintained ; https://www.thurrott.com/?p=336913 . [U] the primary Google blog post was not fetched. Canonical ships Ubuntu's own App Center, Firmware Updater and Security Center in Flutter. [V, secondary] same.
- The embedder is still GTK3. A GTK4 opt-in variant (`--linux-gtk=gtk4`) is in an open PR (#193405), with tracker #194698. [V] https://github.com/flutter/flutter/pull/193405 , https://github.com/flutter/flutter/issues/194698 . [U] whether it has merged.

**Plugin ecosystem (maintenance check)**

| Need | Package | Version / ~date / publisher | Platforms | Status | Grade |
|---|---|---|---|---|---|
| Local notifications + scheduling | `flutter_local_notifications` | 22.3.3, ~2026-10-25, dexterx.dev (verified), 7.36k likes; 23.0.0-dev.5 in pre-release | Android, iOS, **Linux**, macOS, web, Windows | Active. **Scheduled notifications are NOT supported on Linux.** Handles the exact-alarm permission request (`requestExactAlarmsPermission()`) and falls back to inexact via `androidScheduleMode`. | [V] https://pub.dev/packages/flutter_local_notifications |
| Background Dart callbacks at exact times | `android_alarm_manager_plus` | 5.1.2, ~2026-10-01, fluttercommunity.dev, Flutter Favorite | Android | Active, but "does not provide a way to work with this permission". You handle `SCHEDULE_EXACT_ALARM` yourself (e.g. `permission_handler`). | [V] https://pub.dev/packages/android_alarm_manager_plus |
| Periodic background work (sync) | `workmanager` | 0.10.10, ~2026-09-07, fluttercommunity.dev | Android, iOS, macOS, web. Linux experimental via `workmanager_linux`. | Active | [V] https://pub.dev/packages/workmanager |
| Home-screen widgets | `home_widget` | 0.10.0, ~2026-09-17, antonborri.es (verified), 2.22k likes | Android, iOS | Active, **single maintainer, free time**. Supports interactive widgets that call Dart. Glance is not mentioned (the widget UI is native code you write). | [V] https://pub.dev/packages/home_widget |
| Tray | `tray_manager` | 0.8.2, ~2026-10-31, leanflutter.dev (verified) | Linux, macOS, Windows | Active. Linux tray is now a **StatusNotifierItem over D-Bus**, so `libayatana-appindicator` is no longer needed. Shows on Ubuntu GNOME because Ubuntu ships the extension. | [V] https://pub.dev/packages/tray_manager |
| Tray (alternative) | `system_tray` | 2.0.3, ~3 years old, unverified uploader | Linux, macOS, Windows | **Treat as abandoned. Do not use.** | [V] https://pub.dev/packages/system_tray |
| Global quick-add hotkey | `hotkey_manager` | 0.3.0, ~2026-11-01, leanflutter.dev (verified) | Linux, macOS, Windows (Android: in-app hardware keyboard only) | Active. **Native Wayland via `org.freedesktop.portal.GlobalShortcuts`.** Needs a `.desktop` file whose basename matches the GTK app ID. Throws `PlatformException` if the portal is missing or denied. X11/XWayland path also available. | [V] https://pub.dev/packages/hotkey_manager |
| Launcher quick actions | `quick_actions` | 1.1.1, ~2026-09-09, **flutter.dev** (first-party) | Android, iOS | Active | [V] https://pub.dev/packages/quick_actions |
| Share-sheet target | `receive_sharing_intent` | 1.9.0, ~2026-07, kasem.dev (verified) | Android, iOS | Maintained, slower cadence (single maintainer) | [V] https://pub.dev/packages/receive_sharing_intent |
| Local DB | `drift` | 2.35.3, ~2026-10-23, simonbinder.eu (verified), 2.49k likes | Android, iOS, **Linux**, macOS, web, Windows | Active | [V] https://pub.dev/packages/drift |
| Sync engine | PowerSync Dart/Flutter SDK | — | Android, iOS, macOS, Windows, **Linux (glibc 2.34+)**, web | Listed with no beta/alpha label | [V] https://docs.powersync.com/resources/supported-platforms |

Freshness note: the leanflutter.dev desktop plugins were both published within a day of 2026-11-01. They are actively maintained, but a fresh 0.x release can contain regressions, so **pin exact versions**. [I] from the publish dates.

**AI-agent fit:** 3.44 introduced "Agentic Hot Reload". [V, secondary] https://medium.com/@AlexCodeX/flutter-3-44-release-notes-the-biggest-flutter-release-of-2026-20c82c7a6575 . Dart is single-language across UI, logic and DB (drift codegen). [S] Claude Code is productive in Dart/Flutter, but the dev-speed scores are judgement.

**Performance:** Impeller is the default on Android/iOS (long-standing), and from 3.47 on Linux/Windows/macOS as well. [V] the 3.47 blog. Desktop memory and startup are native AOT, not a JVM or web view. [I]

### 2.2 Kotlin Multiplatform + Compose Multiplatform (KMP/CMP)

- Stability: KMP core is stable on Android, iOS and desktop JVM. Compose Multiplatform UI is stable on Android, iOS and desktop (JVM); web (Wasm) is beta. [V] https://kotlinlang.org/docs/multiplatform/supported-platforms.html . iOS went stable in 1.8.0 (May 2025). [V, secondary] https://www.neowin.net/news/jetbrains-brings-compose-for-ios-to-stable-with-compose-multiplatform-180-release/
- Latest: CMP 1.12.2 (Jetpack Compose 1.12.1). Linux minimum is Ubuntu 20.04 x64/arm64. Desktop needs JDK 11+, packaging needs JDK 17+ (`jpackage`), 64-bit only. [V] https://kotlinlang.org/docs/multiplatform/compose-compatibility-and-versioning.html . 1.11.0 (May 2026) made concurrent rendering the default and added experimental native iOS text input. [V, search snippet of https://blog.jetbrains.com/kotlin/2026/05/compose-multiplatform-1-11-0/ , direct fetch returned 403]
- **Android depth: best of all.** Glance widgets, AlarmManager, WorkManager, ShortcutManager and share intents are all plain Kotlin in `androidMain`, with no bridge, and they share models with the rest of the app. [I] from Glance being Kotlin/androidx (URL above).
- **Local DB:** Room is KMP (Android, iOS, JVM desktop, native macOS/Linux, watchOS/tvOS). Stable 2.8.5 (2026-09-09). [V] https://developer.android.com/jetpack/androidx/releases/room . A new `androidx.room3` line (3.1.0-alpha03) is shown in the KMP docs. [V] https://developer.android.com/kotlin/multiplatform/room . That means a likely future migration if you start on Room 2.x. [I] SQLDelight is the long-standing alternative. [U] not re-checked in this dive. PowerSync Kotlin supports Android, iOS and Linux JVM. [V] PowerSync supported-platforms URL above.
- **Linux GNOME Wayland desktop: the weak spot.**
  - The Compose desktop `Tray()`/`TrayState.sendNotification()` sit on AWT. If `TrayState` is not attached to a tray, the "notification is lost", and `Tray()` only logs to stderr when no tray is supported. [V] https://kotlinlang.org/docs/multiplatform/compose-desktop-tray.html
  - AWT `SystemTray.isSupported()` returns false on GNOME (a user report on Ubuntu, compose-jb #65). [V, older thread] https://slack-chats.kotlinlang.org/t/519872/what-s-the-current-status-about-notifications-on-linux-i-ve- . **So on GNOME, both the tray and native notifications need custom work:** a D-Bus SNI tray plus libnotify/`org.freedesktop.Notifications` via dbus-java or JNA, and a GlobalShortcuts portal client. [I]
  - No official native Wayland backend. JBR-3206 (native Wayland in JetBrains Runtime) is in progress, and community options exist (compose-glfw: single window, no tray). [V, secondary] https://slack-chats.kotlinlang.org/t/34112953/i-ve-been-wondering-for-a-long-time-why-does-compose-desktop . So today it runs through XWayland. [I]
  - The JVM runtime adds startup time and memory for an always-running tray app. [I] from JVM desktop + jpackage.
- Toolchain fit: Kotlin 2.4.21, Gradle 9.8.2 and Java 17/21/27 are already pinned in `/stack/.env`. [V] `/stack/.env:471-498`

### 2.3 React Native / Expo

- Expo SDK 55 (2026-02-25): RN 0.83, Legacy Architecture dropped. SDK 56 (2026-05-21): RN 0.85, **iOS widgets promoted to stable**. SDK 57 (announced 2026-06-30): RN 0.86, non-breaking. [V] https://expo.dev/changelog/sdk-55 , https://expo.dev/sdk/56 , https://expo.dev/changelog/sdk-57
- The New Architecture is always on from RN 0.82. [V] https://docs.expo.dev/guides/new-architecture/
- `expo-widgets` is **iOS-only** (home-screen widgets and Live Activities). SDK 56 says nothing about Android widgets. [V] the SDK 55/56 pages above. Android widgets would need a community library or native Kotlin. [U] community library status not checked.
- **Linux desktop: no official support.** Microsoft maintains Windows/macOS only, react-native-desktop-qt is unsupported, and react-gtk is from 2019. The only 2026 Linux work found is a personal project's "react-native-linux" PR. [V] https://platform.uno/articles/best-cross-platform-frameworks-2026/ , https://github.com/status-im/react-native-desktop-qt , https://github.com/keejkrej/t3code-native/pull/3 . A Linux build would mean a second shell (Electron or Tauri around RN-Web), i.e. a second codebase for desktop integration. [I]
- AI tooling: SDK 55 shipped an MCP server and agent skills for Claude Code. [V, secondary] https://x.com/expo/status/2026811977990025364
- RN/React governance moved to the React Foundation (Linux Foundation), announced October 2025. [V, secondary] https://en.wikipedia.org/wiki/React_Native

### 2.4 Tauri 2

- Core 2.11.5 (2026-07-01). 2.11.2 was a security fix (custom-protocol local-URL confusion on Windows/Android). [V] https://v2.tauri.app/release/core/ (via search). [U] whether a later release exists; the index stopped at July.
- The official plugin index has **no home-screen widget plugin and no background-task or alarm plugin**. The only widget option is the community `tauri-plugin-widgets`. [V] https://v2.tauri.app/plugin/
- The official notification plugin page does not document scheduling. Community forks (Choochmeque, svenpaulsen) add actions, pending management and FCM/APNs push. [V] https://v2.tauri.app/plugin/notification/ , https://github.com/Choochmeque/tauri-plugin-notifications
- Global shortcut: lists Linux, but historically **X11-only**. tao disabled it on Wayland ("highly X11-specific"). [V, 2021–2023 issues] https://github.com/tauri-apps/tao/issues/331 . [U] current portal support not confirmed; the plugin page has no Wayland note: https://v2.tauri.app/plugin/global-shortcut/
- Linux rendering uses WebKitGTK (2.11.0 bumped webkit2gtk-rs to 2.0.2). [V] release notes via search. That brings web-view performance and quirks. [I]
- PowerSync Tauri SDK is **alpha**, untested on iOS and Android. [V] PowerSync URLs above.
- Requires Rust plus a web front end: two languages and three build systems. [I]

### 2.5 Brief: .NET MAUI, Avalonia, Capacitor

- **.NET MAUI:** Microsoft ships no Linux target. Avalonia's MAUI backend Preview 1 (2026-03-16, .NET 11 previews) adds Linux and Wasm; reported as not production-ready. [V] https://avaloniaui.net/blog/maui-avalonia-preview-1 , https://www.devclass.com/development/2026/03/24/avaloniaui-enhances-net-maui-with-linux-and-webassembly-support/5209515 . Not a fit. [I]
- **Avalonia (12.1.4, 2026-10-09):** strong Linux desktop, mobile less proven for deep Android integration. [V, secondary] https://en.wikipedia.org/wiki/Avalonia_(software_framework) . [U] its mobile widget and alarm story was not researched.
- **Capacitor 8** (Dec 2025, 8.3.1 Apr 2026): web-view mobile. Desktop would be the community Electron platform. [V] https://newreleases.io/project/github/ionic-team/capacitor/release/8.3.1 ; [U] the Electron community platform's maintenance was not checked. Not a fit. [I]

---

## 3. Scorecard (1–5, higher is better)

Scores are judgements [S] grounded in the graded facts in §2. The reason column names the facts each score rests on.

| Criterion | Flutter | KMP + CMP | RN / Expo | Tauri 2 | MAUI (+Avalonia) | Capacitor |
|---|---|---|---|---|---|---|
| Android deep integration | **4**: notifications + exact-alarm permission wrapped, quick_actions first-party, share intents, widgets via home_widget plus native code | **5**: everything is native Kotlin | 4 | 2: no alarm or background plugins | 3 | 3 |
| Linux GNOME Wayland desktop | **4**: official embedder, Impeller default, Canonical steward, SNI tray, portal hotkeys. Minus: no scheduled notifications on Linux, experimental multi-window | 2: AWT tray/notifications fail on GNOME, XWayland, JVM resident cost | 1: no official Linux | 3: WebKitGTK, hotkeys X11-only historically | 2: preview only | 1 |
| iOS later (no rewrite) | 4 | 4: stable since 1.8.0 | 5 | 3 | 4 | 4 |
| Widgets | 3: home_widget bridge, UI must be native (Glance/WidgetKit) | 5: Glance in-tree; iOS WidgetKit still Swift | 2: expo-widgets iOS only | 1: community only | 2 [U] | 2 [U] |
| Background / alarms reliability | 4: AlarmManager wrapped, boot reschedule documented; Linux needs own scheduler | 5 | 3 [U] | 1 | 3 [U] | 2 [U] |
| Local DB / sync | **5**: drift on all 6 targets; PowerSync incl. Linux | 4: Room KMP stable, Room3 churn ahead; PowerSync JVM | 4 | 3: PowerSync alpha | 3 | 3 |
| Dev speed, solo + AI agent | **5**: one language, hot reload incl. agentic, one build per target | 4: Gradle/KMP config overhead | 5 on mobile; desktop would be a second project | 3: Rust + JS | 3 | 4 |
| Ecosystem health / maintenance | 4: core strong; several key plugins single-maintainer; system_tray abandoned; Material split churn in 3.50 | 4: JetBrains + Google backed | 4 | 3 | 2 | 3 |
| Performance | 4 | 4: JVM on desktop | 4 | 3 | 3 | 2 |
| Existing toolchain fit | **5**: Flutter 3.47.9 + FVM pinned; Android SDK; Java 17 | 4: Kotlin/Gradle/Java pinned | 3: Node present | 2: Rust in stack, no Tauri | 1 | 3 |
| **Total / 50** | **42** | **41** | 35 | 24 | 26 | 27 |

**Source-age disclosure:** the GNOME AWT-tray finding behind KMP's Linux score comes from an older Slack thread and compose-jb #65 (GTK-StatusIcon era). It was not re-verified for CMP 1.12. Even if the tray now works, XWayland, the lack of a GlobalShortcuts portal client and the AWT notification path cap KMP Linux at 3. That would tie the totals at 42, and the qualitative Linux-depth argument (Canonical-maintained embedder, ready SNI tray and portal hotkey plugins) would still decide. [I]

Tauri mobile maturity: [U] I did not check Tauri's own mobile stability statement. It is scored on its verified plugin gaps instead.

The KMP total is within one point of Flutter. What decides the choice is the **Linux desktop gap (4 vs 2)**: Linux is one of the two day-one platforms, and KMP's gaps there (tray, notifications, hotkeys) are exactly the "deep PC integration" features the user asked for. The Android gap (5 vs 4) is narrower, and the mitigation below closes most of it. [I]

---

## 4. Recommendation: Flutter, structured to neutralise its weak spots

**Architecture**
1. One Flutter app, `material_ui` from day one, drift for local SQLite, and a sync layer chosen in the sync dive (PowerSync covers all targets incl. Linux). [V] for the library facts in §2.1.
2. **`android/` native module (Kotlin), owned by the repo:**
   - Jetpack Glance 1.2.0 widgets (today list, quick-add, toggle done) reading the same SQLite file, or a JSON snapshot written via home_widget. Interactive actions go through Glance `ActionCallback` and then a headless Dart entry or a direct DB write. [I] from home_widget's "interactive widgets that call Dart" and Glance lambdas being stable in 1.2.0.
   - The exact-alarm path: either `flutter_local_notifications.zonedSchedule` with `exactAllowWhileIdle`, or a small Kotlin `AlarmManager.setAlarmClock()` wrapper plus boot receiver, if reminders must survive Doze and OEM killers better. `setAlarmClock` is what alarm-clock apps use. [U] OEM behaviour per device, see dontkillmyapp.com.
3. **Linux desktop:** a tray-resident process (`tray_manager`, SNI) that owns its **own reminder scheduler** (Dart timers re-armed from the DB, plus resume-from-suspend handling), because flutter_local_notifications cannot schedule on Linux. [V] the limitation; [I] the design. A hardened option for the user (a Linux expert) is to also emit a `systemd --user` timer or autostart `.desktop` entry, so reminders fire even if the app was quit. [S]
4. **Global quick-add:** `hotkey_manager` 0.3.0 over the GlobalShortcuts portal. Ship a `.desktop` file whose basename equals the GTK app ID. In the UI, show the trigger the desktop actually assigned, and fall back to a tray menu item and a `--quick-add` CLI flag the user can bind in GNOME Settings, which always works. [V] the portal requirements; [S] the fallback design.
5. **Quick-add window:** until multi-window leaves experimental, use a single small borderless window toggled from the tray/hotkey, or a second process launched with `--quick-add`. [I] from multi-window being experimental in 3.47.

**Specific risks and mitigations**

| # | Risk | Likelihood / impact | Mitigation | Grade |
|---|---|---|---|---|
| R1 | No scheduled notifications on Linux | Certain / high | Resident scheduler + systemd-user fallback (above) | [V] limitation |
| R2 | Widgets are native anyway; home_widget is a one-person project | Medium / medium | Write the Glance widget in-repo Kotlin, use home_widget only as the data bridge (or replace it with a 100-line MethodChannel). If abandoned, fork it. | [V] maintainer note |
| R3 | Exact-alarm permission denied by default (Android 14+) or OEM kills | High / high | Onboarding screen requesting `SCHEDULE_EXACT_ALARM`, inexact fallback, `setAlarmClock`, boot rescheduling, a "reminder reliability" self-test screen. Evaluate `USE_EXACT_ALARM` against the Play policy (separate dive). | [V] permission facts; [U] Play policy |
| R4 | Wayland hotkey not granted or no portal | Medium / medium | `PlatformException` handling → fall back to a CLI flag bound as a GNOME custom shortcut | [V] behaviour documented |
| R5 | Material/Cupertino deprecation in 3.50; breaking windowing API renames | Certain / low | Start on `material_ui`; isolate windowing calls behind one class | [V] |
| R6 | Impeller-on-Linux regressions (Vulkan drivers) | Low-medium / medium | Skia opt-out flag while it exists; test on the user's GPU | [V] opt-out exists; [U] the user's GPU not checked |
| R7 | Desktop plugin regressions in fresh 0.x releases (tray_manager, hotkey_manager released ~1 day ago) | Medium / low | Pin exact versions, Renovate/Dependabot with CI | [I] |
| R8 | Canonical stewardship skews towards Ubuntu | Low for this user (on Ubuntu) / medium for a later public product on other distros | Test on Fedora in CI later | [V, secondary] commentary |
| R9 | GTK3 → GTK4 embedder transition | Low / low | Opt-in only per PR #193405; no action needed | [V] |

**Runner-up rationale:** pick KMP+CMP instead only if Android widgets, wearables and alarms turn out to be the product's core and the Linux desktop is reduced to "nice to have". Even then you would rebuild tray, notifications and hotkeys on Linux over D-Bus yourself. [I]

**A hybrid worth naming:** Flutter for UI and app logic, plus a Kotlin `android/` module for Glance and alarms. This is the recommendation, and it captures most of KMP's Android advantage without losing Flutter's Linux story. [I]

---

## 5. Open items not verified in this dive

- The primary Google/Flutter blog post on Canonical stewardship (only secondary coverage was fetched).
- Whether a Flutter 3.47.9 changelog entry exists. It is pinned in `/stack/.env`, but only 3.47.7 and 3.47.8 were seen in sources.
- The user's Ubuntu/GNOME version (portal behaviour differs between GNOME 48/49 and 50).
- Google Play policy wording for `USE_EXACT_ALARM` eligibility for a todo/reminder app (belongs to the Android dive).
- Tauri global-shortcut on Wayland in 2026. Only 2021–2023 issues were found.
- SQLDelight's current version and status; Avalonia mobile capabilities; Capacitor's Electron community platform.
