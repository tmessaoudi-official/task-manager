# Holdfast (working name)

A local-first task, todo and reminder app that never lets a commitment slip. It is built to integrate deeply with daily life: Android phone, Ubuntu desktop, calendars (Google, Microsoft 365, Nextcloud/CalDAV, iCloud), home-screen widgets and notifications.

**Status: research and design only. No app code yet.**

## Documents

| Document | What it is |
|---|---|
| [`docs/research/exploration.md`](docs/research/exploration.md) | Full exploration report (vision, competitors, feasibility, framework decision, architecture, stack, effort, roadmap, risks, costs, open decisions, 352 sources). Every load-bearing claim carries an evidence grade. |
| [`docs/research/exploration.html`](docs/research/exploration.html) | The same report as a designed page. |
| [`docs/research/raw/`](docs/research/raw/) | The six research dives and their adversarial verification reports. |
| [`docs/plans/task-manager-exploration.plan.md`](docs/plans/task-manager-exploration.plan.md) | Decisions log and plan status. |

## Proposed stack (pending validation)

Flutter 3.47.9 with an in-repo Kotlin Android module (Glance widgets, AlarmManager reminder engine) and a Linux tray daemon with a systemd-user fallback. Data is in drift/SQLite, synced by self-hosted PowerSync to Postgres 18 with Keycloak auth. A server-side calendar bridge runs in Docker, and a local MCP server lets AI agents manage tasks.

## Licence

Not chosen yet (decision D11 in the report).
