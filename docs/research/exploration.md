# Task / Reminder App: Definitive Exploration Report

*Synthesis date: 2026-11-01. Author role: lead product architect. Inputs: six research dives (competitors, innovation, OS integration, framework, sync/backend, risks/AI/cost) plus six adversarial verification passes. **Where a verification pass corrected a research dive, the correction is used here.***

**Evidence grades used throughout**
- **[Verified: url]**: the source was fetched or searched during the research (by the dive or its verifier), and the URL is given.
- **[Verified-2nd: url]**: verified only against third-party or secondary sources.
- **[Verified-local]**: probed on the user's own machine (Ubuntu 26.04.1, GNOME Shell 50.1, Wayland).
- **[Inferred: basis]**: reasoned from verified facts; the basis is named.
- **[Unverified: why]**: could not be checked; the reason is given.
- **[Speculative]**: product or design judgement with no checkable fact behind it (all effort numbers are this grade).

---

## 1. Executive summary

**Verdict: feasible, with one hard engineering core.** Every surface the user asked for can be built on Android and Ubuntu GNOME Wayland today. No hard blocker exists for the personal MVP. The difficulty is concentrated in one place: **reminders that always fire**. On Android that means a native exact-alarm engine that survives Doze, reboots, permission changes and OEM battery killers. On Linux it means a resident scheduler, because the OS has no scheduled-notification store and the main Flutter notification plugin cannot schedule on Linux. [Verified: https://developer.android.com/develop/background-work/services/alarms ; https://pub.dev/packages/flutter_local_notifications] Two-way calendar sync is the second-hardest part. It is feasible for Google (events), Microsoft (events and To Do tasks), Nextcloud/CalDAV (events and tasks) and iCloud (events only). **iCloud Reminders cannot be reached from Android or Linux at all.** [Verified-2nd: https://www.busymac.com/docs/faqs/112990-reminders-in-ios-13-and-macos-catalina-drops-support-for-caldav/]

**Recommended stack in one line:** **Flutter 3.47.9 (move to 3.50 when stable) + an in-repo Kotlin Android module (Glance widgets, AlarmManager engine) + a Linux tray daemon with a systemd-user fallback, drift/SQLite synced by PowerSync Open Edition to the user's existing Postgres 18 and Keycloak, a server-side calendar bridge in Docker, and a local MCP server.**

**MVP time (personal daily driver on Android + Ubuntu, with sync, no external calendars yet):** about **540 focused hours (likely; range 360–850)**. That is **~7 months as a side project at 15–20 h/week**, or **~4 months full-time at 35 h/week**, plus ~60 h (3–4 weeks on the side) of de-risking spikes first. A first useful build (Android + Ubuntu, local only, no sync) comes after about 180 h, or ~2.5 months on the side. Through Phase 3, the full "genius" feature set is ~1,500 h: ~20 months on the side, ~10 months full-time. [Speculative: §8 gives the derivation.]

**The framework decision is closer than the user may expect.** Kotlin Multiplatform + Compose Multiplatform ties Flutter on an unweighted score. The best open-source competitor, Tasks.org, is built on KMP. Flutter wins on Linux desktop depth, which is ready today (portal hotkeys, SNI tray, Linux sync SDK), and on toolchain fit. The margin is small, and §5 says so plainly.

---

## 2. Product vision

### 2.1 Thesis (one sentence)

> **"A local-first reminder system that never lets a commitment slip: capture it in one breath, attach it to the real-life moment you'll act on it, and let an honest, calendar-aware plan, and your AI agents, keep it moving until it's done."** [Speculative: positioning, built on the evidence in §2.3]

### 2.2 Working names (none is a well-known app; conflicts disclosed)

| Name | Idea | Collision check |
|---|---|---|
| **Holdfast** | "It holds on until it's done": the nag-until-done promise | Only a minor Chrome Pomodoro/site-blocker extension uses it. No task app found. [Verified: single standard search; https://alternativeto.net/software/holdfast/] |
| **Cuewise** | Implementation-intention cues ("when X, do Y") | Only a Devpost hackathon project (streaming recommendations) uses it. No store listing found. [Verified: single standard search; https://devpost.com/software/cuewise] |
| **Whenly** | "When" as the core concept: when-cues, when-plans | **Weakest**: a small iOS world-holidays calendar app *with reminders* uses it, which is an adjacent category. [Verified: single standard search; https://apps.apple.com/ru/app/id6759036480] |

Recommendation: **Holdfast**. A trademark search (EUIPO/USPTO) and a domain check are still needed. [Unverified: not done]

### 2.3 Target persona: the user

- A solo developer, expert in bash, Docker and Linux. He uses an **Android phone** daily and an **Ubuntu 26.04 GNOME Wayland** PC, and lives in Claude Code. He is in Europe/Paris, so GDPR applies and the parser must handle French and English. [Verified-local for OS/GNOME; timezone from the user's setup]
- Pains, as mapped by the competitor dive:
  - commercial apps are weak or web-only on Linux;
  - there is no Wayland quick-add;
  - Android reminders are unreliable;
  - calendar sync usually means one provider, one-way;
  - AI planners cost $15–34/month and keep pivoting or shutting down (Clockwise closed 2026-03-27).

  [Verified: https://www.morgen.so/pricing ; https://akiflow.com/pricing ; https://www.sunsama.com/pricing ; Clockwise shutdown confirmed by reputable third-party reports in competitors.verify.md]
- Later public audience [Speculative]: technical and privacy-minded prosumers on Android and Linux, then ADHD/time-blindness users drawn by the cue and nag features.

### 2.4 Three signature features

1. **Moments: cues that won't let go.** Every task can carry a *Moment*:
   - a time;
   - a place (arrive or leave);
   - a device (car or headphones Bluetooth connect/disconnect);
   - driving start/stop;
   - a PC event (login, unlock, workday start).

   Delivery escalates and varies: a gentle notification, then a repeat at a learned cadence with a different sound and style, then an opt-in full-screen alarm tier, until the task is Done, Rescheduled or Dropped. Chronic snoozing triggers "break it down, move it, or drop it?".
   - Evidence: implementation intentions improve goal attainment with **d ≈ 0.27–0.66 depending on the outcome**. That is small-to-large, not uniformly large. [Verified: Sheeran, Listrom & Gollwitzer, Eur. Rev. Soc. Psychol. 36(1), https://kops.uni-konstanz.de/handle/123456789/69905]
   - Habit apps that lean on reminders hinder automaticity, while event cues build it, and most apps lack event cues. [Verified-2nd: Stawarz et al. CHI 2015, https://discovery.ucl.ac.uk/id/eprint/1468224/]
   - Repeated identical alerts habituate users, so the alert must vary. [Verified-2nd: https://news.byu.edu/news/study-android-users-how-get-people-pay-attention-mobile-security-notifications]
   - Bluetooth ACL broadcasts can wake a killed app. [Verified: https://developer.android.com/develop/background-work/background-tasks/broadcasts/broadcast-exceptions]
2. **Brain dump → Honest Day.** Speak or paste a messy paragraph. The AI returns tasks, durations and sub-steps as a **reviewable diff**. The planner then builds today against your calendars, using **your own estimate-vs-actual history** ("you usually take 1.5× on #admin"), and re-flows the plan when you slip. A morning plan and an evening shutdown close the loop.
   - Evidence: the planning fallacy, and its correction with past data. [Verified-2nd: https://en.wikipedia.org/wiki/Planning_fallacy]
   - Making a concrete plan quiets unfinished-goal intrusion. [Verified: https://pubmed.ncbi.nlm.nih.gov/21688924/]
   - No competitor found calibrates durations against personal history. [Unverified: absence not exhaustively checked]
3. **Agent-native and local-first.** The data lives in on-device SQLite and works fully offline, with optional end-to-end encryption per project and a self-hostable backend. A **local MCP server** ships on the Ubuntu app, so Claude Code can add, query, complete and plan tasks. A remote MCP server comes later.
   - Honest note: MCP is now **table stakes**. Todoist runs an official hosted MCP, and Super Productivity v19.3.0 added an opt-in **local** MCP. [Verified: https://www.todoist.com/help/articles/use-claude-code-with-todoist-cli-and-mcp-b1USJ4HB3 ; https://github.com/super-productivity/super-productivity/releases]
   - The differentiator is therefore the *combination* of native GNOME depth, reliability and a private agent interface, not MCP alone. [Inferred: from the two facts above]

### 2.5 Full feature map

Tags: **MVP** (Phase 1), **v1** (Phase 2, daily driver), **v2** (Phase 3, AI and context), **later** (Phase 4+). Effort and value judgements are [Speculative]; the constraints are graded.

| Pillar | Feature | Tag | Key constraint / note |
|---|---|---|---|
| **Capture** | Natural-language quick-add, FR + EN, deterministic and offline ("dentist mar 15h toutes les 2 semaines #santé !1") | MVP | `chrono_dart` is English-only and ~2 years stale, so write our own grammar with golden tests. [Verified: https://pub.dev/packages/chrono_dart] |
| | Android share-sheet target (text, URL, image) | MVP | `ACTION_SEND` filters. When *sending*, add the grant flags explicitly, because implicit URI grants end in Android 18. [Verified: https://developer.android.com/training/sharing/receive ; https://developer.android.com/about/versions/17/behavior-changes-all] |
| | Quick Settings tile + launcher shortcuts + notification inline reply | MVP | `TileService`, `requestAddTileService()`; launchers show at most 4 shortcuts. [Verified: https://developer.android.com/develop/ui/views/quicksettings-tiles ; https://developer.android.com/develop/ui/views/launch/shortcuts] |
| | Ubuntu global quick-add hotkey + tray + `--quick-add` CLI | MVP | GlobalShortcuts portal (GNOME 48+, needs AppID) with a GNOME custom-shortcut fallback. [Verified: https://release.gnome.org/48/developers/index.html] |
| | Voice capture, on-device STT | v1 | `createOnDeviceSpeechRecognizer()` (API 31+). Whether it works fully offline is unverified. [Verified: https://developer.android.com/reference/android/speech/SpeechRecognizer.html] |
| | Brain dump → AI triage (preview diff) | v2 | Structured-output LLM behind a proxy. ~$0.001 per dump. [Inferred: §6.7] |
| | Task breakdown ("make it smaller") | v2 | Same plumbing as the brain dump |
| | Screenshot/OCR capture | v2 | ML Kit Text Recognition v2 on-device. [Verified: https://developers.google.com/ml-kit/vision/text-recognition/v2/android] |
| | Email-to-task forwarding address | later | Needs inbound mail infrastructure. **Never use Gmail API scopes** (they are restricted and trigger a CASA assessment). [Verified: https://developers.google.com/identity/protocols/oauth2/production-readiness/restricted-scope-verification] |
| **Organize** | Projects, sub-tasks, tags, priorities, fractional ordering | MVP | — |
| | Recurrence: RRULE + "repeat after completion" + edit this/future instance | MVP | Dart `rrule` 0.2.18 lacks WKST and is a partial port, so wrap it and test it. [Verified: https://pub.dev/packages/rrule] |
| | Saved filters / smart lists, search | v1 | — |
| | Attachments (lazy download, MinIO/S3) | v1 | — |
| | Shared lists / households | later | Needs accounts, permissions and invitations |
| **Remind** | Exact reminders that survive Doze, reboot, updates and timezone changes; reliability self-test | **MVP** | `SCHEDULE_EXACT_ALARM` user grant plus `setAlarmClock`. [Verified: §4.1] |
| | Desktop reminders that fire with the app closed | **MVP** | Tray daemon plus a systemd-user timer fallback. [Inferred: §6.5] |
| | Nag until done (escalating, varied) | v1 | Full-screen tier only with a user-granted `USE_FULL_SCREEN_INTENT`. [Verified: https://support.google.com/googleplay/android-developer/answer/13392821] |
| | Moments: car/headphones Bluetooth, driving start/stop, PC login/unlock | v1 | BT is exempt from broadcast limits. Activity Recognition needs Play Services. [Verified: broadcast-exceptions URL ; https://developer.android.com/develop/sensors-and-location/location/transitions] |
| | Location (geofence) reminders | v2 | 100 geofences per app, ~100 m radius, minutes of latency, background location plus a Play declaration. [Verified: https://developer.android.com/develop/sensors-and-location/location/geofencing] |
| | Adaptive snooze (personal on-device statistics) | v2 | Personal models beat generic ones (>60% precision after 9 days). [Verified-2nd: https://discovery.ucl.ac.uk/1502364/1/CHI16.pdf] |
| | Wear OS: notification bridging (free), Wear widgets | v1 / later | Wear widgets arrive with Wear OS 7. `glance-wear-tiles` is deprecated. [Verified-2nd: https://9to5google.com/2026/05/22/google-explains-new-wear-os-widgets/ ; Verified: https://developer.android.com/jetpack/androidx/releases/glance] |
| **Plan** | Today / Upcoming / calendar overlay | MVP (local) / v1 (with calendars) | — |
| | Duration estimates + estimate-vs-actual calibration | v2 | Planning-fallacy correction |
| | Morning plan + shutdown review; weekly review on "fresh start" landmarks | v2 | Fresh-start effect. [Verified-2nd: Dai, Milkman & Riis 2014, https://faculty.wharton.upenn.edu/wp-content/uploads/2014/06/Dai_Fresh_Start_2014_Mgmt_Sci.pdf] |
| | Focus timer as an Android 16 Live Update; tray countdown | v1 | Only a **user-started** timer qualifies. A persistent "next commitment countdown" is an explicitly inappropriate use. [Verified: https://developer.android.com/develop/ui/compose/notifications/live-update] |
| | Auto time-blocking + re-plan on slip (proposal-first; never moves meetings) | v2 | Greedy/constraint packer, no LLM. Focus Time events only on primary calendars, "only to some users". [Verified: https://developers.google.com/calendar/api/guides/calendar-status] |
| | "Energy" slots as a user preference, not as science | v2 | 2025 review: >80% of studies show no main chronotype effect. [Verified-2nd: https://researchinnovation.kingston.ac.uk/en/publications/chronotype-and-synchrony-effects-in-human-cognitive-performance-a/] |
| **Integrate** | Sync between devices (phone ↔ PC) | MVP | PowerSync OE on the user's Postgres 18 |
| | CalDAV two-way: Nextcloud (VEVENT + VTODO), iCloud (VEVENT only) | v1 | App-specific passwords. iCloud allows max 25, and all are revoked on an Apple ID password change. [Verified: https://support.apple.com/en-us/102654] |
| | Google Calendar two-way (events); Google Tasks as a date-only mirror at most | v1 | The Tasks API cannot carry a time of day. [Verified: https://developers.google.com/workspace/tasks/reference/rest/v1/tasks] |
| | Microsoft 365 / Outlook: events + **To Do tasks two-way** | v1 | `todoTask` has reminders and recurrence. [Verified: https://learn.microsoft.com/en-us/graph/api/resources/todotask?view=graph-rest-1.0] |
| | Read the phone's calendars via CalendarContract (no OAuth) | v1 | Android only. No tasks. [Verified: https://developer.android.com/guide/topics/providers/calendar-provider] |
| | Local MCP server (stdio) + CLI | MVP | Must target MCP **2026-07-28**. Write tools need consent UI. [Verified: https://modelcontextprotocol.io/specification/2026-07-28/changelog] |
| | Remote MCP (OAuth, Client ID Metadata Documents) | v2 | Dynamic Client Registration is deprecated. [Verified: same changelog] |
| | Android AppFunctions (Gemini) | later | Experimental preview; Gemini integration is in private preview. [Verified: https://developer.android.com/ai/appfunctions] |
| | Export (JSON / ICS), import from Todoist/TickTick | v1 | Portability is the answer to vendor exits. [Inferred: Clockwise shutdown] |
| **Reflect** | Completed log, streak-free "forgiving" routines (habit horizon ~66 days median) | v2 | [Verified-2nd: https://www.ucl.ac.uk/news/2009/aug/how-long-does-it-take-form-habit] |
| | Weekly/monthly review with estimate accuracy and "backlog bankruptcy" | v2 | — |
| **AI** | Deterministic parser → on-device LLM (where available) → cloud small model (opt-in) | v2 | Gemini Nano Prompt API is Beta, foreground-only, and structured output is **Alpha**. [Verified: https://developers.google.com/ml-kit/genai] |
| | Daily-plan proposal (mid-tier model, metered) | v2 | ~$0.36/user/month on Sonnet 5.5; ~$0.05 on Haiku 5.5. [Inferred: §6.7] |
| | RRULE synthesis with mandatory expansion preview | v2 | Never trust a generated RRULE without showing the next dates |
| | Optional offline desktop LLM (llama.cpp) | later | [Unverified: not researched in depth] |

---

## 3. Competitive landscape

| App | Android | Linux desktop | Price | Calendars | AI / agent | Grade |
|---|---|---|---|---|---|---|
| Todoist | Yes + Wear OS | Official app | Pro $60/yr | Google **or** Outlook, one account; events read-only; no filter | Ramble voice, Assist; **official MCP** | [Verified: https://www.todoist.com/pricing ; help article in competitors.md] |
| TickTick | Yes | Listed | Premium **$49.99/yr** | Calendar feed; Google two-way reported | AI assistant; MCP (date unverified) | [Verified: https://ticktick.com/about/upgrade] |
| Microsoft To Do | Yes | **No official app** | Free | Outlook | Copilot elsewhere | [Verified-2nd: https://itsfoss.com/kuro-to-do-app] |
| Google Tasks | Yes | No (web) | Free | Inside Google Calendar; **API drops the time of day** | Gemini creates tasks | [Verified: Tasks API reference] |
| Things 3 / Apple Reminders | No | No | — | iCloud | Apple Intelligence | [Verified-2nd: https://www.macrumors.com/2026/09/14/things-3-updated-for-ios-27/] |
| Sunsama | Companion app | Yes | $17–22/mo | Google, Outlook | AI + MCP | [Verified: https://www.sunsama.com/pricing] |
| Akiflow | Yes | No official (AUR) | $19–34/mo, no free tier | Google, Outlook | Hosted MCP | [Verified: https://akiflow.com/pricing] |
| Motion | Yes | Not confirmed | $19–29/seat/mo with credits | Google, Outlook | Auto-scheduling | [Verified: https://usemotion.com/pricing] |
| Morgen | Yes | **Official** | $15/mo yearly | **Google, M365, iCloud, Fastmail, CalDAV**. Nextcloud tested, but the server must be **publicly reachable** (cloud-side sync) | AI Planner | [Verified: https://www.morgen.so/pricing ; https://www.morgen.so/guides/caldavs-supported-by-v4] |
| Reclaim (Dropbox) | No | No | $10–22/seat | Google, Outlook; no iCloud | AI agents | [Verified: https://reclaim.ai/pricing] |
| Amie / Superlist / Structured / Tiimo | Second-class or rebuilt in 2026 | No | $5–21/mo | Partial; Tiimo and Structured lack Google sync on Android | Voice AI, MCP | [Verified: https://amie.so/changelog ; https://www.tiimoapp.com/resource-hub/tiimo-android-relaunch] |
| **Tasks.org** (OSS) | Strongest Android CalDAV app | **Desktop alpha** (KMP + Compose MP) | Free, GPLv3 | CalDAV, Google Tasks, EteSync | — | [Verified: https://tasks.org/ ; repository build files, per competitors.verify.md] |
| **Super Productivity** (OSS) | Yes (Capacitor) | Yes (Electron) | Free | Multi-account Google, CalDAV | **Local MCP** (v19.3.0) | [Verified: https://github.com/super-productivity/super-productivity/releases] |
| Planify / Errands (OSS, GNOME) | No | Native GNOME | Free | CalDAV / EDS | — | [Verified: https://github.com/alainm23/planify/releases ; https://github.com/mrvladus/Errands/releases] |
| Vikunja (OSS, self-hosted) | APKs | Web | Free | CalDAV | MCP (v2.7.0) | [Verified: https://github.com/go-vikunja/vikunja/releases] |

**The gap this app fills.** It is narrower than it first looks, which is a reason to be precise.

Coverage alone is no longer a gap: Super Productivity and Tasks.org already span Android and Linux, and MCP is everywhere. [Verified: competitors.verify.md]

What nobody combines is this [Inferred: from the table]:
1. **reminder reliability as an engineered, tested guarantee** on Android, including OEM guidance and a self-test (G5);
2. **native-feeling GNOME Wayland integration**: portal hotkey, tray, notifications that fire with the app closed, and a CLI (G1, G2, G12);
3. **true multi-provider two-way sync** with Google, Microsoft (with timed To Do tasks), CalDAV/Nextcloud **done on infrastructure the user controls**, so a home Nextcloud behind NAT works, unlike Morgen (G3);
4. **evidence-based cues and nagging** (Moments) instead of plain time reminders;
5. **local-first data and a private agent interface** at an individual price (G7, G8, G10).

---

## 4. Feasibility by integration surface

Legend: ✅ possible · ⚠️ possible with limits · ⛔ hard blocker.

### 4.1 Android (primary)

| Surface | Verdict | Limits / mechanism | Grade |
|---|---|---|---|
| Exact reminders | ✅⚠️ | `USE_EXACT_ALARM` is for alarm/timer apps and "calendar app that shows event notifications". **A to-do app is not a listed case**, so plan on `SCHEDULE_EXACT_ALARM`. It is denied by default on fresh installs targeting API 33+ on Android 14+ devices, kept on device upgrade, and denied after a backup/restore. **Revoking it kills the app process and cancels all exact alarms.** The state-change broadcast fires on **grant**, not on revoke. A `USE_EXACT_ALARM` declaration may be arguable once the app shows calendar events; that is a review risk, not a design basis. | [Verified: https://support.google.com/googleplay/android-developer/answer/16558241 ; https://developer.android.com/about/versions/14/changes/schedule-exact-alarms ; https://developer.android.com/develop/background-work/services/alarms/schedule] |
| Doze | ✅ | `setAlarmClock()` is never deferred, and the system leaves Doze before it fires. `*AllowWhileIdle` fires at most once per 9 minutes per app. | [Verified: https://developer.android.com/training/monitoring-device-state/doze-standby] |
| Standby buckets | ⚠️ | The Restricted bucket allows **one alarm per day**. **Whether `setAlarmClock` escapes that cap is undocumented**, so it becomes Spike S1. | [Verified: https://developer.android.com/topic/performance/power/power-details ; exemption: Unverified, no source says it] |
| Reboot / update / timezone | ✅ | Reschedule on `BOOT_COMPLETED` (only after first launch), `MY_PACKAGE_REPLACED`, `TIME_SET`, `TIMEZONE_CHANGED`, and the permission-grant broadcast. Android 15+: no `dataSync` foreground service from the boot receiver. | [Verified: https://developer.android.com/about/versions/15/behavior-changes-15 ; broadcast-exceptions URL] |
| OEM battery killers | ⚠️ (residual risk) | Huawei, Xiaomi, OnePlus and Samsung score worst. Mitigate with per-OEM deep links, a self-test and a late-fire log. The direct battery-optimisation exemption prompt is policy-restricted for non-listed app types; send the user to the settings list instead. | [Verified: https://dontkillmyapp.com/ ; doze-standby URL] |
| Full-screen "alarm" nag | ⚠️ opt-in | Not default-granted (since 2025-01-22). The user enables it through `ACTION_MANAGE_APP_USE_FULL_SCREEN_INTENT`, plus a Play declaration. | [Verified: https://source.android.com/docs/core/permissions/fsi-limits] |
| Notifications | ✅ | `POST_NOTIFICATIONS` runtime permission (Android 13+): ask in context. Action buttons and RemoteInput are standard. | [Verified: https://developer.android.com/develop/ui/views/notifications/notification-permission ; actions: Unverified, well-known API not fetched] |
| Home-screen widgets | ✅ | Glance 1.2.0 stable (2026-08-26): checkboxes and actions, **no text input** (a button opens quick-add). API 37 throws on over-budget bitmaps. | [Verified: https://developer.android.com/jetpack/androidx/releases/glance ; https://developer.android.com/about/versions/17/behavior-changes-17] |
| QS tile, shortcuts, share sheet | ✅ | See §2.5 | [Verified: as cited there] |
| Live Updates | ⚠️ | Only a user-started focus timer. Needs `POST_PROMOTED_NOTIFICATIONS`. Not for reminders. | [Verified: live-update URL] |
| Bluetooth / driving / geofence cues | ✅ / ✅ / ⚠️ | BT ACL is exempt; Activity Recognition needs Play Services; geofences: 100 per app, minutes of latency, background location. Play policy changes on location from 2027-01-27 (Android 17 location button). | [Verified: URLs in §2.5 ; https://support.google.com/googleplay/android-developer/answer/16558241] |
| Assistant (Gemini) integration | ⛔ for now | AppFunctions is an experimental preview. Design domain "intents" now; wire them later. | [Verified: https://developer.android.com/ai/appfunctions] |
| Sideload distribution | ⚠️ 2027 | Developer verification is enforced in BR/ID/SG/TH since 2026-09-30, for store installs only, and goes global in 2027. **ADB installs are exempt.** A free Limited Distribution account covers ≤20 devices with no ID. | [Verified: https://developer.android.com/developer-verification ; https://developer.android.com/developer-verification/guides/faq] |
| Play target API | — | API 36 is required for new apps and updates. API 37 is expected around Aug 2027. | [Verified: https://support.google.com/googleplay/android-developer/answer/11926878 ; the 37 date is Inferred from the policy-deadline table] |

### 4.2 Ubuntu 26.04 GNOME 50 Wayland (primary, the user's machine)

| Surface | Verdict | Limits / mechanism | Grade |
|---|---|---|---|
| Native notifications | ✅ | Notification portal **v1** on this machine (title, body, buttons, priority). v2 features (sounds, `alarm.ringing`) are not available here. | [Verified-local ; spec: https://flatpak.github.io/xdg-desktop-portal/docs/doc-org.freedesktop.portal.Notification.html] |
| Reminders with the app closed | ⚠️ (design work) | Linux has no OS scheduled-notification store, and `flutter_local_notifications` cannot schedule on Linux. Use a resident tray daemon plus a systemd user timer fallback. | [Verified: plugin page ; design Inferred] |
| Global quick-add | ✅⚠️ | GlobalShortcuts portal: the user approves the binding and may reassign it. **xdp ≥ 1.21.0 requires a valid AppID** (a `.desktop` file for unsandboxed builds). `ConfigureShortcuts` **is exposed** locally (whether the GNOME 50 backend implements it is unchecked). Fallback: a GNOME custom shortcut running `holdfast --quick-add`, which always works. | [Verified: https://github.com/flatpak/xdg-desktop-portal/blob/main/NEWS.md ; Verified-local: busctl introspect] |
| Tray | ⚠️ best-effort | Works only through the Ubuntu AppIndicators extension (active here). Open bug #628 on GNOME 50 drops some icons; the suspected cause is `IconPixmap` handling, so it **may not hit every SNI client**. | [Verified: https://github.com/ubuntu/gnome-shell-extension-appindicator/issues/628] |
| Autostart | ✅ | `~/.config/autostart/*.desktop` for a `.deb`. The Background portal also needs an AppID since xdp 1.21.0. | [Verified: NEWS.md ; XDG autostart spec: Unverified, not fetched] |
| Secrets (OAuth tokens) | ✅ | Secret portal / libsecret present | [Verified-local] |
| Packaging | ✅ | `.deb` first (personal use). Flatpak or Snap later. Flathub requires an offline build from source, so the Flutter SDK and pub packages must be vendored. | [Verified: https://docs.flathub.org/docs/for-app-authors/requirements ; Snap: Unverified] |
| X11 hotkey libraries | ⛔ | Ubuntu 26.04 is Wayland-only | [Verified-2nd: https://documentation.ubuntu.com/release-notes/26.04/summary-for-lts-users/] |

### 4.3 iOS (later)

| Surface | Verdict | Limits | Grade |
|---|---|---|---|
| Local reminders | ⚠️ | **64 pending notifications per app.** The app must cap and choose them itself (the "soonest kept" rule is unsupported by sources). Refill a rolling window on launch, background refresh and push. Use APNs for server-driven changes. | [Verified: https://developer.apple.com/forums/thread/811171 ; https://developer.apple.com/forums/thread/765490] |
| Must-not-miss alerts | ✅ (iOS 26+) | AlarmKit overrides Focus and Silent. A countdown needs a widget extension. Forum reports of misfires exist. | [Verified: https://developer.apple.com/documentation/alarmkit/scheduling-an-alarm-with-alarmkit.md ; Verified-2nd: https://developer.apple.com/forums/thread/846063] |
| Background timing | ⚠️ | `BGAppRefreshTask` runs at the system's discretion | [Verified-2nd: https://developer.apple.com/forums/thread/770479] |
| Widgets / Siri / Shortcuts / Focus | ✅ | App Intents is the single surface. Widgets are native SwiftUI/WidgetKit, buttons and toggles only. | [Verified: https://developer.apple.com/documentation/appintents.md ; https://developer.apple.com/documentation/widgetkit/adding-interactivity-to-widgets-and-live-activities.md] |
| iCloud Reminders | ✅ (EventKit, on iOS only) | Full access only | [Verified: https://developer.apple.com/documentation/eventkit/accessing-the-event-store.md] |
| On-device LLM | ✅ | Apple Foundation Models. The context size is disputed (4K vs 8K). | [Verified-2nd: WWDC26 session snippets] |

### 4.4 Windows / macOS (later)

| Platform | Verdict | Notes | Grade |
|---|---|---|---|
| Windows | ✅ | **Scheduled toasts** exist (the OS firing them with the app closed is inferred). Unpackaged apps need an AUMID plus an activator; MSIX avoids this. Hotkey and tray are standard. | [Verified: https://learn.microsoft.com/en-us/windows/apps/design/shell/tiles-and-notifications/send-local-toast-other-apps ; hotkey/tray: Unverified, recall] |
| macOS | ✅ | `UNUserNotificationCenter` needs a signed bundle. Menu-bar items may be hidden on macOS 26. App Intents and EventKit as on iOS. | [Verified-2nd: https://engineered.at/articles/mac-menu-bar-chaos ; Verified: App Intents URL] |

### 4.5 Calendar and task providers

| Provider | Events | Tasks | Change detection | Auth / compliance | Grade |
|---|---|---|---|---|---|
| **Google** | ✅ two-way (REST v3; write to an app-created calendar) | ⛔ timed tasks: **the Tasks API is date-only**, with no reminders or recurrence. ⛔ via CalDAV: Google CalDAV has no VTODO. | `events.watch` HTTPS webhook, which expires and is not guaranteed; always run `syncToken` incremental sync | OAuth. "Reading Calendar events" is Google's own example of a **sensitive** scope: verification takes up to 10 days, needs a verified domain, a privacy policy and a YouTube demo video, and **no CASA**. Unverified apps are capped at 100 users. Testing status gives 7-day refresh tokens. Whether `calendar.app.created` is classed sensitive is **unknown** (check the Cloud Console Data Access screen). | [Verified: https://developers.google.com/identity/protocols/oauth2/production-readiness/sensitive-scope-verification ; https://developers.google.com/identity/protocols/oauth2 ; https://support.google.com/cloud/answer/7454865 ; https://developers.google.com/workspace/calendar/caldav/v2/guide] |
| **Microsoft 365 / Outlook** | ✅ two-way (Graph, delta) | ✅ **two-way To Do** (`reminderDateTime`, `recurrence`, checklist). The id changes when a task moves between lists. | Subscriptions: events <7 days, todoTask <3 days; HTTPS webhook + renewal job | Personal MSA works without verification. Corporate tenants may block unverified multitenant apps; publisher verification is free but needs a Partner Program account and an Entra work tenant. `baseTask` is deprecated. | [Verified: todotask URL ; https://learn.microsoft.com/en-us/graph/api/resources/subscription?view=graph-rest-1.0 ; https://learn.microsoft.com/en-us/entra/identity-platform/publisher-verification-overview] |
| **iCloud** | ✅ two-way via CalDAV | ⛔ off-Apple (upgraded Reminders are not on CalDAV) | Polling (ctag/etag) | App-specific password (2FA, max 25) | [Verified: https://support.apple.com/en-us/102654 ; Verified-2nd: BusyMac FAQ] |
| **Nextcloud / CalDAV** | ✅ | ✅ VTODO with DUE, RRULE, VALARM | sync-token polling (RFC 6578) | App password; plain passwords fail with 2FA | [Verified: https://docs.nextcloud.com/server/latest/user_manual/en/groupware/sync_android.html ; https://manual.davx5.com/tasks_notes.html] |

### 4.6 Notifications summary

- Android: local alarms are the only reliable source of firing. FCM is free, and v2 `RemoteNotification` wakes the device in Doze; use it only as a "wake and sync" nudge. [Verified: https://firebase.google.com/pricing ; doze-standby URL]
- UnifiedPush 6.2.0 covers de-Googled phones. [Verified: https://pub.dev/packages/unifiedpush]
- Desktop: portal or freedesktop notifications fired by the daemon or the systemd helper.

**Conclusion [Inferred: from §4.1–4.6]: there is no hard blocker for the personal MVP.** The ⛔ items (Google timed tasks, iCloud Reminders off-Apple, Gemini AppFunctions) are routed around by design: the app's own store is the source of truth, and external services are adapters.

---

## 5. Framework decision

### 5.1 Matrix (1–5; verifier corrections applied)

Scores are [Speculative] judgements grounded in the facts graded in framework.md and framework.verify.md. Corrections applied:
- KMP Linux goes from 2 to 3: ComposeNativeTray provides a D-Bus SNI tray (released July 2026, maturity unverified). [Verified-2nd: https://mvnrepository.com/artifact/dev.nucleusframework?p=2]
- RN widgets go from 2 to 3: Expo SDK 58 added an Android `expo-widgets` implementation, which is fresh and has a release-build crash bug. [Verified: https://expo.dev/changelog/sdk-58 ; https://github.com/expo/expo/issues/51337]

| Criterion (weight for *this* user) | Flutter | Flutter + in-repo Kotlin module (**recommended**) | KMP + CMP | RN / Expo | Tauri 2 |
|---|---|---|---|---|---|
| Android deep integration (×3) | 4 | 4.5 | 5 | 4 | 2 |
| Linux GNOME Wayland desktop (×3) | 4 | 4 | 3 | 1 | 3 |
| iOS later, no rewrite (×1) | 4 | 4 | 4 | 5 | 3 |
| Widgets (×2) | 3 | 4 | 5 | 3 | 1 |
| Background / alarm reliability (×3) | 4 | 5 | 5 | 3 | 1 |
| Local DB + sync on all targets (×2) | 5 | 5 | 4 | 4 | 3 |
| Dev speed, solo + Claude Code (×2) | 5 | 5 | 4 | 4 | 3 |
| Ecosystem health (×1) | 4 | 4 | 4 | 4 | 3 |
| Performance (×1) | 4 | 4 | 4 | 4 | 3 |
| Toolchain fit (×1) | 5 | 5 | 4 | 3 | 2 |
| **Unweighted / 50** | **42** | **44.5** | **42** | **35** | **24** |
| **Weighted / 95** | **79** | **85.5** | **81** | **62** | **43** |

**Fairness check:** the hybrid column credits Flutter for in-repo Kotlin on Android. A symmetric "KMP + in-repo Linux D-Bus module" (tray, notifications and portal client via dbus-java) would lift KMP's Linux score to about 3.5 and its weighted total to about 82.5. That is still behind the Flutter hybrid at 85.5, but by 3 points, not 4.5. [Speculative]

Why each row scores as it does:
- **Flutter Linux 4**: there is an official GTK embedder, and Impeller has been the default since 3.47 [Verified: https://flutter.dev/blog/whats-new-in-flutter-3-47]. `tray_manager` 0.8.2 is SNI over D-Bus and `hotkey_manager` 0.3.0 speaks the native portal [Verified: https://pub.dev/packages/tray_manager ; https://pub.dev/packages/hotkey_manager]. Canonical is reported as steward of Flutter desktop [Verified-2nd only: https://www.omgubuntu.co.uk/2026/05/flutter-desktop-canonical-maintained ; no primary Google post found]. Points lost: no Linux scheduling in the notifications plugin; multi-window is experimental.
- **KMP Linux 3**: Compose desktop tray and notifications sit on AWT, and "notification is lost" without a tray [Verified: https://kotlinlang.org/docs/multiplatform/compose-desktop-tray.html]. There is no GlobalShortcuts portal client and no native Wayland, so it runs through XWayland. The JVM is a resident cost.
- **Sync on Linux**: PowerSync lists **Flutter Linux (glibc 2.34+)** explicitly, while for Kotlin only "Linux JVM" is listed and native Linux is "maybe soon" [Verified: https://docs.powersync.com/resources/supported-platforms].
- **RN Linux 1**: there is no official Linux desktop [Verified-2nd: https://platform.uno/articles/best-cross-platform-frameworks-2026/].
- **Tauri 1–2**: there are no official widget, alarm or background plugins [Verified: https://v2.tauri.app/plugin/]. Wayland global shortcuts remain unconfirmed [Unverified: only 2021–2023 issues found].
- **Toolchain**: `/stack` pins Flutter **3.47.9** (the master `.env`), a local 3.41.9 image, FVM 4.3.2, Kotlin 2.4.21, Gradle 9.8.2 and Java 17/21/27 [Verified-local: grep of /stack/.env, per framework.verify.md]. 3.47.9 is the newest stable patch [Verified: https://raw.githubusercontent.com/flutter/flutter/stable/CHANGELOG.md].

### 5.2 Recommendation

**Flutter 3.47.9, with an in-repo Kotlin Android module**, upgrading to 3.50 when it lands (3.50 is in beta and due in November) [Verified: https://github.com/flutter/flutter/issues/195428]. Start on the standalone `material_ui` package from day one, because the in-SDK Material/Cupertino libraries are scheduled for formal deprecation in 3.50 [Verified: 3.47 blog]. The user's 3.41.9 image is too old for this matrix (AGP 9.1, KGP 2.4.0, minSdk 24, targetSdk 36), so use the 3.47.9 pin. [Inferred: from the 3.47 dependency matrix]

### 5.3 Runner-up: KMP + Compose Multiplatform

Choose it instead if the user decides that **Android depth, Wear and widgets are the product, and the Linux desktop is secondary**. On an unweighted score it **ties** pure Flutter (42 vs 42). On the weighted score it slightly **beats** pure Flutter (81 vs 79). It is also real-world proven for exactly this app class: **Tasks.org is KMP + CMP with Android, Linux desktop and iOS targets.** [Verified: Tasks.org build files, per competitors.verify.md]

### 5.4 "Why Flutter", honestly, since the user leans Flutter

The lean is justified, but **not by a wide margin**. Four reasons decide it:
1. **Linux desktop is a day-one platform, and Flutter's Linux integration exists today.** The portal hotkey and SNI tray plugins exist, a Linux sync SDK exists, and Canonical ships its own Ubuntu apps in Flutter [Verified-2nd]. With KMP the user would write a D-Bus tray, notification and portal client in Kotlin himself.
2. **The pure-Flutter weakness on Android disappears with the hybrid.** Widgets are native Kotlin and Glance in *every* framework [Inferred: home_widget states widgets must be native code]. The alarm engine is best written as Kotlin `setAlarmClock` anyway. The recommended hybrid therefore gets most of KMP's Android advantage (weighted 85.5 vs 81).
3. **One language (Dart) for UI, domain, DB codegen (drift), the sync SDK and even the API server (`shelf`)**, plus hot reload. That suits a solo developer working with an AI agent. [Speculative]
4. **Toolchain** is already pinned in the user's Docker stack. [Verified-local]

**Why not Flutter, the honest risks:**
- Several key plugins have one maintainer (`home_widget`, `receive_sharing_intent`). [Verified: pub.dev pages]
- `system_tray` is abandoned. [Verified: https://pub.dev/packages/system_tray]
- Linux has no scheduled notifications.
- Multi-window is experimental, so the quick-add popup needs a workaround.
- 3.50 brings Material split churn.
- Impeller on Linux needs Vulkan, and the Skia fallback will be removed.
- The `leanflutter` desktop plugins were republished within a day of this research (fresh 0.x releases).

**Optional tie-breaker:** if the user wants the decision to be data-driven, run a 2-day KMP Linux spike (ComposeNativeTray + dbus-java portal hotkey + a scheduled notification) next to the Flutter spike S2 (§9). [Speculative]

### 5.5 Framework risks and mitigations

| Risk | Likelihood / impact | Mitigation |
|---|---|---|
| No Linux scheduling in the notifications plugin | Certain / high | Own scheduler (§6.5) [Verified: plugin page] |
| Single-maintainer `home_widget` | Medium / medium | Glance UI in-repo Kotlin; the bridge is a ~100-line MethodChannel; fork if it is abandoned [Inferred] |
| Flutter 3.50 Material deprecation | Certain / low | Start on `material_ui`; run `dart fix --code=migrate_design_widgets` [Verified: 3.47 blog] |
| Impeller/Vulkan regressions on Linux | Low-medium / medium | Test on the user's GPU early. Do not rely on the Skia opt-out long-term. [Verified: opt-out slated for removal] |
| Fresh 0.x desktop plugin regressions | Medium / low | Pin exact versions; Renovate + CI [Inferred] |
| Stale AI-generated code adds EOL packages (`sqlite3_flutter_libs`, `powersync_flutter_libs`) | High / low | SQLite now loads via `package:sqlite3` 3.x build hooks. Add a CI lint banning those packages. [Verified: https://pub.dev/packages/powersync_flutter_libs] |
| `flutter_local_notifications` 23.0 major bump (in dev) | Certain / low | Pin 22.3.3; migrate deliberately [Verified: pub.dev] |

---

## 6. Architecture

### 6.1 Layers

1. **UI (Flutter, `material_ui`)**: screens (Today, Upcoming, Projects, Plan, Review), quick-add sheet, settings, onboarding (permissions, OEM guide, reliability self-test).
2. **Domain (pure Dart)**: entities, a recurrence expander (wraps `rrule`), the NL parser, the planner/scheduler algorithm, conflict rules, and the **intent API** (`createTask`, `complete`, `snooze`, `queryToday`, `planDay`). The intent API is the single surface used by the UI, the MCP server, notification actions, and later Siri/AppFunctions. [Inferred: os-integration §7.6]
3. **Local DB**: SQLite via drift, managed by the PowerSync SDK (the client is plain SQLite). [Verified: https://pub.dev/packages/drift ; https://pub.dev/packages/powersync]
4. **Sync**: the PowerSync client streams down. An upload queue pushes field-level patches to the App API. [Verified: sync-backend §3]
5. **Platform shells**:
   - *Android Kotlin module*: Glance widgets, AlarmManager reminder engine, boot/time/permission receivers, notification action receiver, BT/Activity cue receivers, QS tile, share target.
   - *Linux shell*: tray daemon, GlobalShortcuts client, resident scheduler, systemd-user helper, `holdfast` CLI (`--quick-add`, `--fire-due`, `mcp`), notifications.
   - *Later*: iOS Swift extensions (WidgetKit, AlarmKit, App Intents); Windows toasts; macOS.
6. **Backend (Docker, the user's stack)**: Postgres 18, PowerSync Open Edition service, Keycloak (OIDC), App API (Dart `shelf`), calendar-bridge worker, AI proxy, push sender (FCM/UnifiedPush), MinIO for attachments.

### 6.2 Component list (textual, for the page author to draw)

```
DEVICES
 [Android app]
   Flutter UI ── Domain/intent API ── drift/SQLite (PowerSync SDK) ── upload queue
   Kotlin module: ReminderEngine(AlarmManager.setAlarmClock) · Boot/Time/Perm receivers
                  NotificationActionReceiver · Glance widgets · QS tile · ShareTarget
                  CueReceivers(BT ACL, ActivityRecognition, later Geofence)
   FCM / UnifiedPush receiver ──> trigger sync ──> ReminderEngine.rescheduleAll()
 [Ubuntu app]
   Flutter UI (hidden window) + tray_manager (SNI) + hotkey_manager (GlobalShortcuts portal)
   Resident scheduler (timers re-armed from DB; resume-from-suspend hook)
   systemd --user timer -> `holdfast --fire-due` (headless) -> freedesktop notification
   Local MCP server (stdio, `holdfast mcp`) over the same SQLite + intent API
 [later] iOS (WidgetKit/AlarmKit/App Intents/APNs) · Windows (scheduled toasts) · macOS

NETWORK  JWT (OIDC+PKCE from Keycloak) · PowerSync stream · uploadData patches · HTTPS webhooks

BACKEND (Docker)
 Keycloak ──JWKS──> PowerSync service (OE, >=1.27.2) <── logical replication ── Postgres 18
 App API (shelf): validate patches, permissions, conflict rules, reminder fan-out, push
 Calendar bridge worker: Google events.watch + syncToken · Graph subscriptions + delta
                         CalDAV polling (iCloud, Nextcloud) · renewal cron · sync_conflicts
 AI proxy: provider keys, per-user quotas, schema validation, no content logging
 Push sender: FCM v2 / UnifiedPush (ntfy)        MinIO: attachment blobs
 Reachability for the phone off-LAN: Tailscale / Cloudflare Tunnel / small VPS
```

[Inferred: adapted from sync-backend §11, with the platform shells from os-integration §7 added]

### 6.3 Data model

Postgres is the source of truth and is mirrored to on-device SQLite. All rules below are [Inferred: from sync-backend §5 and the RFC 5545 mappings] unless graded otherwise.

- **IDs**: UUIDv7 minted on the device. Postgres 18 has `uuidv7()` server-side. [Verified-2nd: https://www.dbvis.com/thetable/uuidv7-in-postgresql-18-what-you-need-to-know/ ; whether a Dart v7 package exists: Unverified]
- **Common columns**: `owner_id, created_at, updated_at (server), deleted_at (tombstone), version (server-bumped), origin_device_id`.
- **Entities**:
  - `users(id, display_name, default_tz, locale)`
  - `projects(id, owner_id, name, color, parent_id, sort_key, e2ee BOOL, archived_at)`
  - `tasks(id, project_id, parent_task_id, title, notes, status, priority, sort_key /*fractional index*/, due_kind{none,date,datetime}, due_date, due_local, due_tz, start_local, start_tz, duration_est_min, rrule, rrule_dtstart_local, rrule_tz, recurrence_mode{fixed,after_completion}, completed_at)`
  - `task_occurrence_overrides(task_id, recurrence_id_local, status{done,skipped,moved}, moved_to_local, completed_at)`. These map to RECURRENCE-ID and EXDATE, so completing one instance never mutates the series.
  - `moments(id, task_id, kind{time,place_enter,place_exit,bt_connect,bt_disconnect,driving_start,driving_stop,pc_login,pc_unlock,workday_start}, params JSONB)`. These are the if-then cues.
  - `reminders(id, task_id, kind{absolute,relative,moment}, fire_at_utc, offset_minutes, anchor{due,start}, escalation_profile, snoozed_until_utc, dismissed_at)`. These map 1:1 to VALARM.
  - `reminder_deliveries(reminder_id, occurrence_local, device_id, fired_at, late_by_s, acted)`. This gives cross-device dedupe and the **late-fire log** behind the reliability self-test.
  - `time_entries(task_id, started_at, ended_at)`, used for estimate-vs-actual calibration.
  - `tags`, `task_tags`, `attachments(blob_key, mime, size, sha256)`.
  - `calendar_accounts` (server-only, never synced; tokens encrypted).
  - `calendar_links(task_id, account_id, remote_calendar_id, remote_uid, remote_etag, last_pushed_version)`.
  - `sync_conflicts`, `devices(platform, push_kind, push_token_enc)`.
- **RRULE**: store the RFC 5545 string, which is portable to CalDAV, Google and Graph. Expand on the device over a rolling ~60-day window. Wrap the Dart `rrule` package, which lacks WKST and is a partial port [Verified: https://pub.dev/packages/rrule], behind an interface, with a conformance corpus taken from the rrule.js / dateutil test vectors. "After completion" recurrence is not expressible in RRULE, so it gets its own mode and only the next instance is exported.
- **Timezones**: absolute instants are stored as UTC `timestamptz`. Wall-clock intents ("09:00 Europe/Paris every day") are stored as **local datetime + IANA TZID**, never as UTC, so DST and travel are handled correctly. Floating tasks ("pills at 08:00 wherever I am") are an explicit flag. Ship tzdata with the app. Test across DST transitions in Europe/Paris, America/New_York and Australia/Lord_Howe (a 30-minute DST shift).

### 6.4 Sync design

- **Engine**: PowerSync Open Edition (self-hosted) with the user's Postgres 18 as the source.
  - Postgres needs `wal_level=logical`, a publication named `powersync`, and a role with `REPLICATION BYPASSRLS`. Bucket storage can live in the same server (PG ≥ 14). [Verified: https://docs.powersync.com/configuration/source-db/setup]
  - **Corrections from the verifier:**
    - Postgres bucket storage is pinned to **legacy storage v2**. Storage v4, GA since Service 1.27.0 (2026-10-12), is MongoDB-only. Every sync-config change therefore triggers a **full reprocess and a client re-sync**. That is fine personally; at product scale, use MongoDB storage or PowerSync Cloud. [Verified: https://docs.powersync.com/sync/advanced/storage-version-4]
    - Run Service **≥ 1.27.2**, which carries security fixes. [Verified: https://releases.powersync.com/]
    - The licence is **FSL-1.1-ALv2**: source-available, with a non-compete clause, converting to Apache-2.0 later. The client SDKs are Apache-2.0. [Verified: GitHub licence API, per sync-backend.verify.md]
  - Set `max_slot_wal_keep_size` so that a stopped PowerSync container cannot fill the disk with retained WAL. [Inferred: Postgres logical-replication behaviour]
- **Writes**: the client uploads **field-level patches** with a base `version` to the App API. The server applies last-writer-wins per field and bumps `version`. Domain rules:
  - completion beats a concurrent edit (both are applied);
  - delete beats edits;
  - recurring completion writes an override row;
  - tree moves are validated for cycles;
  - reorders use fractional indexes.

  [Inferred: sync-backend §6]
- **Auth**: Keycloak OIDC with PKCE (passkeys officially supported since 26.4). PowerSync verifies JWTs via `jwks_uri`. Use `offline_access` refresh tokens so a phone that has been offline for a long time can resume. Keycloak must be reachable off-LAN. [Verified: https://www.keycloak.org/2025/09/keycloak-2640-released ; offline-token point Inferred]
- **Exit ramp**: the data is always in your Postgres, and the client is plain SQLite. If PowerSync changes, the fallbacks are sqlite-sync (CRDT, Dart and Kotlin, Apache-2.0, 1.2.0) or a hand-rolled outbox with a cursor-pull API (~2–4 weeks). [Verified: https://pub.dev/packages/sqlite_sync ; estimate Speculative]
- **Disqualified engines**:
  - Electric: no write path, no Dart client [Verified: https://electric.ax/docs/guides/writes].
  - Firestore: no Linux [Verified: https://pub.dev/packages/cloud_firestore].
  - InstantDB: sunsets 2027-08-31 [Verified: https://www.instantdb.com/docs].
  - Zero/Replicache: no Dart [Verified: https://zero.rocicorp.dev/docs/status].

### 6.5 Notification / reminder engine (the product core)

**Principle: a reminder must fire with zero network.** Push only wakes a device so it syncs and re-arms. [Inferred: push can be dropped by Doze or OEMs]

**Android (Kotlin `ReminderEngine`, owned in the repo)**
1. **Source of truth**: the `reminders` table plus the expanded occurrences. The engine is **idempotent**: `rescheduleAll()` cancels and re-arms the next window (e.g. 48 h or ≤ 50 alarms). Reminders due in the same minute are coalesced. The window stays far below the reported **Samsung 500-alarm cap**. [Verified: cap "reportedly", https://pub.dev/packages/flutter_local_notifications]
2. **API**: `setAlarmClock()` for every user-facing reminder. It is never deferred, and the system leaves Doze for it. [Verified: alarms page] If `canScheduleExactAlarms()` is false, fall back to `setWindow()` (≥ 10 min windows) and show a persistent "reminders may be late, tap to fix" banner. [Verified: https://developer.android.com/develop/background-work/services/alarms]
3. **Triggers for `rescheduleAll()`**:
   - `BOOT_COMPLETED` (and `LOCKED_BOOT_COMPLETED` only if the DB is in device-protected storage; [Inferred]);
   - `MY_PACKAGE_REPLACED`, `TIME_SET`, `TIMEZONE_CHANGED`;
   - `ACTION_SCHEDULE_EXACT_ALARM_PERMISSION_STATE_CHANGED` (sent on **grant**);
   - **every app start**, because a revoke kills the process and sends no broadcast;
   - after every sync that touches reminders.

   [Verified: broadcast-exceptions ; alarms/schedule]
4. **Firing**: an alarm receiver posts the notification on a dedicated channel, with the actions Done / Snooze 10 min / Tomorrow / Open and an inline reply. A `NotificationActionReceiver` applies the action **without starting the Flutter engine**. Design caveat: PowerSync tracks local writes through its own SQLite views and triggers. Writing the DB from Kotlin therefore needs either the PowerSync extension loaded in the Kotlin connection, or a small `pending_actions` table that Dart drains on the next start. **Decide this in Spike S1.** [Inferred: PowerSync's CRUD-queue design; Unverified: not tested]
5. **Escalation (nag until done)**: on no action, the engine re-arms at an increasing cadence (e.g. 5, 10, 20 minutes), switching channel, sound and vibration pattern between steps (varied, to resist habituation). The final tier is a full-screen intent, only if the user granted `USE_FULL_SCREEN_INTENT`. [Verified: FSI policy ; cadence Speculative]
6. **Sound under Android 17**: play reminder sound through the notification channel. Background audio from a service needs a foreground service, with the while-in-use waiver only for exact-alarm holders using `USAGE_ALARM`. [Verified: https://developer.android.com/about/versions/17/behavior-changes-all]
7. **Reliability tooling**:
   - onboarding requests `POST_NOTIFICATIONS`, then `SCHEDULE_EXACT_ALARM`, then the full-screen intent (optional);
   - per-OEM deep links (dontkillmyapp);
   - a **self-test** that schedules an alarm 2 minutes ahead and records `late_by_s`;
   - a delivery-log screen;
   - a fix-it banner when the permission is missing.

   **Spike S1 must test the Restricted standby bucket** with `adb shell am set-standby-bucket <pkg> restricted`, because whether `setAlarmClock` escapes the one-alarm-per-day cap is undocumented. [Verified: power-details page ; exemption Unverified]
8. **Never use WorkManager for timing.** Use it only for sync, which is subject to Android 16 quotas. [Verified: https://developer.android.com/about/versions/16/behavior-changes-all]

**Ubuntu (the app is closed or open)**
1. **Resident daemon**: the Flutter app autostarts hidden, with a tray icon. It keeps in-process timers re-armed from the DB at start, after each sync, and **on resume from suspend** (via the logind `PrepareForSleep` D-Bus signal), because timers drift across suspend. [Inferred: logind D-Bus is standard; Unverified: not fetched]
2. **Fallback when the GUI is quit**: on every change, the daemon writes `~/.config/systemd/user/holdfast-next.timer` with `OnCalendar=<next due>` and `Persistent=true`. The timer runs `holdfast --fire-due`, a headless Dart CLI build that reads SQLite, posts a freedesktop notification and arms the next timer. A late-fire after a power-off is reported as "missed at HH:MM". [Inferred design; systemd `Persistent=` and `WakeSystem=` semantics Unverified: not fetched]
3. **Display**: portal or freedesktop notifications. Action buttons (Done/Snooze) are available at portal v1. Whether `flutter_local_notifications` exposes Linux actions is **unverified**, so the fallback is a direct D-Bus call. [Verified-local: portal v1 ; plugin actions Unverified]
4. **Dedupe with the phone**: a user setting decides "notify on phone only / PC only when active / all devices". `reminder_deliveries` prevents a double "done". [Inferred]

**iOS later**: a rolling self-capped window of ≤ 64 notifications, refilled on launch, background refresh and APNs. AlarmKit is opt-in for the "must not miss" tier. [Verified: §4.3]

### 6.6 Calendar sync design

All rules below are [Inferred: from sync-backend §10 and os-integration §6] unless graded otherwise.

- **Where it runs**: a **server-side calendar bridge** (a Dart worker in Docker). Google and Graph webhooks need a public HTTPS endpoint with a CA-signed certificate. Renewal jobs need an always-on scheduler (Graph events < 7 days, To Do < 3 days). OAuth refresh tokens live in one hardened place and never reach devices. [Verified: https://developers.google.com/calendar/api/guides/push ; Graph subscription URL]
  - Because the bridge runs on the user's own server, a **home Nextcloud behind NAT works**. Morgen requires public reachability, so this is a differentiator. [Verified: Morgen CalDAV guide]
- **Mapping**:

| App object | Google | Microsoft | Nextcloud / CalDAV | iCloud |
|---|---|---|---|---|
| Task (dated or timed, with reminders) | Event in a dedicated app-created calendar (timed tasks only); Google Tasks as an optional date-only mirror | **To Do `todoTask`** (two-way: due, reminder, recurrence, checklist) | **VTODO** (DUE, RRULE, VALARM) | Not possible (Reminders not on CalDAV) |
| Time-blocked task | VEVENT-equivalent event + reminders | Outlook event | VEVENT + VALARM | VEVENT |
| External calendar events | Read for Today/Plan; two-way edits for events the app created | same | same | same |

- **Change detection**: Google `syncToken` incremental sync, plus `events.watch` as an accelerator only, since delivery is not guaranteed. Graph delta queries plus subscriptions with lifecycle notifications, renewed at 50% of lifetime. CalDAV uses `sync-collection` and ctag/etag polling every N minutes. A full re-sync runs on `410 Gone` or token invalidation.
- **Conflicts**: compare `remote_etag` and `last_pushed_version`. App-owned fields win for tasks. For events that originated in the calendar, the remote time wins. Everything is logged to `sync_conflicts` and shown in the UI. Lossy RRULE edits must never be pushed: a failed round-trip test means "detach the instance", not "rewrite the series".
- **Personal phase without a public server**: CalDAV first (no OAuth verification needed), and Google/Graph via polling. Webhooks come later behind a tunnel. Android `CalendarContract` can read every phone-synced calendar with no OAuth, as a Today-view fallback.
- **Google compliance**: personal use (< 100 users) is exempt from verification. **Testing status gives 7-day refresh tokens.** Whether "In production, unverified" avoids that is **undocumented**, so put a 7-day token test in the Phase 1 exit criteria. Check each scope's class in the Cloud Console. [Verified: https://support.google.com/cloud/answer/13464323 ; https://developers.google.com/identity/protocols/oauth2]

### 6.7 AI design

- **Three-layer parsing** [Inferred: risks-ai-cost §1.4]:
  1. A deterministic FR + EN grammar in Dart, with golden tests. It costs nothing, works offline, and handles ~70% of captures (assumed).
  2. An on-device LLM where available: Gemini Nano via ML Kit GenAI is Beta, **foreground-only**, device-listed, and its structured output is Alpha [Verified: https://developers.google.com/ml-kit/genai]. Apple Foundation Models come later.
  3. A cloud small model, opt-in, **through the AI proxy** (never ship a key in a public repo or APK). Every result appears as a preview diff before saving, and generated RRULEs are expanded to their next 5 dates.
- **Model choice (pin current IDs)**:
  - Default: **Claude Haiku 5.5**, $0.10 / $0.50 per Mtok (≤ 100K prompt).
  - Daily planning, metered: **Sonnet 5.5**, $2 / $10.
  - Fallback: gpt-6-luna ($0.10 / $0.50).

  [Verified: https://platform.claude.com/docs/en/about-claude/pricing ; https://developers.openai.com/api/docs/pricing]
- **Do not build on**:
  - Gemini 2.5 Flash-Lite: closed to new users [Verified: https://ai.google.dev/gemini-api/docs/deprecations].
  - Gemini 3.1 Flash-Lite: shuts down 2027-05-07 [Verified: same].
  - MediaPipe LLM Inference: maintenance-only, so use LiteRT-LM [Verified: https://developers.google.com/edge/mediapipe/solutions/genai/llm_inference].
  - Gemini 3.8 Flash: its price **doubles on 2027-01-01** [Verified: https://ai.google.dev/gemini-api/docs/pricing].
- **Cost per active user per month**:
  - Typical profile (10 captures/day, 2 dumps/week, 1 plan/day): Haiku-only ≈ **$0.053**; recommended mix (Sonnet for planning) ≈ **$0.40**; deterministic-first ≈ $0.37 for the mix and ≈ $0.032 Haiku-only.
  - Add **~30%** for the Claude tokenizer (4.7+ models produce ~30% more tokens).
  - A heavy user (3×) on the mix is ≈ $1.2. The innovation dive's heavier capture-only profile gives ≈ $0.30.

  **Conclusion: cents per user per month on Haiku, under $1.60 even for heavy planners.** [Inferred: arithmetic, recomputed by risks-ai-cost.verify.md; tokenizer note Verified on the pricing page]
- **EU rule**: the free Gemini tier may not serve EEA users, and that includes the developer himself in France. [Verified: https://ai.google.dev/gemini-api/terms]
- Anthropic does not train on API content, a DPA is included, and data is deleted within 30 days (ZDR by agreement). [Verified: https://www.anthropic.com/legal/commercial-terms ; https://privacy.claude.com/en/articles/7996866-how-long-do-you-store-my-organization-s-data]
- OpenAI's API data terms are **unverified**.
- **Personal phase**: BYOK in libsecret / Android Keystore is acceptable. [Inferred]

### 6.8 Security and privacy

- **Local-first**: the device works fully offline. The server holds a copy, or ciphertext.
- **E2EE option, per project**: encrypt `title`, `notes` and attachments client-side with libsodium XChaCha20-Poly1305 (`sodium` 4.1.2+1 supports Android and Linux) [Verified: https://pub.dev/packages/sodium].
  - The per-user key is wrapped per device with X25519. Enrolment uses a QR code from an existing device, or a recovery passphrase with Argon2id.
  - **E2EE projects cannot use** the calendar bridge, server AI, server search, or push text containing content. The UI says this plainly.
  - Timestamps stay plaintext so the server can still nudge.

  [Inferred: sync-backend §7]
- **Local DB encryption**: PowerSync v2 provides `EncryptionOptions` with SQLite3MultipleCiphers, replacing SQLCipher. [Verified-2nd: https://releases.powersync.com/announcements/ann_P6AmNl0W84xt8 ; exact API not fetched]
- **Secrets**:
  - Calendar OAuth tokens are server-only and encrypted at rest.
  - Device secrets use Android Keystore / libsecret. [Verified-local: Secret portal]
  - CI runs secret scanning.
  - No provider keys go in binaries.
- **MCP safety**: the local stdio MCP exposes write tools. Require confirmation for destructive tools (delete, bulk-complete), keep an audit log, and offer a read-only mode. This guards against prompt-injected agent actions. [Inferred: the spec's tool-safety guidance, per innovation.verify.md]
- **GDPR (France/EU)**: tasks, calendar entries and voice notes are personal data, and possibly special-category data (health appointments). Requirements: a lawful basis (contract), minimisation, export (Art. 20), erasure (Art. 17), a privacy policy, processor DPAs (Anthropic: yes), records of processing, and AI **opt-in per feature** with disclosure. Transfers rely on DPF/SCCs or EU-region endpoints (OpenAI regional endpoints cost +10%). [Unverified: GDPR articles from recall, not fetched; OpenAI uplift Verified on the pricing page]
- **Retention proposal**: server ciphertext is kept until account deletion + 30 days; AI requests are not logged by the proxy; crash logs are scrubbed and kept 90 days. [Speculative]

---

## 7. Tech stack

| Component | Choice | Version (as verified) | Why | Alternative | Grade |
|---|---|---|---|---|---|
| UI framework | Flutter | **3.47.9** stable; 3.50 in beta (Nov) | §5 | KMP + CMP 1.12.2 | [Verified: Flutter CHANGELOG ; https://kotlinlang.org/docs/multiplatform/compose-compatibility-and-versioning.html] |
| Design widgets | `material_ui` | 1.0 | Avoids the 3.50 deprecation migration | in-SDK Material (deprecated) | [Verified: 3.47 blog] |
| Android toolchain | AGP 9.1.0 / KGP 2.4.0 / Gradle 9.3.1 / Java 17; compile & target SDK 36; minSdk 24 | per the 3.47 matrix | Required by Flutter 3.47; Play requires target 36 | — | [Verified: 3.47 blog ; Play target page] |
| Local DB | drift | 2.35.3 | Typed SQL, all 6 targets incl. Linux | sqflite (no Linux) [Unverified] | [Verified: https://pub.dev/packages/drift] |
| SQLite loading | `package:sqlite3` 3.x build hooks | — | `sqlite3_flutter_libs` / `powersync_flutter_libs` are EOL | — | [Verified: https://pub.dev/packages/powersync_flutter_libs] |
| Sync SDK | PowerSync Dart | 2.5.0 (2026-10-19) | Flutter Linux supported; server-authoritative writes | sqlite_sync 1.2.0 | [Verified: https://pub.dev/packages/powersync] |
| Sync service | PowerSync Open Edition (`journeyapps/powersync-service`) | ≥ 1.27.2 | Free self-host on the user's Postgres; FSL licence | PowerSync Cloud Pro ($49/mo) | [Verified: https://docs.powersync.com/self-hosting/getting-started ; releases page] |
| Server DB | Postgres 18 (existing container) | 18.x | Already running; logical replication | Supabase (managed) | [Verified-local: /stack pin per sync-backend] |
| Auth | Keycloak | 26.4+ (26.7.0 latest seen) | Already running; OIDC, passkeys | Supabase Auth (passkeys beta) | [Verified: https://www.keycloak.org/2026/07/keycloak-2670-released] |
| App API + calendar bridge + AI proxy | Dart `shelf` services | [Unverified: version not checked] | One language end to end | Kotlin Ktor | [Inferred] |
| Notifications (Android/iOS/desktop display) | flutter_local_notifications | **22.3.3** (pin; 23.0 in dev) | Permission helpers, channels; **no Linux scheduling** | Own Kotlin/Swift code | [Verified: pub.dev] |
| Android alarms | Own Kotlin `ReminderEngine` (AlarmManager `setAlarmClock`) | Android API 36 target | Full control over reschedule, escalation, self-test | android_alarm_manager_plus 5.1.2 (Dart callbacks) | [Verified: https://pub.dev/packages/android_alarm_manager_plus ; design Inferred] |
| Widgets | Jetpack Glance (Kotlin, in repo) + `home_widget` as the data bridge | Glance 1.2.0; home_widget 0.10.0 | Widgets are native anyway | MethodChannel bridge | [Verified: Glance releases ; https://pub.dev/packages/home_widget] |
| Background sync | workmanager | 0.10.10 | Periodic sync only (never timing) | Native WorkManager | [Verified: https://pub.dev/packages/workmanager] |
| Share target | receive_sharing_intent | 1.9.0 | Android/iOS share | Native intent filter in Kotlin | [Verified: https://pub.dev/packages/receive_sharing_intent] |
| Launcher shortcuts | quick_actions (first-party) | 1.1.1 | — | ShortcutManagerCompat | [Verified: https://pub.dev/packages/quick_actions] |
| Tray (Linux) | tray_manager | 0.8.2 (pin) | SNI over D-Bus, no appindicator lib needed | ComposeNativeTray (KMP) ; **not** system_tray (abandoned) | [Verified: https://pub.dev/packages/tray_manager] |
| Global hotkey (Linux) | hotkey_manager | 0.3.0 (pin) | Native GlobalShortcuts portal; needs a matching `.desktop` | GNOME custom shortcut + CLI | [Verified: https://pub.dev/packages/hotkey_manager] |
| Linux scheduling fallback | systemd user timer + `holdfast --fire-due` | systemd as shipped by Ubuntu 26.04 [Unverified: version] | Fires with the GUI closed | XDG autostart daemon only | [Inferred] |
| Recurrence | `rrule` (wrapped) + conformance corpus | 0.2.18 | RFC 5545; lacks WKST, partial port | Own Dart expander | [Verified: https://pub.dev/packages/rrule] |
| Timezones | `timezone` package (bundled tzdata) | [Unverified: not fetched] | DST-correct expansion | OS tzdata | [Unverified] |
| NL parser | Own Dart grammar FR + EN (chrono-inspired) | — | chrono_dart is English-only and stale | chrono (JS) as a reference grammar | [Verified: https://pub.dev/packages/chrono_dart ; https://github.com/wanasit/chrono] |
| Crypto (E2EE) | `sodium` (libsodium) | 4.1.2+1 | XChaCha20-Poly1305, X25519, Argon2id | cryptography (Dart) [Unverified] | [Verified: https://pub.dev/packages/sodium] |
| Push | FCM (free) + UnifiedPush 6.2.0 (+ ntfy) | — | Wake-for-sync only | none (poll) | [Verified: https://firebase.google.com/pricing ; https://pub.dev/packages/unifiedpush] |
| AI models | Claude Haiku 5.5 default; Sonnet 5.5 for planning; gpt-6-luna as alternative | pricing 2026-11-01 | Cheapest capable; DPA; no training | Gemini 3.5 Flash-Lite ($0.30 / $2.50) | [Verified: pricing pages] |
| On-device AI | ML Kit GenAI (Gemini Nano), opportunistic | Prompt API Beta | Free and private on supported phones | LiteRT-LM | [Verified: https://developers.google.com/ml-kit/genai] |
| OCR | ML Kit Text Recognition v2 | — | On-device | Tesseract (Linux) | [Verified: ML Kit URL] |
| MCP | MCP SDK that speaks spec **2026-07-28** (stateless, `server/discover`) | [Unverified: which Dart SDK supports 2026-07-28] | Agent interface | TypeScript MCP sidecar over the CLI | [Verified: https://modelcontextprotocol.io/specification/latest] |
| Attachments | MinIO (S3-compatible) | [Unverified] | Self-host | Cloudflare R2 | [Unverified] |
| CI | GitHub Actions (public repo) | — | Linux + Android builds; macOS runners later for iOS | GitLab CI | [Unverified: free-minute terms not fetched this session] |
| Packaging | `.deb` → Flatpak/Snap; APK via ADB/Limited Distribution → Play | — | §4 | AppImage (poor portal fit) | [Verified: Flathub requirements ; developer-verification FAQ] |
| Toolchain host | User's Docker stack (Flutter 3.47.9 image, Android SDK, FVM 4.3.2) | per `/stack/.env` | Already provisioned | Host install | [Verified-local] |

---

## 8. Effort estimate

**Assumptions.** All numbers in this section are [Speculative].
- One developer with Claude Code writing most of the boilerplate.
- "Focused hours" means hands-on time, including tests and review.
- Ranges are optimistic / **likely** / pessimistic.
- The per-module figures come from the research dives' developer-week estimates. The innovation and sync dives estimated in developer-weeks, converted here at ~35 focused hours per week, then adjusted for overlap.
- Calendar weeks are computed at **17.5 h/week (side project, midpoint of 15–20)** and **35 h/week (full-time)**.
- Rule 14-style debugging of OEM reminder issues is the main source of the pessimistic tail.

### 8.1 Per module

| Phase | Module | Opt h | **Likely h** | Pess h |
|---|---|---|---|---|
| 0 | Repo, CI, Docker dev env, 5 spikes (§9) | 40 | **60** | 90 |
| 1 | Core domain + drift schema + main UI (Today/Upcoming/Projects/Tags/Search) | 70 | **100** | 150 |
| 1 | NL quick-add parser FR + EN + golden corpus | 45 | **70** | 105 |
| 1 | Recurrence engine wrapper + conformance tests | 30 | **45** | 70 |
| 1 | Android ReminderEngine (permissions, setAlarmClock, reschedule-all, actions, self-test, OEM guide) | 55 | **80** | 130 |
| 1 | Android capture surfaces (Glance widget, QS tile, share target, shortcuts) | 40 | **60** | 100 |
| 1 | Linux shell (tray, portal hotkey, resident scheduler, systemd helper, CLI, notifications, `.deb`) | 45 | **70** | 120 |
| 1 | Sync backend (PG logical, PowerSync OE, Keycloak realm, App API patches + conflict rules + tests) + client integration | 45 | **65** | 110 |
| 1 | Local MCP server (2026-07-28) + consent rules | 15 | **25** | 40 |
| 1 | Onboarding, settings, export, hardening, CI coverage | 15 | **25** | 45 |
| | **Phase 1 subtotal** | **360** | **540** | **870** |
| 2 | CalDAV bridge (Nextcloud VTODO + VEVENT, iCloud VEVENT) | 60 | **85** | 120 |
| 2 | Google Calendar + Graph (events + To Do), OAuth, webhooks, renewal, conflicts UI | 70 | **100** | 150 |
| 2 | Nag-until-done escalation + full-screen tier | 30 | **45** | 70 |
| 2 | Moments v1: Bluetooth, driving, PC login/unlock | 40 | **60** | 100 |
| 2 | Push wake (FCM + UnifiedPush), multi-device dedupe | 20 | **30** | 50 |
| 2 | Polish: focus timer + Live Update, voice capture, attachments, import from Todoist/TickTick, widget variants | 60 | **90** | 130 |
| | **Phase 2 subtotal** | **280** | **410** | **620** |
| 3 | AI proxy + brain dump + breakdown + RRULE synthesis | 45 | **65** | 100 |
| 3 | Duration calibration + morning plan + shutdown/weekly review | 60 | **90** | 130 |
| 3 | Auto time-blocking scheduler + re-plan on slip | 80 | **115** | 170 |
| 3 | Adaptive snooze statistics | 35 | **55** | 80 |
| 3 | Geofence Moments | 40 | **60** | 90 |
| 3 | Remote MCP (OAuth, CIMD) | 40 | **60** | 90 |
| | **Phase 3 subtotal** | **300** | **445** | **660** |
| 4 | Multi-user accounts, sharing, per-project E2EE | 130 | **190** | 280 |
| 4 | iOS port (WidgetKit, AlarmKit, 64-slot window, APNs, App Intents) | 120 | **170** | 260 |
| 4 | Windows + macOS shells | 70 | **110** | 170 |
| 4 | Store and compliance (Play closed test, Google OAuth verification, MS publisher verification, privacy policy, Flathub/Snap) | 50 | **80** | 130 |
| 4 | Billing / monetisation, hosted ops, observability | 70 | **110** | 170 |
| | **Phase 4 subtotal** | **440** | **660** | **1,010** |
| | **Grand total** | **1,420** | **2,115** | **3,250** |

### 8.2 Calendar time

| Milestone | Likely h (cumulative) | Side project @17.5 h/wk | Full-time @35 h/wk |
|---|---|---|---|
| Phase 0 done (spikes) | 60 | **~3.5 weeks** (opt 2 / pess 6) | **~2 weeks** |
| First useful build (Android + Ubuntu, local, reminders, no sync), ≈ 60 + 120 of Phase 1 | ~180 | ~10 weeks | ~5 weeks |
| **Phase 1: personal MVP** (+540) | 600 | **~34 weeks ≈ 8 months** from start (Phase 1 alone ≈ 31 wk; opt 21 / pess 58 wk) | **~17 weeks ≈ 4 months** (Phase 1 alone ≈ 15.5 wk; opt 10 / pess 25 wk) |
| **Phase 2: daily driver + calendars** (+410) | 1,010 | ~58 weeks ≈ 13 months | ~29 weeks ≈ 7 months |
| **Phase 3: AI & context** (+445) | 1,455 | ~83 weeks ≈ 19–20 months | ~42 weeks ≈ 10 months |
| **Phase 4: product-ready** (+660) | 2,115 | ~121 weeks ≈ 2.3 years | ~60 weeks ≈ 14 months |

The arithmetic is cumulative likely hours ÷ hours per week; optimistic and pessimistic weeks use the module ranges (e.g. Phase 1 alone is 360/17.5 ≈ 21 weeks and 870/15 ≈ 58 weeks). The single biggest lever is **scope discipline in Phase 1**. Moving the Linux systemd helper, the MCP server and export to Phase 2 would cut ~60–80 h. [Speculative]

---

## 9. Roadmap

### Phase 0: setup and de-risking spikes (≈ 60 h; 3–4 weeks side / 2 weeks full-time)

**Goals:** prove the five riskiest assumptions before writing product code.
**Deliverables:** a public repo, CI (Android APK + Linux `.deb` builds, unit tests, a lint banning EOL packages), a dev compose stack (Postgres logical, PowerSync OE, Keycloak), and a spike report.

| # | Spike | Pass criterion | De-risks |
|---|---|---|---|
| **S1** | **Android exact alarm + Glance widget from Flutter.** Kotlin `setAlarmClock` engine, boot/permission reschedule, notification action handled without the Flutter engine (and how it writes through PowerSync), Glance list widget with a checkbox | Fires within 5 s across: reboot; app swipe-away; Doze (`adb shell dumpsys deviceidle force-idle`); **Restricted bucket (`adb shell am set-standby-bucket <pkg> restricted`)**; permission revoke then re-grant; timezone change. Widget toggle reflects in the app. | §4.1, R1, R2 |
| **S2** | **GNOME 50 Wayland desktop shell.** `hotkey_manager` portal binding from an **unsandboxed `.deb`** with a matching `.desktop` AppID; `tray_manager` icon vs bug #628; autostart; resident timer plus systemd-user `--fire-due` firing with the GUI quit; notification action buttons | Hotkey opens quick-add in < 300 ms; tray visible after a re-login; a reminder fires with the app quit and after suspend/resume | §4.2, R4, R5 |
| **S3** | **PowerSync on Flutter Linux + Android.** Build via `package:sqlite3` hooks on a GitHub Actions Linux runner; Postgres 18 storage; field-patch upload; offline edit on both devices then merge; `max_slot_wal_keep_size` set | Two devices converge after 24 h offline; CI build green | §6.4, R7 |
| **S4** | **Google OAuth personal mode.** Consent screen in "In production, unverified" with the calendar scopes; read each scope's class in the Cloud Console Data Access screen; refresh-token survival test | Refresh still works on **day 8** (or the documented re-login path is accepted); scope classes recorded | §6.6, R9 |
| **S5** | **Recurrence + timezone correctness.** Wrap `rrule`; run an RFC 5545 corpus (dateutil / rrule.js vectors) including WKST, BYSETPOS, the 31st and Feb 29; DST tests for Paris, New York and Lord Howe | ≥ 95% of the corpus passes; the gaps are listed with fork-or-implement decisions | R8 |

**Exit criteria:** all five spikes pass, or a documented fallback is accepted (e.g. S1 failing on Restricted means documenting the "keep the app active" mitigation). Framework confirmed, or switched to KMP if S1 or S2 reveal a blocker. [Speculative]

### Phase 1: personal MVP on Android + Ubuntu (≈ 540 h; ~31 weeks side / ~15.5 weeks full-time)

**Goals:** the user stops using any other task app.
**Deliverables:**
- tasks, projects, tags, sub-tasks;
- FR + EN quick-add;
- recurrence (fixed and after-completion);
- exact reminders with actions and a self-test;
- Glance widget, QS tile, share target, shortcuts;
- Ubuntu tray, hotkey, resident and systemd reminders, CLI;
- phone ↔ PC sync through the self-hosted backend;
- local MCP server;
- JSON/ICS export.

**Exit criteria:**
- 30 consecutive days of daily use;
- **zero missed reminders** in the delivery log, and a p95 lateness < 60 s on the user's phone;
- conflict-free sync across 2 devices with offline periods;
- Claude Code can create and complete tasks through MCP;
- the reliability self-test is green.

### Phase 2: daily-driver polish + calendar sync (≈ 410 h; ~23 weeks side / ~12 weeks full-time)

**Goals:** calendars and escalation make it "complete".
**Deliverables:**
- CalDAV (Nextcloud tasks + events, iCloud events);
- Google Calendar and Microsoft (events + To Do) two-way through the bridge, with webhooks behind a tunnel;
- a conflicts screen;
- nag-until-done with a full-screen tier;
- Moments v1 (Bluetooth car/headphones, driving, PC login/unlock);
- FCM/UnifiedPush wake;
- focus timer Live Update;
- voice capture;
- attachments;
- Todoist/TickTick import.

**Exit criteria:**
- 2 weeks with every connected calendar round-tripping without a lossy edit;
- Graph and Google subscriptions auto-renewing for 30 days;
- at least 3 Moment types used daily.

### Phase 3: AI & context (≈ 445 h; ~25 weeks side / ~13 weeks full-time)

**Goals:** the "Honest Day" and the smart features.
**Deliverables:**
- AI proxy with quotas;
- brain dump and breakdown with preview diffs;
- duration calibration;
- morning, shutdown and weekly reviews;
- auto time-blocking with re-plan;
- adaptive snooze with the chronic-snooze prompt;
- geofences;
- remote MCP.

**Exit criteria:**
- the planner's proposed day is accepted with ≤ 2 edits on most days;
- AI cost stays below $1/month for the user;
- estimate-error tracking shows calibration converging.

### Phase 4: product readiness (≈ 660 h; ~38 weeks side / ~19 weeks full-time)

**Goals:** a public launch without a rewrite.
**Deliverables:**
- multi-user accounts on hosted PowerSync Cloud, or self-host with MongoDB storage;
- sharing;
- per-project E2EE;
- iOS (Swift widget, AlarmKit and App Intents extensions);
- Windows and macOS;
- Google sensitive-scope verification (domain, privacy policy, demo video; up to 10 days);
- Microsoft publisher verification (Partner Program + Entra tenant);
- a Play closed test (**12 testers for 14 days**) and a production release;
- Apple Developer Program ($99/yr);
- Flathub or Snap;
- billing, status page, backups.

**Exit criteria:** passes store reviews; 100+ beta users with a crash-free rate above 99.5%; the self-test reports reliability across the top OEMs. [Verified: https://support.google.com/googleplay/android-developer/answer/14151465 ; https://developer.apple.com/programs/ ; sensitive-scope page]

---

## 10. Risks register

| # | Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|---|
| R1 | Missed or late Android reminders (OEM killers, Restricted bucket) | High | **Critical** | Kotlin `setAlarmClock` engine, reschedule-on-everything, self-test + late-fire log, OEM deep links; Spike S1 tests the Restricted bucket [Verified: §4.1 mechanisms; exemption Unverified] |
| R2 | `SCHEDULE_EXACT_ALARM` denied or revoked (a revoke kills the process, no broadcast) | High | High | Onboarding, reschedule on every start, inexact fallback + banner [Verified: alarms/schedule] |
| R3 | Play rejects a `USE_EXACT_ALARM` or full-screen declaration | Medium | Medium | Ship on `SCHEDULE_EXACT_ALARM`; full-screen as an opt-in user grant [Verified: Play policy] |
| R4 | Wayland hotkey refused (AppID, portal) | Medium | Medium | `.desktop` AppID; `PlatformException` → GNOME custom shortcut + CLI [Verified: NEWS.md ; hotkey_manager docs] |
| R5 | GNOME tray icons dropped (bug #628) | Medium | Low | The tray is not the primary entry point; test in S2 [Verified: issue #628] |
| R6 | Linux reminders missed when the app is quit or after suspend | Medium | High | systemd-user helper + resume hook [Inferred] |
| R7 | PowerSync licence, pricing or storage-format changes (FSL; Postgres storage = legacy v2) | Medium | Medium | Postgres-centric data; sqlite-sync or hand-rolled outbox as exit ramps; Cloud or MongoDB storage at scale [Verified: licence + storage v4 docs] |
| R8 | Recurrence/timezone bugs corrupt calendar series | Medium | High | Conformance corpus, DST tests, never push a lossy RRULE edit [Verified: rrule limits] |
| R9 | Google 7-day refresh tokens / verification friction | High (personal) | Medium | S4 test; CalDAV first; verification budgeted in Phase 4 [Verified: OAuth docs] |
| R10 | Webhook renewal failures silently stop calendar sync | Medium | Medium | Renewal at 50% of lifetime, lifecycle notifications, polling fallback, health dashboard [Verified: Graph lifetimes] |
| R11 | Home server unreachable from the phone | Medium | Low | Offline-first; Tailscale/Cloudflare Tunnel; small VPS option [Inferred] |
| R12 | Single-maintainer Flutter plugins abandoned | Medium | Medium | Thin wrappers, ready to fork; native code for the critical paths [Verified: pub.dev] |
| R13 | AI key leak from the public repo or APK | Medium | High | Proxy + per-user auth + quotas; secret scanning [Inferred] |
| R14 | GDPR non-compliance with cloud AI | Low (personal) / Medium (product) | High | Opt-in per feature, Anthropic DPA, no free Gemini tier, E2EE projects excluded from AI [Verified: terms] |
| R15 | Android developer verification hits sideloading in 2027 | Certain (2027) | Low (ADB exempt) | Limited Distribution account (free, ≤ 20 devices) now; Full account at launch [Verified: FAQ] |
| R16 | Scope creep and side-project fatigue (2+ years to product) | High | High | Hard Phase 1 exit criteria; ship the first useful build at ~180 h; defer Phase 4 until Phase 2 is used daily [Speculative] |
| R17 | Competitors close the gap (Tasks.org desktop leaving alpha; Super Productivity) | Medium | Medium | Lean on reliability, Moments and GNOME depth; open source as a community moat [Verified: competitor status] |
| R18 | iOS 64-notification cap and background limits | Certain (later) | Medium | Rolling window, AlarmKit, APNs [Verified: Apple forums] |
| R19 | Stale AI-generated code (EOL packages, old MCP handshake) | High | Low | CI lint rules; pin the MCP 2026-07-28 SDK [Verified: EOL notices ; MCP changelog] |

---

## 11. Costs

| Item | Personal (1 user) | 1k users | 10k users | Grade |
|---|---|---|---|---|
| Google Play developer account | $25 one-time (only needed for Play; ADB/Limited Distribution is free) | $25 | $25 | [Verified: developer-verification FAQ] |
| Apple Developer Program | $0 (no iOS yet) | $99/yr | $99/yr | [Verified: https://developer.apple.com/programs/] |
| Flathub / Snap | $0 | $0 | $0 | [Verified: Flathub mentions no fee; Snap Unverified] |
| Sync infra | $0 (self-hosted on the user's Docker); + optional tunnel ($0 Tailscale/Cloudflare free tiers) or a small VPS (~$5–10/mo) | ~150 peak concurrent clients exceeds PowerSync Free (50). Cloud Pro $49 + Supabase Pro $25 ≈ **$75–100/mo**, or a self-host VPS ≈ $20–40 | PowerSync Pro + overage ($30 per 1,000 extra concurrent) + Supabase Pro + compute ≈ **$105–150/mo + compute add-on**; self-host ≈ $20–80 + ops time | [Verified: https://www.powersync.com/pricing ; https://supabase.com/pricing ; VPS/tunnel prices Unverified] |
| Calendar bridge, AI proxy, API hosting | $0 (same box) | included above / +$10–20 | +$20–50 | [Unverified: VPS prices not fetched] |
| Push (FCM) | $0 | $0 | $0 | [Verified: Firebase pricing] |
| AI (Haiku default; Sonnet planning opt-in; +30% tokenizer) | **< $1/mo** | Haiku-only ≈ $70; mix ≈ $520 (1k × $0.40 × 1.3) | Haiku-only ≈ $700; mix ≈ $5,200. **Meter planning or make it a paid tier.** | [Inferred: §6.7 per-user figures × users × 1.3] |
| Google OAuth sensitive verification | $0 (personal exempt) | $0 fee (time: up to 10 days + video) | $0 | [Verified: sensitive-scope page] |
| CASA assessment | $0 (avoided: no Gmail or other restricted scopes) | $0 | $0 | [Verified: restricted-scope page] |
| MS publisher verification | $0 | $0 (needs Partner Program + Entra tenant) | $0 | [Verified: publisher-verification overview] |
| Domain + privacy-policy site | optional ~€10–15/yr | ~€15/yr | ~€15/yr | [Unverified: registrar prices not fetched] |
| CI (GitHub Actions, public repo) | $0 expected | $0 expected; macOS runners for iOS builds | same | [Unverified: free-tier terms not fetched] |
| **Total run-rate** | **≈ $0–10/mo + $25 once** | **≈ $150–650/mo**, mostly AI depending on the planning mix | **≈ $850–5,500/mo**, AI-dominated if Sonnet planning is unmetered | [Inferred: sum of rows] |

**Monetisation implications.** Since 2026-06-30, Google Play fees in the US, UK and **EEA** are **10% + 5% billing fee on subscriptions** and **20–25% + 5% on one-time purchases** in the first $1M. [Verified: https://support.google.com/googleplay/android-developer/answer/112622] That penalises one-time unlocks more than subscriptions.

Recommended model [Speculative]:
- **free and open source for local use**;
- an **optional "Sync & AI" subscription at about €3–4/month or €30/year**, below TickTick's $49.99/yr and Todoist's $60/yr [Verified: their pricing pages];
- AI planning metered inside the subscription;
- BYOK allowed for self-hosters.

At ~$0.5–1.3 cost per active paying user per month, the margin is healthy.

---

## 12. Open decisions for the user

Every recommendation below is [Speculative] product judgement unless graded otherwise. The facts behind each are graded in the section cited.

| # | Decision | Recommendation | Why it matters |
|---|---|---|---|
| D1 | **Framework**: Flutter + Kotlin module, or KMP + CMP | Flutter (+ optional 2-day KMP Linux spike as a tie-breaker) | §5: close call; Linux depth decides |
| D2 | Flutter version: move from the 3.41.9 image to the 3.47.9 pin, then 3.50 | Yes | The 3.47 matrix is required; avoids the Material migration |
| D3 | Sync hosting in Phase 1: self-host on the home stack + tunnel, or a small VPS | Self-host + Tailscale | $0; the phone stays usable offline anyway |
| D4 | First calendar provider: CalDAV/Nextcloud, Google or Microsoft | **Whichever the user lives in.** CalDAV is technically easiest; Google needs the OAuth spike | Phase 2 ordering |
| D5 | Google Tasks: ignore, or mirror date-only | Ignore (use calendar events for timed items) | The API cannot carry times |
| D6 | Distribution in Phase 1: ADB / Limited Distribution APK vs a Play internal track | ADB + Limited Distribution | Avoids the Play policy review until Phase 4 |
| D7 | `USE_EXACT_ALARM` declaration attempt at launch | No; `SCHEDULE_EXACT_ALARM` only | Policy risk |
| D8 | AI provider and BYOK vs proxy in the personal phase | Anthropic Haiku 5.5 via BYOK now; proxy at Phase 3 | Cost and privacy |
| D9 | E2EE in v1, or later per project | Later (Phase 4), schema ready now | It conflicts with the calendar bridge and AI |
| D10 | Product name | Holdfast (trademark check pending) | Branding, domain, AppID (`.desktop`, package name) must be fixed early because the portal binds to the AppID |
| D11 | Licence of the public repo (GPL/AGPL vs MIT/Apache) | AGPL-3.0 for the app + server if a hosted product is planned [Speculative] | Protects against hosted forks; interacts with the PowerSync FSL non-compete only for the sync service itself |
| D12 | Monetisation model | Free/OSS + "Sync & AI" subscription | Play's EEA fee structure favours subscriptions |
| D13 | Packaging for Linux: `.deb` only, or Flatpak from the start | `.deb` for personal use; Flatpak in Phase 4 | The portal identity differs (sandboxed AppID) |
| D14 | Wear OS priority | Notification bridging only until Wear OS 7 widgets | Effort vs value |

---

## 13. Sources (deduplicated)

**Android and Google Play**
- https://developer.android.com/about/versions/14/changes/schedule-exact-alarms
- https://developer.android.com/develop/background-work/services/alarms
- https://developer.android.com/develop/background-work/services/alarms/schedule
- https://developer.android.com/training/monitoring-device-state/doze-standby
- https://developer.android.com/topic/performance/power/power-details
- https://developer.android.com/develop/background-work/background-tasks/broadcasts/broadcast-exceptions
- https://developer.android.com/about/versions/15/behavior-changes-15
- https://developer.android.com/about/versions/16/behavior-changes-all
- https://developer.android.com/about/versions/16/behavior-changes-16
- https://developer.android.com/about/versions/16/features/progress-centric-notifications
- https://developer.android.com/about/versions/17
- https://developer.android.com/about/versions/17/behavior-changes-all
- https://developer.android.com/about/versions/17/behavior-changes-17
- https://developer.android.com/about/versions/17/release-notes
- https://developer.android.com/develop/ui/compose/notifications/live-update
- https://developer.android.com/develop/ui/views/notifications/live-update
- https://developer.android.com/develop/ui/views/notifications/notification-permission
- https://developer.android.com/jetpack/androidx/releases/glance
- https://developer.android.com/develop/ui/views/appwidgets/advanced
- https://developer.android.com/develop/ui/views/quicksettings-tiles
- https://developer.android.com/develop/ui/views/launch/shortcuts
- https://developer.android.com/training/sharing/receive
- https://developer.android.com/ai/appfunctions
- https://android-developers.googleblog.com/2026/05/android-ai-intelligence-system.html
- https://developer.android.com/ai/gemini-nano
- https://developers.google.com/ml-kit/genai
- https://developers.google.com/ml-kit/genai/prompt/android
- https://developers.google.com/ml-kit/vision/text-recognition/v2/android
- https://developer.android.com/develop/sensors-and-location/location/geofencing
- https://developer.android.com/develop/sensors-and-location/location/transitions
- https://developer.android.com/guide/topics/providers/calendar-provider
- https://developer.android.com/reference/android/speech/SpeechRecognizer.html
- https://developer.android.com/sdk/api_diff/31/changes/android.speech.SpeechRecognizer
- https://developer.android.com/training/wearables/tiles
- https://developer.android.com/training/wearables/widgets
- https://developer.android.com/developer-verification
- https://developer.android.com/developer-verification/guides/faq
- https://android-developers.googleblog.com/2026/06/android-developer-verification.html
- https://developer.android.com/jetpack/androidx/releases/room
- https://developer.android.com/kotlin/multiplatform/room
- https://source.android.com/docs/core/permissions/fsi-limits
- https://support.google.com/googleplay/android-developer/answer/16558241
- https://support.google.com/googleplay/android-developer/answer/13986130
- https://support.google.com/googleplay/android-developer/answer/13392821
- https://support.google.com/googleplay/android-developer/answer/11926878
- https://support.google.com/googleplay/android-developer/answer/12253906
- https://support.google.com/googleplay/android-developer/answer/14151465
- https://support.google.com/googleplay/android-developer/answer/6112435
- https://support.google.com/googleplay/android-developer/answer/112622
- https://dontkillmyapp.com/
- https://manual.davx5.com/tasks_notes.html
- https://www.davx5.com/tested-with/icloud
- https://9to5google.com/2026/05/22/google-explains-new-wear-os-widgets/
- https://github.com/juev/lists/pull/261

**Linux / GNOME / Flatpak / Ubuntu**
- https://release.gnome.org/48/developers/index.html
- https://github.com/flatpak/xdg-desktop-portal/blob/main/NEWS.md
- https://flatpak.github.io/xdg-desktop-portal/docs/doc-org.freedesktop.portal.Notification.html
- https://flatpak.github.io/xdg-desktop-portal/docs/doc-org.freedesktop.portal.GlobalShortcuts.html
- https://flatpak.github.io/xdg-desktop-portal/docs/doc-org.freedesktop.portal.Background.html
- https://github.com/ubuntu/gnome-shell-extension-appindicator/issues/628
- https://docs.flatpak.org/zh-cn/latest/desktop-integration.html
- https://discourse.flathub.org/t/permissions-for-showing-tray-icon/10785
- https://docs.flathub.org/docs/for-app-authors/requirements
- https://documentation.ubuntu.com/release-notes/26.04/summary-for-lts-users/
- https://linuxiac.com/ubuntu-26-04-lts-resolute-raccoon-released/
- https://discourse.gnome.org/t/how-do-you-enable-disable-global-shortcuts-in-gnome-48/29119
- https://packages.debian.org/sid/amd64/xdg-desktop-portal-gnome
- https://issues.chromium.org/issues/404298968
- https://github.com/aaddrick/claude-desktop-debian/blob/main/docs/learnings/wayland-global-shortcuts-portal.md
- https://pypi.org/project/global-shortcut-portal/
- https://git.dorkbox.com/dorkbox/SystemTray/raw/commit/0b53d5388fee855bcb3f436e184f63efccee38fa/README.md

**Apple / Windows / macOS**
- https://developer.apple.com/forums/thread/811171
- https://developer.apple.com/forums/thread/765490
- https://developer.apple.com/forums/thread/770479
- https://developer.apple.com/forums/thread/846063
- https://developer.apple.com/documentation/alarmkit/scheduling-an-alarm-with-alarmkit.md
- https://developer.apple.com/documentation/appintents.md
- https://developer.apple.com/documentation/appintents/focus.md
- https://developer.apple.com/documentation/widgetkit/adding-interactivity-to-widgets-and-live-activities.md
- https://developer.apple.com/documentation/activitykit/displaying-live-data-with-live-activities.md
- https://developer.apple.com/documentation/eventkit/accessing-the-event-store.md
- https://developer.apple.com/videos/play/wwdc2026/8121/
- https://developer.apple.com/videos/play/wwdc2026/241/
- https://developer.apple.com/programs/
- https://support.apple.com/en-us/102654
- https://www.busymac.com/docs/faqs/112990-reminders-in-ios-13-and-macos-catalina-drops-support-for-caldav/
- https://www.2doapp.com/kb/print-763.html
- https://github.com/Lingnik/icloud-reminders-mcp
- https://learn.microsoft.com/en-us/windows/apps/design/shell/tiles-and-notifications/send-local-toast-other-apps
- https://docs.rs/crate/mac-usernotifications/latest/source/src/lib.rs
- https://desktop-notifier.readthedocs.io/en/latest/_modules/desktop_notifier/macos.html
- https://engineered.at/articles/mac-menu-bar-chaos

**Calendars, OAuth, providers**
- https://developers.google.com/workspace/tasks/reference/rest/v1/tasks
- https://developers.google.com/workspace/tasks/auth
- https://developers.google.com/workspace/calendar/api/auth
- https://developers.google.com/workspace/calendar/caldav/v2/guide
- https://developers.google.com/calendar/api/guides/push
- https://developers.google.com/calendar/api/guides/calendar-status
- https://developers.google.com/identity/protocols/oauth2
- https://developers.google.com/identity/protocols/oauth2/production-readiness/sensitive-scope-verification
- https://developers.google.com/identity/protocols/oauth2/production-readiness/restricted-scope-verification
- https://support.google.com/cloud/answer/7454865
- https://support.google.com/cloud/answer/13464321
- https://support.google.com/cloud/answer/13464323
- https://support.google.com/cloud/answer/13464325
- https://learn.microsoft.com/en-us/graph/api/resources/todotask?view=graph-rest-1.0
- https://learn.microsoft.com/en-us/graph/api/resources/todo-overview?view=graph-rest-1.0
- https://learn.microsoft.com/en-us/graph/api/resources/subscription?view=graph-rest-1.0
- https://learn.microsoft.com/en-us/graph/outlook-calendar-concept-overview
- https://learn.microsoft.com/en-us/entra/identity-platform/publisher-verification-overview
- https://learn.microsoft.com/entra/identity-platform/application-consent-experience
- https://docs.nextcloud.com/server/latest/user_manual/en/groupware/sync_android.html
- https://cli.nylas.com/guides/icloud-caldav-settings
- https://www.onecal.io/blog/how-to-integrate-icloud-calendar-api-into-your-app
- https://developer.nylas.com/docs/provider-guides/google/google-verification-security-assessment-guide/
- https://deepstrike.io/blog/google-casa-security-assessment-2025
- https://aiemaily.com/blog/google-casa-verification-explained-for-buyers

**Frameworks and libraries**
- https://docs.flutter.dev/release/release-notes
- https://docs.flutter.dev/release/release-notes/release-notes-3.47.0
- https://flutter.dev/blog/whats-new-in-flutter-3-47
- https://raw.githubusercontent.com/flutter/flutter/stable/CHANGELOG.md
- https://github.com/flutter/flutter/issues/195428
- https://github.com/flutter/flutter/pull/193405
- https://github.com/flutter/flutter/issues/194698
- https://x.com/FlutterDev/status/2113740228686946668
- https://docs.shorebird.dev/getting-started/flutter-version.md
- https://www.omgubuntu.co.uk/2026/05/flutter-desktop-canonical-maintained
- https://www.thurrott.com/?p=336913
- https://www.bidev.dev/blog/flutter-2026-roadmap-everything-from-google-io
- https://medium.com/@AlexCodeX/flutter-3-44-release-notes-the-biggest-flutter-release-of-2026-20c82c7a6575
- https://pub.dev/packages/flutter_local_notifications
- https://pub.dev/packages/android_alarm_manager_plus
- https://pub.dev/packages/workmanager
- https://pub.dev/packages/home_widget
- https://pub.dev/packages/tray_manager
- https://pub.dev/packages/system_tray
- https://pub.dev/packages/hotkey_manager
- https://pub.dev/packages/quick_actions
- https://pub.dev/packages/receive_sharing_intent
- https://pub.dev/packages/drift
- https://pub.dev/packages/rrule
- https://pub.dev/packages/chrono_dart
- https://pub.dev/packages/sodium
- https://pub.dev/packages/unifiedpush
- https://pub.dev/packages/sqlcipher_flutter_libs
- https://pub.dev/packages/powersync_flutter_libs
- https://github.com/wanasit/chrono
- https://kotlinlang.org/docs/multiplatform/supported-platforms.html
- https://kotlinlang.org/docs/multiplatform/compose-compatibility-and-versioning.html
- https://kotlinlang.org/docs/multiplatform/compose-desktop-tray.html
- https://blog.jetbrains.com/kotlin/2026/05/compose-multiplatform-1-11-0/
- https://www.neowin.net/news/jetbrains-brings-compose-for-ios-to-stable-with-compose-multiplatform-180-release/
- https://slack-chats.kotlinlang.org/t/519872/what-s-the-current-status-about-notifications-on-linux-i-ve-
- https://slack-chats.kotlinlang.org/t/34112953/i-ve-been-wondering-for-a-long-time-why-does-compose-desktop
- https://mvnrepository.com/artifact/dev.nucleusframework?p=2
- https://scriptagc.wasmer.app/https_togithub_com/kdroidFilter/ComposeNativeTray
- https://expo.dev/changelog/sdk-55
- https://expo.dev/sdk/56
- https://expo.dev/changelog/sdk-57
- https://expo.dev/changelog/sdk-58
- https://github.com/expo/expo/issues/51337
- https://docs.expo.dev/guides/new-architecture/
- https://x.com/expo/status/2026811977990025364
- https://en.wikipedia.org/wiki/React_Native
- https://platform.uno/articles/best-cross-platform-frameworks-2026/
- https://github.com/status-im/react-native-desktop-qt
- https://github.com/keejkrej/t3code-native/pull/3
- https://v2.tauri.app/release/core/
- https://v2.tauri.app/plugin/
- https://v2.tauri.app/plugin/notification/
- https://v2.tauri.app/plugin/global-shortcut/
- https://github.com/Choochmeque/tauri-plugin-notifications
- https://github.com/tauri-apps/tao/issues/331
- https://avaloniaui.net/blog/maui-avalonia-preview-1
- https://www.devclass.com/development/2026/03/24/avaloniaui-enhances-net-maui-with-linux-and-webassembly-support/5209515
- https://en.wikipedia.org/wiki/Avalonia_(software_framework)
- https://newreleases.io/project/github/ionic-team/capacitor/release/8.3.1

**Sync and backend**
- https://pub.dev/packages/powersync
- https://releases.powersync.com/
- https://releases.powersync.com/announcements/ann_P6AmNl0W84xt8
- https://releases.powersync.com/announcements/ann_mPwGQpcccCyvL
- https://releases.powersync.com/announcements/powersync-dart-flutter-sdk-v2-0
- https://github.com/powersync-ja/powersync-kotlin
- https://www.powersync.com/pricing
- https://powersync.com/legal/licensing-terms
- https://docs.powersync.com/resources/supported-platforms
- https://docs.powersync.com/configuration/source-db/setup
- https://docs.powersync.com/self-hosting/getting-started
- https://docs.powersync.com/self-hosting/installation/powersync-service-setup
- https://docs.powersync.com/sync/advanced/storage-version-4
- https://docs.powersync.com/client-sdks/reference/tauri
- https://docs.powersync.com/client-sdk-references/kotlin-multiplatform
- https://pub.dev/packages/sqlite_sync
- https://github.com/sqliteai/sqlite-sync
- https://api.github.com/repos/vlcn-io/cr-sqlite
- https://electric.ax/docs/guides/client-development
- https://electric.ax/docs/guides/writes
- https://electric.ax/pricing
- https://pub.dev/packages/electricsql
- https://zero.rocicorp.dev/docs/status
- https://www.infoq.com/news/2026/06/zero-version-1/
- https://rocicorp.dev/blog/retiring-reflect
- https://replicache.dev/
- https://www.instantdb.com/docs
- https://api.github.com/repos/aspen-cloud/triplit
- https://www.ycombinator.com/companies/triplit
- https://jazz.tools/docs/install/client
- https://docs.ditto.live/sdk/latest/compatibility/flutter
- https://pub.dev/packages/ditto_live
- https://pub.dev/packages/cbl
- https://pub.dev/packages/foodb/changelog
- https://pub.dev/packages/fluttercouch
- https://automerge.org/docs/migrating-from-automerge-2-to-automerge-3
- https://pub.dev/packages/yjs_dart
- https://pub.dev/packages/y_crdt
- https://pub.dev/packages/supabase_flutter
- https://github.com/supabase-community/supabase-kt
- https://supabase.com/pricing
- https://supabase.com/changelog/46458-passkeys-for-supabase-auth-beta
- https://pub.dev/packages/cloud_firestore
- https://firebase.google.com/pricing
- https://github.com/appwrite/appwrite/discussions/5326
- https://pub.dev/packages/appwrite_offline
- https://pub.dev/packages/pocketbase
- https://github.com/pocketbase/pocketbase/blob/master/CHANGELOG_23_39.md
- https://www.keycloak.org/2026/07/keycloak-2670-released
- https://www.keycloak.org/2025/09/keycloak-2640-released
- https://www.dbvis.com/thetable/uuidv7-in-postgresql-18-what-you-need-to-know/
- https://frenchydigital.com/blog/offline-first-mobile-architecture-2026

**AI, MCP and legal**
- https://platform.claude.com/docs/en/about-claude/pricing
- https://www.anthropic.com/news/claude-haiku-4-5
- https://www.anthropic.com/legal/commercial-terms
- https://privacy.claude.com/en/articles/7996866-how-long-do-you-store-my-organization-s-data
- https://ai.google.dev/gemini-api/docs/pricing
- https://ai.google.dev/gemini-api/docs/deprecations
- https://ai.google.dev/gemini-api/terms
- https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/gemini/2-5-flash-lite
- https://pricepertoken.com/pricing-page/model/google-gemini-2.5-flash-lite
- https://www.maximem.ai/blog/gemini-api-pricing-2026
- https://developers.openai.com/api/docs/pricing
- https://developers.google.com/edge/mediapipe/solutions/genai/llm_inference
- https://github.com/google-ai-edge/LiteRT-LM
- https://modelcontextprotocol.io/specification/latest
- https://modelcontextprotocol.io/specification/2026-07-28/changelog
- https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization
- https://modelcontextprotocol.io/specification/2025-11-25/basic/authorization
- https://developers.cloudflare.com/email-service/platform/pricing/
- https://f-droid.org/2026/02/24/open-letter-opposing-developer-verification.html
- https://pinggy.io/blog/f_droid_2_0_android_developer_verification/
- https://testerbee.com/blog/google-play-console-registration-fee-and-verification-time
- https://www.helpnetsecurity.com/2026/06/19/android-developer-verification-rollout-markets/

**Competitors**
- https://www.todoist.com/pricing
- https://www.todoist.com/downloads
- https://www.todoist.com/help/articles/troubleshoot-google-calendar-integration-issues-in-todoist-SsSatVeir
- https://www.todoist.com/help/articles/use-claude-code-with-todoist-cli-and-mcp-b1USJ4HB3
- https://www.todoist.com/help/todoist/product-updates/2026-changelog-HD3jJAtLd
- https://github.com/Doist/todoist-mcp
- https://www.developersdigest.tech/mcp-servers/todoist
- https://www.pkgstats.com/pkg:@doist/todoist-mcp
- https://www.morningstar.com/news/pr-newswire/20260121la67618/introducing-todoist-ramble-ai-that-turns-natural-speech-into-structured-tasks
- https://prodsens.live/2026/04/16/building-todoist-ramble-how-doist-turned-voice-braindumps-into-real-time-task-capture/
- https://www.asianefficiency.com/task-management/todoist-ai-setup-guide/
- https://reddit.birdcat.cafe/r/todoist/comments/1jg1t31/new_google_calendar_integration_doesnt_sync
- https://ticktick.com/download
- https://ticktick.com/about/upgrade
- https://help.ticktick.com/articles/7082552170989486080
- https://vyneron.com/en/blog/ticktick-pricing-2026
- https://curtismchale.ca/2023/11/29/issues-with-ticktick
- https://aur.archlinux.org/pkgbase/ticktick
- https://clickup.com/learn/topic/task-management/tools/ticktick/
- https://www.macrumors.com/2026/09/14/things-3-updated-for-ios-27/
- https://www.macrumors.com/2026/08/19/things-3-23-brings-long-requested-overhaul-of-repeating-to-dos/
- https://www.macrumors.com/2026/06/08/ios-27-calendar-reminders/
- https://toolradar.com/tools/things-3/pricing
- https://itsfoss.com/kuro-to-do-app
- https://www.neowin.net/news/microsoft-confirms-major-2026-update-to-remove-several-planner-features-add-new-ones/
- https://support.microsoft.com/en-us/todo/troubleshoot-reminder-issues-in-microsoft-to-do-for-android
- https://workspaceupdates.googleblog.com/2026/09/create-content-schedule-events-and-coordinate-tasks-across-Workspace-regardless-of-what-app-you-are-in.html
- https://astig.ph/google-workspace-intelligence-google-tasks-gemini-2026/
- https://amie.so/changelog
- https://www.usecarly.com/blog/what-happened-to-amie/
- https://www.any.do/
- https://www.saasworthy.com/product/any-do
- https://www.capterra.com/p/173614/Any-do/
- https://www.notion.com/releases/2024-07-17
- https://snapcraft.io/notion-calendar-snap
- https://www.sunsama.com/pricing
- https://aitoolsatlas.ai/tools/sunsama-ai/review
- https://www.usecarly.com/blog/sunsama-review/
- https://usemotion.com/pricing
- https://temporal.day/blog/motion-ai-agents-individual-users-2026
- https://www.usagepricing.com/blueprint/motion
- https://www.lindy.ai/blog/motion-app-pricing
- https://akiflow.com/pricing
- https://aur.archlinux.org/packages/akiflow-bin
- https://play.google.com/store/apps/details?id=io.unorderly.structured&hl=en_US
- https://calmevo.com/structured-app-review/
- https://dupple.com/reviews/superlist
- https://www.omgubuntu.co.uk/2024/02/wunderlist-returns-as-superlist-more-or-less
- https://www.superlist.com/updates/widgets-are-here
- https://reclaim.ai/pricing
- https://siliconflorist.com/2024/08/20/early-ai-startup-reclaim-ai-acquired-by-dropbox/
- https://techcrunch.com/2024/08/22/dropbox-acquires-index-ventures-backed-ai-scheduling-tool-reclaim-ai
- https://www.usecarly.com/blog/reclaim-ai-review/
- https://www.morgen.so/pricing
- https://www.morgen.so/guides/caldavs-supported-by-v4
- https://www.morgen.so/blog-posts/akiflow-review
- https://apps.apple.com/us/app/tweek-minimal-to-do-list/id1535438855
- https://gappsy.com/tools/tweek/
- https://makerstack.co/reviews/llama-life-review/
- https://www.tiimoapp.com/resource-hub/tiimo-android-relaunch
- https://www.tiimoapp.com/product/ai-planning
- https://www.tiimoapp.com/
- https://apps.apple.com/app/tiimo/id1480220328
- https://techcrunch.com/2025/12/04/ai-finds-its-way-into-apples-top-apps-of-the-year
- https://www.pulse.bot/ceo-pulse/news/clockwises-team-joins-salesforce-sunsetting-platform-as-they-go-0a371bee-5e75-4ed3-88e6-7ed0db628c5a/
- https://github.com/super-productivity/super-productivity/releases
- https://tasks.org/
- https://github.com/go-vikunja/vikunja/releases
- https://github.com/alainm23/planify/releases
- https://github.com/mrvladus/Errands/releases
- https://gitlab.gnome.org/World/Endeavour
- https://apps.apple.com/us/app/due-%E5%BF%AB%E9%80%9F%E6%8F%90%E9%86%92-%E5%AE%9A%E6%97%B6%E5%99%A8/id390017969?l=ar
- https://android.goblin.tools/ToDo
- https://www.androidpolice.com/google-keep-ticktick-to-do-app/

**Behavioural science**
- https://kops.uni-konstanz.de/handle/123456789/69905
- https://www.frontiersin.org/journals/psychology/articles/10.3389/fpsyg.2021.565202/full
- https://cancercontrol.cancer.gov/brp/constructs/implementation_intentions/goal_intent_attain.pdf
- https://eprints.whiterose.ac.uk/107519/
- https://bpb-us-e1.wpmucdn.com/wp.nyu.edu/dist/c/6235/files/2019/02/prestwich-et-al-2015-implementation-intentions.pdf
- https://en.wikipedia.org/wiki/Prospective_memory
- https://encyclopedia.pub/entry/31898
- https://discovery.ucl.ac.uk/id/eprint/1468224/
- https://research-information.bris.ac.uk/en/publications/beyond-self-tracking-and-reminders-designing-smartphone-apps-that/
- https://core.ac.uk/download/79500911.pdf
- https://www.thebehavioralscientist.com/articles/how-long-to-form-a-habit
- https://www.ucl.ac.uk/news/2009/aug/how-long-does-it-take-form-habit
- https://pubmed.ncbi.nlm.nih.gov/21688924/
- https://en.wikipedia.org/wiki/Planning_fallacy
- https://bear.warrington.ufl.edu/brenner/mar7588/Papers/buehler-et-al-1994.pdf
- https://faculty.wharton.upenn.edu/wp-content/uploads/2014/06/Dai_Fresh_Start_2014_Mgmt_Sci.pdf
- https://econpapers.repec.org/RePEc:inm:ormnsc:v:60:y:2014:i:10:p:2563-2582
- https://news.byu.edu/news/study-android-users-how-get-people-pay-attention-mobile-security-notifications
- https://www.psypost.org/?p=231293
- https://discovery.ucl.ac.uk/1502364/1/CHI16.pdf
- https://typeset.io/pdf/designing-content-driven-intelligent-notification-mechanisms-35mu5sj3pp.pdf
- https://www.simplypsychology.org/adhd-time-blindness.html
- https://evalab.artun.ee/dual-timer/
- https://researchinnovation.kingston.ac.uk/en/publications/chronotype-and-synchrony-effects-in-human-cognitive-performance-a/

**Name checks (this synthesis)**
- https://alternativeto.net/software/holdfast/
- https://devpost.com/software/cuewise
- https://apps.apple.com/ru/app/id6759036480

---

*Limits of this synthesis.*
- No new primary research was done here beyond the three name searches. Every grade is inherited from the dives and their verifiers.
- Effort, ranking and pricing recommendations are [Speculative].
- Items still [Unverified] that matter most:
  - whether `setAlarmClock` is exempt from the Restricted-bucket cap (Spike S1);
  - the token lifetime of production-unverified Google apps (S4);
  - the per-scope sensitivity class of `calendar.app.created` (S4);
  - Linux notification actions in `flutter_local_notifications` (S2);
  - systemd timer semantics (S2);
  - which Dart MCP SDK supports spec 2026-07-28;
  - GitHub Actions free-tier terms;
  - OpenAI API data-use terms.
