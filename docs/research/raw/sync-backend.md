# Dive: Data, sync and backend architecture

*Researched 2026-11-01. Evidence grades: **[V]** = verified (I fetched or searched a source, URL given) · **[I]** = inferred (reasoned from verified facts, the basis is stated) · **[U]** = unverified (I could not check it, the reason is given). "pub.dev says N days ago" is relative to the fetch date, 2026-11-01.*

---

## 0. TL;DR recommendation

**Phase 1 (personal, about $0/month):** Flutter, with **SQLite on the device (drift) driven by the PowerSync Dart SDK**. Sync goes through the **self-hosted PowerSync Open Edition** container against the user's **existing Postgres 18**. Bucket storage lives in Postgres itself, so no MongoDB is needed. Auth is the **existing Keycloak** (OIDC, passkeys on 26.4+), whose keys PowerSync reads via `jwks_uri`. Writes go through a **small backend API** (an "upload handler") that applies the client's mutation queue to Postgres. **Reminders fire locally** (Android AlarmManager; on Linux, a tray process or a systemd user timer). They never depend on push. Push (**FCM**, plus optional **UnifiedPush**) is only a "wake up and sync" nudge. Calendar sync runs as a **server-side worker** ("calendar bridge") in Docker. Google and Microsoft need HTTPS webhooks, and CalDAV (iCloud, Nextcloud) is polled with sync-tokens.

**Why it grows into a product without a rewrite:** the same client SDK and schema work against **PowerSync Cloud** (Pro from $49/month) or a managed Postgres such as Supabase. You swap configuration, not code. Keycloak can stay as the IdP or be replaced by any JWKS-issuing IdP, Supabase Auth included.

**Runner-up:** `sqlite-sync` (sqlite.ai). It is a CRDT SQLite extension with **both Dart and Kotlin** packages and a Postgres or self-hosted Supabase backend. It ranks second on **track record** (1.x only since September 2026), not on capability.

**Disqualified for this stack:**
- **Electric**: read-path only, and no Dart or Kotlin client.
- **Zero/Replicache**: no Dart or Kotlin; Replicache is in maintenance mode.
- **InstantDB**: sunsetting on 2027-08-31.
- **Triplit**: appears inactive.
- **Firestore**: no Linux support in FlutterFire.
- **Jazz**: no Dart or Kotlin.

---

## 1. Requirements that drive the choice

| Requirement | Consequence |
|---|---|
| Offline-first on Android **and Ubuntu desktop** | Needs an embedded local DB with the full schema, plus a sync engine whose SDK runs on **Flutter Linux**. Many candidates fail on Linux, not on Android. |
| Sync between devices now, multi-user later | Partial replication per user, project or team (buckets or streams), plus an auth model that carries scopes. |
| Exact reminders | Must be scheduled **locally on every device**. Push is best-effort and can be dropped by Doze or OEMs. |
| Two-way calendar sync (Google, Outlook, iCloud, Nextcloud) | Google and Microsoft change notifications need a publicly reachable HTTPS webhook, so that part is server-side by nature. |
| Public GitHub repo, solo dev + AI agent | Prefer open or source-available components that run in Docker locally, have one schema language (SQL), and keep custom sync code small. |
| Later product | Hosted option with predictable pricing, and no lock-in to a dying vendor. |

---

## 2. Candidate matrix: Dart/Flutter and Kotlin SDK reality check

| Option | Model | Official Dart/Flutter SDK (Linux?) | Kotlin SDK | Status 2026 | Grade / source |
|---|---|---|---|---|---|
| **PowerSync** | Server-authoritative sync: Postgres/Mongo/MySQL/MSSQL → client SQLite; writes via your API | **Yes**. `powersync` 2.5.0, publisher powersync.com, Android/iOS/**Linux**/macOS/Windows/web (web beta). Apache-2.0 SDK. v2.0 (Apr 2026) merged packages. | **Yes**. powersync-kotlin (KMP); extension bundled for Android **and JVM**, Apple via SPM. Kotlin v1.14.1 on 2026-07-29 (Room 3.0 integration). | Active, monthly releases | [V] Dart platforms + Kotlin Android/JVM/Apple targets: https://pub.dev/packages/powersync · https://releases.powersync.com/ · https://github.com/powersync-ja/powersync-kotlin. **[I]** Kotlin on *Linux desktop* via the JVM target is inferred from "bundled for Android and JVM"; the README does not name Linux. |
| **sqlite-sync (sqlite.ai)** | CRDT extension inside SQLite; syncs with SQLite Cloud, **Postgres**, Supabase (incl. self-hosted) | **Yes**. `sqlite_sync` 1.2.0, publisher sqlite.ai, Android/iOS/**Linux**/macOS/Windows (no web listed) | **Yes**. `ai.sqlite:sync` on Maven Central | 1.2.0 on 2026-09-28, 1.1.4 on 2026-09-21; Apache-2.0 | [V] https://pub.dev/packages/sqlite_sync · https://github.com/sqliteai/sqlite-sync · https://api.github.com/repos/sqliteai/sqlite-sync/releases |
| **cr-sqlite (vlcn)** | CRDT SQLite extension | No official Dart package found | No | Not archived; last push **2026-08-10**, 56 open issues. (A search summary claimed it was stale since 2024; the GitHub API contradicts that.) Slow cadence, v2 promised. | [V] https://api.github.com/repos/vlcn-io/cr-sqlite · [U] Dart binding: none found by search |
| **Electric (electric.ax)** | **Read-path only** shape sync over HTTP from Postgres | **No**. Official clients are TypeScript and Elixir. Community `electricsql` Dart targets old pre-1.0 Electric and is deprecated. | No | Active, but "Electric does not do write-path sync" | [V] https://electric.ax/docs/guides/client-development · https://electric.ax/docs/guides/writes · https://pub.dev/packages/electricsql |
| **Zero (Rocicorp)** | Server-authoritative query-driven sync, Postgres → zero-cache | **No**. React, SolidJS, React Native only. | No | 1.0 GA (status page says March 2026; InfoQ reported June 2026) | [V] https://zero.rocicorp.dev/docs/status · https://zero.rocicorp.dev/llms.txt · https://www.infoq.com/news/2026/06/zero-version-1/ |
| **Replicache** | Client-side sync framework | No | No | Users steered to Zero [V]; "maintenance mode" [I] (third-party 2026 guide + Rocicorp's 2024 "no new features" stance, read via search summary) | https://rocicorp.dev/blog/retiring-reflect · https://frenchydigital.com/blog/offline-first-mobile-architecture-2026 |
| **InstantDB** | Hosted triple-store sync | Community ports only (`flutter_instantdb`) | No | **"Instant is sunsetting. Services will continue until August 31st, 2027."** | [V] https://www.instantdb.com/docs |
| **Triplit** | Full-stack syncing DB (TS) | No | No | Last push 2026-01-19; YC lists it as Inactive; AGPL | [V] https://api.github.com/repos/aspen-cloud/triplit · https://www.ycombinator.com/companies/triplit |
| **Jazz** | CoValues CRDT + sync cloud | No. TS frameworks + **Rust**; React Native alpha. | No | Active | [V] https://jazz.tools/docs/install/client |
| **Ditto** | P2P + cloud CRDT sync | **Yes**. `ditto_live` 5.1.0. Docs list **Linux Ubuntu 20.04+ x64/arm64**, but the pub.dev 5.x platform table shows Linux as "Planned" (conflict). | [U] Kotlin/Android SDK exists per my recall; not fetched | Proprietary; pricing "contact us"; third-party reports list a Free tier of 10 cloud device connections / 2 GB | [V] https://docs.ditto.live/sdk/latest/compatibility/flutter · https://pub.dev/packages/ditto_live · [U] pricing (official page not fetched; pricingsaas.com) |
| **Couchbase Lite** | Document DB + Sync Gateway replication | Community `cbl` 3.6.0+2 (stable published ~15 months ago), 4.0.0-dev.10 prerelease; Android/iOS/macOS/Windows/**Linux**; CE vs EE split | [U] Official Android/Kotlin SDK exists per my recall; not fetched | Community Dart binding, slow stable cadence | [V] https://pub.dev/packages/cbl |
| **PouchDB/CouchDB** | Multi-master doc replication | No maintained official lib; `foodb` (community, replication), `fluttercouch` discontinued | [U] none checked | Weak in Dart | [V] https://pub.dev/packages/foodb/changelog · https://pub.dev/packages/fluttercouch |
| **Automerge** | JSON CRDT (Rust core, JS) | **No Dart binding found** | [U] none found | Automerge 3.x (JS v3.2.3) | [V] https://automerge.org/docs/migrating-from-automerge-2-to-automerge-3 · [U] Dart/Kotlin absence (search-only) |
| **Yjs** | Text/structure CRDT | Community: `yjs_dart` 1.1.15 (pure Dart, claims Yjs v1/v2 binary compatibility), `y_crdt` 0.2.0 (WASM) | [U] (y-crdt has Rust/JNI bindings, not checked) | Ports are community | [V] https://pub.dev/packages/yjs_dart · https://pub.dev/packages/y_crdt |
| **Supabase** | Postgres BaaS (REST/Realtime/Auth/Storage); **no offline data mode** | `supabase_flutter` 2.18.3 (3.0.0-dev.15 pre), all 6 platforms incl. Linux | `supabase-kt` (community-maintained, BOM `io.github.jan-tennert.supabase`) | Active | [V] https://pub.dev/packages/supabase_flutter · https://github.com/supabase-community/supabase-kt |
| **Firebase / Firestore** | Doc DB with offline cache | `cloud_firestore` 7.1.0: Android/iOS/macOS/web/Windows. **Linux NOT listed.** | Official Android (Kotlin) | Active | [V] https://pub.dev/packages/cloud_firestore |
| **Appwrite** | BaaS (self-host or cloud) | Official Flutter SDK; **no shipped offline mode** (only an old discussion + community `appwrite_offline` 0.0.5 from 2024-12) | [U] Kotlin SDK exists per recall | Active | [V] https://github.com/appwrite/appwrite/discussions/5326 · https://pub.dev/packages/appwrite_offline |
| **PocketBase** | Single Go binary BaaS (SQLite) | `pocketbase` 0.25.3 Dart SDK, all platforms; **no offline mode**; auth store memory-only by default | No official | Server v0.39.x, pre-1.0 | [V] https://pub.dev/packages/pocketbase · https://github.com/pocketbase/pocketbase/blob/master/CHANGELOG_23_39.md |

**What this means for the stack** [I, from the table]: only **PowerSync**, **sqlite-sync** and (proprietary) **Ditto** have first-party offline sync for Dart **and** Kotlin with desktop Linux in scope. That keeps the framework decision (Flutter vs KMP) open without changing the backend. Couchbase Lite is a fourth option on Dart, but only through a community binding.

---

## 3. Deep comparison of the finalists

| Criterion | PowerSync | sqlite-sync | Ditto | Supabase alone (no sync engine) |
|---|---|---|---|---|
| Local store | SQLite (SQL queries, works with drift/Room) | SQLite + CRDT extension | Proprietary store, DQL | You build it yourself (drift + hand-written outbox) |
| Write path | Client upload queue → **your** backend (`uploadData` connector) → Postgres. Server authoritative. [V via search summary, page not fetched] connector pattern https://docs.powersync.com/client-sdk-references/kotlin-multiplatform | CRDT merge (column-level), no custom backend logic needed [V] repo description | CRDT, P2P mesh + cloud | REST calls; offline = DIY |
| Conflict policy | Whatever your backend decides (LWW, field merge, reject + validation) [I from the connector design] | Automatic CRDT (LWW per column per the CRDT design) [I] | Automatic CRDT | DIY |
| Server-side business rules (validation, permissions, reminder fan-out) | **Natural**: every write passes through your API | Harder: merges happen in the extension; rules must be enforced after the fact [I] | Harder [I] | Natural |
| Partial replication / multi-user | Sync Streams / buckets, JWT claims-scoped [V] pricing page lists "Sync Streams" https://www.powersync.com/pricing | Per-table/DB [U] details not fetched | Subscriptions | RLS |
| Self-host | **Open Edition: free, source-available** (not OSI open source), Docker image `journeyapps/powersync-service`; no dashboard [V] https://www.powersync.com/pricing · https://docs.powersync.com/self-hosting/getting-started | Apache-2.0 extension; backend = your Postgres/Supabase [V] | No (cloud/enterprise) [I] | Yes (Docker) [U] not fetched |
| Postgres prerequisites | PG ≥ 11, `wal_level=logical`, publication **named `powersync`**, role `REPLICATION BYPASSRLS` with SELECT; PG ≥ 14 lets source + bucket storage share one server [V] https://docs.powersync.com/configuration/source-db/setup · https://docs.powersync.com/self-hosting/installation/powersync-service-setup | [U] not fetched | n/a | n/a |
| Bucket storage | MongoDB (replica set) **or Postgres** [V] same setup page. Caveat: Postgres storage does **not** support storage version 4 (the version that offloads large bucket data to S3) [V]. Whether Sync Streams or any other feature this design relies on needs v4: [U] not checked. | n/a | n/a | n/a |
| Auth | Any JWT: `jwks_uri`, inline JWKS, `supabase: true`; asymmetric keys recommended [V] same setup page | [U] | Ditto auth webhook [U] | Supabase Auth |
| Maturity | Years in production, Dart SDK v2 | 1.x since Sep 2026 [V] release dates | Mature, commercial | Mature |
| Lock-in risk | Medium-low: data stays in **your Postgres**; client is plain SQLite [I] | Low-medium [I] | High (proprietary format + pricing) [I] | Low |

**Verdict** [I]: PowerSync fits a task manager best. Reminders, recurrence expansion, calendar bridging and later sharing permissions all want **server-authoritative writes with business logic**. Plain SQL on both ends also suits an AI coding agent. Two caveats:
- **PowerSync Open Edition is source-available, not open source.** That is fine for self-hosting in a public repo, but a fork or competitor-hosting scenario must read its licence. [V] "free, source-available": https://www.powersync.com/pricing. Licence text not read: [U].
- Rich-text notes could later embed a **Yjs document blob per task description** (`yjs_dart`), synced as a bytes column, if real-time collaborative editing is wanted. [I]

---

## 4. Cost

### 4.1 Personal scale (1 user, 2–3 devices)

| Setup | Monthly cost | Notes |
|---|---|---|
| Self-hosted PowerSync OE + existing Postgres 18 + Keycloak in the user's Docker | **$0 software** | Needs the server reachable from the phone off-LAN. A tunnel or VPN (Tailscale, Cloudflare Tunnel, WireGuard) or a small VPS. [U] tunnel/VPS pricing not fetched. Home server down = phone stays fully usable offline and syncs later [I from offline-first design]. |
| PowerSync Cloud Free + Supabase Free | **$0** | PowerSync Free: 2 GB synced/month, 500 MB hosted, 50 peak concurrent clients; **deactivates after inactivity** (the page says 1 week in one place and 2 weeks in another). Supabase Free: 500 MB DB, 50k MAU, **pauses after 1 week of inactivity**, 2 projects. Daily personal use avoids both pauses [I]. [V] https://www.powersync.com/pricing · https://supabase.com/pricing |
| FCM | $0 | "No-cost" [V] https://firebase.google.com/pricing |

### 4.2 10k users (assumptions stated)

Assumptions [I]: 10k registered users; **peak concurrent connected clients ≈ 15% = 1,500**; ~500 tasks/user × ~1 KB ≈ 0.5–1 MB hosted per user; ~3 MB synced/user/month; ~50 writes/user/day → ~15M writes/month.

| Stack | Derivation | ≈ Monthly |
|---|---|---|
| **PowerSync Cloud Pro + Supabase Pro (DB + Auth)** | PowerSync Pro $49 incl. 30 GB synced (need ~30 GB → ~$0–5 over), 10 GB hosted (need ~5–10 GB), 1,000 concurrent incl. + $30 per extra 1,000 → +$30. Supabase Pro $25 incl. 100k MAU, 8 GB disk, $10 compute credit (a larger compute add-on is likely: [U] compute prices not fetched). | **~$105–$150** + compute add-on [I from https://www.powersync.com/pricing, https://supabase.com/pricing] |
| **Self-hosted PowerSync OE + Postgres + Keycloak on a VPS** | Software $0; one or two mid VPS + backups | **~$20–80** [U] VPS prices not fetched; ops time is the real cost |
| **Electric Cloud (for comparison only, disqualified)** | 15M writes × ($1 + $2 Postgres Sync)/1M = $45; usage under $5 waived; reads free | ~$45 + Postgres [I from https://electric.ax/pricing] |
| **Firebase Firestore (disqualified by Linux)** | Spark free: 50k reads/day, 20k writes/day, 1 GiB. 10k users exceed it; Blaze per-op prices not on the page | [U] Blaze rates not fetched [V free quotas https://firebase.google.com/pricing] |
| **Supabase Auth alone at 10k MAU** | Within Pro's 100k MAU; overage $0.00325/MAU | included [V] https://supabase.com/pricing |
| **Appwrite Cloud** | Official table not scraped; third-party pages disagree ($15 vs $25 Pro; 75k MAU free; free projects pause after 7 days since Feb 2026) | [U] https://appwrite.io/pricing content not retrieved |

**Takeaway** [I]: on any viable stack, sync infrastructure for a 10k-user task app stays roughly **$100–200/month**. Cost does not decide between them; SDK fit and write semantics do.

---

## 5. Data model (Postgres = source of truth, mirrored to SQLite)

Principles [I unless noted]:
- **Client-generated IDs**: UUIDv7 (time-ordered, index-friendly). Postgres 18 has a built-in `uuidv7()` [V, secondary sources: https://www.dbvis.com/thetable/uuidv7-in-postgresql-18-what-you-need-to-know/], but IDs must be minted **on the device** while offline, so generate v7 in Dart/Kotlin. [U] I did not verify a Dart `uuid` package with v7 support.
- Every row carries `owner_id`, `created_at`, `updated_at` (server-assigned on apply), `deleted_at` (**soft delete / tombstone**, required for sync), `version` (integer, bumped by the server), plus `origin_device_id`.
- All instants are stored **UTC (`timestamptz`)**. "Wall-clock" intents (e.g. "every day at 09:00 Europe/Paris") are stored as **local datetime + IANA TZID**, never as UTC, so DST shifts are handled correctly when occurrences are expanded.

```
users(id, display_name, default_tz, ...)
projects(id, owner_id, name, color, parent_id NULL, sort_key, archived_at, ...)       -- hierarchy optional
project_members(project_id, user_id, role)                                             -- phase 3 sharing
tasks(id, project_id, parent_task_id NULL,            -- subtasks = self-reference (depth-limited in UI)
      title, notes, notes_ydoc BYTEA NULL,            -- optional Yjs blob later
      status, priority, sort_key (fractional index),
      due_kind ENUM(none,date,datetime),              -- all-day vs timed
      due_date DATE NULL, due_local TIMESTAMP NULL, due_tz TEXT NULL,
      start_local/start_tz (optional, for calendar time-blocking),
      rrule TEXT NULL,                                -- RFC 5545 RRULE, e.g. FREQ=WEEKLY;BYDAY=MO,WE
      rrule_dtstart_local, rrule_tz,                  -- DTSTART;TZID=
      recurrence_mode ENUM(fixed,after_completion),   -- "every 3 days after done" is NOT expressible in RRULE
      completed_at, ...)
task_occurrence_overrides(task_id, recurrence_id_local, -- = RFC 5545 RECURRENCE-ID
      status(done|skipped|moved), moved_to_local, completed_at)   -- = EXDATE / overridden instance
reminders(id, task_id, kind ENUM(absolute,relative,location),
      fire_at_utc NULL,                                -- absolute
      offset_minutes NULL, anchor ENUM(due,start),     -- relative (VALARM TRIGGER;RELATED=)
      lat, lon, radius_m, on_enter BOOL,               -- geofence (later)
      channel_mask (push|local|email), snoozed_until_utc, dismissed_at)
reminder_deliveries(reminder_id, occurrence_local, device_id, fired_at, acted)  -- dedupe across devices
tags(id, owner_id, name, color) ; task_tags(task_id, tag_id)
attachments(id, task_id, blob_key, mime, size, sha256, uploaded_at)   -- metadata syncs; bytes go to S3/MinIO, lazy download
calendar_accounts(id, user_id, provider, auth_ref, sync_state JSONB)  -- server-only, never synced to clients
calendar_links(task_id, account_id, remote_calendar_id, remote_uid, remote_etag, last_pushed_version)
devices(id, user_id, platform, push_kind(fcm|unifiedpush|none), push_token_enc, last_seen)
```

Why **reminders are separate entities** [I]:
- A task can carry N reminders, each absolute, relative or (later) location-based. Each has its own snooze state and per-device delivery dedupe.
- They map 1:1 to iCalendar `VALARM` components for calendar round-tripping.
- Each device's scheduler can be **rebuilt from the table** after a sync, boot or timezone change.

**Recurrence engine** [I + V]:
- Store the RRULE string (portable to CalDAV, Google and Graph) and expand occurrences on the device in a rolling window, e.g. the next 60 days, for alarms and views.
- The Dart `rrule` package (0.2.18) handles RFC 5545 strings and jCal, but **does not support WKST** (Monday only), is a **partial port of rrule.js**, and was last published about 9 months ago. [V] https://pub.dev/packages/rrule
- So: (a) wrap it behind an interface; (b) build an RFC 5545 conformance test corpus (the python-dateutil / rrule.js test vectors); (c) be prepared to fork, or implement a pure Dart expander, for WKST and BYSETPOS edge cases. Round-trip fidelity with Google and Outlook matters, since a lossy RRULE edit can corrupt a user's calendar series. [I]
- "After completion" recurrence (Todoist-style) is not RRULE-expressible. Model it as a separate mode; when it is exported to a calendar, write only the next instance. [I]

**Timezones** [I]:
- On a timezone change, Android broadcasts `ACTION_TIMEZONE_CHANGED`. Reschedule every wall-clock reminder, but keep absolute (`fire_at_utc`) reminders fixed. [U] I did not fetch the broadcast's current docs.
- Ship IANA tzdata with the app (Dart `timezone` package) so rules do not depend on OS tzdata. [U] package not fetched.

---

## 6. Conflict resolution

| Data | Strategy (PowerSync, server-authoritative) |
|---|---|
| Scalar task fields (title, due, priority, status) | **Field-level last-writer-wins**. The client uploads a *patch* (only the changed fields + base `version`); the server applies it per field. Two devices editing different fields both win. Same field: the later server-received write wins, and the server bumps `version`. [I] |
| Completion vs edit race | Domain rule: `completed` beats a concurrent title edit (both are applied); `deleted` beats edits unless restored. Recurring completion writes an **override row**, never mutating the series → no conflict. [I] |
| Ordering | Fractional index (`sort_key` strings) → concurrent reorders never collide; ties broken by id. [I] |
| Subtask tree moves | Server validates no cycles; rejects → the client's local change is reverted on next sync (PowerSync replaces the local view with the server state). [I from PowerSync's server-authoritative model] |
| Notes / long text | Phase 1 LWW; later a Yjs doc per task (`yjs_dart`) merged server-side or as a CRDT blob. [I] |
| Calendar (external) vs app edits | Compare `remote_etag` / `last_pushed_version`. If both sides changed, take the **app** for task-owned fields, the **remote** for event times when the event originated in the calendar, and log the conflict to a `sync_conflicts` table shown in the UI. [I] |

---

## 7. End-to-end encryption options

| Option | What it protects | What it breaks | Fit |
|---|---|---|---|
| None (TLS + DB-at-rest) | Transport only | Nothing | Phase 1 [I] |
| **Column-level E2EE**: encrypt `title`/`notes`/attachments client-side with libsodium XChaCha20-Poly1305 (`sodium` 4.1.2+1 supports Android + Linux [V] https://pub.dev/packages/sodium); key per user, wrapped per device (X25519) | Content invisible to the server | **Server-side features that read content**: calendar bridge (it must write titles to Google/Outlook), server-generated push text, server search, AI features. Timestamps stay plaintext so the server can still schedule push. | Product "privacy mode" per project [I] |
| Full E2EE (CRDT on encrypted ops, e.g. Jazz-style) | Everything | Server-side business rules; calendar sync impossible server-side | Poor fit for a calendar-integrated app [I] |
| Local DB encryption (SQLCipher) | Device theft | Slight performance cost | PowerSync v2 **removed** the separate `powersync_sqlcipher` package during consolidation [V, search summary of https://releases.powersync.com/announcements/powersync-dart-flutter-sdk-v2-0]. [U] whether SQLCipher is now a flag inside `powersync`; check before relying on it. |

Recommendation [I]:
- Design the schema now so `title`, `notes` and attachment bytes can later hold `enc:v1:<ciphertext>`.
- Offer E2EE per project; calendar-linked projects stay non-E2EE, and the UI says so honestly.
- Multi-device key enrolment: QR code from an existing device, or a recovery passphrase via Argon2id (libsodium). [I]

---

## 8. Auth

| Option | Status | Fit |
|---|---|---|
| **Keycloak (already running)** | Passkeys **officially supported since 26.4** (not on by default; enabled in the WebAuthn Passwordless Policy); 26.7.0 released 2026-07 with passkey UX fixes [V] https://www.keycloak.org/2026/07/keycloak-2670-released · https://www.keycloak.org/2025/09/keycloak-2640-released | **Phase 1–2.** OIDC + PKCE from the app; PowerSync verifies via `jwks_uri` [V setup page]; Google/Microsoft "social login" federation built in [U] not re-fetched. Ops-heavy at product scale. |
| **Supabase Auth** | Passkeys **beta since 2026-05-28**, all projects; JS SDK support in a v3 pre-release; reported rough edges; 5-origin limit (third-party) [V] https://supabase.com/changelog/46458-passkeys-for-supabase-auth-beta | Product option; PowerSync has `supabase: true` [V]. Flutter passkey support not verified [U]. |
| Firebase Auth | 50k MAU no-cost [V] https://firebase.google.com/pricing | Linux Flutter support for Firestore absent [V]; Auth on Linux [U] |
| Phase-0 shortcut | A single user, static RS256 key pair, JWT minted by the backend | Fine for week-1 dev only [I] |

Native passkeys on Android go through Credential Manager. On Linux desktop, passkeys depend on the browser, so use the system-browser OIDC flow: a loopback redirect or a custom URI scheme. [I] [U] There is no verified Flutter Linux passkey plugin.

**Calendar OAuth is separate from app login**:
- Google Calendar scopes (`calendar.events`) are **sensitive**, needing brand + scope verification with a video. Third-party guides say **no CASA** assessment (the restricted tier is Gmail/Drive-class); one source disagrees. [V] https://developers.google.com/identity/protocols/oauth2/production-readiness/sensitive-scope-verification · https://developer.nylas.com/docs/provider-guides/google/google-verification-security-assessment-guide/. [U] I did not read Google's official restricted-scope list.
- For personal use, an unverified app in "testing" mode works for the owner. [U] I did not check the 7-day refresh-token expiry for testing-mode apps; it is a known gotcha from recall.

---

## 9. Push and reminder delivery (the critical design constraint)

**Rule: a reminder must fire with zero network.** Push is only used to *wake a device so it syncs*, so newly created or edited reminders from another device get scheduled. [I]

| Platform | Firing mechanism | Evidence |
|---|---|---|
| Android | AlarmManager exact alarms via `flutter_local_notifications` (22.3.3) `zonedSchedule`. Android 14+: `SCHEDULE_EXACT_ALARM` is **denied by default**, so the user grants it in-app. `USE_EXACT_ALARM` is auto-granted but restricted by Play policy to apps whose *core* function is alarms/timers/**calendar**; reports say Play rules on reminder apps case by case. Needs `RECEIVE_BOOT_COMPLETED` to reschedule. **Samsung caps scheduled alarms at 500** → schedule a rolling window, not all future occurrences. OEM battery killers (Xiaomi/Huawei) can still break it. | [V] https://pub.dev/packages/flutter_local_notifications · https://developer.android.com/about/versions/14/changes/schedule-exact-alarms · https://support.google.com/googleplay/android-developer/answer/16558241 |
| Ubuntu (GNOME/Wayland) | `flutter_local_notifications` on Linux **does not support scheduled notifications** ("lack of a scheduler API"). So: a long-running **tray/background process** with in-process timers, re-armed after resume (suspend skews timers), **plus** an optional **systemd user timer** or a small headless helper so reminders fire even when the GUI is closed. Display via freedesktop notifications. | [V] https://pub.dev/packages/flutter_local_notifications · [I] systemd approach |
| Wake-for-sync | **FCM** data messages (free) for Google-services phones; **UnifiedPush** (`unifiedpush` 6.2.0, Android, optional Linux via `unifiedpush_linux`, last published ~12 months ago) for de-Googled phones, with a self-hostable distributor such as ntfy [U ntfy not fetched]. Desktop keeps the PowerSync streaming connection open while running → no push needed. | [V] https://firebase.google.com/pricing · https://pub.dev/packages/unifiedpush |

**Dedup across devices** [I]: when a reminder fires on one device and the user acts on it (done/snooze), the action syncs. Other devices cancel their pending alarm on the next sync or push. A `reminder_deliveries` row prevents double "done" effects. Some double-buzz across devices is unavoidable offline; let the user choose "notify on phone only / all devices".

---

## 10. Calendar sync: server-side vs on-device

| Provider | Protocol | Change detection | Where it must live | Evidence |
|---|---|---|---|---|
| Google Calendar | REST v3 (+ Tasks API) | `events.watch` push channels to an **HTTPS webhook**; channels expire and **must be re-created manually** (no auto-renew); `syncToken` incremental sync | **Server** (webhook) or on-device polling as a fallback | [V] https://developers.google.com/calendar/api/guides/push · [U] max channel lifetime not in the fetched snippet |
| Outlook / M365 | Microsoft Graph | Subscriptions on `event`: **max 10,080 min (<7 days)**, 1,440 min with resource data; `todoTask` 4,230 min; HTTPS webhook + lifecycle notifications; delta queries | **Server** | [V] https://learn.microsoft.com/en-us/graph/api/resources/subscription?view=graph-rest-1.0 |
| iCloud | CalDAV (RFC 4791) at `caldav.icloud.com`, **app-specific password** only (revoked when the Apple ID password changes); PROPFIND discovery, partition hosts `pXX-caldav` | Polling (ctag/etag, sync-collection) | Either; server keeps one credential | [V] https://cli.nylas.com/guides/icloud-caldav-settings · https://www.onecal.io/blog/how-to-integrate-icloud-calendar-api-into-your-app |
| Nextcloud | CalDAV (+ VTODO natively) | Polling with sync-token (RFC 6578) | Either | [U] Nextcloud docs not fetched; standard CalDAV |

**Recommendation: a server-side "calendar bridge" worker** (Dart or Kotlin/JVM service in Docker) [I]:
1. Webhooks (Google, Graph) are server-only by nature, and renewal jobs (Graph every <7 days) need an always-on scheduler.
2. One implementation instead of per-platform code, and OAuth refresh tokens live in one hardened place (encrypted, never synced to devices). That is good for a product too.
3. The bridge writes into Postgres, and PowerSync fans changes out to every device. Offline devices see calendar changes on the next sync.
4. Map tasks to **VTODO** where the provider supports it (CalDAV/Nextcloud, optionally Google Tasks or MS To Do). Map time-blocked tasks to **VEVENT** with `VALARM` mirrored from `reminders`.

**On-device read-only alternative** [I]: Android's `CalendarContract` would show the phone's synced calendars locally without any server. This is useful for the "today" view if the bridge is down, but it does not exist on Linux, so it stays an optional add-on.

---

## 11. Recommended architecture

```
 ┌───────────────────── Devices (Flutter; KMP-compatible SDKs exist too) ─────────────────────┐
 │ UI ─ Repository ─ drift/SQLite (PowerSync-managed) ─ Upload queue ─┐                         │
 │ Recurrence expander (rrule + tests) → Local scheduler (Android exact alarms │ Linux tray+timer)│
 │ FCM / UnifiedPush receiver → trigger sync → re-arm alarms                                    │
 └──────────────────────────────────────────────┬───────────────────────────────────────────────┘
          JWT (OIDC/PKCE)        sync stream     │ uploadData (patches)
 ┌──────────┐            ┌────────────────────┐  ▼
 │ Keycloak │──JWKS────▶ │ PowerSync service  │  ┌──────────────────────┐
 └──────────┘            │ (OE, Docker)       │  │ App API (Dart shelf / │── push: FCM / UnifiedPush
                         └─────────┬──────────┘  │ Kotlin Ktor)          │
                     logical repl  │ bucket store│ validation, perms,    │
                                   ▼ (same PG)   │ conflict rules        │
                         ┌────────────────────────┴──────────────────────┐
                         │ Postgres 18 (wal_level=logical, pub "powersync")│
                         └────────────────────────┬──────────────────────┘
                                                  │
                          Calendar bridge worker (Google/Graph webhooks, CalDAV polling)
                          + MinIO/S3 for attachments
```

### Phase 1: personal (weeks 1–N), $0
- Postgres 18 (existing container): `ALTER SYSTEM SET wal_level = logical`, restart, then `CREATE PUBLICATION powersync FOR TABLE ...` and a `powersync_role` with REPLICATION BYPASSRLS. [V setup page]
- PowerSync OE container with **Postgres bucket storage** (PG ≥ 14 can share the server). [V]
- Keycloak realm with passkeys enabled; app client with PKCE. [V Keycloak 26.4+]
- App API: thin service (Dart `shelf` keeps one language with Flutter [I]) applying patches.
- Reachability: tunnel or VPN. [U]
- Calendar: start with the CalDAV bridge (Nextcloud/iCloud) because it needs no OAuth verification; Google in "testing" mode for the owner. [I]

### Phase 2: multi-device polish / beta testers
- Sync Streams per user/project; RLS-equivalent checks in the API; FCM + UnifiedPush; Graph subscriptions; attachments on MinIO. [I]

### Phase 3: product
- Move Postgres to a managed service (Supabase or another) and PowerSync to **Cloud Pro** (or keep self-hosted). The client code does not change, because the endpoint and JWKS are configuration. [I from §3]
- Sharing via `project_members` + stream parameters; per-project E2EE option; Google OAuth verification; optional move from Keycloak to a managed IdP. [I]

### Exit ramps (avoiding lock-in) [I]
- Data is always in your Postgres and the client is plain SQLite.
- If PowerSync's licence or pricing changes, the fallbacks are: (a) sqlite-sync on the same Postgres; (b) a hand-rolled outbox + `updated_at`-cursor pull API (drift) for the data model above, about 2–4 weeks of work for a solo dev + AI [I/speculative estimate].

---

## 12. Risks

| Risk | Severity | Mitigation |
|---|---|---|
| Exact alarms denied / Play rejects `USE_EXACT_ALARM` | High | `SCHEDULE_EXACT_ALARM` + onboarding prompt; detect revocation and fall back to inexact alarms with a warning. [V/I] |
| Linux has no OS scheduler in the notification plugin | High | Tray process + systemd user timer helper. [V/I] |
| `rrule` gaps (WKST, partial port) corrupt calendar series | Medium-high | Conformance corpus, wrapper, fork-ready. [V/I] |
| PowerSync source-available licence / vendor change | Medium | Postgres-centric design; sqlite-sync fallback. [V/I] |
| Graph/Google webhook renewal failures → silent sync stop | Medium | Renewal cron at 50% of lifetime, lifecycle notifications, full re-sync on `410 Gone`/invalid token. [V lifetimes / I] |
| iCloud app-password revocation | Low-medium | Surface "reconnect" in the UI. [V] |
| Home-server outage (phase 1) | Low | Offline-first → only sync pauses. [I] |
| Ditto pub.dev Linux "Planned" contradiction | n/a (not chosen) | — [V] |

## 13. Effort estimate for this dive's scope (solo + Claude Code) [Speculative]
- Backend infra (PG config, PowerSync OE, Keycloak realm, compose): 2–4 days.
- Schema + API upload handler + conflict rules + tests: 1–2 weeks.
- Client sync integration + local scheduler (Android) + Linux tray/timer: 2–3 weeks.
- Recurrence engine + conformance tests: 1–2 weeks.
- Calendar bridge, CalDAV first: 2–3 weeks; Google + Graph with webhooks: +2–3 weeks.
- E2EE per project (later): 2–3 weeks.
