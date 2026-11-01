# task-manager exploration Plan

## Decisions Log
- [2026-11-01 22:10] AGREED: Task is Large, research/design only — deliverable is a report (artifact page + Markdown copy), no app code.
- [2026-11-01 22:10] AGREED: Audience is "me first, product later" — must integrate deeply with daily life (phone, PC, calendar, home-screen widgets, OS reminders/notifications).
- [2026-11-01 22:10] AGREED: Deliverable is a rich Artifact page plus a Markdown copy in this repo.
- [2026-11-01 22:20] AGREED: Phone strategy — cross-platform codebase, Android first; iOS kept ready for a later phase (user uses Android daily).
- [2026-11-01 22:20] AGREED: Calendar integration targets Google Calendar, Outlook/Microsoft 365, Apple/iCloud and Nextcloud/CalDAV, plus phone and desktop notifications.
- [2026-11-01 22:20] AGREED: Desktop strategy — cross-platform native desktop app, Ubuntu (GNOME/Wayland) first; Windows/macOS later from the same code.
- [2026-11-01 22:25] AGREED: Framework — user leans Flutter, but research decides neutrally among Flutter / KMP+Compose / RN-Expo / Tauri 2 and presents the decision with reasons for the user to validate or change.
- [2026-11-01 22:25] AGREED: Certification tier — advisor() at 3C/6C gates.
- [2026-11-01 22:25] AGREED: This directory is its own git repo (master), local identity Takieddine MESSAOUDI <takieddine.messaoudi.official@gmail.com>, signing key 0C537690706EB7F5, commit+tag gpgsign on.
- [2026-11-01 22:25] AGREED: Remote is a PUBLIC GitHub repo tmessaoudi-official/task-manager (free unlimited CI), SSH, plain push of master.

## Formal Plan
1. Repo setup: git init + local identity/signing (done), README + .gitignore, gh repo create --public, signed initial commit, push.
2. Research workflow: 6 web-sourced researchers (competitors; differentiators; OS integration Android/iOS/Linux/Win/macOS; framework comparison; sync/backend local-first vs BaaS; privacy/AI/policy/cost risks) each pipelined into an adversarial verifier (Rule 9 deprecation lens) → synthesis architect (concept, architecture, effort in hours + calendar weeks, roadmap with personal MVP first, framework decision + why).
3. Write the report: Markdown copy at docs/research/exploration.md + Artifact page.
4. 6C advisor() certification, signed commit + push, present link, stop.

## Status
<!-- progress-block v1 -->
| # | Step | Size | State | Evidence | Files |
|---|------|------|-------|----------|-------|
| 1 | Repo setup + public remote | S | done | - | README.md, .gitignore |
| 2 | Research workflow (research → verify → synthesize) | L | done | - | - |
| 3 | Report: Markdown + Artifact page | M | done | - | docs/research/** |
| 4 | 6C certify, commit, push, present | S | todo | - | - |
<!-- /progress-block -->
### Blocked
### Needs input
### Needs research
### Fragile
### Known issues
