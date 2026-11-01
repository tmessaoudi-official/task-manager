# OS integration surface — what a task/reminder app can actually do (as of 2026-11-01)

Scope: Android (15/16/17), iOS (later), Ubuntu GNOME Wayland (the user's desktop), Windows/macOS (brief), calendar/task back-ends (Google, Microsoft, iCloud, Nextcloud/CalDAV).

Evidence grades:
- **verified**: fetched or searched a source, URL given. "verified-secondary" means only third-party sources were found.
- **verified-local**: probed on the user's own machine during this session (command given).
- **inferred**: reasoned from verified facts, basis stated.
- **unverified**: not checked, reason given.

---

## 0. TL;DR: the hard limits that shape the architecture

| # | Hard limit | Consequence for the design | Grade |
|---|---|---|---|
| H1 | Google Play allows **`USE_EXACT_ALARM` only for alarm/timer apps and calendar apps that show event notifications**. Other apps that ship it "may not be allowed to publish". | A to-do app has to use **`SCHEDULE_EXACT_ALARM`**. On a fresh install (target 33+) it is **denied by default**, and the user can revoke it at any time. Revoking it **cancels every pending exact alarm**. Build onboarding around granting it, plus an inexact fallback. | verified: https://support.google.com/googleplay/android-developer/answer/16558241 , https://developer.android.com/about/versions/14/changes/schedule-exact-alarms |
| H2 | `setAlarmClock()`, `setExact()` and `setExactAndAllowWhileIdle()` all need the exact-alarm permission. `*AllowWhileIdle` fires **at most once every 9 minutes per app** in Doze. `setAlarmClock()` is not throttled: the system leaves Doze shortly before it fires. | Use `setAlarmClock()` for each user-facing reminder, and coalesce several reminders into one alarm. | verified: https://developer.android.com/training/monitoring-device-state/doze-standby , https://developer.android.com/about/versions/14/changes/schedule-exact-alarms |
| H3 | **Full-screen intents**: since 2025-01-22, `USE_FULL_SCREEN_INTENT` is enabled by default only for apps whose core purpose is calling or alarms. | A to-do app has to send the user to "Manage full-screen intents" (`ACTION_MANAGE_APP_USE_FULL_SCREEN_INTENT`) and handle a refusal gracefully. | verified: https://support.google.com/googleplay/android-developer/answer/13392821 |
| H4 | Android 16 **Live Updates** (promoted ongoing notifications) explicitly exclude "alerts, upcoming calendar events". | Do not use them for reminders. They fit only a running focus or timer session. | verified: https://developer.android.com/develop/ui/compose/notifications/live-update |
| H5 | Alarms are cleared on reboot. A `BOOT_COMPLETED` receiver works only after the user has launched the app once. Since Android 15, a `BOOT_COMPLETED` receiver may **not** start `dataSync` (and other) foreground services. | Reschedule from a boot receiver with AlarmManager. Do not start a sync FGS there. | verified: https://developer.android.com/develop/background-work/services/alarms , https://developer.android.com/about/versions/15/behavior-changes-15 |
| H6 | OEM battery killers (Huawei, Xiaomi, OnePlus, Samsung rank worst) can stop alarms whatever the API does. | Add per-OEM onboarding (deep links to the vendor's settings) and a "reminder reliability self-test". | verified (ranking undated): https://dontkillmyapp.com/ |
| H7 | **Google Tasks API**: `due` is date-only ("the time portion of the timestamp is discarded"). There is **no reminder field and no recurrence field**. | A timed reminder **cannot round-trip through Google Tasks**. Google Tasks can only be a date-level mirror. Timed items go to Google Calendar events, or stay in the app's own sync. | verified: https://developers.google.com/workspace/tasks/reference/rest/v1/tasks |
| H8 | Google's CalDAV endpoint "Doesn't support VTODO" and requires OAuth 2.0 (Basic auth gives 401). | You cannot reach Google tasks through CalDAV. Google needs its own REST adapters. | verified: https://developers.google.com/workspace/calendar/caldav/v2/guide |
| H9 | **iCloud Reminders that were "upgraded" (iOS 13 / Catalina format) are not reachable over CalDAV.** Only lists that were never upgraded still appear as VTODO. | iCloud gives **calendar events only** over CalDAV (app-specific password). iCloud Reminders need native EventKit on iOS/macOS. There is **no** path from Android or Linux. | verified-secondary: https://www.busymac.com/docs/faqs/112990-reminders-in-ios-13-and-macos-catalina-drops-support-for-caldav/ , https://github.com/Lingnik/icloud-reminders-mcp (no Apple statement found) |
| H10 | Google OAuth: a *public* app with sensitive scopes needs verification (homepage on a verified domain, privacy policy, demo video). Without it the app is capped at 100 users. Personal use (<100 users) is exempt. A project in **"Testing" status gets refresh tokens that expire after 7 days**. There is a cap of 100 refresh tokens per Google account per client ID. | For personal use, either publish the consent screen "In production, unverified" (warning screen; no 7-day expiry, which is inferred because only Testing status is documented as 7-day) or accept a weekly re-login. Budget the verification step before going public. **No CASA security assessment is needed**, because Calendar and Tasks are not restricted scopes. | verified: https://support.google.com/cloud/answer/13464323 , https://developers.google.com/identity/protocols/oauth2 , https://support.google.com/cloud/answer/13464325 ; "no restricted" = inferred (Calendar/Tasks are absent from the restricted list) |
| H11 | **GNOME tray** exists only through the Ubuntu AppIndicators shell extension (StatusNotifierItem). That extension has an **open bug on GNOME 50 / Ubuntu 26.04** (#628) where items register and then get dropped. | Treat the tray as best-effort. The primary desktop entry points have to be the global shortcut, notifications and a normal window. | verified: https://github.com/ubuntu/gnome-shell-extension-appindicator/issues/628 ; verified-local (watcher present, extension ACTIVE) |
| H12 | **GlobalShortcuts portal** works on GNOME 48+. Since xdg-desktop-portal 1.21.0 it "Require[s] a valid AppID". The user's machine runs 1.21.1. | A non-Flatpak, non-Snap build has to install a matching `.desktop` file (and use the host-app registry) or the bind is refused. The user approves the binding in a GNOME dialog, so the app cannot force a key combo. | verified: https://release.gnome.org/48/developers/index.html , https://github.com/flatpak/xdg-desktop-portal/blob/main/NEWS.md ; verified-local |
| H13 | **iOS keeps only 64 pending local notifications per app** (the soonest-firing 64). Background refresh runs when the system decides; there is no guaranteed time. | iOS needs a "rolling window" scheduler that refills the 64 slots on every launch, background task and push. Push (APNs) from a backend is the reliable path for a public product. | verified-secondary: https://developer.apple.com/forums/thread/811171 , https://developer.apple.com/forums/thread/770479 |
| H14 | Android 17 (targetSdk 37) **rejects widget `RemoteViews` updates over the bitmap memory budget with an IllegalArgumentException**. Lower targets only log a warning. | Keep widget images small (RGB_565, no partial bitmap swaps) and use Glance's `onCompositionError` fallback. | verified: https://developer.android.com/about/versions/17/behavior-changes-17 |
| H15 | Microsoft: since Nov 2020, if a tenant has risk-based step-up consent enabled, its users cannot consent to an *unverified multitenant* app. Publisher verification is free, but it needs a Microsoft AI Cloud Partner Program account and an app **registered in an Entra work/school tenant**. An app registered with a personal Microsoft account cannot be verified. | Personal-MSA users and the developer's own account work without verification. Corporate M365 users of a public product would need the developer to get a CPP account and a work tenant. | verified: https://learn.microsoft.com/en-us/entra/identity-platform/publisher-verification-overview |

---

## 1. Platform baseline (dates)

| Fact | Grade |
|---|---|
| Android 17 = API 37. Stable rollout to Pixel 6+ started 16/17 June 2026. QPR1/QPR2 betas listed on the official page. | Official page: verified (API 37, QPR tracks), https://developer.android.com/about/versions/17 . Release date: verified-secondary, https://www.businesstoday.in/technology/news/story/google-starts-rolling-out-android-17-to-pixel-phones-check-eligibility-and-whats-new-537393-2026-06-17 , https://www.gizchina.com/android-devices/android-17-stable-is-here-rolls-out-to-pixel-6-and-later |
| Google Play: since 2026-08-31, new apps and updates must target **API 36 (Android 16)**; extension possible to 2026-11-01. | verified: https://support.google.com/googleplay/android-developer/answer/11926878 |
| Play policy deadline table lists a Local Network permission declaration for API 37 apps, due 2027-08-25. That suggests an API 37 target requirement around Aug 2027. | Deadline verified (Play policy deadlines page https://support.google.com/googleplay/android-developer/answer/12253906); the target-requirement date is inferred from that deadline |
| User's desktop: **Ubuntu 26.04.1 LTS, GNOME Shell 50.1, Wayland session**, xdg-desktop-portal 1.21.1, xdg-desktop-portal-gnome 50.0. | verified-local: `/etc/os-release`, `gnome-shell --version`, `$XDG_SESSION_TYPE`, `dpkg -l` |
| Ubuntu 26.04 / GNOME 50 desktop is Wayland-only (no Xorg session; X11 apps run through XWayland). | verified-secondary: https://linuxiac.com/ubuntu-26-04-lts-resolute-raccoon-released/ |
| iOS 26 is current (AlarmKit availability "iOS 26.0+"). | verified: https://developer.apple.com/documentation/alarmkit/scheduling-an-alarm-with-alarmkit.md |

---

## 2. ANDROID

### 2.1 Exact reminders: the core of a reminder app

| Capability | Possible? | How | Restriction / policy | Grade + source |
|---|---|---|---|---|
| Exact reminder at a set minute | Yes | `AlarmManager.setAlarmClock()` (never deferred, shows an alarm icon, system leaves Doze shortly before) or `setExactAndAllowWhileIdle()` | Needs `SCHEDULE_EXACT_ALARM` (or `USE_EXACT_ALARM`), otherwise a `SecurityException`. `setExact*` with an `OnAlarmListener` needs no permission, but only while the process lives, so it is useless for reminders. | verified: https://developer.android.com/develop/background-work/services/alarms , https://developer.android.com/about/versions/14/changes/schedule-exact-alarms |
| `USE_EXACT_ALARM` (auto-granted, not revocable) | **No for a to-do app** | manifest | Play: only "an alarm or timer app" or "a calendar app that shows event notifications". Others may be blocked from publishing. A Play Console declaration is needed. | verified: https://support.google.com/googleplay/android-developer/answer/16558241 |
| `SCHEDULE_EXACT_ALARM` | Yes | manifest + `canScheduleExactAlarms()` + `ACTION_REQUEST_SCHEDULE_EXACT_ALARM` | Denied by default for new installs targeting 33+ (Android 14+). Also denied after backup/restore to a new device. User-revocable: "your app stops, and all future exact alarms are canceled". Listen for `ACTION_SCHEDULE_EXACT_ALARM_PERMISSION_STATE_CHANGED` and reschedule. No Play declaration needed. | verified: same two URLs above. "No declaration" = verified-secondary (GitHub PR discussions in search results) |
| Fallback without the permission | Yes, degraded | `setWindow()` (window ≥10 min), `setAndAllowWhileIdle()`, inexact `set()` (within ~1 h on Android 12+) | Show the user "reminders may be up to N minutes late" | verified: https://developer.android.com/develop/background-work/services/alarms |
| Fire in Doze | Yes | `setAlarmClock()` (unthrottled), `*AllowWhileIdle` (≤1 per 9 min per app) | Regular alarms, jobs and WorkManager do not run in Doze | verified: https://developer.android.com/training/monitoring-device-state/doze-standby |
| Battery-optimization exemption prompt | Mostly no | `ACTION_REQUEST_IGNORE_BATTERY_OPTIMIZATIONS` | Play: "may not request direct exemption … unless core function adversely affected". The acceptable list includes "Task automation app" and safety apps. A to-do or reminder app is not clearly listed. Use `ACTION_IGNORE_BATTERY_OPTIMIZATION_SETTINGS` (the list screen) instead. Even with the exemption, "regular alarms still don't fire" in Doze. | verified: https://developer.android.com/training/monitoring-device-state/doze-standby ; to-do eligibility = inferred (not in the acceptable list) |
| Survive reboot | Yes | `RECEIVE_BOOT_COMPLETED` receiver → reschedule all alarms from the local DB | Only after the user has launched the app once. Alarms are cancelled at shutdown. Android 15+: a `BOOT_COMPLETED` receiver cannot start `dataSync`, `camera`, `mediaPlayback`, `phoneCall`, `mediaProjection` or `microphone` FGS. | verified: https://developer.android.com/develop/background-work/services/alarms , https://developer.android.com/about/versions/15/behavior-changes-15 |
| Also reschedule on | — | `TIME_SET`, `TIMEZONE_CHANGED`, `MY_PACKAGE_REPLACED`, exact-alarm permission change | Time-zone correctness matters for reminders | unverified (standard Android broadcasts, recall; not fetched this session) |
| OEM killers | Hard reality | dontkillmyapp.com guidance, per-vendor autostart and battery settings deep links | Worst: Huawei, Xiaomi, OnePlus, Samsung (score 5). The ranking is undated. | verified: https://dontkillmyapp.com/ |
| Android 17 alarm-related changes | Small | — | No alarm change found for all apps. At target 37, background audio needs a WIU FGS **or** the exact-alarm permission plus `USAGE_ALARM` audio, which matters if the reminder plays an alarm sound. | verified: https://developer.android.com/about/versions/17/behavior-changes-all , https://developer.android.com/about/versions/17/behavior-changes-17 |
| Android 16 jobs | — | — | JobScheduler/WorkManager runtime quotas tightened (including jobs alongside an FGS); `setImportantWhileForeground` ignored. Keep sync in WorkManager, but **never rely on WorkManager for timing**. | verified: https://developer.android.com/about/versions/16/behavior-changes-all |

### 2.2 Notifications

| Capability | Possible? | How | Restriction | Grade |
|---|---|---|---|---|
| Post notifications | Yes | `POST_NOTIFICATIONS` runtime permission (Android 13+) | Off by default on new installs. Ask in context, for example when the first reminder is created. | verified: https://developer.android.com/develop/ui/views/notifications/notification-permission |
| Channels (per type: reminders, overdue, daily digest, sync errors) | Yes | `NotificationChannel` with importance; the user controls each channel | The app cannot raise importance after creating a channel (needs a new channel ID) | unverified (well-known API; not fetched) |
| Action buttons (Done, Snooze 10 min, Tomorrow) and inline reply (quick-add from a notification) | Yes | `NotificationCompat.Action` + `RemoteInput`; the BroadcastReceiver acts without opening the app | — | unverified (well-known API; not fetched) |
| Full-screen "alarm-style" reminder | Conditional | `setFullScreenIntent` | Not default-granted for a to-do app since 2025-01-22. The user enables it in settings (`ACTION_MANAGE_APP_USE_FULL_SCREEN_INTENT`). | verified: https://support.google.com/googleplay/android-developer/answer/13392821 , https://source.android.com/docs/core/permissions/fsi-limits |
| Live Updates / ProgressStyle (Android 16) | Only for a running timer or focus session | `POST_PROMOTED_NOTIFICATIONS` + `setRequestPromotedOngoing`, ongoing, contentTitle, no custom view, not colorized | "Inappropriate uses: … alerts, upcoming calendar events" | verified: https://developer.android.com/develop/ui/compose/notifications/live-update |
| Auto-grouping | System behaviour | Android 16 auto-groups notifications on the app's behalf | Design for grouping (one summary per day) | verified-secondary (Android 16 summary in search results: https://developer.android.com/about/versions/16/summary) |
| DND | Indirect | Apps targeting 15+ cannot change global DND; they contribute an `AutomaticZenRule` | Could build a "focus mode" that creates a zen rule | verified: https://developer.android.com/about/versions/15/behavior-changes-15 |
| Push from a server | Yes | FCM. The docs now describe v2 `RemoteNotification` (user-visible, wakes the app in Doze) vs `BackgroundSync` (may be throttled); v1 high-priority is deprecated | Needed for server-driven reminders (shared lists, calendar changes) | verified: https://developer.android.com/training/monitoring-device-state/doze-standby |

### 2.3 Home screen, quick capture, assistant

| Capability | Possible? | How | Restriction | Grade |
|---|---|---|---|---|
| Interactive home-screen widgets (list with checkboxes, add button) | Yes | Jetpack **Glance 1.2.0 stable (2026-08-26)**, 1.3.0-alpha02. Actions: `actionRunCallback`, lambdas; Checkbox/Switch/RadioButton composables; generated previews (`providePreview`, `setWidgetPreview`) | `updatePeriodMillis` ≥ 30 min (push updates yourself when data changes). Broadcast receivers have 10 s before ANR. **API 37: bitmap budget enforced with an exception.** | verified: https://developer.android.com/jetpack/androidx/releases/glance , https://developer.android.com/develop/ui/views/appwidgets/advanced , https://developer.android.com/about/versions/17/behavior-changes-17 |
| Widget **text input** | No | Widgets have no text field. Use an "add" button that opens a translucent quick-add activity or dialog. | Hard limit | inferred (Glance component list has buttons/checkbox/switch/radio, no text field: https://developer.android.com/jetpack/androidx/releases/glance) |
| Quick Settings tile ("quick add") | Yes | `TileService` (active tile recommended); `requestAddTileService()` prompt (Android 13+); `startActivityAndCollapse()` to open quick-add; `unlockAndRun` when locked | Max 2 recommended per app. The prompt stops after repeated denials. Android 14 PendingIntent overload of startActivityAndCollapse: unverified (the page itself was silent). | verified: https://developer.android.com/develop/ui/views/quicksettings-tiles |
| App shortcuts (long-press icon: "New task", "Today") | Yes | static `shortcuts.xml` + dynamic `ShortcutManagerCompat` + pinned | Launchers show up to 4. `capabilities` in shortcuts.xml are how Assistant surfaces them. | verified: https://developer.android.com/develop/ui/views/launch/shortcuts |
| Share target ("share to tasks" from any app) | Yes | `ACTION_SEND` / `SEND_MULTIPLE` intent filters (`text/plain`, `image/*`), optional Direct Share via sharing shortcuts | Validate MIME and size. Android 17 docs say implicit URI grants for ACTION_SEND will stop being automatic (planned for 18), so add the grant flags explicitly when *sending*. | verified: https://developer.android.com/training/sharing/receive , https://developer.android.com/about/versions/17/behavior-changes-all |
| Voice quick capture | Yes (in-app) | `SpeechRecognizer` / `RecognizerIntent`, or on-device ASR | System assistant integration is the weak point (next row) | unverified (APIs not fetched this session) |
| Gemini / AI agents calling the app | **Not yet for third parties** | **AppFunctions** (Android 16+, `EXECUTE_APP_FUNCTIONS` callers) | "experimental preview … subject to change". Gemini integration "in a private preview with trusted testers" as of May 2026. An Early Access form exists. Treat it as a roadmap item behind a flag. | verified: https://developer.android.com/ai/appfunctions , https://android-developers.googleblog.com/2026/05/android-ai-intelligence-system.html |
| Legacy App Actions / built-in intents | Exists, unclear future | `shortcuts.xml` capabilities | No official deprecation found. Secondary sources call App Actions the predecessor of AppFunctions. | verified-secondary: https://ecorpit.com/android-app-agent-ready-gemini-app-actions-2026/ ; reference page https://developer.android.com/reference/app-actions/built-in-intents |

### 2.4 Wear OS, location, calendar provider

| Capability | Possible? | How | Restriction | Grade |
|---|---|---|---|---|
| Wear notifications | Yes, free | Phone notifications bridge to the watch with their actions | — | unverified (standard behaviour, not fetched) |
| Wear Tiles | Yes | Jetpack `protolayout` + `tiles` (declarative, not scrollable, no long work in the service) | Glance's `glance-wear-tiles` is **deprecated** in favour of `androidx.glance.wear` | verified: https://developer.android.com/training/wearables/tiles , https://developer.android.com/jetpack/androidx/releases/glance |
| Wear **Widgets** (successor of tiles) | Coming | Announced at I/O 2026 for **Wear OS 7**, 2×1 / 2×2, built on Glance / Remote Compose. Tiles still supported "for the time being". Todoist named as an early adopter. | Build for widgets if you do Wear at all; low priority for v1 | verified-secondary: https://9to5google.com/2026/05/22/google-explains-new-wear-os-widgets/ , https://developer.android.com/training/wearables/widgets |
| Location reminders (geofence) | Yes | Play services `GeofencingClient` | **Max 100 geofences per app per user.** Needs `ACCESS_FINE_LOCATION` + `ACCESS_BACKGROUND_LOCATION` (Play background-location declaration). Latency usually <2 min, 2–3 min typical, up to 6 min when stationary. Re-register after reboot, reinstall or data clear. | verified: https://developer.android.com/develop/sensors-and-location/location/geofencing ; Play declaration requirement = unverified (not fetched) |
| CalendarContract (device calendars) | Yes | `READ_CALENDAR` / `WRITE_CALENDAR`; Events, Instances (read-only), Reminders tables; writes to a Google-account calendar sync through that account's sync adapter. Can create an `ACCOUNT_TYPE_LOCAL` calendar as a sync adapter (never synced). | **No tasks in CalendarContract.** It is a cheap way to read every calendar the phone already syncs (Google, Exchange, DAVx⁵ CalDAV) without OAuth. | verified: https://developer.android.com/guide/topics/providers/calendar-provider ; "writes sync up" = inferred from the deleted-flag and sync-adapter text |
| Device task providers (CalDAV VTODO without own CalDAV code) | Yes, niche | DAVx⁵ syncs VTODO into jtx Board, OpenTasks or Tasks.org (one at a time) | Could integrate with the OpenTasks/jtx provider, or implement CalDAV directly (cleaner for Linux too) | verified: https://manual.davx5.com/tasks_notes.html |

### 2.5 Foreground services (for completeness)

| Rule | Grade |
|---|---|
| Android 14+: each FGS must declare a type, plus a Play Console declaration with a video. A reminder app should not need an FGS. Use alarms + notifications. | verified: https://support.google.com/googleplay/android-developer/answer/13392821 |
| Android 15: `dataSync` FGS limited to 6 h per 24 h (`onTimeout`); SYSTEM_ALERT_WINDOW exemption narrowed | verified: https://developer.android.com/about/versions/15/behavior-changes-15 |
| Android 15 BAL hardening; Android 17 extends BAL protections to `IntentSender` (use `MODE_BACKGROUND_ACTIVITY_START_ALLOW_IF_VISIBLE`) | verified: https://developer.android.com/about/versions/15/behavior-changes-15 , https://developer.android.com/about/versions/17/behavior-changes-17 |
| Android 16 target: edge-to-edge forced, predictive back on, orientation/resizability ignored on ≥600dp | verified: https://developer.android.com/about/versions/16/behavior-changes-16 |

---

## 3. iOS / iPadOS (for later)

| Capability | Possible? | How | Restriction / hard limit | Grade |
|---|---|---|---|---|
| Local reminders | Yes | `UNUserNotificationCenter` | **64 pending requests per app**; the system keeps the soonest-firing 64 (a repeating request counts as one). Scheduling past 64 "succeeds" but items are dropped. Needs a rolling-window refill. | verified-secondary: https://developer.apple.com/forums/thread/811171 , https://developer.apple.com/forums/thread/765490 |
| Background refresh | Best-effort | `BGAppRefreshTask` | The system picks the time. "There is no such thing as scheduling a background task at a certain time" (Apple DTS) | verified-secondary: https://developer.apple.com/forums/thread/770479 |
| Alarm-grade alerts | Yes (iOS 26+) | **AlarmKit**: `AlarmManager.requestAuthorization()`, `NSAlarmKitUsageDescription` required; one-time, weekly-repeating, countdown | "It overrides both a device's focus and silent mode". A **widget extension is required** if a countdown is used, "Otherwise, the system may unexpectedly dismiss alarms". Forum reports of late or missed fires. | verified: https://developer.apple.com/documentation/alarmkit/scheduling-an-alarm-with-alarmkit.md ; reliability = verified-secondary https://developer.apple.com/forums/thread/846063 |
| Siri / Shortcuts / Spotlight / Action button / Apple Intelligence | Yes | **App Intents** (iOS 16+), App Shortcuts, app entities, app schema domains | App Intents is the single integration surface (also for widgets, controls, Focus) | verified: https://developer.apple.com/documentation/appintents.md |
| Interactive widgets | Yes (iOS 17+) | `Button(intent:)` / `Toggle(isOn:intent:)` backed by App Intents; small through extra-large and lock-screen accessory families | **Only buttons and toggles**, no text input. Inactive on a locked device until unlocked. Toggle is optimistic, so reconcile with the DB. | verified: https://developer.apple.com/documentation/widgetkit/adding-interactivity-to-widgets-and-live-activities.md |
| Live Activities | Yes (running timers or focus sessions) | ActivityKit, buttons via `LiveActivityIntent`; updates by APNs push | **Max 8 h active, ≤12 h on lock screen; 4 KB data**. Usually started in the foreground (or via an App Intent). Can be scheduled for a date. | verified: https://developer.apple.com/documentation/activitykit/displaying-live-data-with-live-activities.md |
| Focus filters | Yes | `SetFocusFilterIntent` (show only work lists in the Work Focus) | — | verified: https://developer.apple.com/documentation/appintents/focus.md |
| EventKit reminders / calendar | Yes | `requestFullAccessToReminders`; events: write-only or full | **Reminders have only full access (no read-only or write-only).** iOS 17+ auto-denies if the new Info.plist keys are missing. EventKitUI editor needs no permission. | verified: https://developer.apple.com/documentation/eventkit/accessing-the-event-store.md |
| Reading iCloud Reminders from non-Apple platforms | **No** | — | See H9 | verified-secondary (see H9) |

---

## 4. LINUX: Ubuntu 26.04, GNOME 50, Wayland (the user's machine)

Local probe (verified-local; `busctl --user introspect org.freedesktop.portal.Desktop /org/freedesktop/portal/desktop`, `get-property … version`, `gnome-extensions list --enabled`):
- Portals present include `Notification` (**version 1**), `GlobalShortcuts` (**version 1**), `Background` (**version 2**), `Secret`, `OpenURI`, `Settings`, `Inhibit`, `DynamicLauncher`.
- `org.kde.StatusNotifierWatcher` is owned by gnome-shell. `ubuntu-appindicators@ubuntu.com` is ENABLED/ACTIVE, together with `status-icons@gnome-shell-extensions` and `dash-to-panel`.

| Capability | Possible? | How | Restriction / hard limit | Grade |
|---|---|---|---|---|
| Native notifications | Yes | `org.freedesktop.portal.Notification` (works sandboxed or not) or `org.freedesktop.Notifications` (libnotify) directly when unsandboxed | The spec documents v2 (buttons with purpose, `alarm.ringing` category, sounds, display-hint), but **GNOME on this machine exposes v1**, so expect plain title/body/buttons/priority. Clicking after the app exits re-activates it via D-Bus activation (exported `app.` actions), which needs a D-Bus-activatable `.desktop`. | spec verified: https://flatpak.github.io/xdg-desktop-portal/docs/doc-org.freedesktop.portal.Notification.html ; v1 = verified-local |
| Reminders when the app is closed | Only if a process runs | A **background daemon/tray process** (autostarted), or a **systemd user timer** that fires a small notifier | Linux has no OS-level scheduled-notification store like Android or Windows. Someone must be running at the time. | inferred (the Notification portal has no "schedule at" field — spec above) |
| Autostart at login | Yes | Background portal `RequestBackground(autostart=true, commandline=…)` (v2, `SetStatus`), or the classic `~/.config/autostart/*.desktop` for unsandboxed builds | Portal doc speaks of sandboxed apps only | portal verified: https://flatpak.github.io/xdg-desktop-portal/docs/doc-org.freedesktop.portal.Background.html ; XDG autostart = unverified (spec not fetched) |
| Tray icon (quick add, today count) | Best-effort | StatusNotifierItem (`org.kde.StatusNotifierWatcher`) through the Ubuntu AppIndicators extension | Upstream GNOME has no tray. Ubuntu ships the extension (active here). **Open bug #628 on GNOME 50 / Ubuntu 26.04**: items register then get dropped (suspected `IconPixmap` handling; a Dash-to-Panel conflict was also floated, and Dash-to-Panel is enabled here). In Flatpak: `--talk-name=org.kde.StatusNotifierWatcher`, avoid `--own-name=org.kde.*`. | verified: https://github.com/ubuntu/gnome-shell-extension-appindicator/issues/628 , https://docs.flatpak.org/zh-cn/latest/desktop-integration.html , https://discourse.flathub.org/t/permissions-for-showing-tray-icon/10785 ; verified-local |
| Global quick-add shortcut | Yes, user-approved | `org.freedesktop.portal.GlobalShortcuts`: `CreateSession` → `BindShortcuts` (description + optional `preferred_trigger`) → `Activated` signal (with activation token to raise the window) | GNOME 48+ "fully supported"; apps "must request permission" and users can remove shortcuts in Settings > Apps. xdp ≥1.21.0 "Require[s] a valid AppID", so an unsandboxed app needs a resolvable `.desktop`/app-id (host app registry). Machine has portal **v1**, so **no `ConfigureShortcuts`** (v2) here. Bind once per session. | verified: https://release.gnome.org/48/developers/index.html , https://flatpak.github.io/xdg-desktop-portal/docs/doc-org.freedesktop.portal.GlobalShortcuts.html , https://github.com/flatpak/xdg-desktop-portal/blob/main/NEWS.md ; v1 = verified-local |
| Fallback shortcut | Yes | GNOME Settings > Keyboard > Custom Shortcut running `myapp --quick-add` (single-instance IPC) | Always works on Wayland, no portal needed; a robust plan B | inferred (standard GNOME feature; not fetched this session) |
| Secret storage for OAuth tokens | Yes | `org.freedesktop.portal.Secret` / libsecret (GNOME Keyring) | — | verified-local (portal present) |
| Packaging | Several | **Flatpak** (Flathub; portals are first-class; Flathub reviews permissions), **Snap** (Ubuntu default store; portals via snapd), **.deb** (simplest for the user himself; unsandboxed, needs `.desktop` for the shortcuts portal), **AppImage** (no install integration, so a poor fit for portals and autostart) | Flathub and Snap Store review processes add lead time for a public release | Flatpak tray detail verified (above); Snap/AppImage behaviour = unverified (not researched in depth) |

---

## 5. Windows and macOS (brief, for later)

| Platform | Capability | Notes | Grade |
|---|---|---|---|
| Windows | Toast notifications incl. **scheduled toasts** (verified to exist; that the OS holds them and fires them with the app closed is inferred) | Unpackaged apps must register an AUMID + toast activator CLSID (Start-menu shortcut). AUMID >129 chars makes scheduled toasts fail with 0x8007007A. MSIX packaging avoids this. | verified: https://learn.microsoft.com/en-us/windows/apps/design/shell/tiles-and-notifications/send-local-toast-other-apps |
| Windows | Global hotkey, tray | `RegisterHotKey`, `Shell_NotifyIcon`: standard and unrestricted | unverified (recall; not fetched) |
| macOS | Notifications | `UNUserNotificationCenter` needs a signed bundle with a bundle ID; interruption levels (time-sensitive) on macOS 12+ | verified-secondary: https://docs.rs/crate/mac-usernotifications/latest/source/src/lib.rs , https://desktop-notifier.readthedocs.io/en/latest/_modules/desktop_notifier/macos.html |
| macOS | Menu bar item | `NSStatusItem`; macOS 26 Tahoe may hide crowded items silently | verified-secondary: https://engineered.at/articles/mac-menu-bar-chaos |
| macOS | EventKit, App Intents, widgets | Same frameworks as iOS (App Intents macOS 13+) | verified: https://developer.apple.com/documentation/appintents.md |
| macOS | Global hotkey | Carbon `RegisterEventHotKey` (no Accessibility permission) vs event taps (need permission) | unverified (not fetched) |

---

## 6. CALENDARS and TASK BACK-ENDS

### 6.1 Google

| Item | Fact | Grade |
|---|---|---|
| Calendar API scopes | 20 scopes incl. `calendar`, `calendar.events`, `calendar.events.owned`, `calendar.app.created` (create secondary calendars and manage their events, the narrowest write option), `calendar.readonly`, `calendar.calendarlist.readonly` | verified: https://developers.google.com/workspace/calendar/api/auth |
| Classification | Calendar and Tasks scopes are **not** on Google's restricted-scope list (restricted = Gmail, Drive, Fit, Chat, Data Portability, Photos Ambient, Health) → **sensitive**, so verification without a CASA assessment | restricted list verified: https://support.google.com/cloud/answer/13464325 ; "sensitive" = inferred (absence from restricted + multiple secondary sources) |
| Sensitive verification requirements | Homepage on a verified domain, privacy policy (same on homepage and consent screen), Search Console domain verification, English demo video of the full OAuth flow. No cost or timeline published. | verified: https://support.google.com/cloud/answer/13464321 |
| Unverified cap / exemptions | 100 new users total after the unverified screen. Exempt: personal use (<100 users), dev/test/staging, internal Workspace org. | verified: https://support.google.com/cloud/answer/7454865 , https://support.google.com/cloud/answer/13464323 |
| Token pitfalls | "Testing" status → refresh tokens expire in **7 days** (unless only basic profile scopes). Max **100 refresh tokens per Google Account per client ID**; the oldest is silently invalidated. | verified: https://developers.google.com/identity/protocols/oauth2 |
| Restricted-scope cost (only if ever needed) | Annual CASA by an empanelled assessor; third-party price quotes vary widely ($500–$4,500+/yr) | requirement verified: https://developers.google.com/identity/protocols/oauth2/production-readiness/restricted-scope-verification ; price verified-secondary: https://deepstrike.io/blog/google-casa-security-assessment-2025 |
| Google Tasks API | Resources: tasklists, tasks (`clear`, `move`, …). Task fields: title, notes, status (`needsAction`/`completed`), parent (read-only, use `move`), links, due (**date only**), deleted, hidden, webViewLink. **No reminders, no recurrence, no time.** | verified: https://developers.google.com/workspace/tasks/reference/rest/v1/tasks |
| Tasks scopes | `tasks`, `tasks.readonly` | verified: https://developers.google.com/workspace/tasks/auth |
| Calendar push | Webhook channels (HTTPS, valid cert), header-only notifications, channels expire and must be recreated, delivery not guaranteed, so always keep incremental sync (sync tokens) | verified: https://developers.google.com/workspace/calendar/api/guides/push |
| CalDAV | No VTODO, OAuth only, no MKCALENDAR, no free-busy | verified: https://developers.google.com/workspace/calendar/caldav/v2/guide |

### 6.2 Microsoft (Outlook / Microsoft 365 / To Do)

| Item | Fact | Grade |
|---|---|---|
| Calendar | Graph calendar API; most features work for personal MSA and work/school accounts; delta query + change notifications | verified: https://learn.microsoft.com/en-us/graph/outlook-calendar-concept-overview |
| To Do API | `todoTaskList` → `todoTask` (title, body, `dueDateTime`, **`reminderDateTime` + `isReminderOn`**, **`recurrence`**, `startDateTime`, importance, status incl. `waitingOnOthers`/`deferred`, categories) + `checklistItem` subtasks + `linkedResource`; delta query on tasks and lists; delegated and application permissions. **Far richer than Google Tasks: a true two-way task target.** Task id "changes when the item is moved from one list to another". | verified: https://learn.microsoft.com/en-us/graph/api/resources/todotask?view=graph-rest-1.0 , https://learn.microsoft.com/en-us/graph/api/resources/todo-overview?view=graph-rest-1.0 |
| Webhooks | Outlook event subscriptions max 10,080 min (<7 days); `todoTask` max 4,230 min (<3 days), latency <2 min avg / 15 min max; HTTPS endpoint required, so a backend is needed for push | verified: https://learn.microsoft.com/en-us/graph/api/resources/subscription?view=graph-rest-1.0 |
| Publisher verification | See H15 (free, needs CPP + Entra work tenant) | verified: https://learn.microsoft.com/en-us/entra/identity-platform/publisher-verification-overview |

### 6.3 Apple iCloud

| Item | Fact | Grade |
|---|---|---|
| Auth for third-party CalDAV | App-specific password; requires 2FA; max **25 active**; **all revoked when the Apple Account password changes** | verified: https://support.apple.com/en-us/102654 |
| Calendar events | Reachable over CalDAV | verified-secondary: https://www.davx5.com/tested-with/icloud |
| Reminders | Upgraded lists are **not** reachable over CalDAV (see H9); EventKit only, on Apple devices | verified-secondary |

### 6.4 Nextcloud / generic CalDAV

| Item | Fact | Grade |
|---|---|---|
| Endpoint | `https://<server>/remote.php/dav` | verified: https://docs.nextcloud.com/server/latest/user_manual/en/groupware/sync_android.html |
| Auth | Use an **app password**; a plain password fails when 2FA is enabled | verified: same URL |
| Tasks | VTODO supported (Nextcloud Tasks; DAVx⁵ → OpenTasks/jtx/Tasks.org) | verified: https://manual.davx5.com/tasks_notes.html , Nextcloud URL above |
| Protocol fit | CalDAV VTODO (RFC 5545 VTODO with DUE, RRULE, VALARM) is the **only open standard** that carries timed, recurring tasks with alarms, so it is a strong canonical model for interop | inferred (VTODO feature set is standard; contrast with Google Tasks fields above) |

---

## 7. Design implications (inferred from the verified facts above)

1. **Reminder engine is per-platform native code, whatever the UI framework.** Android: `setAlarmClock` + exact-alarm onboarding + boot/time-zone/permission-change rescheduling + OEM guidance. iOS: 64-slot rolling window + AlarmKit opt-in for "must not miss". Linux: an autostarted background process or systemd user timer. Windows: scheduled toasts. A framework that makes platform channels or native modules painful is a real cost here. (inferred from H1–H6, H13, §4)
2. **The app's own store is the source of truth; external services are adapters.** Google Tasks cannot carry times; iCloud Reminders cannot be reached off-Apple; Microsoft To Do and CalDAV VTODO can carry the full model. Map: timed items → Google Calendar events (in a dedicated `calendar.app.created` calendar, so the narrowest scope) or MS To Do / CalDAV VTODO. (inferred from H7–H9, §6)
3. **Push-capable sync needs a small backend** with HTTPS webhooks: Google channels and Graph subscriptions both require one and expire within days (MS To Do in under 3 days). A local-only v1 can poll with sync tokens or delta queries instead. (inferred from §6.1/§6.2)
4. **Ship personal-use first, legally.** Google OAuth "In production / unverified" under 100 users avoids the 7-day token expiry; Microsoft personal MSA needs no verification; iCloud/Nextcloud use app passwords. Going public later adds a verified domain, privacy policy, demo video (Google) and a CPP + work tenant (Microsoft, for org users). (inferred from H10, H15)
5. **Desktop entry points on GNOME, ranked by reliability:** notification (portal v1) > custom GNOME keyboard shortcut calling a CLI > GlobalShortcuts portal (needs app-id + user approval) > tray (extension-dependent, currently buggy on GNOME 50). (inferred from §4 + local probe)
6. **AI-assistant integration is not shippable on Android yet** (AppFunctions is an experimental preview, Gemini integration is a private preview). On iOS, App Intents is mature and doubles as the widget, Siri and Focus API. Plan the domain layer around "intents" (create task, complete, snooze, query today) so both can be wired later. (inferred from §2.3, §3)

## 8. What was NOT verified in this dive (explicit)
- Notification channel / action / RemoteInput API details, `TIME_SET`/`TIMEZONE_CHANGED` rescheduling, `SpeechRecognizer`: well-known APIs, not fetched (unverified).
- Play background-location declaration specifics for geofencing (unverified).
- Whether the GNOME 50 notification backend will move to portal v2 (only the v1 local observation is verified).
- Snap and AppImage portal and autostart behaviour (not researched in depth).
- Windows `RegisterHotKey`/tray, macOS global hotkey APIs (recall only).
- Exact Google OAuth verification turnaround time (Google publishes none).
