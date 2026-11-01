# HoldfastDo — name check (2026-11-01)

Verdict: minor-conflicts. The exact name "HoldfastDo" is unused. The base word "Holdfast" is crowded, and two apps in the Productivity category already use it.

## Exact "HoldfastDo"
- Web search for "HoldfastDo" (extended mode): no hits; the engine fell back to "holdfast".
- iTunes Search API, term=holdfastdo: 2 unrelated fuzzy hits (Intimo Radio, a dance school). No HoldfastDo.
- Play search page q=HoldfastDo: no package containing "holdfastdo".
- RDAP: holdfastdo.app, holdfastdo.com, holdfastdo.io and getholdfastdo.app all returned 404, so all four are unregistered.
- holdfast.do: rdap.org says "No RDAP service is available". The IANA RDAP bootstrap has no .do entry. Its status is UNKNOWN; check it through nic.do or WHOIS.
- github.com/holdfastdo: 404. pub.dev/api/packages/holdfastdo: 404. registry.npmjs.org/holdfastdo: 404. All three are free.

## "Holdfast" conflicts (confusing similarity)
- Google Play **app.holdfast** = "Holdfast: Block Reels & Shorts", category PRODUCTIVITY. It holds the `app.holdfast` id prefix, so `app.holdfast.do` would look like that developer's app. `app.holdfastdo` is a distinct id.
- Google Play com.studio.osori.holdfast (abstinence counter, Health) and com.twofoldfamily.holdfast (game).
- App Store: "Holdfast: App & Site Blocker" (Productivity, id6819522827), plus about 14 Health/Fitness "Holdfast" apps (fasting, men's health, crisis support, etc.), "HoldFast Anchor Alarm" and "My Holdfast".
- AlternativeTo: "Holdfast", a Pomodoro timer and website blocker Chrome extension by Focus Tools.
- Steam: Holdfast: Nations At War (Anvil Game Studios).
- F-Droid (do.holdfast): 404. Flathub search: no holdfast hit. Snapcraft find "holdfast": 0 results.

## Trademarks
- Trademarkia (a secondary aggregator of USPTO data): 37 HOLDFAST records and no HOLDFASTDO. The Class 9 marks I saw: 97914391 Anvil Studios (classes 9 and 28, games, live) and 77416213 Bel-Art Products (class 9, lab goods, live). 88273845 Humans First Detroit (classes 9 and 42, financial-assistance software) is ABANDONED. Trademarkia reports 7 class 9 and 2 class 42 records, but I could not see the details of all of them.
- USPTO tmsearch API: the POST was rejected (MethodNotAllowed), so no query worked.
- TMview API: "Request Rejected" (WAF). EUIPO eSearch and WIPO GBD only run as JS apps and were not queried. **The EU, French (INPI) and WIPO registers were NOT checked.**

## FR/EN
- EN: "hold fast" means grip firmly or persevere; "do" is the action word. It reads naturally, and the "do" cue helps signal a task app.
- FR: speakers drop the h ("old-fast"). "do" reads as "doh" (the note do) rather than "doo", which is fine. "fast" looks like French "faste" (pomp), which is harmless. No vulgar or embarrassing meaning in either language.
- The camel-case compound "Holdfastdo" is hard to parse when lowercased, as in a package id. "fast" also pulls toward fasting apps (many "Holdfast" apps are fasting trackers).
