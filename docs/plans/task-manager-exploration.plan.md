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
- [2026-11-01 23:17] AGREED: D1 framework validated by user: Flutter 3.47.9 + in-repo Kotlin Android module (KMP runner-up not pursued).
- [2026-11-01 23:17] AGREED: D11 licence validated by user: AGPL-3.0 for the repository.
- [2026-11-01 23:18] AGREED: D10 name: Holdfast acceptable but user objects it does not evoke a task/todo app; requires a verified-unused name - availability research (stores, trademarks, domains, registries) before fixing it.
- [2026-11-01 23:46] AGREED: D10 name chosen by user: Nagdone (app id app.nagdone; domain nagdone.app). Formal EUIPO/INPI clearance and domain/GitHub-org registration remain the user's manual steps.

## Formal Plan
1. Repo setup: git init + local identity/signing (done), README + .gitignore, gh repo create --public, signed initial commit, push.
2. Research workflow: 6 web-sourced researchers (competitors; differentiators; OS integration Android/iOS/Linux/Win/macOS; framework comparison; sync/backend local-first vs BaaS; privacy/AI/policy/cost risks) each pipelined into an adversarial verifier (Rule 9 deprecation lens) → synthesis architect (concept, architecture, effort in hours + calendar weeks, roadmap with personal MVP first, framework decision + why).
3. Write the report: Markdown copy at docs/research/exploration.md + Artifact page.
4. 6C advisor() certification, signed commit + push, present link, stop.

## Status
<!-- progress-block v1 -->
| # | Step | Size | State | Evidence | Files |
|---|------|------|-------|----------|-------|
| 1 | Repo setup + public remote | S | done | 0946d99 | README.md, .gitignore |
| 2 | Research workflow (research → verify → synthesize) | L | done | 0946d99 | - |
| 3 | Report: Markdown + Artifact page | M | done | 0946d99 | docs/research/** |
| 4 | 6C certify, commit, push, present | S | done | - | - |
<!-- /progress-block -->
### Blocked
### Needs input
- D1, D10, D11 decided. Remaining open decisions D2–D9, D12–D14 in docs/research/exploration.md §12 (defaults recommended there).
- User: register nagdone.app + GitHub org; formal EUIPO/INPI trademark check.
### Needs research
### Fragile
### Known issues
