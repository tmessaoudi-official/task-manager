# Holdtask — name availability evidence (2026-11-01)

## Stores
- Google Play: fetched play.google.com/store/search?q=Holdtask&c=apps → results Chore Focus, TodoTask, TaskLock, MeisterTask (list cut off); no "Holdtask". Web search: no Holdtask listing (HOLD Hungarian investment app, HOLD trainerize app only).
- Apple: iTunes Search API term=holdtask (US) → 9 results, none named Holdtask (GoodTask, Task Flow, Super Productivity…); term="hold task" (FR) → nearest is "Hold My Thought: To Do List" (Newton Ameh). Web search site:apps.apple.com: none.
- F-Droid API packages/holdtask → 404. Snapcraft info/holdtask → resource-not-found. Flathub search → no holdtask hit (generic "Tasks" results).
- Product Hunt: search page 403 to curl; web search finds only "HOLD X 2.0" (Chrome to-do/focus extension, 2021).
- Nearby productivity names: Hold X (Asana-integrated Chrome extension), Hold My Thought (task manager, Android/iOS/web, holdmythought.com), Hold (phone-distraction app). Similar, not identical.

## Trademarks
- USPTO: queried tmsearch backend API (prod-stage-v1-0-0/tmsearch) → "holdtask" 0 hits, "hold task" 0 hits. Positive control works: todoist 3, holdfast 34.
- EUIPO eSearch: curl failed (connection/JS app) — NOT queried.
- TMview API: "Request Rejected" (WAF) — NOT queried.
- WIPO Global Brand Database: 200 but JS-only page, no results data — NOT queried.
- Web search for HOLDTASK trademark: no record; near marks MAINTASK, 24TASK (class 42).

## Domains (RDAP)
- holdtask.app → 404 not found (rdap.org via curl, and pubapi.registry.google) → AVAILABLE.
- holdtask.com → REGISTERED 2026-08-12 via Amazon Registrar, Cloudflare NS, status "client hold", expires 2027-08-12; no DNS A record, site not live.
- holdtask.io → 404 (Identity Digital RDAP) → available.
- getholdtask.app → 404 → available. (getholdtask.com → 404 as well.)
- Note: WebFetch to rdap.org got 403; curl worked.

## Registries
- github.com/holdtask → 404; GitHub repo search "holdtask" → 0.
- pub.dev holdtask → 404. npm holdtask → 404.

## FR/EN
- EN: "hold task" reads clearly as "hold (on to) a task"; "on hold" connotation (task paused) is a mild ambiguity — the opposite of "nag until done".
- FR: "hold-task", both syllables easy; French has no "h" aspiration so "oldtask"; no bad meaning. "tâche" vs "task" fine.

## Verdict: minor-conflicts
No app, package or US trademark named Holdtask. holdtask.com was registered recently (Aug 2026) by an unknown party (parked on client hold) — possible unseen squatter/competitor. EU/WIPO trademark registers not checked.
