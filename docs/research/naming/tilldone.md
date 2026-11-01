# TillDone — name availability check (2026-11-01)

**Verdict: CONFLICTED**

## Google Play — DIRECT CONFLICT
- "TillDone: Task Reminders" by ObayLabs, package `com.obaylabs.tilldone`
  https://play.google.com/store/apps/details?id=com.obaylabs.tilldone
  Category Productivity, 100+ downloads, updated Oct 22, 2026. Tagline: "Task reminders that repeat until done. Snooze, recurring schedules, and no ads."
  It has the same name and the same signature concept (nag until done).
  [Verified: curl of the Play search page + details page, title/genre/updated text extracted]

## Apple App Store (iTunes Search API)
- term=tilldone: 0 results.
- term="till done": "Pico: Nags Till It's Done" (hyoungmin lee, id6756974216) has the same concept with the phrase in its subtitle. "Debug Till Done LLC" is a developer name (fitness apps). "Until Done" (id6752790841) is an aggressive-reminder app with a near-identical concept.

## Other stores and registries
- F-Droid: package-id API 404. On search.f-droid.org, `?q=tilldone` showed "tilldone" only where the page repeats the search term, with no app names, so no app was found.
- Flathub POST /api/v2/search "tilldone": 11 fuzzy hits (Tasks, Errands, Planify, TickTick, ...), none named TillDone.
- Snapcraft `tilldone`: 404.
- Product Hunt: web search found no TillDone. It does list "tick done" (different name).
- GitHub user `tilldone` exists (User, created 2019-05-27, 1 repo `gatsby_tutorial_1`). This looks like an inactive squatter.
- pub.dev `tilldone`: 404. npm `tilldone`: 404 (both free).

## Domains (RDAP via curl)
- tilldone.app: REGISTERED 2026-08-26, Cloudflare. It was likely registered by the Play app's developer [Inferred: the date predates that app's Oct 2026 update].
- tilldone.com: REGISTERED 2017-10-27, NameBright (last changed 2026-10-28).
- tilldone.io: 404, available.
- gettilldone.app: 404, available.
- (WebFetch to rdap.org returned 403; curl returned the records.)

## Trademarks
- USPTO tmsearch: the page is JS-only and returned no results text, so it was NOT effectively queried.
- EUIPO eSearch, TMview, WIPO GBD: not queried (JS-only).
- Web search for "TillDone trademark": no hit. Absence is NOT proven.

## FR/EN
- EN: "till done" means "until done". It is clear, and "till" also means a cash register, so the name is mildly ambiguous.
- FR: "TillDone" is roughly "til-donne", easy to say. No bad meaning was found. French speakers may not parse "till", and spelling it out loud is easy.

## Conclusion
An active Play Store productivity app has this exact name and the same nag-until-done concept, and both .app and .com are taken. Do not use.
