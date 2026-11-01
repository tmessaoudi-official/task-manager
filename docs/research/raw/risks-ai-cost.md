# Dive: AI features, costs, privacy, store policies and risks

Date of research: 2026-11-01. Grades: **[verified: URL]** means the source was fetched or searched this session. **[inferred: basis]** means it was reasoned from verified facts. **[unverified: why]** means it could not be checked (recall, or the page did not render).

---

## 1. AI features: cost, quality and on-device options

### 1.1 Cloud LLM prices per million tokens, current as of 2026-11-01

| Provider / model | Input $/Mtok | Cached input | Output $/Mtok | Grade |
|---|---|---|---|---|
| **Claude Haiku 5.5** (prompts ≤100k tokens) | 0.10 | 0.01 (hit) | 0.50 | [verified: https://platform.claude.com/docs/en/about-claude/pricing] |
| Claude Haiku 5.5 (prompts >100k tokens) | 0.50 | 0.05 | 2.50 | [verified: same] |
| Claude Haiku 4.5 | 1.00 | 0.10 | 5.00 | [verified: same] |
| **Claude Sonnet 5.5** | 2.00 | 0.10 | 10.00 | [verified: same] |
| Claude Sonnet 4.6 | 3.00 | 0.30 | 15.00 | [verified: same] |
| Claude Opus 5.5 | 4.00 | 0.20 | 20.00 | [verified: same] |
| Anthropic Batch API | 50% off input and output | | | [verified: same] |
| **Gemini 3.8 Flash** | 0.75 | n/a | 3.75 | [verified: https://ai.google.dev/gemini-api/docs/pricing] |
| **Gemini 3.5 Flash-Lite** | 0.30 | n/a | 2.50 | [verified: same] |
| **Gemini 3.1 Flash-Lite** | 0.25 | n/a | 1.50 | [verified: same] (a third-party source gives a 2027-05-07 shutdown date: [unverified: https://www.maximem.ai/blog/gemini-api-pricing-2026]) |
| Gemini 2.5 Flash-Lite (older) | 0.10 | 0.01 | 0.40 | [verified: same] |
| Gemini price changes | many models rise on 2026-12-01 or 2027-01-01 | | | [verified: same page; which models are affected is not fully itemised] |
| **gpt-6-luna** (OpenAI's newest small model) | 0.10 | 0.01 | 0.50 | [verified: https://developers.openai.com/api/docs/pricing] |
| gpt-5.6-luna | 0.20 | 0.02 | 1.20 | [verified: same] |
| gpt-5.4-mini | 0.75 | 0.075 | 4.50 | [verified: same] |
| gpt-5.4-nano | 0.20 | 0.02 | 1.25 | [verified: same] |
| gpt-5-mini | 0.25 | 0.025 | 2.00 | [verified: same] |
| gpt-5-nano | 0.05 | 0.005 | 0.40 | [verified: same] |
| OpenAI regional endpoints | +10% for models released on or after 2026-03-05 | | | [verified: same] |

**Rule for the EU: the free Gemini tier cannot be used.** The Gemini API terms say *"You may use only Paid Services when making API Clients available to users in the European Economic Area"* (the same applies to Switzerland and the UK). Free-tier inputs may be read by human reviewers and used to improve Google products. Paid services are not used for training. [verified: https://ai.google.dev/gemini-api/terms]

**Anthropic.** The Commercial Terms say *"Anthropic may not train models on Customer Content from Services"*, and a DPA is incorporated by reference [verified: https://www.anthropic.com/legal/commercial-terms]. API inputs and outputs are deleted within 30 days; content flagged for a Usage Policy violation is kept up to 2 years; zero data retention is available by agreement [verified: https://privacy.claude.com/en/articles/7996866-how-long-do-you-store-my-organization-s-data].

### 1.2 The AI features that matter for this app, by model tier

| Feature | What it does | Tier needed | Notes |
|---|---|---|---|
| NL quick-add ("dentist tue 3pm every 2 weeks #health !high") | extracts fields into JSON | **Deterministic parser first**, then a small LLM as fallback | Most inputs are parsed by chrono-style grammars; the LLM handles only leftovers and ambiguous text [inferred: chrono covers relative and absolute dates, see 1.4] |
| Brain-dump triage (paste or dictate 20 lines, get tasks, projects, dates and dedup) | multi-item structured output | Small to mid model (Haiku 5.5, gpt-6-luna, Gemini Flash-Lite) | Use JSON schema / structured outputs; validate before writing [inferred] |
| Daily or weekly planning ("plan my day around 3 meetings, 2h focus") | reasons over calendar and tasks | Mid model (Sonnet 5.5, Gemini 3.8 Flash), opt-in | The costliest feature; meter it [inferred: from the cost table below] |
| Recurrence-rule synthesis ("last weekday of each month except Dec") | produces an RRULE | Small LLM plus **mandatory validation** with an RFC 5545 library | Never trust a generated RRULE without expanding it and showing the user the next 5 dates [inferred] |
| Voice capture | speech to text, then parse | On-device speech recognition (Android ML Kit GenAI Speech Recognition / platform SpeechRecognizer), then the parser | [verified that the API exists: https://developer.android.com/ai/gemini-nano] |
| Semantic search / dedup | embeddings | On-device (ML Kit GenAI embedding) or cheap cloud embeddings | [verified that the API exists: same] |

### 1.3 Estimated monthly cost per active user

Assumed usage profile [inferred: a design assumption, not measured]. A "typical" user makes 10 LLM-assisted captures a day (≈600 input tokens including a cached system prompt and ≈80 output tokens each), 2 brain-dumps a week (2,000 in / 800 out), and 1 daily plan (3,000 in / 600 out). That is about **286k input + 48k output tokens per user per month**. A "heavy" user is 3× that.

| Model | Typical user / month | Heavy user (3×) | With deterministic-first parsing (~70% of captures skip the LLM) |
|---|---|---|---|
| gpt-5-nano | ≈ $0.034 | ≈ $0.10 | ≈ $0.025 |
| Claude Haiku 5.5 | ≈ $0.053 | ≈ $0.16 | ≈ $0.04 |
| gpt-6-luna | ≈ $0.053 | ≈ $0.16 | ≈ $0.04 |
| Gemini 3.1 Flash-Lite | ≈ $0.14 | ≈ $0.43 | ≈ $0.10 |
| Gemini 3.8 Flash | ≈ $0.40 | ≈ $1.21 | ≈ $0.30 |
| gpt-5.4-mini | ≈ $0.43 | ≈ $1.30 | ≈ $0.32 |
| Claude Sonnet 5.5 (all features) | ≈ $1.06 | ≈ $3.17 | ≈ $0.80 |
| **Recommended mix**: Haiku 5.5 / gpt-6-luna for parsing and triage, Sonnet 5.5 only for daily planning | **≈ $0.40** (planning ≈ $0.36 + Haiku share ≈ $0.035) | ≈ $1.19 | ≈ $0.38 |

[inferred: arithmetic on the verified prices in 1.1. Prompt caching lowers the input share further. A 5-minute cache write on Haiku 5.5 is $0.125 per Mtok, so a long shared system prompt is cheap.] **Conclusion:** at one-user scale the AI cost is close to zero (cents a month). At product scale it is **$0.05 to $1 per active user per month**, well under a $3 to $5/month subscription price. The real risk is abuse, such as unmetered planning calls, so add per-user quotas [inferred].

**Architecture implication.** Never put a provider API key in the mobile or desktop binary, because a public repo plus an APK means the key gets extracted. Route calls through a thin proxy (a Cloudflare Worker, a small server, or Firebase AI Logic with App Check, which Google documents for exactly this case [verified: https://developer.android.com/ai/gemini-nano, which links Firebase AI Logic hybrid plus App Check]). For personal use only, BYOK (the user pastes their own key into secure storage) is acceptable [inferred].

### 1.4 On-device and deterministic alternatives

| Option | Status (2026-11) | Fit | Grade |
|---|---|---|---|
| **Gemini Nano via AICore / ML Kit GenAI** (Android) | Has a Prompt API (`GenerativeModel`), an AgentRunner, summarization, proofreading, rewriting, image description, speech recognition, embeddings. **The Prompt API is beta**: *"not subject to any SLA or deprecation policy"*. AICore *"doesn't store any record of the input data or outputs"*. Device support is limited to recent flagships | Good for free, private, offline parsing on supported phones; must fall back to the cloud or the deterministic parser elsewhere | [verified: https://developer.android.com/ai/gemini-nano ; https://developers.google.com/ml-kit/genai/prompt/android]. Device list and token limits: [unverified: not shown on the fetched pages] |
| Firebase AI Logic hybrid API | routes to Nano when it is available, otherwise to cloud Gemini Flash/Pro | The cleanest hybrid on Android | [verified: same page] |
| **Apple Foundation Models** (iOS/macOS 26+) | on-device model with guided generation (`@Generable`) and tool calling; an Apple WWDC26 lab answer says the context is **4,096 tokens, shared** between input and output (one third-party source says 8,192); WWDC26 adds a 32k-context server model via Private Cloud Compute | Ideal for the later iOS port; requires an Apple Intelligence device | [verified: https://developer.apple.com/videos/play/wwdc2026/8121/ , https://developer.apple.com/videos/play/wwdc2026/241/ via search snippets]. The context-size conflict is unresolved |
| **MediaPipe LLM Inference** (Android/iOS) | **deprecated on mobile, maintenance-only**; Google points to **LiteRT-LM** | Do NOT build on MediaPipe LLM Inference | [verified: https://ai.google.dev/edge/mediapipe/solutions/genai/llm_inference] |
| **LiteRT-LM + Gemma 4 E2B / Gemma 3n** | Google's current open on-device LLM runtime (Android, desktop) | Possible for Linux desktop and non-AICore phones, but the model download (~1 to 3 GB) is heavy for a todo app | [verified: https://github.com/google-ai-edge/LiteRT-LM]. Model size: [unverified: recall] |
| llama.cpp (desktop, Ubuntu) | mature; runs small GGUF models (Qwen/Gemma/Llama ~1 to 4B) on CPU | Optional "offline AI" for power users on the PC | [unverified: not fetched this session; widely known] |
| **chrono (JS)** | MIT; supports **French (`fr`)** and English, partial de/es/pt | Usable in Tauri / RN / web, or as a reference grammar | [verified: https://github.com/wanasit/chrono] |
| **chrono_dart** (Flutter) | MIT port of chrono; **English only**; last release 2.0.2, last push Aug 2024 (stale) | Starting point; would need a fork plus French rules | [verified: https://pub.dev/packages/chrono_dart] |
| Duckling (Facebook, Haskell), dateparser (Python), Natty (Java, abandoned) | server-side only, or dead | Not for an on-device client | [unverified: recall] |

**Recommendation.** A three-layer parser:
1. A deterministic grammar written in the app's own language, covering FR + EN, with golden tests. This is the zero-cost path that works offline.
2. On-device LLM where available (Nano / Apple FM) for brain-dump structuring.
3. A cloud small model (Haiku 5.5 or gpt-6-luna) as opt-in fallback, behind a proxy.

Always show a confirmation preview ("Tue 4 Nov 15:00, repeats every 2 weeks") before saving. [inferred]

---

## 2. Stores and publishing

| Item | Fact | Grade |
|---|---|---|
| Google Play registration fee | **$25 one-time**, personal or organisation; government ID required; the fee is not refunded if the ID is invalid | [verified via search: https://support.google.com/googleplay/android-developer/answer/6112435 , https://testerbee.com/blog/google-play-console-registration-fee-and-verification-time] |
| Closed testing for new personal accounts | Accounts created after **2023-11-13** need **at least 12 testers opted in continuously for the preceding 14 days** before production access (an older 20-tester figure was reduced to 12) | [verified: https://support.google.com/googleplay/android-developer/answer/14151465]. The 20-to-12 history: [unverified: recall, consistent with the page showing 12] |
| **Android developer verification** (applies to sideloading and F-Droid) | Timeline: **Aug 2026**: developer APIs, *limited distribution accounts* and the power-user *advanced flow* launched. **2026-09-30**: protections begin in **Brazil, Indonesia, Singapore and Thailand** on certified Android 7+ devices. **2027**: global expansion | [verified: https://developer.android.com/developer-verification] |
| Android Developer Console (distribution outside Play) | **$25 fee** for a Full Distribution account; **fee waived** for a Limited Distribution account (students and hobbyists, **up to 20 devices**, no government ID) | [verified: https://developer.android.com/developer-verification/guides/faq , https://developer.android.com/developer-verification] |
| ADB | *"Apps installed using ADB won't require verification"*; no waiting period for ADB installs | [verified: FAQ above] |
| Advanced flow (users installing unverified apps) | a one-time, one-day (24h) wait, then biometric or PIN confirmation | [verified: FAQ above] |
| Effect on **you** in France | In Nov 2026 France is not yet enforced; enforcement comes in 2027. For personal use, ADB installs or a Limited Distribution account (≤20 devices, free) cover you. For a public product, register (Play auto-registers 99% of Play apps) | [inferred: from the timeline + FAQ] |
| F-Droid | F-Droid calls verification "existential" (an open letter on 2026-02-24); it shipped F-Droid 2.0 (a Kotlin/Compose rewrite) on 2026-09-24; for reproducible builds it can ship under the upstream developer's signing key, which implies the upstream developer must be verified | [verified: https://f-droid.org/2026/02/24/open-letter-opposing-developer-verification.html ; https://pinggy.io/blog/f_droid_2_0_android_developer_verification/ (secondary)] |
| **Exact alarm on Play** | `USE_EXACT_ALARM` only if *"core functionality supports the need for an exact alarm"*; the examples given are *"an alarm or timer app"* and *"a calendar app that shows event notifications"*. Otherwise use the user-granted `SCHEDULE_EXACT_ALARM` | [verified: https://support.google.com/googleplay/android-developer/answer/13986130] |
| Implication for a task/reminder app | A reminder app is **not explicitly listed**. The safe path is `SCHEDULE_EXACT_ALARM` with an onboarding screen and a `canScheduleExactAlarms()` check. If the app shows calendar events, a `USE_EXACT_ALARM` declaration is arguable but carries review risk | [inferred: policy wording] |
| Full-screen intent (`USE_FULL_SCREEN_INTENT`) | auto-granted on Android 14+ only to alarm and calling apps; others must ask the user and should submit a declaration | [verified: same policy page] |
| Apple Developer Program | **$99/year**; fee waivers exist (non-profits etc.) | [verified: https://developer.apple.com/programs/] |
| Flathub | no fee is mentioned; the app must be legally redistributable and **built entirely from source with no network access during the build**; portals are mandatory where they exist | [verified: https://docs.flathub.org/docs/for-app-authors/requirements] |
| Flathub for a Flutter app | needs the Flutter SDK and pub packages vendored as sources; this is doable (other Flutter apps exist there) but is real packaging work | [inferred: the build-from-source/no-network rule; that Flutter apps exist on Flathub is unverified recall] |
| Snap Store | free to publish, `snapcraft` with a Flutter extension | [unverified: not fetched this session; recall] |

---

## 3. Privacy and legal (France / EU)

| Topic | What applies | Grade |
|---|---|---|
| GDPR | Tasks, reminders, calendar events and voice notes are personal data, and can be special-category data (health appointments, religion). This calls for a lawful basis (contract), data minimisation, export (Art. 20 portability), erasure (Art. 17), a privacy policy, processor DPAs (Art. 28) and records of processing | [unverified: GDPR articles from recall, not fetched; they are standard] |
| Lowest-risk architecture | **Local-first** (SQLite on device) + **end-to-end-encrypted sync** means the operator cannot read the content, which greatly reduces the GDPR scope and breach impact | [inferred] |
| AI processing | Sending task text to an LLM makes the provider a processor. Anthropic includes a DPA and does not train on API content; Gemini **paid** tier is not used for training, and the free tier is forbidden for EEA users; OpenAI is not verified this session. Make AI **opt-in per feature** and disclose it | [verified: Anthropic + Gemini terms above; OpenAI: unverified] |
| Transfers outside the EU | US providers rely on the EU-US Data Privacy Framework or SCCs; consider EU-region endpoints (OpenAI regional processing costs +10%) | [verified: the OpenAI +10% regional uplift on the pricing page; the DPF status is unverified recall] |
| **Google OAuth: Calendar** | Calendar read access is given by Google as an example of a **sensitive** scope (not restricted). Sensitive-scope verification means brand verification, a privacy policy on a verified domain, a per-scope justification and a **YouTube demo video**, and takes **up to 10 days**. **No CASA security assessment** is required for sensitive scopes | [verified: https://developers.google.com/identity/protocols/oauth2/production-readiness/sensitive-scope-verification] |
| CASA | Required only for **restricted** scopes (Gmail, full Drive, etc.) whose data is accessed from or through a third-party server; reassessment *"at least every 12 months"*; third-party labs charge roughly $540 to $4,500 a year | [verified: https://developers.google.com/identity/protocols/oauth2/production-readiness/restricted-scope-verification ; the cost comes from secondary sources: https://aiemaily.com/blog/google-casa-verification-explained-for-buyers] |
| How to avoid CASA | Do **not** request Gmail scopes (no "email to task" via the Gmail API; use a forwarding address or the share sheet instead) | [inferred] |
| Personal use before verification | An unverified app in "testing" mode works for a small user cap (100 is the known figure); refresh tokens in testing mode expire after ~7 days | [unverified: the cap number and token expiry are recall; the page only says "a user cap restricts..."] |
| Microsoft 365 (Graph) | Publisher verification is free; unverified multi-tenant apps show "Unverified" and can be blocked by tenant admin policy; publisher verification needs a Microsoft partner (MPN/AI Cloud Partner Program) ID | [verified: free + "Unverified" label via search, https://learn.microsoft.com/entra/identity-platform/application-consent-experience]. MPN requirement: [unverified: recall] |
| iCloud | CalDAV with an app-specific password (no OAuth for third parties) | [unverified: recall] |
| Data retention policy (proposed) | Completed tasks stay locally indefinitely (user's choice); server-side ciphertext is kept until account deletion + 30 days; AI requests are not logged by us; crash logs are scrubbed and kept 90 days | [inferred: design proposal] |

---

## 4. Top technical risks for a reminder app, and mitigations

| # | Risk | Evidence | Mitigation | Grade |
|---|---|---|---|---|
| 1 | **Missed or late reminders (Doze, standby buckets)** | Restricted bucket: *"One alarm per day, either an exact alarm or an inexact alarm"*; Rare: 1 per hour; Frequent: 2 per hour; Working set: 10 per hour | Use `setAlarmClock()` for user-facing reminders (*"the system never adjusts their delivery time... leaves low-power modes if necessary"*), or `setExactAndAllowWhileIdle()`; ask the user to exempt the app from battery optimisation; keep the app out of Restricted through regular use and its widget | [verified: https://developer.android.com/topic/performance/power/power-details ; https://developer.android.com/develop/background-work/services/alarms/schedule] |
| 2 | **Alarms lost on reboot, update, permission revoke or time change** | *"all alarms are canceled when a device shuts down"*; revoking `SCHEDULE_EXACT_ALARM` cancels all future exact alarms | Single source of truth in the DB; a `reschedule-all` routine on BOOT_COMPLETED, MY_PACKAGE_REPLACED, TIME_SET, TIMEZONE_CHANGED and `ACTION_SCHEDULE_EXACT_ALARM_PERMISSION_STATE_CHANGED`; schedule only the next N alarms (a rolling window) | [verified: same alarms page; the extra broadcasts are inferred] |
| 3 | **OEM battery killers** | dontkillmyapp ranks Huawei, Xiaomi, OnePlus and Samsung worst (5/5); one report shows OnePlus Android 16 turning exact alarms into inexact ones, while setAlarmClock was not delayed | Per-OEM onboarding guides (deep links to autostart / battery settings), `setAlarmClock` for critical reminders, an in-app "reminder health" self-test, and a delivery log that detects late fires | [verified: https://dontkillmyapp.com/ ; https://github.com/juev/lists/pull/261 (single-device report)] |
| 4 | **Android 16+ job quotas** | Android 16 limits jobs even under a foreground service, by standby bucket | Don't rely on WorkManager for timing-critical work; use it only for sync | [verified: power-details page] |
| 5 | **Timezone / DST bugs** | the classic failures: a 02:30 local time that doesn't exist (spring forward) or happens twice (fall back); travel; floating vs fixed times | Store a **wall time + IANA zone** for local-time tasks and UTC for absolute events; model "floating" tasks explicitly (e.g. "take pills 08:00 wherever I am"); test with fake clocks across DST transitions in Europe/Paris, America/New_York and Australia/Lord_Howe (30-minute DST); keep tzdata updated | [inferred: domain knowledge; RFC 5545 floating vs zoned time is recall] |
| 6 | **Recurrence edge cases** | the 31st of each month, Feb 29, "last weekday", completing a recurring task early or late (repeat from due date vs from completion), editing "this and future", EXDATE | Use RFC 5545 RRULE + an established library (rrule.js, dart `rrule`, ical4j / lib-recur) behind a property-test suite; separate "series" from "instance" overrides; offer both "repeat from completion" and "repeat on schedule" | [inferred; library maturity unverified] |
| 7 | **Sync conflicts (offline phone + PC + calendars)** | concurrent edits, deletes vs edits, clock skew | Local-first with a CRDT or a per-field LWW using hybrid logical clocks; tombstones; an idempotent op log; for CalDAV/Graph/Google use ETags/sync-tokens and treat the remote calendar as authoritative for events, the app for tasks | [inferred] |
| 8 | **Two-way calendar mapping loss** | VTODO support differs: Google has no VTODO in Calendar (Google Tasks is a separate API); Outlook uses To Do (Graph `todoTask`); iCloud/Nextcloud support VTODO via CalDAV | Map tasks to native task APIs where they exist, and show time-blocked tasks as events; keep app-only fields in your own store, keyed by remote UID | [unverified: API capabilities from recall; to verify in the integrations dive] |
| 9 | **Wayland global shortcut** | GNOME 48 *"fully supports"* the XDG GlobalShortcuts portal, but there are bug reports (Chromium disabled it on GNOME in M140/141; repeating setup popups) | Use the portal (`org.freedesktop.portal.GlobalShortcuts`) with a fallback: tell the user to bind a GNOME custom keyboard shortcut to `app --quick-add` (CLI/D-Bus activation); tray via the AppIndicator extension / StatusNotifierItem (GNOME needs an extension) | [verified: https://release.gnome.org/48/developers/index.html , https://issues.chromium.org/issues/404298968]. The tray-extension need is unverified recall |
| 10 | **Notification permission and channels** | Android 13+ requires runtime POST_NOTIFICATIONS; users can silence channels | Onboarding check + a health screen; separate channels for reminders, digests and sync | [unverified: recall, standard] |
| 11 | **API key leak via the public repo / APK** | | Proxy + App Check / per-user auth; secret scanning in CI | [inferred] |
| 12 | **Play policy rejection over exact alarms** | the policy wording in section 2 | Ship `SCHEDULE_EXACT_ALARM` first; add `USE_EXACT_ALARM` only with a declaration and a demo video if the calendar features justify it | [inferred from verified policy] |

---

## 5. Monetisation for later: competitor price anchors

| Competitor | Model | Price | Grade |
|---|---|---|---|
| Todoist | freemium + sub | Pro **$5/mo billed annually ($60/yr)** or $7 month-to-month; Business $8/user/mo annual; the free tier has 5 projects | [verified: https://todoist.com/pricing] |
| TickTick | freemium + sub | Premium **$49.99/yr** on the official page (third parties still quote $35.99, which is probably old or regional) | [verified: https://ticktick.com/about/upgrade] |
| Things 3 | one-time, per platform (Apple only) | ~$49.99 Mac, $19.99 iPad, $9.99 iPhone | [unverified: third-party listing, https://toolradar.com/tools/things-3/pricing] |
| Todoist AI ("Ramble", Task Assist) | AI features gated to Pro | | [verified: Todoist pricing page] |

**Options:**
- **(A) Free + open source + optional "sync & AI" subscription (~€3 to €4/mo or €30 to €40/yr).** The subscription covers real costs (sync server + LLM ≈ $0.4 to $1.2 per user per month by §1.3). This is the best fit for a public GitHub repo.
- **(B) One-time "Pro" unlock (~€15 to €25)** for local features plus BYOK AI. Low churn risk, but there is no recurring revenue to pay for the server.
- **(C) Hybrid:** a one-time unlock for local features, plus a sub only for hosted sync and AI.

Play Billing and App Store take 15% on the first $1M under small-business programmes [unverified: recall]. An open-source repo does not prevent selling the store build or hosted services [inferred]. **Recommendation: (C), priced under TickTick (€29.99/yr) to undercut both anchors.** [inferred, Speculative on price]

---

## 6. Bottom line

- AI is **cheap**: cents per user per month with Haiku 5.5 / gpt-6-luna / gpt-5-nano. The design constraints are privacy (opt-in, a proxy, no keys in the client, no free Gemini tier in the EU) and reliability (a deterministic parser first, schema validation, user confirmation). [inferred from verified prices/terms]
- Publishing costs are low: $25 for Play (+ 12 testers for 14 days), $99/yr for Apple later, Flathub free but build-from-source. Android developer verification does not touch a personal ADB install, and reaches France only in 2027. [verified, see section 2]
- Google Calendar is a **sensitive** scope with no CASA requirement. **Avoid Gmail scopes** to stay out of CASA entirely. [verified]
- The #1 product risk is **reminder delivery on Android OEMs**, so the reminder engine must be built natively with `setAlarmClock`, reschedule-on-everything and a health self-test, whatever the cross-platform framework. [verified mechanisms + inferred design]
