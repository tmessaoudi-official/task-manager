# Competitive landscape and gaps: task, todo, reminder and planner apps (as of 2026-11-01)

Scope: task, todo, reminder and planner apps that matter to an **Android phone + Ubuntu GNOME/Wayland desktop** user who wants deep phone, desktop and calendar integration.

## Evidence grades

Every claim carries one of these grades:

- **[V]** verified. I fetched the page or ran a search, and the URL is given. "[V-primary]" means an official vendor page, official docs or a GitHub release.
- **[V-3p]** verified only against a third-party page, such as a review blog or SEO site. It is weaker evidence, and these sources often contradict each other.
- **[I]** inferred. I reasoned from verified facts, and the basis is stated.
- **[U]** unverified. I could not check it, and the reason is given.

**Method:** WebSearch (extended mode for prices and platform policy) plus WebFetch of vendor pricing and download pages, GitHub releases (through the API, for exact dates), and Google, Microsoft and Android developer docs. Reddit could not be reached directly (only mirrors showed up in search), so user complaints come from vendor help pages, mirrored threads, store-review aggregators and blogs, and they are graded to match.

---

## 1. Master comparison table

The **Linux** column means a native or official desktop app, not a browser tab.

| App | Android | Linux desktop | Pricing (USD) | Calendar integration | AI features (2026) | Grade / source |
|---|---|---|---|---|---|---|
| **Todoist** | Yes, plus Wear OS | Yes, official (downloads page lists Linux) | Free; Pro $60/yr ($5/mo) or $7/mo; Business $96/user/yr | Google **or** Outlook, **one account**. Tasks go out to a separate "Todoist" calendar; calendar events come in **read-only**; no filtering of what syncs. No iCloud or CalDAV | Ramble (voice to tasks; limited on free, unlimited on Pro), Task Assist, Filter Assist, Email Assist | [V-primary] https://www.todoist.com/pricing, https://www.todoist.com/downloads, https://www.todoist.com/help/articles/troubleshoot-google-calendar-integration-issues-in-todoist-SsSatVeir ; Ramble launch date of 2026-01-21 is [V-3p] https://www.asianefficiency.com/task-management/todoist-ai-setup-guide/ |
| **TickTick** | Yes | Yes, listed on the official download page (package format not stated) | Free (capped); Premium **$49.99/yr**. One source says it rose from $35.99 by Sept 2026 | Calendar subscription feed (TickTick to calendar). Third parties report Google two-way sync on Premium | AI Assistant (Oct 2026), MCP (Apr 2026), voice parsing | Platforms [V-primary] https://ticktick.com/download ; price [V-3p] https://vyneron.com/en/blog/ticktick-pricing-2026 ; AI and MCP [V-3p, a snippet of the official help centre; the full page did not render] https://help.ticktick.com/articles/7082552170989486080 |
| **Things 3** | **No** | **No** | Paid once per Apple platform [U: price not fetched] | Apple ecosystem only | 3.24 (Sept 2026): iOS 27 Siri AI, Spotlight; 3.23 reworked repeating to-dos | [V-3p] https://www.macrumors.com/2026/09/14/things-3-updated-for-ios-27/ , https://www.macrumors.com/2026/08/19/things-3-23-brings-long-requested-overhaul-of-repeating-to-dos/ |
| **Microsoft To Do** | Yes | **No official app**; only unofficial Electron wrappers (Kuro, Ao, a Snap package) | Free with a Microsoft account | Built into Outlook Tasks. Planner's Outlook calendar and iCal feed were retired in early 2026 with no replacement | Copilot gets the investment; To Do itself is not being retired | Linux [V-3p] https://itsfoss.com/kuro-to-do-app ; Planner retirement [V-3p] https://www.neowin.net/news/microsoft-confirms-major-2026-update-to-remove-several-planner-features-add-new-ones/ ; Graph API reminder fields [V-primary] https://learn.microsoft.com/en-us/graph/api/resources/todotask |
| **Google Tasks** | Yes | No (web/PWA only) | Free | Shown inside Google Calendar. **The Tasks API drops the time of day**: "It isn't possible to read or write the time that a task is scheduled for using the API." The API has no reminder fields | Gemini creates tasks from any Workspace app (rollout from 2026-09-02); Tasks data goes into Workspace Intelligence from 2026-11-14 | API [V-primary] https://developers.google.com/workspace/tasks/reference/rest/v1/tasks ; Gemini [V-primary] https://workspaceupdates.googleblog.com/2026/09/create-content-schedule-events-and-coordinate-tasks-across-Workspace-regardless-of-what-app-you-are-in.html ; Nov-14 date [V-3p] https://astig.ph/google-workspace-intelligence-google-tasks-gemini-2026/ |
| **Apple Reminders** | **No** | **No** | Free | iCloud. Third parties sync through CalDAV/EventKit (Amie added two-way sync in Feb 2026) | iOS 27: natural-language entry, an Apple Intelligence "describe" option (needs iPhone 15 Pro or later) | [V-3p] https://www.macrumors.com/2026/06/08/ios-27-calendar-reminders/ ; Amie sync [V-primary] https://amie.so/changelog |
| **Any.do** | Yes | Unknown or web [U: not checked] | Premium about $4.99/mo billed yearly; Family $8.33/mo; 14-day trial | Built-in calendar (providers not verified) | AI suggestions; "works with AI agents" (an agent can create tasks and set reminders) | [V-3p] https://www.saasworthy.com/product/any-do , [V-primary homepage claim] https://www.any.do/ ; complaints about "unreliable and intrusive notifications" [V-3p] https://www.capterra.com/p/173614/Any-do/ |
| **Notion Calendar** | Yes (since July 2024) | **No official app**; only unofficial Electron and Snap builds | Free | Google; Outlook.com was missing on Android at launch | Notion AI lives in Notion, not in the calendar [I] | [V-primary] https://www.notion.com/releases/2024-07-17 ; Linux [V-3p] https://snapcraft.io/notion-calendar-snap |
| **Sunsama** | Yes, but a "companion" app | **Yes**, a Linux desktop app is listed | Pro $17/mo yearly ($204) or $22/mo; 14-day trial; no free tier | Google and Outlook (the pricing page does not mention iCloud) | "AI + MCP + Zapier" (narrow: time estimates, channel tagging) | [V-primary] https://www.sunsama.com/pricing ; AI scope [V-3p] https://aitoolsatlas.ai/tools/sunsama-ai/review |
| **Motion** | Yes | Desktop apps (Linux not confirmed) | Pro AI $19/seat/mo, Business AI $29/seat/mo; credit pools; trial only | Google, Outlook [I] | Pivoted to a business "AI Employees" suite, then went back to Pro AI and Business AI tiers with credits (June 2026) | [V-primary] https://usemotion.com/pricing ; pivot [V-3p] https://temporal.day/blog/motion-ai-agents-individual-users-2026 , https://www.usagepricing.com/blueprint/motion |
| **Akiflow** | Yes (out of beta Sept 2024) | **No official app** (unofficial AUR `akiflow-bin`) | Pro $19/mo yearly ($228) or $34/mo; **no free tier**; 7-day trial with card | Google and Outlook | Aki AI assistant, hosted MCP connector, AI Meeting Assistant add-on at $19/mo | [V-primary] https://akiflow.com/pricing ; Linux [V-3p] https://aur.archlinux.org/packages/akiflow-bin |
| **Amie** | Not mentioned in 2026 changelogs [U] | No | [U: pricing not fetched] | Google; two-way Outlook sync **only since 2026-08-05**; Apple Reminders two-way (Feb 2026) | Pivoted to AI meeting notes; MCP server (2026-07-14) | [V-primary] https://amie.so/changelog ; pivot [V-3p] https://www.usecarly.com/blog/what-happened-to-amie/ |
| **Structured** | Yes, but the Android app was rebuilt in May 2026 and is "catching up"; Google Calendar sync was "coming next" | No | Free; Pro about $6.99/mo, $29.99/yr or $99.99 lifetime | Apple Calendar on iOS; Android Google sync pending | "Structured AI" [U] | Play listing [V-primary] https://play.google.com/store/apps/details?id=io.unorderly.structured&hl=en_US ; price [V-3p] https://calmevo.com/structured-app-review/ |
| **Superlist** | Yes (widgets since 2024; complaints about minimal widgets) | **No** (an open feature request) | Free (5 lists); Basic about $5/mo; Super about $21/mo | [U] | Voice AI, meeting notes, MCP (v1.54, May 2026) | [V-3p] https://dupple.com/reviews/superlist ; Linux [V-3p] https://www.omgubuntu.co.uk/2024/02/wunderlist-returns-as-superlist-more-or-less ; widgets [V-primary] https://www.superlist.com/updates/widgets-are-here |
| **Reclaim (Dropbox)** | No mobile app on the pricing page | No (web; Raycast for Mac) | Lite free; Starter $10, Business $15, Enterprise $22 /seat/mo yearly | Google and Outlook; **no iCloud** | "AI agents", Claude and ChatGPT connections | [V-primary] https://reclaim.ai/pricing ; Dropbox acquisition (Aug 2024) [V-3p] https://siliconflorist.com/2024/08/20/early-ai-startup-reclaim-ai-acquired-by-dropbox/ |
| **Morgen** | Yes | **Yes, official Linux desktop** | Individual $15/mo yearly or $30/mo; Team $10–25/seat; 14-day trial | **Google, Outlook/M365, iCloud, Fastmail, CalDAV.** The broadest of any app in this survey | AI Planner (time-blocking), "Kai" assistant | [V-primary] https://www.morgen.so/pricing |
| **Tweek** | Yes (one source disagrees) | Web | Free; Premium $5.99/mo or $49.99/yr (App Store) | Google, Outlook, Apple | "Tweek GPT" | [V-3p] https://apps.apple.com/us/app/tweek-minimal-to-do-list/id1535438855 , https://gappsy.com/tools/tweek/ |
| **Llama Life** | Yes (Play listing exists) | Web | About $6/mo or $39/yr (unconfirmed with the vendor) | [U] | A timer on every task (ADHD single-tasking) | [V-3p] https://makerstack.co/reviews/llama-life-review/ |
| **Tiimo** | Rebuilt **2026-05-04**: daily plan, focus timer, AI task breakdown. **Google Calendar and the AI Co-Planner are still "upcoming"** | No | Pro about $54/yr (App Store) | Pending on Android | AI task breakdown; Co-Planner on iOS | [V-primary] https://www.tiimoapp.com/resource-hub/tiimo-android-relaunch ; price [V-3p] https://apps.apple.com/app/tiimo/id1480220328 |
| **Clockwise** | – | – | **SHUT DOWN 2026-03-27.** Salesforce hired the team (Agentforce); the product and data were not acquired | – | – | [V-3p, several consistent reports] https://www.pulse.bot/ceo-pulse/news/clockwises-team-joins-salesforce-sunsetting-platform-as-they-go-0a371bee-5e75-4ed3-88e6-7ed0db628c5a/ |

### Open source

All release dates below are [V-primary], from the GitHub API `releases/latest` endpoint on 2026-11-01.

| App | Platforms | Latest release | Sync / calendar | Notes |
|---|---|---|---|---|
| **Super Productivity** | Android APK, Linux `.deb` and `.snap`, Windows, macOS | v19.3.1, 2026-10-25 | SuperSync (self-hostable), WebDAV, Dropbox, OneDrive, Nextcloud; CalDAV; Google Calendar with several accounts (v19.1); Trello two-way; GitLab, Gitea and Azure DevOps | Developer and time-tracking focus. Electron [I: the ~110 MB `.deb` and Electron-style artefacts] https://github.com/super-productivity/super-productivity/releases |
| **Tasks.org** | Android (Play, F-Droid); **desktop in alpha**; Pebble | 15.13, 2026-10-14 | Tasks.org sync, Google Tasks, DAVx5/CalDAV, EteSync (end-to-end encrypted), DecSync | GPLv3, no ads. The strongest Android CalDAV task app [I] https://tasks.org/ |
| **Vikunja** | Self-hosted server plus web; Android APKs in the assets | v2.7.1, 2026-10-20 | CalDAV (fixes in v2.5) | v2.7.0 added an **MCP server**; v2.7.1 shipped **41 security fixes** https://github.com/go-vikunja/vikunja/releases |
| **Planify** | GNOME (Flathub) | v4.20.0, 2026-09-10 | Todoist; Nextcloud/CalDAV (Radicale, Posteo, Mailbox.org, Vikunja); Nextcloud Deck; two-way reminder sync | GNOME Online Accounts, shell search provider. **No Android app** [I: none found] https://github.com/alainm23/planify/releases |
| **Errands** | GNOME (Flatpak) | 51.0.1, 2026-11-01 (51.0 was a **full rewrite from Python to C**, 2026-10-31) | Now goes through evolution-data-server (so GNOME Online Accounts and CalDAV) [I: EDS is GNOME's PIM backend] | Very active; no Android https://github.com/mrvladus/Errands/releases |
| **Endeavour** (formerly GNOME To Do) | GNOME | Last repo activity 2026-09-19 (GitLab API) | EDS-based | Low activity [I] https://gitlab.gnome.org/World/Endeavour |

---

## 2. Per-app notes: standout features and known complaints

- **Todoist.** Its strengths are the best natural-language quick add, Ramble voice capture and the broadest platform coverage, including official Linux and Wear OS [V-primary, pricing and downloads]. Its calendar integration is weak:
  - it connects one account only;
  - Google events are read-only inside Todoist;
  - new events created in the "Todoist" calendar do not become tasks;
  - there is no project or tag filter;
  - tasks without a time are not synced unless "Sync all-day tasks" is on;
  - shared-project tasks sync only when they are assigned to you.

  The first four points are [V-primary, help article]; the shared-project point is [V-3p, mirrored r/todoist thread] https://reddit.birdcat.cafe/r/todoist/comments/1jg1t31/new_google_calendar_integration_doesnt_sync. Prices went up around Dec 2025 alongside the AI features [V-3p].
- **TickTick.** The all-in-one app: tasks, calendar views, Pomodoro and a habit tracker [V-3p]. Premium appears to have gone up about 39% in 2026 [V-3p, a single source]. In 2023 the Linux app was reported to crash daily on Fedora and to have **no system-wide quick-entry on Linux** (macOS has one) [V-3p, 2023] https://curtismchale.ca/2023/11/29/issues-with-ticktick. Whether that is still true in 2026 is [U]. The AUR package needs a workaround to run it under Wayland [V-3p] https://aur.archlinux.org/pkgbase/ticktick.
- **Microsoft To Do.** Free, deep in Outlook, with a good Graph API (`reminderDateTime`, `linkedResources`, delta query) [V-primary]. It has no Linux client. Its Android reminders need OEM battery whitelisting: Microsoft publishes Samsung-specific steps for this [V-primary] https://support.microsoft.com/en-us/todo/troubleshoot-reminder-issues-in-microsoft-to-do-for-android.
- **Google Tasks.** The default on Android, inside Calendar and Gemini. It is a **dead end for third-party sync of timed tasks**, because the API cannot carry a time of day [V-primary].
- **Things 3 and Apple Reminders.** Polished, but Apple-only. They are irrelevant to an Android and Linux user except as sync targets (iCloud) [V].
- **Sunsama, Akiflow, Motion, Morgen, Reclaim.** The "daily planning and time-blocking" class costs **$15–34/month** [V-primary]. Of these, only Morgen and Sunsama list Linux desktop apps, and only Morgen supports iCloud and CalDAV [V-primary]. Sunsama's Android app is a companion, and planning has to start on desktop [V-3p].
- **Amie, Superlist, Structured, Tiimo.** Design-led apps. Each one treats Android or Linux as second-class:
  - Amie's 2026 changelog mentions only macOS [V-primary];
  - Superlist has no Linux app and Android widgets are minimal [V-3p];
  - Structured and Tiimo **rebuilt their Android apps in May 2026, and both still lack Google Calendar sync** on Android [V-primary Tiimo; V-3p Structured].

---

## 3. Shifts in 2025–2026

1. **MCP became a standard feature in 2026.** TickTick (April) [V-3p], Superlist (May) [V-3p], Amie (July) [V-primary], Vikunja (October) [V-primary], Akiflow (hosted) [V-primary], Sunsama ("AI + MCP") [V-primary] and Reclaim (Claude/ChatGPT) [V-primary] all added it. Any.do markets itself as "works with AI agents" [V-primary]. **Implication:** an agent interface is now the minimum, not something that sets an app apart [I].
2. **Platform owners are absorbing AI capture.** Gemini creates Google Tasks from any Workspace app (rollout from 2026-09-02) [V-primary]. iOS 27 Reminders gained natural-language entry and Apple Intelligence [V-3p]. Things gained Siri AI [V-3p]. **Implication:** basic "AI turns a sentence into a task" no longer sets an app apart [I].
3. **Consolidation and exits.**
   - Clockwise shut down on 2026-03-27 after a Salesforce acqui-hire, and user data was deleted [V-3p, consistent reports].
   - Reclaim is now owned by Dropbox [V-3p].
   - Motion pivoted toward business AI agents, then reshuffled its tiers into credit pools [V-primary pricing; V-3p history].
   - Amie moved its emphasis from calendar to AI meeting notes [V-3p plus changelog].

   **Implication:** solo, prosumer users of AI schedulers have been repeatedly stranded, so **data portability and local-first design are real selling points** [I].
4. **Prices went up.** Todoist after Dec 2025 [V-3p], TickTick +39% [V-3p], and AI is metered through credits (Motion) [V-primary]. Akiflow has no free tier [V-primary].
5. **Big-vendor regressions.** Microsoft retired Planner's Outlook calendar view and iCal feed with no replacement [V-3p].
6. **Android delivery is tighter.** Since Android 14, `SCHEDULE_EXACT_ALARM` is not pre-granted. `USE_EXACT_ALARM` is restricted by Play policy to alarm-clock and calendar apps [V-primary docs] https://developer.android.com/about/versions/14/changes/schedule-exact-alarms. OEM battery layers (Samsung, Xiaomi, Huawei) still kill alarms [V-primary Microsoft support; V-3p dev.to]. Whether a pure to-do app qualifies for `USE_EXACT_ALARM` depends on the current wording of Play policy, which I did not fetch [U].
7. **The open-source GNOME stack is very active.** Errands was rewritten in C on 2026-10-31, Planify 4.20 adds CalDAV and Deck, and Tasks.org has a desktop alpha [V-primary]. But **no open-source project spans native GNOME and polished Android with multi-provider calendars** [I: from the platform columns above].

---

## 4. Gaps: what no app does well, especially on Android and Linux

| # | Gap | Evidence |
|---|---|---|
| G1 | **A native, first-class Linux desktop experience.** Most commercial apps are web-only or unofficial Electron wrappers on Linux (To Do, Notion Calendar, Akiflow, Superlist, Reclaim, Amie). Where official Linux apps exist (Todoist, TickTick, Sunsama, Morgen), they are Electron or cross-platform shells, and TickTick was reported to lack a system-wide quick-add on Linux | [V] platform table; quick-add point [V-3p, 2023]. Electron for Todoist and Morgen specifically: [U, not verified] |
| G2 | **A global quick-add hotkey on GNOME Wayland.** Wayland does not allow arbitrary global key grabs; it needs the XDG GlobalShortcuts portal or a GNOME custom shortcut that calls the app's CLI or D-Bus interface. I found no competitor documenting this | [I: Wayland security model; the 2023 TickTick report]. Portal support in GNOME: [U, not fetched for this dive; belongs in the platform dive] |
| G3 | **Real two-way sync with ALL calendar ecosystems.** Typical support is Google plus Outlook only (Todoist with one account, Sunsama, Akiflow, Reclaim). iCloud and CalDAV/Nextcloud are rare: only Morgen among paid apps, and Planify and Tasks.org among open-source apps (CalDAV). "Two-way" often means read-only events plus pushed tasks (Todoist) | [V-primary] Todoist help, the pricing pages |
| G4 | **Google Tasks cannot carry a time of day through its API**, so every third-party app "syncing Google Tasks" loses times and reminders. An app has to model timed reminders as Calendar events or keep its own store | [V-primary] Tasks API reference |
| G5 | **Reminders that reliably fire on Android.** OEM battery killers and the exact-alarm permission changes. Vendors push the fix onto users (Microsoft's Samsung guide). Any.do is called out for "unreliable notifications" | [V-primary] Microsoft support, Android docs; [V-3p] Capterra |
| G6 | **Rich, interactive Android widgets plus quick-capture surfaces** (Quick Settings tile, share sheet, voice). Superlist users complain the widgets are bare. Structured and Tiimo rebuilt their Android apps only in May 2026, without calendar sync. iOS gets first-class treatment (Things' new widget, Live Activities) [I] | [V-3p] android-stats summary; [V-primary] Tiimo |
| G7 | **Affordable AI planning for individuals.** Auto-scheduling and time-blocking cost $15–34/month, metered by credits, or come from vendors pivoting away from individual users (Motion, Clockwise gone, Amie pivoted) | [V-primary] pricing pages; [V-3p] shutdown and pivot |
| G8 | **Data ownership, local-first and portability.** Clockwise users lost their data within a week, and the paid apps are all cloud silos. Open-source options exist but are fragmented by platform | [V-3p] Clockwise; [I] |
| G9 | **One model for tasks, reminders, events and habits.** Users juggle Google Tasks plus Calendar plus Keep, or To Do plus Outlook. Apps either do tasks, or calendar-with-tasks, or time-blocking, and rarely all of them across Android and Linux | [I] from the survey; Android Police "Goodbye Google Keep" [V-3p] https://www.androidpolice.com/google-keep-ticktick-to-do-app/ |
| G10 | **An agent interface that is both local and private.** MCP servers in 2026 are mostly vendor-hosted (Akiflow, TickTick, Amie). A self-hosted or on-device MCP plus a CLI for power users (the bash and Linux persona) is rare; Vikunja is the open-source exception | [V-primary] Akiflow, Vikunja; [I] |
| G11 | **Location and context reminders that work across devices.** Tasks.org and Apple Reminders have them; most cross-platform SaaS apps do not advertise them | [U: not verified per app] |
| G12 | **Developer and power-user integration with GNOME** (search provider, GNOME Online Accounts, Evolution Data Server). Only the GNOME-native open-source apps (Planify, Errands) do this, and they have no Android counterpart | [V-primary] Planify and Errands releases |

### What users ask for again and again

- Two-way calendar sync with per-project or tag filtering [V-3p] (TaskCalSync on Product Hunt, r/todoist).
- Linux quick entry [V-3p] (TickTick blog post).
- Better widgets [V-3p] (Superlist).
- Reliable notifications [V-3p] (Any.do, Capterra).
- Linux apps (Superlist's open feature request) [V-3p].
- No subscription increases [V-3p].
- ADHD-friendly time-boxing (Llama Life, Tiimo, Structured have a strong niche) [V-3p].

---

## 5. Implications for the user's product (inferred, for the synthesis step)

All points in this section are [I], based on the gaps above.

- **The defensible wedge** is not "AI sentence to task", which is commoditised (Gemini, iOS 27, everyone). It is:
  1. Android reminder reliability as an engineered, tested guarantee (G5);
  2. true multi-provider two-way calendar sync, including CalDAV and iCloud (G3, G4);
  3. a native-feeling GNOME/Wayland desktop with portal hotkey, tray and notifications (G1, G2, G12);
  4. local-first data with export, plus a self-hostable MCP/CLI for agents (G8, G10).
- **Match on day one:** natural-language quick add, voice capture, widgets, share sheet, recurring tasks with "edit this instance vs the rule" (Things 3.23 made this the bar), and an MCP server.
- **Avoid:** depending on the Google Tasks API for timed reminders (G4). Do not price AI scheduling at Motion or Akiflow levels before launch.

## 6. Limits of this dive

- Reddit could not be reached directly; complaints come from mirrors, aggregators and vendor help pages.
- Many prices come from third-party sites that contradict each other. Primary pricing was fetched for Todoist, Motion, Sunsama, Akiflow, Morgen and Reclaim only.
- Not verified:
  - prices and platforms for Things, Amie and Any.do (Linux);
  - Notion Calendar's 2026 Outlook status on Android;
  - Llama Life's official pricing;
  - TickTick's Linux package format and its 2026 Wayland quick-add status;
  - the current wording of Play policy on `USE_EXACT_ALARM`.
- The advisor review was rate-limited during this dive, so it was not run. These findings are self-checked only.
