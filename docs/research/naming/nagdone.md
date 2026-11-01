# Nagdone: name availability check (2026-11-01)

Verdict: **clear** for an exact-name conflict, with a crowded "Nag*" field nearby (details below).

## Stores
- Google Play: curl of `play.google.com/store/search?q=nagdone&c=apps&hl=en` returned 1 MB of HTML. The only app id in it is `com.littlepolarapps.day100` (Day 100: Fitness Challenge, unrelated filler). No Nagdone. A search for "nag done" returned the same single filler.
- Apple App Store: the iTunes Search API (`term=nagdone`, US/FR/GB) returned 1 result, NAFS by Mehdi Naghel (Lifestyle, fuzzy match, unrelated). No Nagdone.
- Nearby names in the same niche (not exact, but they share the "nag until done" concept):
  - "nag reminder" (com.robinkunz.nag, iOS id6478837661)
  - "Nag - Relentless Reminders" (iOS id6472675266)
  - "Nag: constant reminder" (iOS id6760954480)
  - "Nag: To Do List Daily Planner" (iOS id6445812798)
  - "Nagger: Reminder & To-Do" (iOS id6761343936)
  - "Nagger - Couple Todo Sharing"
  - NagTag, NagMe, NagMeLater, Naggr

## Other
- Flathub (POST /api/v2/search): only a fuzzy hit, Newelle. No Nagdone.
- Snapcraft: `find?q=nagdone` returned `[]`, and `info/nagdone` returned 404.
- F-Droid search: the page only echoed the query. No Nagdone app.
- Product Hunt / SaaS: an extended web search found nothing named Nagdone.

## Trademarks
- TMview (Playwright, JS UI) found **no rows** for "nagdone". A control search for "todoist" returned 16 hits in the same UI (USPTO, WIPO, UKIPO, CNIPA, IGE, INPI, in classes 9 and 42), so the search works. TMview covers EUIPO, INPI France, WIPO-designated marks and the USPTO, among others.
- Direct queries failed: EUIPO eSearch (curl got no connection), the TMview API (WAF "Request Rejected"), WIPO Brand DB (altcha captcha) and the USPTO tmsearch API (405).
- A web search for "NAGDONE" trademark found only unrelated marks: NAGD (dead, class 37), NADONG and NAGO.

## Domains (registry RDAP; rdap.org returned 403/302)
- nagdone.app: 404 not found (pubapi.registry.google). Control google.app returned 200.
- getnagdone.app: 404 not found.
- nagdone.com: 404 (rdap.verisign.com). Control google.com returned 200.
- nagdone.io: 404 (rdap.identitydigital.services). Control github.io returned 200.

## Registries
- github.com/nagdone: 404
- pub.dev/api/packages/nagdone: 404
- registry.npmjs.org/nagdone: 404

## FR/EN
- EN: "nag-done", clear, and it says exactly what the app does. "Nag" has a mildly negative tone (nagging) but nothing embarrassing.
- FR: "nag" is not a French word. "Nager" means to swim, so a French speaker may say "nag-donne" or "nage-done". Nothing vulgar. "Done" is widely understood in French. The spelling is easy in both languages.
- Risk: the name is descriptive ("nag" + "done") and sits close to "Nag" / "Nagger" reminder apps, which weakens its distinctiveness, both as a trademark and in store search.
