# Name shortlist: local-first task / reminder app

Date: 2026-11-01. Source: availability checks for 18 candidate names (Play, App Store, F-Droid, Flathub, Snapcraft, Product Hunt, trademark registers, RDAP, GitHub, pub.dev, npm, and a French/English meaning check).

Only 7 of the 18 names came back `clear` or `minor-conflicts`: Nagdone, Holdtask, HoldfastDo, Bouclo, Tenu, Cuedo and Tenace. The best 5 are ranked below, and the other 2 are listed under rejected names as "eligible but cut on score".

## How names were scored

The criteria are listed by weight, highest first:

1. **Availability evidence.** The deciding fact is whether a trademark query actually ran and was shown to work with a control search. Only two names have that:
   - **Nagdone**: TMview UI via Playwright, with a `todoist` control search that returned 16 hits.
   - **Holdtask**: the USPTO tmsearch backend, with controls `todoist` (3 hits) and `holdfast` (34 hits).

   HoldfastDo and Tenu were checked only through third-party aggregators. Every trademark query failed for Bouclo, Cuedo and Tenace.
2. Whether the name evokes tasks or reminders. The owner's bar is that it does so CLEARLY, in both French and English.
3. Ease in both French and English: easy to say, easy to spell, no bad meaning.
4. Distinctiveness, i.e. how strong it would be as a trademark.
5. Domain availability (.app, .com, .io, get*.app).

## Ranked shortlist

### 1. Nagdone (verdict: clear)

- **Pitch:** It nags you until it's done. The name says what the signature escalating reminder does.
- **Proposed id:** `app.nagdone` (Android package and Linux app id), with the domain nagdone.app.
- **What was checked and found:**
  - **Stores:** no exact match on Google Play (`https://play.google.com/store/search?q=nagdone&c=apps&hl=en`, about 1 MB of HTML; the only package id in it was unrelated filler). No exact match on the App Store: the iTunes Search API was queried for the US, FR and GB stores.
  - **Other channels:** no match on Flathub, Snapcraft, F-Droid or Product Hunt.
  - **Trademarks:** TMview returned "No rows found". TMview aggregates EUIPO, the national EU offices including INPI France, WIPO-designated marks and the USPTO. A `todoist` control search in the same UI returned 16 hits, so the query works.
  - **Domains:** nagdone.app, nagdone.com, nagdone.io and getnagdone.app all returned 404 from their registries' RDAP servers. Each server was checked with a registered control domain that returned 200.
  - **Registries:** github.com/nagdone, pub.dev/packages/nagdone and npm `nagdone` are all free.
  - **Evidence file:** `docs/research/naming/nagdone.md`
- **Remaining risks:**
  - The name is descriptive (nag + done), which makes it a weak trademark. That matters more if the paid "Sync & AI" tier goes ahead.
  - The "Nag" reminder niche is crowded, so the name will be hard to find in store search:
    - nag reminder by Robin Kunz (`com.robinkunz.nag`, https://apps.apple.com/ie/app/nag-reminder/id6478837661)
    - Nag - Relentless Reminders (https://apps.apple.com/us/app/id6472675266)
    - Nag: constant reminder (https://apps.apple.com/us/app/-/id6760954480)
    - Nagger: Reminder & To-Do (https://apps.apple.com/us/app/id6761343936)
    - NagMe and Naggr
  - "Nag" has a slightly negative tone.
  - In French "nag" means nothing, and a French speaker may say "nage-done". Nothing offensive.
- **Human checks still needed:**
  - A formal EUIPO eSearch and a WIPO Global Brand Database search. The direct queries failed (connection failure and a captcha), so these registers were covered only through TMview.
  - An INPI search.
  - A TMview phonetic or similarity search. Only the basic search was run, and NAG\* marks in classes 9 and 42 have not been reviewed.
  - A trademark attorney's opinion on whether the name is too descriptive to register.

### 2. Holdtask (verdict: minor-conflicts)

- **Pitch:** A task you hold on to until it's done. It keeps the Holdfast "never let go" idea that the owner liked, and puts "task" in the name.
- **Proposed id:** `app.holdtask`, with the domain holdtask.app.
- **What was checked and found:**
  - **Stores:** no exact match on Google Play (`https://play.google.com/store/search?q=Holdtask&c=apps`; the list was truncated). No exact match in the iTunes Search API for the US and FR stores. The nearest name is "Hold My Thought: To Do List" (holdmythought.com).
  - **Other channels:** no match on F-Droid (404), Snapcraft or Flathub. Product Hunt returned 403, so it was not checked directly.
  - **Trademarks:** the USPTO tmsearch backend returned 0 hits for both "holdtask" and "hold task". The control searches (todoist: 3 hits, holdfast: 34 hits) show the query works. Nearby class 42 marks: MAINTASK and 24TASK.
  - **Domains:** holdtask.app, holdtask.io and getholdtask.app are free (RDAP 404). holdtask.com is registered, see the risks below.
  - **Registries:** github.com/holdtask (no repos found), pub.dev and npm are all free.
- **Remaining risks:**
  - **holdtask.com** was registered on 2026-08-12 through Amazon Registrar. It is on client hold, has no DNS and no live site, and the owner's identity is redacted. It could be a squatter or a future competitor.
  - In English, "on hold" means paused, which is the opposite of nag-until-done.
  - Nearby productivity names: Hold My Thought, HOLD X (a Chrome to-do extension) and Hold (a phone-distraction app).
  - The EU and WIPO registers could not be queried, so trademark status outside the US is unknown.
- **Human checks still needed:**
  - A formal EUIPO eSearch, plus TMview and WIPO GBD (all failed or were blocked here), and an INPI search.
  - A WHOIS or registrar enquiry about who registered holdtask.com in August 2026, and whether a product is coming.

### 3. HoldfastDo (verdict: minor-conflicts)

- **Pitch:** Hold fast and do it. It keeps the name the owner liked and adds an action verb, so it reads as a doing app.
- **Proposed id:** `app.holdfastdo`. **Do not use `app.holdfast.do`**: `app.holdfast` is another developer's Play package prefix.
- **What was checked and found:**
  - **Stores:** the exact name is unused on Google Play. On the App Store, the iTunes Search API for term=holdfastdo returned only 2 unrelated fuzzy hits.
  - **Other channels:** no match on F-Droid, Flathub or Snapcraft.
  - **Trademarks:** Trademarkia, an aggregator of USPTO data, shows 37 HOLDFAST records and no HOLDFASTDO.
  - **Domains:** holdfastdo.app, holdfastdo.com, holdfastdo.io and getholdfastdo.app are all free (RDAP 404).
  - **Registries:** GitHub, pub.dev and npm `holdfastdo` are all free.
- **Remaining risks:**
  - **The word that identifies the product, "Holdfast", is already used by productivity apps:**
    - Play: "Holdfast: Block Reels & Shorts" (`app.holdfast`, category Productivity, https://play.google.com/store/apps/details?id=app.holdfast)
    - iOS: "Holdfast: App & Site Blocker" (https://apps.apple.com/us/app/holdfast-app-site-blocker/id6819522827)
    - Chrome: a "Holdfast" Pomodoro and site-blocker extension (https://alternativeto.net/software/holdfast/)
    - About 14 Health/fasting "Holdfast" apps on iOS.
  - **Trademarks:** HOLDFAST 97914391 (Anvil Studios, classes 9 and 28, games) is live. Trademarkia counts 7 class 9 records and 2 class 42 records, and their details were not all visible.
  - The lowercase compound "holdfastdo" is hard to read.
  - In French it is said "old-fast" and "do" reads like the note "do". Acceptable.
- **Human checks still needed:**
  - Direct USPTO searches for HOLDFAST in classes 9 and 42: the goods lists of the Anvil mark and of the other class 9 records.
  - EUIPO eSearch, TMview, WIPO GBD and INPI. None of these was queried.
  - A legal opinion on confusion with the existing Holdfast productivity apps.
  - The status of holdfast.do (there is no RDAP service for .do; use WHOIS or nic.do).

### 4. Bouclo (verdict: minor-conflicts)

- **Pitch:** "C'est bouclé" means it's wrapped up and done. It is the "close the loop" promise behind the brain dump → done flow.
- **Proposed id:** `app.bouclo`, with the domain bouclo.app.
- **What was checked and found:**
  - **Stores:** no match on Google Play (`https://play.google.com/store/search?q=Bouclo&c=apps`; the page was truncated) or the App Store (web search only). The nearest name is Boulot, a GTD task manager (https://apps.apple.com/mx/app/boulot/id6751812250).
  - **Other channels:** no match on F-Droid, Snapcraft, Flathub or Product Hunt.
  - **Domains:** bouclo.app, bouclo.io and getbouclo.app are free (RDAP 404).
  - **Registries:** pub.dev and npm are free.
- **Remaining risks:**
  - **The name fails the owner's bar for English speakers.** "Boo-klo" means nothing in English and does not evoke tasks. This is the same objection that sank Holdfast.
  - **Other users of the name in France:**
    - https://bouclo.com is a live French vintage clothing shop called "Bouclo" (registered 2025-12-02, Hostinger), in the owner's own market.
    - bouclo.fr was registered on 2026-09-25 (OVH) and has no content yet.
  - github.com/bouclo is taken by an individual.
  - Every trademark query failed (TMview, EUIPO, WIPO, INPI, USPTO), so status in classes 9 and 42 is unknown.
  - In French, "boucle-la !" means "shut up!" and "boucler" can mean to lock someone up. These are mild.
- **Human checks still needed:**
  - INPI and EUIPO searches. Check whether the clothing shop holds a BOUCLO mark: classes 25 and 35 would not block software, but the overlap in brand recognition in France would remain.
  - Who registered bouclo.fr.
  - WIPO GBD and USPTO searches.

### 5. Tenu (verdict: minor-conflicts)

- **Pitch:** "Promesse tenue" means a promise kept. Four ASCII letters that carry the app's promise, and a story for a French owner.
- **Proposed id:** `app.tenu`, which depends on buying tenu.app (otherwise use the gettenu.app domain).
- **What was checked and found:**
  - **Stores:** no app named exactly Tenu on Google Play (`https://play.google.com/store/search?q=Tenu&c=apps`) or the App Store (web search). The Play search shows Temu instead, a near-homograph.
  - **Other channels:** no todo or productivity "Tenu" on Flathub, Snapcraft, F-Droid or Product Hunt. Tenu (Singapore) is a wardrobe app.
  - **Trademarks:** the aggregators markinton and trademarkdir show only TENU 79220360 (class 33, spirits), which is DEAD, plus TENUGIN (class 33). No class 9 or 42 mark was found.
  - **Domains:** gettenu.app is free.
  - **Registries:** GitHub, npm and pub.dev `tenu` are all free.
  - **Evidence file:** `docs/research/naming/tenu.md`
- **Remaining risks:**
  - **The name fails the owner's bar for English speakers.** It has no meaning in English, English speakers say "TEN-oo", and it does not signal tasks.
  - It is easy to confuse with Temu.
  - **Domains:**
    - tenu.app is registered (2026-04-02, Namecheap) and shows a parked auction page, so it may be for sale.
    - tenu.com is registered (2001, IONOS).
    - tenu.io is registered (Backordr, expires 2026-12-30).
  - The EU, WIPO and INPI registers were not queried. The USPTO data comes only from aggregators.
- **Human checks still needed:**
  - EUIPO eSearch, TMview, WIPO GBD and INPI searches.
  - A direct USPTO search.
  - The asking price for tenu.app, and whether tenu.io drops when it expires on 2026-12-30.

## Rejected candidates

### Eligible but cut on score

- **Cuedo** (minor-conflicts): Cuedo Controls Pty Ltd, an Australian software company, has held cuedo.com, cuedo.app and github.com/cuedo since 2019–2020. The name is one letter from Hasbro's CLUEDO, which French speakers know well, and the opening "cu" invites jokes in French. No trademark query worked, and the check itself called the name "borderline conflicted".
- **Tenace** (minor-conflicts):
  - tenace.app, tenace.com and tenace.io are all registered.
  - TENACE (USPTO Reg 7171259) is a live mark in class 42 (IT services).
  - "TENACE: Workout Tracker" with streaks and planning is on both stores.
  - It evokes persistence, not tasks, and English and French speakers pronounce it differently.

### Conflicted

- **Holdfast Todo:** the word that identifies the product, Holdfast, is used by a Play focus app (`app.holdfast`) and a Chrome productivity extension, and is registered as HOLDFAST in USPTO class 9 and UKIPO class 42. The name is also long.
- **Taskfast:** TaskFast (Akuja Inc.) is a live AI-agent task marketplace at taskfast.app. taskfast.com and the GitHub handle are taken.
- **TillDone:** "TillDone: Task Reminders" (`com.obaylabs.tilldone`) is an active Play app with the same name and the same nag-until-done concept. tilldone.app and tilldone.com are registered.
- **Nagly:** "Nagly: Mom's Water & Pill Nags", a persistent-reminder app, is live on Play and iOS. nagly.app, nagly.com and the GitHub handle are taken.
- **Duely:** several live Duely reminder, planner and bill apps exist on iOS and Android. duely.app, duely.com, getduely.app, npm and the GitHub handle are taken.
- **Dueline:** "Dueline: Bill Planner" and "Dueline - Invoice Maker" are on the App Store, and a Dueline bill PWA exists. dueline.app is registered (2026-09) and dueline.com is for sale. The name leans toward bills.
- **Ondue:** three live OnDue bill and due-date reminder apps exist, ondue.io is a reminder-driven collections SaaS, and ondue.app, ondue.com and the GitHub handle are taken.
- **Taskeep:** four or more task products already use the name: a Play app (`com.xaviscript.taskeep`), a task SaaS on taskeep.app, the TasKeep organiser on taskeep.me, and a GitHub task app. taskeep.com is registered too.
- **Pacto:** PACTO is a live USPTO class 9 software mark (serial 90736165). Several Pacto apps exist, including two in Productivity. All the main domains, npm and GitHub are taken. French and English speakers would spell it Pacte or Pact.
- **Promis:** "Promis - Engineer" holds `com.promis.*` on Play and two Promis iOS apps exist. PROMIS is an HHS mark in classes 9 and 42 and the name of the NIH PROMIS system, which floods search. Several promise-tracker apps have near-identical names. promis.com is corporate-held, and npm and GitHub are taken.
- **Relanz:** relanz.fr is a live French SaaS (2026) that sends automated SMS follow-ups and reminders, in the same market and on the same "relance" concept. relanz.com and the GitHub handle are taken.

## Checks that apply to every shortlisted name before choosing

1. A formal EUIPO eSearch and a WIPO Global Brand Database search in classes 9 and 42. Every automated attempt was blocked by JavaScript-only pages, firewalls or captchas. Run identical and phonetic searches.
2. An INPI France search, since the owner is in Paris. Only Nagdone was covered, through TMview.
3. A clearance opinion from a trademark professional before registering domains or filing a mark, especially if the paid tier goes ahead.
4. A full, non-truncated Google Play search, done by hand in a browser. Several automated Play result pages were cut off.
5. Register the chosen name's .app domain and GitHub org quickly. Several candidates' domains were registered during 2026: holdtask.com, tilldone.app, dueline.app and bouclo.fr.
