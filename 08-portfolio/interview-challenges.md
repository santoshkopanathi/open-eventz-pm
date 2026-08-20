# Open Eventz — Challenges & Hard Problems (Interview Prep)

*This document captures the real obstacles encountered during the Open Eventz project — both what the challenge was and how it was resolved or handled. Use these in interviews to demonstrate PM judgment, problem-solving, and honest scoping.*

---

## 1. There's No "Free Kids Events" API — and the Obvious Ones Are Dead Ends

**The challenge.** The most natural assumption when building an events aggregator is that some API exists you can tap. It doesn't. The most prominent option — Eventbrite — deprecated its public event search endpoint in February 2020 specifically to prevent third-party aggregators from competing with it. What remains is event-by-ID access, which is useless for open discovery. PredictHQ exists and is broad, but it's an enterprise B2B data product sold via sales conversation, not curated for "free" as a category, and would add cost before there's a dollar of revenue.

**What this forced.** The entire data strategy had to pivot to direct, source-by-source integrations with the actual publishers of free programming — libraries and city governments. This means the data problem is fundamentally per-metro, which has real implications for how the product scales: adding a new metro is a repeatable process (a checklist + schema), not a single API call.

**Summary.** "My first instinct was to find an events API. I spent time verifying that the obvious candidates don't work for this use case — Eventbrite killed public search in 2020, PredictHQ is enterprise-priced and unfocused on 'free.' That constraint actually clarified the product's moat: if there were an easy API, someone would have already built this. The difficulty of the data sourcing is part of what makes it defensible."

---

## 2. The Same City's Own Documents Disagreed With Each Other

**The challenge.** Frisco Library's unattended minor policy appeared in two places with two different ages: the city code said 9, the library's own Summer FAQ said 10. This wasn't an abstract discrepancy — the supervision field is a core differentiating feature, and if the app shows a wrong age threshold, a parent could leave a child unsupervised based on bad data.

**How it was resolved.** Tracked down the official 2026 Service Policy PDF directly from the library's website. It states "Children aged nine (9) or younger must be accompanied by an adult" — which means 10-and-older can attend without a parent. This aligned with the Summer FAQ (not the city code). The city code was the stale source. Having the current, authoritative policy document in hand is now the logged source.

**What this forced.** It validated the three-tier confidence system (Tier 1: event-specific statement; Tier 2: venue general policy; Tier 3: unverified). The conflict also made it clear that "Tier 2" doesn't just mean you found a policy — it means you found the *right* document and know which one to trust when they disagree.

**Summary.** "I hit a real data quality problem early: the library's own documents contradicted each other on the age threshold. My rule was: never guess at this field, because a parent making a drop-off decision based on wrong data is a real safety issue. I tracked down the primary policy document, found the authoritative answer, and logged the source and date so it can be re-verified. That's how I'd approach any trust-sensitive data field."

---

## 3. Play Frisco Had No Official Feed — But Unexpectedly Had Something Better

**The challenge.** Unlike the libraries (which have vendor-supported RSS/XML feeds), the City of Frisco Parks & Recreation had no official data feed. The only option appeared to be HTML scraping of the city event calendar — which is fragile (structure can change), legally ambiguous, and blocked in the city's robots.txt for the RSS version (`/RSS.aspx`).

**What was discovered.** The city's calendar page included a "Subscribe to iCalendar" link — a publicly accessible `.ics` feed at `/iCalendar.aspx`. This is a structured, machine-readable format the city already publishes for public subscription. It's not blocked in robots.txt, it's not HTML (so it won't break when the page redesigns), and using a subscription feed the city explicitly provides for public use is on much firmer ground than scraping HTML.

**What this forced.** Updated the PRD to make the iCalendar feed the preferred integration method and HTML scraping the fallback. The discovery came directly from looking at what the city calendar page actually offered rather than assuming scraping was the only path.

**Summary.** "My assumption going in was that Play Frisco would require HTML scraping. I did the ToS investigation anyway, and while doing that I found that the city publishes an iCalendar subscription feed that's publicly accessible. That's a much cleaner integration — structured data, already intended for machine consumption, no gray area. The lesson is: read the full page before assuming the hard path is the only path."

---

## 4. VisitFrisco.com Looked Like a Data Source Until It Wasn't

**The challenge.** VisitFrisco.com appeared to be a promising source of Frisco events with a well-organized calendar. Before building against it, investigated the actual network calls the site makes to load its calendar data.

**What was found.** The site is built on Next.js and loads event data from internal URLs that include a build ID (`esoUENnin8q1gQN52KTW3`) — a value that changes every time the site redeploys. Any scraper built against those URLs would break on the next deployment with no warning and no recoverable path. Beyond the instability, the content itself was wrong for the audience: concerts, sports, nightlife — not kid-friendly free events.

**What this forced.** Ruled out VisitFrisco.com entirely, logged the reason in the Source Decision Log, and flagged it as a backlog candidate only (meaning: would revisit only if they ever publish a stable feed). The investigation saved build time that would have been wasted on a source that would have broken immediately.

**Summary.** "I spent time investigating VisitFrisco.com before deciding not to build against it. The technical reason is that it uses Next.js internal data URLs with a build ID that changes on every deploy — there's no stable endpoint to target. Even setting aside the technical fragility, the content was wrong: concerts and sports, not free kid activities. Sometimes the right PM decision is ruling something out clearly so it stays out of the backlog indefinitely rather than getting relitigated every sprint."

---

## 5. BiblioCommons RSS: Confirmed Existence, Can't Directly Verify Format

**The challenge.** BiblioCommons (Frisco Library's platform) is documented to support RSS export for events. But locating the specific RSS endpoint URL — as opposed to the HTML event page — required investigation, and the fetcher used returned an empty body for the RSS URL (likely valid XML that the tool couldn't display as text).

**What was found.** The endpoint `https://friscolibrary.bibliocommons.com/events/search.rss` appears to exist — it returned a 200-class response without error, just no HTML body, which is consistent with XML content the fetcher filtered out. The events page itself is confirmed live with 280+ active events. Audience filter IDs needed for children's-only filtering were also identified.

**What this forced.** A clear note in the PRD that this needs a 30-second browser/curl verification at build time. Not a blocker, but it's the kind of thing that looks like a blocker if you don't distinguish "can't verify format" from "endpoint doesn't exist."

**Summary.** "One of my early lessons on this project was learning to distinguish 'I can't confirm this' from 'this doesn't work.' The RSS URL returned a valid response — the tooling I was using just couldn't display XML. I logged it as 'confirm format at build time' rather than closing it as broken. Good PM documentation is honest about what's verified versus what's inferred."

---

## 6. Supervision Data: The Product's Best Feature Is Also Its Riskiest

**The challenge.** The supervision / drop-off flag is one of the sharpest, most defensible differentiators in the product — no other app tells parents whether they need to stay or can drop off. But it's also the highest-stakes data field: if the app shows a wrong threshold and a parent leaves a child who shouldn't be left alone, that's a real safety issue the product is implicated in.

**The approach taken.** Designed a three-tier confidence system (event-specific policy → venue general policy → unverified) and applied a hard rule: Tier 3 always shows "check with venue," never a guessed number. When the Frisco Library's documents conflicted, the answer was to find the authoritative source rather than pick the more convenient answer. When Plano Library had no age threshold at all (parent's discretion), that was captured precisely as stated — not defaulted to a number.

**What Plano Library's answer revealed.** Calling the library and getting "it's parent's discretion, we just expect the child to behave" is itself a meaningful data point — it tells a parent something true and useful. The display for Plano events isn't "check with venue" (implying uncertainty), it's "No age requirement — parent's discretion" (which is accurate and actually reassuring). Getting the exact wording right matters.

**Summary.** "The supervision field is the feature I'm proudest of and most careful about. The whole value proposition only works if parents trust it — and they'll only trust it if it's accurate. My rule was: never show a threshold we haven't verified, and never show the same generic 'check with venue' when we actually have real information. I called Plano Library directly to get their policy. Their answer — 'no age requirement, parent's discretion' — is different from Frisco's hard cutoff at 10, and both are worth showing accurately."

---

## 7. Scope Discipline: Saying No to Obvious Sources

**The challenge.** During research, several seemingly useful data sources came up: PlanoMoms.com (a well-curated local parenting blog), Eventbrite organizer pages, Plano Parks & Rec, Frisco ISD, YMCA, museum free-day calendars. Each one had a real argument for inclusion.

**The discipline applied.** Each source was evaluated against: Is there a clean, stable integration method? Is the content reliably in scope (free, kid-appropriate)? Does adding it now materially improve v1 over not adding it? PlanoMoms.com was explicitly ruled out as a scrape target — their curated editorial work is their IP, and republishing it without permission is a partnership conversation, not a technical decision. Plano Parks & Rec has a PDF-based catalog, which adds parsing complexity for minimal gain over what the library feeds already cover. Each ruled-out source is logged in the Source Decision Log with the reason, so the decision doesn't get relitigated every quarter.

**Summary.** "One of the hardest things about this project was saying no to sources that had real value. PlanoMoms.com is genuinely good content — but scraping someone else's curated editorial without permission isn't the right move. I logged it as a potential partnership instead. The Source Decision Log is my artifact for preventing scope creep disguised as obvious improvements."

---

## 8. Two Different Problems Hidden Inside One Feature: Geographic Search

**The challenge.** "Find events near me" sounds like one feature. In practice it's two separate problems: the geographic unit for *data sourcing* (which is city/branch — because that's how libraries and parks departments are organized) versus the geographic unit for *user-facing search* (which ideally would be zip code or radius). Conflating the two would mean either building a geocoded radius search before you have enough sources to make it meaningful, or confusing the data model by trying to store things in terms the user cares about rather than terms the sources use.

**The decision.** Explicitly split these in the data model from day one: sources are stored and identified by city/branch, and the user-facing location experience in v1 is source-based filtering (Frisco Library, Plano Library, Play Frisco) rather than geocoded radius. True radius search is logged as a future enhancement for when there are enough sources across a metro to make it worth the geocoding complexity.

**Summary.** "I caught an assumption buried in the feature spec early: 'near me' filtering could mean two completely different things depending on which layer of the product you're talking about. I separated them in the data model explicitly so we weren't building geocoding complexity before we needed it, and so the v1 experience still made geographic sense even without it."

---

---

## 9. BiblioCommons RSS: Confirmed Dead at Build Time

**The challenge.** Challenge 5 above noted that the BiblioCommons RSS endpoint needed build-time verification. When we actually built the ingest pipeline, the endpoint returned a 404 — the feed no longer exists. This wasn't a tooling display issue; BiblioCommons has retired RSS entirely in favor of their JavaScript-rendered web app. None of the alternative API paths worked either: the internal JSON API was private, iCal redirected to the HTML page, and XHR interception in DevTools found no separate events API call at all.

**How it was resolved.** Discovered that events are fully server-rendered in the HTML as plain markup — no JavaScript needed to see the data if you fetch the page server-side. Built a structured HTML parser that splits the page on the card boundary (`<li><div class="cp-events-search-item">`), then regex-extracts each field from the card HTML. Added audience-based pagination (three feeds × up to 14 pages × 20 events per page) to cover the full two-month event window.

**What this forced.** The integration is more brittle than an RSS feed — if BiblioCommons changes their HTML structure, the parser breaks. Mitigated with per-source error isolation so a parser failure only takes down that source, not the whole app. Also forced the decision to fetch each event's detail page individually to get accurate age data (see Challenge 11).

**Summary.** "The RSS feed I'd planned to use didn't exist when I went to build against it. Rather than treat that as a blocker, I reverse-engineered the HTML the website renders server-side and built a structured parser against it. It's a more fragile integration, and I documented that honestly — the tradeoff is full data coverage now versus some maintenance risk if their HTML changes later. I made that tradeoff consciously rather than quietly."

---

## 10. Play Frisco iCal Only Covered Events Starting January 2027

**The challenge.** The city's iCalendar feed (`catID=81`) was confirmed as the official integration method and appeared clean in ToS terms. But when actually fetched during the build, every event in the feed had a start date of January 2027 or later — a five-month gap with no near-term events. The RSS feed was already in use as a fallback, but it was capped at 8 items covering only the next six weeks.

**How it was resolved.** Investigated where the near-term events actually lived. The city's own calendar page (`calendar.aspx?CID=85,81`) is server-rendered and paginated by month — fetching it for the current and next two months via `&month=M&year=Y` returns the full event set the public sees on the website. Replaced both the RSS and iCal feeds with a month-by-month HTML scrape: extract EIDs from each monthly listing page, then fetch each event's detail page for structured data (title, date, location, description) using CivicPlus's schema.org `itemprop` attributes.

**What this forced.** The integration now makes more HTTP requests (3 listing pages + 1 detail page per event), but the data coverage is accurate and complete. Both the RSS and iCal feeds were removed entirely — two sources replaced by one more reliable one.

**Summary.** "The official feed existed but only surfaced events five months out — useless for a 'what's on this week' product. I traced back to where the city's own website was getting its near-term data from, which turned out to be a server-rendered monthly calendar page. The right integration wasn't the officially documented one; it was the one that actually had the data parents needed."

---

## 11. BiblioCommons Audience "Bleed" — Same Event, Wrong Age Label

**The challenge.** BiblioCommons serves events through three separate audience feeds (Children 0–5, Children 6–12, Teens). The same event frequently appears in multiple feeds — a story time might appear in both the 0–5 and 6–12 feeds. The first approach was to use the feed's audience as the age label. This produced obviously wrong results: a "Dinosaur George" exhibit tagged as Children 0–5 from one feed would then appear again tagged as Children 6–12 from another, with conflicting labels in the UI.

**How it was resolved.** Each BiblioCommons event page includes a "Suitable for:" section listing the actual intended audiences. Since the ingest pipeline was already fetching each event's detail page for description, the same HTTP request was used to scrape the "Suitable for:" section and derive age_min/age_max from the actual page content rather than the feed it was discovered in. Deduplication across feeds then merges age ranges, but since the page scrape is authoritative, merged values typically stay consistent.

**What this forced.** The page scrape adds latency to ingestion (one HTTP request per event), but it's the only reliable source of truth for age. It also required handling four detection cases: adults + young children (0–17), adults + teens only (13–17), adults only (excluded), and kids/teens only (computed range).

**Summary.** "Vendor feed design doesn't always match product needs. BiblioCommons serves events through audience-segmented feeds, but the same event bleeds across multiple feeds with no reliable way to know which audience assignment is correct from the feed alone. I solved it by going to the authoritative source — the event page itself — which explicitly states the intended audience. One extra HTTP request per event at ingest time bought accurate age data for every event permanently."

---

## 12. Adults + Teens Events Surfacing Under Toddler and Kids Filters

**The challenge.** After deploying the "Suitable for:" page scrape, some clearly adult-targeted events (ukulele class, music production workshop) were still appearing under the Toddlers and Kids age filters. The scraper logic treated `hasTeen = true` as evidence of a kid-appropriate event — so when a page said "Suitable for: Adults, Teens," the code hit the `adults + any kids` branch and set age_min=0, age_max=17, making it appear for all ages including toddlers.

**How it was resolved.** Distinguished between "adults with young children present" (0–17 appropriate) and "adults with teens only" (13–17 appropriate). Introduced a `hasYoungKid` check separate from `hasTeen`, and added a specific branch for Adults + Teens that sets age_min=13 rather than 0. Also added a permanent API-level exclusion for adults-only events (age_min ≥ 18) so they never surface regardless of filter state.

**What this forced.** A tighter four-case decision tree for audience detection, and the habit of checking real events on the source website to validate the logic rather than just testing with synthetic cases.

**Summary.** "An edge case in the age detection logic was surfacing adult events to parents looking for toddler activities. The bug was subtle — treating 'teens' as equivalent to 'young children' when adults were also tagged. Fixing it required thinking through the full decision tree: adults + toddlers is different from adults + teens, which is different from adults only. I validated the fix by looking up the specific events that were misbehaving on the source site and confirming what their 'Suitable for:' field actually said."

---

## 13. Multi-Day Events Disappeared Mid-Run

**The challenge.** Soccer Celebration — a Parks & Rec event running June 11 through July 19 — was correctly ingested with its start and end dates, but disappeared from the app the day after it started. The events API filtered to `start_datetime >= today`, which correctly excluded past-starting events but also incorrectly excluded events that had already started and were still running.

**How it was resolved.** Added a supplemental database query that runs alongside the primary one: fetch events where `start_datetime < today AND end_datetime >= today`. Results from both queries are merged and deduplicated before being returned. This ensures ongoing multi-day events always surface at the top of the list for as long as they're running.

**What this forced.** A two-query pattern in the events API instead of one, plus a deduplication step in the merge. Also required fixing how all-day iCal dates were parsed — they were coming in as midnight UTC, which shifted the date one day earlier in Central Time. Fixed by parsing all-day dates as noon UTC instead.

**Summary.** "A date filter that seemed obvious — only show future events — had an unintended consequence for multi-day events. An event that started last week and runs through next month is both past-starting and currently relevant. I added a supplemental query for the 'ongoing' case rather than bending the primary query into something complicated, then merged and deduplicated the results. Simple queries composed cleanly are easier to reason about than one complex query trying to handle every case."

---

## 14. Date Filter Off by One Day — a Timezone Boundary Problem (the **read** path)

> **Distinguishing note — this document has three timezone entries, at three different layers. Don't let them read as the same bug three times; that they're distinct is the point.**
> - **#14 (this one) — the READ path.** A user-selected *calendar date* compared against stored UTC timestamps. The stored data was correct; the **query boundary** was wrong, so filtering let the wrong day through. Fix: convert the date to CT-aware UTC boundaries before comparing.
> - **#18 — the WRITE path.** A source's wall-clock time parsed with no timezone, so it resolved in the *runtime's* zone. The queries were fine; the **stored value itself** was wrong by 5–6 hours. Fix: resolve every source time as the venue's timezone, explicitly.
> - **#21 — the architectural response.** Not a bug at all: making the write path **fail-closed**, so an entire *class* of clock error can't reach a user — including shapes I haven't thought of.
>
> **Together they're a stronger answer than any one alone.** Read → write → prevention is the arc of coming to treat timezone handling as a **system property** rather than a bug you fix once. If asked "tell me about a hard bug," pick one. If asked "how do you think about correctness," walk all three.

**The challenge.** When a user set the date filter to "From: July 1," events from June 30 still appeared. Specifically, an event at 7:00 PM CDT on June 30 was stored in the database as `2026-07-01T00:00:00Z` (midnight UTC, because CDT is UTC−5). The date filter was sending `2026-07-01` to the API, which Supabase compared as `>= 2026-07-01T00:00:00Z` — and the June 30 event passed because it was stored at exactly that UTC timestamp. The same bug ran the other way on the end date: events on the selected to-date didn't appear because their UTC timestamps were ahead of midnight UTC.

**How it was resolved.** Converted user-selected dates (which represent local CT calendar days) to CT-aware UTC boundaries before querying. `date_from` maps to midnight CT = `T05:00:00Z` (CDT offset). `date_to` maps to end of that calendar day in CT = the next day at `T05:00:00Z`, queried with `<` rather than `<=`. Both helpers were extracted into named functions (`dateToCtMidnightUtc`, `dateToCtEndOfDayUtc`) to make the intent explicit.

**What this forced.** Re-examining every place in the API where dates are compared — the ongoing multi-day supplemental query also needed the same treatment.

**Summary.** "Timezone bugs in date filters are one of the most common sources of off-by-one errors in consumer apps, and they're invisible until a real user tries filtering on a boundary day. The root cause here was treating a user-selected calendar date as a UTC timestamp. The fix is always the same: be explicit about which timezone a date string represents and convert it before comparing against stored UTC values. I extracted the conversion into named helper functions so the intent is clear to anyone reading the code later."

---

## 15. Plano Libraries Age Data: No Path Forward, Accepted as a Known Limitation

**The challenge.** Plano Libraries runs on Communico, which does have age/audience fields on individual events. However, the RSS feed — the only accessible integration path — does not include age fields. The Communico API supports an `ages` filter parameter, but passing any value (`children`, `teen`, `all`) returns an empty feed. The API exists but isn't usable without vendor-issued authentication tokens that Plano Library does not make publicly available.

**How it was resolved.** Documented as a known, accepted limitation rather than blocking the release on a fix that doesn't exist. All Plano events are stored with null age_min/age_max, which means they pass all age filters — a parent filtering for toddlers will see Plano events even if some of those events are adult-targeted. Compensated in the UI with a branch sub-filter (choose a specific Plano library branch) as the contextual refinement tool when Plano is selected, since branch-level filtering is data we do have.

**What this forced.** A clear decision about what "acceptable" looks like when a data gap can't be closed. The age filter's null pass-through is a deliberate product decision — showing more events than strictly match is better than showing fewer, given that Plano's events are generally family-oriented by default. Documented in the PM OS as a limitation, not silently papered over.

**Summary.** "Not every data gap has a technical solution. Plano's age data exists in their system, but the only accessible integration path doesn't expose it, and the API that would expose it requires vendor auth they don't issue publicly. Rather than pretend the limitation doesn't exist or block the feature, I documented it explicitly, decided that null-inclusive filtering was the right default behavior, and added a different type of refinement — branch filtering — that works with the data we do have. Knowing when to accept a constraint and compensate elsewhere is as important as knowing how to work around one."

---

## 16. A Shipped Feature Silently Disappeared — and I Was Confidently Wrong About Why, Twice

**The challenge.** The product's most differentiated feature — a supervision flag telling parents whether they could drop their child off or needed to stay — was once shown across all three data sources. Weeks later it was only rendering for one; the other two showed nothing. No error, no failed test, no alert. It surfaced only because old screenshots proved the feature used to do more. A silent regression on the exact feature the product's trust story depends on.

**What it revealed (and the false trails).** Diagnosing *why* took three passes, and being willing to overturn the first two answers was the whole game. Theory one — a git history rewrite (a secret-scrub `filter-branch` + force-push) had erased it — was stated confidently and was wrong: that rewrite preserved every commit. Theory two was also wrong. The actual cause only held up on the third pass: the all-source version had been refactored down to a single-source inline version **before the project was ever committed to git.** A month of development had run with no version control, so the working feature died in an unversioned window with no diff and nothing to recover.

**What this forced.** Three durable changes. (1) Get code under version control *before* refactoring, not after it's "ready." (2) Point the strongest guardrail at the differentiator — the supervision logic had been inline in a component with no unit test, so it fell out silently; it now warrants an extracted, per-source-tested module with a doc-parity check. (3) Record the *change*, not just the current state — the build log had described what the feature *is*, which hid the day it started doing less. It also reframed the incident as an asset: a real production-style regression with a documented root cause and fix, which is exactly the "operating history" that AI-PM interviews probe for and most portfolios lack.

**It then happened a second time — and that's the more useful half (2026-08-14).** Building out the per-event page (`/events/[id]`) — the page a **shared link lands on** and the one Google indexes — I found the supervision badge had **never been there at all.** It rendered only in the in-app detail panel. So a parent who received a shared link got exactly nothing of the one signal the product exists to provide, and had done since the day that page shipped. The first disappearance was a regression *across sources*; this one was a gap *across surfaces* — same feature, different axis, and my fix for the first (a per-source unit-tested module) was structurally incapable of catching the second.

**What the second one forced — "the list of surfaces is the spec."** The badge treatment moved into a single shared `SupervisionCallout` component used by **both** detail surfaces, rather than the markup being copy-pasted. The point isn't DRY; it's that one component means a surface either shows it for every source or not at all — it can no longer drift *per-source* or *per-surface*. Backed by a `supervision-surfaces.test.ts` that asserts each detail surface renders `<SupervisionCallout>` **and that no surface calls `getSupervisionBadge` directly**, so a future surface can't quietly reimplement it. It's a source-text check rather than a render test (the repo has no react-testing-library) — the same idea as the existing doc-parity check: **enumerate the places a feature must appear, and make that enumeration executable.** Also worth admitting: the build log had claimed "Add to Apple Calendar" was on both surfaces when it was only ever on one — the same class of drift, caught in the same pass, which is why the enumeration has to be a test rather than a sentence in a document.

**Summary.** *"One of my most differentiated features silently lost two-thirds of its coverage and I didn't notice for weeks — no test watched it. Investigating, I was confidently wrong about the cause twice before I found it: the feature had been refactored away before the code was ever under version control, so there was no history to recover. Then it happened again on a different axis — I discovered the same feature had never rendered on the shared-link page at all, so anyone receiving a link got none of the thing the product exists for. The first fix, a per-source tested module, structurally couldn't catch a per-surface gap. So the real fix was one shared component plus a test asserting every detail surface uses it and none reimplements it — enumerate the places a feature must appear and make that enumeration executable, because a list of surfaces written in a doc drifts and a test doesn't. And the meta-lesson from the first investigation: the first explanation that fits isn't the cause, it's just the first thing that fits. I got there by disproving myself."*

---

## 17. A Scraper That Returned "Everyone" — Server-Side vs. Client-Side Rendering, and Why I Didn't Reach for the LLM

**The challenge.** Parents on the Frisco tab started seeing adult events ("D&D for Adults," "Adult Volunteer Open House"), and the age filters stopped working — selecting "Toddlers (0–5)" still returned 305 of 306 events. The data confirmed it: 304 of 306 Frisco events were stored as age 0–17. The single source of Frisco age truth — BiblioCommons' "Suitable for:" field — had gone blank for our scraper, so every event fell through to the "all ages" fallback, which passes the adults-only gate (`age_min < 18`) and overlaps every kid filter.

**The false start (an honest correction).** My first pass had called the surrounding staleness "pure staleness, not a parser break" — because I checked the *listing* page markup and it was intact. That was incomplete: I hadn't checked the *detail-page* age extraction, which was broken. Two different scrape steps; I'd validated the wrong one.

**What caused it.** BiblioCommons had migrated event pages to a **client-side-rendered `/v2`** architecture. The "Suitable for:" audience is now hydrated by JavaScript *after* the page loads. Our server-side `fetch` retrieves the raw pre-hydration HTML, where the `<span itemprop="name">` audience is empty — no JSON-LD, no `__NEXT_DATA__`, nothing. The value a human sees in the browser simply wasn't in the bytes our scraper received. And the newly-automated nightly ingest had faithfully re-scraped the empty source, overwriting good data with "0–17" everywhere — which is what surfaced the break.

**What I was going to try.** Extend the LLM age-inference we already run for Play Frisco to Frisco Library — infer the age band from title + description. Robust, but it adds a per-event cost and an inference dependency to reconstruct data that, in principle, still existed.

**What we explored instead — and what unlocked it.** The product owner pointed out the obvious-in-hindsight fact: the "Suitable for:" value *is* visible in the browser. That reframed the problem from "the data is gone" to "the data moved to client rendering." So instead of approximating it, I drove a real headless browser to the event page and watched its network tab — and found BiblioCommons' own **unauthenticated JSON API**: `GET /events/events/{id}?client_scope=events` (with `Accept: application/json`) returns `definition.audience_ids`, and `/events/event_audiences` returns the id→name taxonomy (six stable audiences — the same IDs as the original audience feeds). Validated instantly: Family Story Time → `[Children (0-5)]`; D&D for Adults → `[Adults]`.

**How it helped.** Re-pointing the scraper to the API is strictly better than the LLM path: **deterministic, free, authoritative** (the real data source with stable IDs), decoupled from fragile presentational HTML — and it came with a cleaner description field and a `featured_image_id` hook usable for event images later. We avoided spending money and adding an inference dependency to rebuild data the site was already handing its own front-end.

**What this forced.** A durable habit: when a scrape suddenly returns empty, first decide *is the data gone, or did it move to the client?* — a field visible in the browser but absent from `curl` is the tell — and before reaching for an LLM to reconstruct missing data, check whether the site's own front-end fetches it from an API you can call directly. The network tab is the shortcut past HTML scraping.

**Summary.** "Our kids-events app suddenly started showing adult events and the age filters went dead. Root cause: the source library site had moved to client-side rendering, so the age field a human sees in the browser was no longer in the HTML our server-side scraper fetched — every event defaulted to 'all ages.' I was about to solve it with LLM inference, which we already use elsewhere — but the smarter move came from a simple reframe: the data's still visible in the browser, so it isn't gone, it just moved. I opened the network tab, found the site's own JSON API feeding that field, and pointed our scraper at it. Deterministic, free, and authoritative — no model needed. The lesson I carry: distinguish 'the data is gone' from 'the data is client-rendered,' and reverse-engineer the authoritative source before you approximate it. Knowing when *not* to use an LLM is part of using AI well."

---

## 18. Every Event Was 5–6 Hours Early — a Timezone Bug the Automation Activated (the **write** path)

> *Not to be confused with #14 (the read/query boundary) or #21 (the fail-closed response). See the distinguishing note under #14 — the stored value itself was wrong here, which is why no query fix could have helped.*

**The challenge.** I opened a finished event page on production and saw *"Friday, August 14, 2026 at 5:00 AM"* on a Family Story Time. It wasn't one event — every Frisco and Plano event was landing 5–6 hours early. The app was live and publicly shared, so real people could have planned a morning around a time that didn't exist. For a listings product whose whole job is "trust these details," a *wrong* time is worse than a missing one: it gets trusted, acted on, and betrays the trust.

**What caused it.** `new Date("August 14, 2026 10:00 AM")` resolves an **offset-less** string in the **runtime's** timezone. Three of my four sources publish exactly that shape (bare wall-clock, no offset). On my Central dev machine the result was correct — so it passed every manual check I ever did. But the nightly ingest had moved to **GitHub Actions on 2026-08-12**, and Actions runners are **UTC**. The automation that fixed my staleness problem quietly began writing every event 5–6 hours early. The timezone assumption and the automation were each individually fine; the *combination* wasn't, and nothing in the pipeline was looking at a clock. (Plano was its own trap: its feed stamps `+0000` on times that are plainly local — a 9:30 AM storytime published as `09:30:00 +0000`. The old code was *right* to distrust that bogus offset; it just handed the naive remainder to the machine's timezone instead of the venue's.)

**Why it stayed hidden — the four reasons that actually matter.**
- **It was invisible everywhere a human looked.** Every manual check ran on a machine in the venue's timezone. The only environment that produced wrong data was the one nobody watches — the cloud runner.
- **The trigger was a change to the *environment*, not the code.** The timezone assumption was latent from day one; moving ingest to Actions *activated* it. Nobody re-audits runtime assumptions when the runtime changes — the diff was "where it runs," and the review question was only "does it still run?"
- **I fixed an instance and called it a class.** Two days earlier I'd fixed a *different* timezone bug on a new source (Kaleidoscope's misconfigured `utc_*` field) and written it up as a playbook principle — *don't trust a source's UTC.* I never audited the other three sources against that shape. **A principle recorded but not applied retroactively is a note, not a control** — and this is the miss that stings, because the answer was already written down.
- **Every guardrail ran *after* the write.** My data-quality gate does turn the pipeline red — but only once production is already serving the wrong times. **Detection is not prevention.**

**The fix — two layers.** (1) *Prevention:* `parseCentralWallTime` / `centralWallTimeToUtc` resolve every source time as `America/Chicago`, DST-aware (CDT shifts 5h, CST 6h) — timezone now comes from the venue, never the machine. (2) *Detection, hardened:* `implausiblyEarlyEvents` / `startTimeChecks` in `data-quality.ts` — nothing a family attends starts before 7 AM Central; one stray outlier is tolerated (5% threshold), a whole source shifting is not, and the failing line names the source. Exact-midnight `00:00` is excluded so legitimate all-day markers (library closures, civic observances) don't false-red. The important rule is the **uniform-shift** signature: a real reschedule moves *one* event by an arbitrary amount, a clock bug moves *every* event by an identical amount — so the check catches shifts I haven't thought of (a 1-hour DST error, an off-by-one-day parse) **even when they land at plausible hours**. And CI now runs the unit suite **twice — `TZ=UTC` and `TZ=America/Chicago`** — because a suite that only runs in one timezone can't catch a bug that depends on the timezone. An explicit escape hatch (`INGEST_ALLOW_TIME_SHIFT=1`) permits the intended mass correction — the re-ingest that *fixes* a timezone bug is supposed to move everything at once.

**What this forced.** Two durable habits. First: **when you learn a lesson from one source, audit every other source for the same shape *that day*** — otherwise the write-up is just a nicer way of being surprised twice. Second: **put the strongest check on the output the user actually sees, and in front of the write, not behind it.** A gate that runs after the write can only tell you *how long* you've been wrong.

**Summary.** "My kids-events app started showing every event 5–6 hours early — a 10 AM story time read as 5 AM, on a live site. Root cause was a classic: the code parsed each source's local wall-clock time without a timezone, which resolves in the *runtime's* zone. It was correct on my Central dev machine, but I'd just moved the nightly job to a UTC cloud runner — so the automation that fixed staleness quietly started writing wrong times. What I take from it isn't 'timezones are hard.' It's three things: a bug can be invisible everywhere a human looks and only appear in the one environment nobody watches; the trigger can be an *environment* change with no code diff, so 'does it still run?' is the wrong review question; and I'd actually fixed this exact shape on another source days earlier and never audited the rest — a lesson you write down but don't apply backward is a note, not a control. I fixed it by pinning every time to the venue's timezone, and I added a guardrail that flags a *uniform* shift across a source — the fingerprint of a clock bug versus a real reschedule — moved in front of the user-visible output. Detection after the write only tells you how long you were wrong."

---

## 19. The Ingest Outgrew Its Serverless Function — and the Green Run That Wrote Nothing

**The challenge.** The nightly ingest started as a Vercel scheduled function, but a full multi-source run — scrape four sources, then LLM-classify the new events — takes far longer than a request-scoped serverless function is allowed to live (a 10-second ceiling on the plan). The function hit the wall mid-run. The worst part wasn't the crash; it was that a timed-out or partial run didn't announce itself — the schedule looked like it had fired, and the data just quietly went stale or half-updated. **A green-looking scheduled run that actually wrote nothing is worse than a hard failure, because nothing tells you.**

**What caused it, and the fix.** The workload is a *batch job*, not a web request — the wrong shape for a function metered by seconds. I moved ingest to a **GitHub Actions scheduled workflow** (`0 11 * * *` + manual `workflow_dispatch`), with **one independent job per source** (`matrix` + `fail-fast: false`) so a slow or broken source can't block the others, each with isolated logs. Two failures on the way, both instructive:
- **Exit 2 — secrets invisible.** I'd added the keys as *Environment*-scoped ("Production") secrets, which job-level `${{ secrets.* }}` can't see. Fix: declare `environment: Production` on the job so the environment's secrets are in scope. The lesson: *where* a secret is scoped is as load-bearing as its value.
- **Exit 1 — the runtime moved under me.** `@supabase/supabase-js`'s `createClient()` threw *"Node.js 20 detected without native WebSocket support."* The default runner was Node 20; a global `WebSocket` is first stable in Node 22. Fix: pin `node-version: 22`. (Same family of lesson as the timezone bug in #18 — a dependency's behaviour depends on the runtime, and the runner is not your laptop.)

**What this forced.** A scheduled job needs a way to **shout when it fails**, or a silent green run masks staleness for days. So a `notify` job opens (or comments on) a **GitHub Issue** whenever a source job or the data-quality gate fails — chosen over SMTP/Slack because it needs no new secrets (`GITHUB_TOKEN` suffices), GitHub emails the owner, and an issue has to be *closed*, not just skimmed past at 6 AM.

**Summary.** *"My nightly data ingest outgrew its serverless function — a multi-source scrape-plus-LLM-classify job can't finish inside a 10-second request timeout, so runs were silently timing out and the data went stale without any alarm. The real fix was recognising it's a batch job, not a web request, and moving it to a scheduled CI workflow with one isolated job per source. Two failures taught me more than the move: environment-scoped secrets aren't visible to job-level references, and a dependency needed a newer Node than the default runner — both are 'the runtime isn't what you assumed' bugs. And I learned that a scheduled job's most important feature is that it shouts when it fails; a green run that quietly wrote nothing is the dangerous state."*

---

## 20. The Real Scale Test Isn't the First Hard Source — It's Whether the Second Is Cheaper

**The challenge.** My first sources were archaeology — a dead public API, a reverse-engineered feed, a page that renders its data in JavaScript so a scraper sees nothing (see #17). Each was days of bespoke work. For a data-aggregation product, that raises the existential question: does every new city/source cost the *same* slog, or does the cost *fall*? If it doesn't fall, there's no product — just an endlessly growing pile of fragile one-offs. So when I added a brand-new source — a large local park's events — I treated it as a deliberate test of the expansion thesis, not just another integration.

**What made it cheaper — and why that's the whole point.** Two things, both built *before* the source:
- **A written onboarding playbook** (`SOURCE-ONBOARDING.md`, 7 principles). The park's one surprise — its REST API is `403` WAF-blocked to a bare request but returns `200` with `Accept: application/json` + a browser UA + `Referer` — didn't cost me an afternoon of confusion; it just became the next line in the playbook. Surprises turned into documented steps instead of fresh archaeology.
- **A shared LLM classifier** (`classifyEvents`, extracted from the existing Play Frisco path in the same change). I wrote **zero new rules** about what counts as a "kids event" for this source. The same classifier read a brand-new, mixed-audience calendar and correctly **hid a "Pop & Pour" wine night** — reasoning on file: *"explicitly requires guests to be 21+"* — while keeping the story times and festivals. A hand-tuned keyword list would have needed a rewrite per source; the model generalised.

Net: **103 events → 84 kid-facing** on production, and the new source was mostly *wiring*. Adding my second *kind* of source cost **less** than the first, not more — and re-running the old source afterward made **0 extra LLM calls** (the classifier extraction was regression-free).

**Summary.** *"For a data-aggregation product the moat isn't the first integration — anyone can grind through one — it's whether the tenth is cheap. So I treated adding a new source as a test of that. It was mostly wiring, for two reasons I'd built on purpose: a written onboarding playbook that turned each new source's quirk into a documented step instead of fresh research, and a shared LLM classifier so I wrote no new 'is this for kids?' rules — the same model read a brand-new calendar and correctly hid a 21-plus wine event while keeping the family ones. The rules don't generalise; the model does. That's the difference between a product that scales and a pile of one-off scrapers."*

---

## 21. The Write Path That Refuses to Publish — Encoding "Fewer Events Beats Wrong Events"

**The challenge.** After the timezone incident (#18) I had a data-quality gate that turned the pipeline red on bad data. It worked. It was also, on inspection, the wrong *shape* for the problem: **it ran after the write.** Production had already served wrong times for hours by the time anything went red. A detection layer can only tell you *how long you were wrong* — and for a listings product that's the wrong currency, because the damage happens when a parent reads the page, not when I read the dashboard. The product rule I'd already written down — *a wrong event time is worse than a missing event* — had no enforcement point anywhere near where it could actually be violated.

**The change — the write path is now fail-closed.** All four ingest runners write through a single `guardedUpsert` (`src/lib/ingest-guard.ts`) that screens the batch **before** it touches the database. Three independent rules, any of which aborts:

| Rule | Catches | On trip |
|---|---|---|
| **Implausible start** — no event between 12:01 and 7:00 AM CT (exact midnight = all-day, allowed) | individually broken times | that event is **dropped**, never written |
| **Uniform shift** — ≥80% of overlapping events moved by the *same* non-zero offset | the whole class of clock bugs | **whole batch rejected** |
| **Shrink** — batch < half the stored set | a partial or failed scrape read as "events cancelled" | **whole batch rejected** |

Three design details carry most of the judgment:
- **On abort, the purge/cleanup steps are skipped too.** Play Frisco deletes stored events absent from today's batch — so without this, a *rejected* batch would still delete good rows and empty the source. Rejecting the write while still deleting isn't fail-closed; it's the worst of both.
- **The uniform-shift rule is the important one.** A genuine reschedule moves *one* event by an arbitrary amount; a clock bug moves *every* event by an identical amount. That signature catches shifts I haven't thought of — a 1-hour DST error, a source changing timezone, an off-by-one-day parse — **including ones that land at perfectly plausible hours**, which no plausible-hours rule can ever see. It generalises past the incident that inspired it.
- **An explicit escape hatch** (`INGEST_ALLOW_TIME_SHIFT=1`), because the re-ingest that *fixes* a timezone bug is supposed to move everything at once. It forgives only the shift — a batch of implausible times stays blocked even with the hatch set. A guard with no legitimate override gets disabled wholesale the first time it's inconvenient.

**Verification.** Drilled against the real database before shipping: replaying the incident aborts on all three rules (`uniform -300min shift across 100% of 178 existing events`), a simulated partial scrape aborts, and a healthy re-scrape writes normally. A test also asserts there is exactly **one** `events.upsert` in the ingest module, so a future runner can't quietly bypass the guard.

**What this forced.** The general principle: **if bad data must never reach users, the check has to sit in front of the write and be willing to publish nothing at all.** Choosing "fewer events" over "wrong events" is a *product* decision — and a product decision that lives only in a document isn't enforced. It has to be encoded where the violation would occur.

**Summary.** *"After a bad-data incident I had a quality gate that turned the pipeline red, and I realised it was the wrong shape — it ran after the write, so it could only tell me how long production had been wrong. I moved the guard in front of the database write and made it willing to publish nothing: three rules, any of which aborts the batch. The one I'm proudest of is a uniform-shift check — a real reschedule moves one event by an arbitrary amount, a clock bug moves every event by an identical amount, so that signature catches whole classes of bug I hadn't imagined, including ones that land at perfectly plausible times. Two details matter as much as the rules: on abort I skip the cleanup step too, or a rejected batch would still delete good rows; and there's an explicit override for an intended mass correction, because a guard with no legitimate escape hatch gets switched off the first time it's inconvenient. The real point is that 'a wrong time is worse than a missing event' is a product decision, and one that only lives in a document isn't enforced — it has to be encoded where it can actually be violated."*

---

## 22. I Built an Alert to Tell Me When Things Broke. It Was Broken — Three Times.

**The challenge.** With the write path now fail-closed (#21), the weakest link moved. A rejected batch or a failed data-quality check was only a **red tab nobody was watching**. The pipeline could correctly refuse bad data and still leave the site quietly stale for days — I had traded "wrong data reaches users" for "nobody finds out we stopped updating." So I built a `notify` job: when any source job or the quality gate fails, open a GitHub Issue with triage instructions. No new secrets (the built-in token suffices), and an issue has to be *closed* — an email at 6 AM is easy to skim past.

**Then the honest problem.** An alert only ever executes when something else is already broken. Until it has actually fired, you have written code, not installed a control — and by my own rule from Concept Q, a guard you have not seen fail is not a guard. So I added a **fire drill**: a manual-only workflow input that fails one job on purpose while skipping the real ingest for *every* source, so nothing is scraped, classified, or written. Scheduled runs are untouched.

**What the drills found — three separate defects, none visible in code review.**
- **Drill 1.** The job died with a 503 while creating its *label* — a cosmetic step — and never reached the code that opens the issue. **A decorative call took down the entire alert.** No label, no issue, nothing.
- **Drill 2.** The issue was delivered, but **unlabelled** (the platform rejected the label it had just created — propagation lag). Meanwhile my deduplication looked up existing issues *by label*, so it would never have found that issue again: every future failure would have opened a duplicate, burying the inbox the alert exists to protect.
- **Drill 3.** The lookup worked, then **commenting on the existing issue failed** three times and delivery simply gave up. The issue sat there with zero comments.

**The correction that mattered — I had the architecture backwards.** Three failures on the same API is not bad luck; it is a **flaky dependency being treated as a reliable one**. And the channel that worked flawlessly through all three drills was the one I had not written: the platform's own built-in workflow-failure email. It needs no code and cannot be broken by my logic. So I inverted the design — **that email is the primary alert; my issue is a best-effort durable record layered on top**, with delivery that *falls forward* (comment → new labelled issue → new unlabelled issue) rather than failing. For an alert specifically, the tradeoff inverts from the rest of this product: everywhere else *missing* beats *wrong*, but here **a duplicate beats a silence**.

**The proof it was worth doing.** The drill-3 email arrived listing `alert on failure — Failed`. The failure of the secondary channel was **visible through the primary one** — the property that makes layered alerting actually sound, and I only know it holds because I forced it to happen.

**What this forced.** Each of the three failures is now a permanent regression test that extracts the alert script from the workflow file itself (never a copy, so it cannot drift) and replays them against mocked APIs — verified to fail when the fall-forward logic is removed. And a habit: **for anything that only runs in the failure case, the drill is part of building it, not a follow-up.**

**Summary.** *"After I made my data pipeline refuse to publish bad data, the weak point moved: a rejection was only a red tab nobody watched. So I built a failure alert — and then, because an alert only runs when something else is already broken, I forced it to run. Three deliberate drills produced three different bugs in the alert itself: it died on a cosmetic label call before it could alert; then it delivered an alert my own dedup could never find again; then it gave up when one API call failed. The fix I care about is not any of those three — it is realising I had been treating a flaky dependency as reliable, and that the channel which never failed was the one I had not written: the platform's built-in notification. I made that primary and my clever version the enhancement, with delivery that falls forward instead of failing, because for an alert a duplicate beats a silence. Then I turned all three failures into regression tests. The transferable rule: a control that only executes during an incident is unproven until you deliberately cause one."*

---

## 23. The Most Dishonest Sentence in My Product — Writing Down What the User Sees When It Breaks

**The challenge.** My governance review flagged **fallbacks** as a missing instrument. I nearly dismissed it as paperwork: every fallback behaviour *already existed* in the code — an LLM failure hides the event, an unparseable time skips it, a rejected batch keeps the previous rows. I was just going to write them down. The exercise has one rule that makes it more than documentation: every row is a failure scenario, and one column is **exactly what the user sees** — so *a row you cannot fill honestly is an undefined behaviour you have just found.*

**What I found — six of them, on the first pass.** The worst was this: when the database was unreachable, the app displayed **"No events match your filters."** The request failed, the client fell through to an empty list, and the empty list rendered the filter message. So a parent looking for something to do on Saturday is told *their filters* are wrong, and goes off adjusting filters that were never the problem. **We were blaming the user for our own outage.** Others in the same family: a thrown network request left the loading spinner running forever, because the code that cleared it only ran on the success path; a failed "Attending" save left the interface still claiming it had saved; a map failure produced an empty map with no explanation.

**The category this named.** None of these were correctness bugs. The data was right, the pre-write guard was doing its job, and **every test was green** — because every test I had written asked "does the happy path work?" These are a distinct class: **honesty-of-failure** bugs, where the product degrades without telling the truth about *why*. They are invisible to correctness testing by definition, and they occur at exactly the moments a user is most likely to lose trust.

**The fix — and the distinction that was the whole bug.** A failed request and a genuinely empty result had been **one state**; they are now two, with different messages and different *actions*: *"We couldn't load events right now"* + **Try again** (filters preserved) versus *"No events match your filters"* + **Clear filters**. The error copy exists specifically to stop the wrong conclusion the old message caused. The spinner clear moved into a `finally` so no failure path can hang it. A failed engagement save now **reverts the optimistic toggle** rather than leaving the interface asserting something untrue — optimistic UI is good, but when the save fails, leaving it is a lie. Three automated tests now assert the two states stay distinct, verified by reintroducing the original bug and watching them fail.

**A methodological note worth keeping.** My first attempt to prove those tests were not vacuous *passed* — because my edit had removed the wrong guard, and my script reported success without verifying the edit applied. The tests were fine; my verification of them was not. **Checking that a check works is itself a step that can silently succeed.**

**What this forced.** A habit and a bias. The habit: when specifying a feature, write the failure column *in the user's words* before writing the happy path — it is the sentence you never think about and the only one they will read. The bias: prefer saying "this is on us" over a generic empty state, because a message that misattributes blame does not merely fail to help, it actively sends the user to waste effort in the wrong place.

**Summary.** *"A governance review told me my fallback documentation was missing, and I almost dismissed it — every fallback already existed in code, I was just writing them down. But the format forces one column: what the user actually sees. Six rows in, I could not fill it honestly. My app told users 'No events match your filters' when the database was down — blaming a parent's filters for my outage and sending them to adjust filters that were never the problem. Five more like it: a spinner that ran forever on a network error, a save that failed while the UI kept claiming success. None were correctness bugs; the data was right and every test was green, because my tests only asked whether the happy path worked. I would call these honesty-of-failure bugs — the product degrading without telling the truth about why. I split failure from empty into two states with different messages and different actions, made the failing save revert instead of lying, and added tests that keep the two states distinct. The lesson: writing the failure column in plain user language is not documentation, it is design — and if you cannot fill it, you never designed that behaviour."*

---

## 24. The Spend Cap That Would Have Hidden Events Forever

**The challenge.** My governance audit (Concept R) left exactly one instrument fully missing: a **cost cap**. Nothing bounded paid LLM spend. Normal cost was already near-zero and structurally so — classification is nightly, batched, and cached, so re-running an unchanged source makes **zero** model calls. What was unbounded was the *anomaly*: a source that suddenly returns 10,000 events (a changed API default, a pagination bug, a bad date window) would have been classified in full, at my expense, with nothing to stop it.

**The straightforward part.** A per-run ceiling (`MAX_LLM_CALLS_PER_RUN`, default 300), set far above normal volume so tripping it means *something is wrong*, not *we are busy*. Raising it is a deliberate act, and a malformed value falls back to the default rather than silently disabling the cap — the dangerous failure mode for a limit is not "too low", it is `Number("abc") → NaN → no limit`.

**The defect I nearly shipped — and what caught it.** On refusing a call, my first version set `kid_relevant = false`, the same fail-closed value used when the LLM *errors*. Writing the integration path, not the unit tests, exposed that this is wrong in **two opposite directions**:
- **`false` poisons the cache.** The next run reads `prior.kid_relevant !== null`, treats it as a *decision already made*, and hides that event **permanently** — even after the cap is raised. My cost control would have quietly and irreversibly deleted coverage.
- **`null` fails open.** `kid_relevant IS NULL` *passes* the events API gate — that is how library events, which have no LLM inference at all, flow through. An unclassified event would have been **shown**.

Neither value is safe **if the row is written at all**. The fix was to stop writing it: budget-skipped events are excluded from the batch entirely, so they are neither displayed nor cached, and get classified normally on the next run. Fail-closed *and* self-healing.

**The generalisable insight.** The two situations look identical in code — "we do not have a classification for this event" — but they are opposite in meaning. An LLM *error* is a **decision**: we tried, it failed, hide it. A budget skip is a **deferral**: we never looked. A cache cannot tell those apart, so writing the same value for both is what turns a cost control into permanent data loss. **When you add a skip path, ask what the next run will believe about the rows it leaves behind.**

**Verification.** 14 unit tests, including a structural guard that there is exactly one paid call site and that it is gated — proven non-vacuous by deleting the gate and watching two tests fail. Then a live run with `MAX_LLM_CALLS_PER_RUN=0` against the real source: **37 events, 0 paid calls, not flagged as capped**, because every event was a cache hit. That was the integration property most worth checking — if the cap had counted cache hits, every nightly would have gone red.

**What is still open, deliberately.** The in-code cap bounds **calls**; the dashboard figure is `calls × unit price`, an **estimate, not metered spend**. So it is a dashboard number, not a control — the account-level spend limit (Console → Settings → Billing → *Spend limits*) is the outer backstop that does not depend on my arithmetic being right. Recorded as an open gap rather than quietly counted as covered.

**Summary.** *"The last gap in my governance audit was a cost cap — nothing bounded LLM spend. Normal cost was already near zero because classification is batched and cached, so the cap was never about normal volume; it was about the anomaly, a source suddenly returning ten thousand events. The counter was the easy part. The real problem was what a refused call leaves behind: my first version marked the skipped event 'not for kids', which is the correct value when the model *errors* — but a budget skip is not a decision, it is a deferral, and the cache cannot tell those apart. It would have read that placeholder as an answer and hidden the event permanently, even after I raised the cap. Writing null instead would have failed open, because a null classification passes my API gate. So the answer was to not write the row at all — the event is withheld and re-tried next run. The lesson I carry: when you add a skip path, ask what the next run will believe about the rows it leaves. And I kept one gap open in the doc rather than claiming it: my spend figure is an estimate, not metered, so the real dollar backstop is the account-level limit, not my arithmetic."*

---

# Technical Concepts & Talking Points

*Not project obstacles — foundational concepts worth being able to explain crisply. Same format: the concept, then a summary.*

> **Framing note — the governance arc (use this when asked "tell me about a failure").** The August 2026 incidents aren't separate lessons; read in order they're a **six-step progression in how I think about governance**, which is the more interesting answer:
> - **Aug 13** (#17) — a source changed shape and every test stayed green. Lesson: *guard the output, not the process.* Built the post-ingest data-quality gate (Concept L). **But it ran after the write.**
> - **Aug 14** (#18) — a second incident, plus that new guard's own first contact with real data, where it flagged 11 legitimate rows. Lesson: *the guard itself is a thing that can be wrong* (Concept Q).
> - **Aug 15** (#21) — moved the guard **in front of** the write and made it willing to publish nothing. Lesson: *detection is not prevention.*
> - **Aug 17** (#22) — drilled the new alert deliberately; it failed three times, in three different places. Lesson: *a control that only runs during a failure is unproven until you force it to run* — and I had the channel hierarchy backwards.
> - **Aug 17** (#23) — wrote down what the user actually sees for every failure. Lesson: *correct data is not the same as an honest product.* Found six undefined behaviours, including an outage that blamed the user’s filters.
>
> - **Aug 19** (#24) — closed the last missing instrument, a spend cap, and nearly shipped it writing a placeholder that would have cached a decision the model never made. Lesson: *a skip is not a decision — ask what the next run will believe about the rows you leave behind.*
> Each step was forced by the limitation of the one before it. The claim isn't "I learned a lesson" — it's that the thinking compounded under pressure, with dates and code to show for it. This is also the honest, current answer to the "Governance was my under-invested layer" line in Concept I.

---

## A. Secrets, API Keys & Environment Configuration

**The concept.** Everything in the app's `.env.local` file is **environment configuration** — a mix of **environment variables**, **API keys**, and **secrets**. The practice is called **secrets management / configuration management**, and the guiding principle is the **12-Factor App** rule: *store config in the environment, not in code*. That's why the file is **git-ignored** (never committed) and the same values live as **environment variables on the hosting platform** (Vercel) in production.

**The keys, named properly:**

| Env var | What it's called | Secret or public? | What it is |
|---|---|---|---|
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | **Anonymous / publishable key** | Public (safe in browser) | Limited-privilege key, scoped by **Row-Level Security (RLS)**; safe to ship to the client |
| `SUPABASE_SERVICE_ROLE_KEY` | **Service-role / secret key** | **Secret** (server only) | Full-privilege key that **bypasses RLS**; used only in server code |
| `GOOGLE_MAPS_API_KEY`, `ANTHROPIC_API_KEY` | **API keys** | Semi/secret | Credentials that authenticate the app to a third-party API |
| `GCP_SA_KEY_B64` | **Service-account key** | **Secret** | A **service account** = a non-human identity for **machine-to-machine (M2M) auth** to BigQuery; JSON credential, base64-encoded to fit an env var |
| `CRON_SECRET` | **Shared secret / bearer token** | **Secret** | A password the cron job sends to authenticate itself to `/api/ingest` |
| `NEXT_PUBLIC_GA_MEASUREMENT_ID` | **Identifier**, not a secret | Public | Just names the GA4 property |

**The concepts an interviewer probes:**
- **Public vs. secret config** — the `NEXT_PUBLIC_` prefix means "bundle this into the browser." Publishable keys get it; secrets never do (they'd leak to every visitor).
- **Principle of least privilege** — the two Supabase keys are the textbook example: browser gets the minimum (anon + RLS), the server gets the powerful key and it never leaves the server.
- **Authentication vs. Authorization** — the *key* authenticates (who is calling); **RLS / scopes** authorize (what they can do).
- **Machine-to-machine auth / service accounts** — the GCP key: the app acting as its own non-human identity, not on behalf of a logged-in user.
- **Secret rotation & storage** — secrets should be **rotatable** and stored in a **secrets manager** (Vercel env vars), never in the repo; if leaked, you **rotate** them.

**Summary.** *"I separated config from code following 12-factor: public identifiers and publishable keys are exposed to the client via a `NEXT_PUBLIC_` prefix, while secrets — the Supabase service-role key, the Anthropic API key, the GCP service-account credential, and the cron shared-secret — stay server-side, git-ignored locally and stored as environment variables in Vercel for production. The two Supabase keys are a clean least-privilege example: the browser gets an anon key constrained by row-level security; the server gets the privileged key that never leaves the backend."*

---

## B. Connecting the Database via MCP (Model Context Protocol)

**The concept.** **MCP (Model Context Protocol)** is an open standard that lets an AI client (Claude Code, claude.ai, Cursor) connect to external tools and data sources through a uniform interface. Supabase publishes an **official Supabase MCP server**: once connected, the agent gets tools to **inspect schema**, **run SQL queries**, **apply migrations**, and **generate types**. It authenticates with a **personal access token (PAT)**, can be **scoped to one project**, and run **read-only**.

**Important distinction.** This is a **development / debugging** connection between the *agent* and the database — it does **not** change the product. The app itself still talks to Supabase through its SDK. (Open Eventz has **no MCP in its runtime**; the app uses direct SDKs — Supabase, the Anthropic API, BigQuery, Google Maps.)

**Where it would help this project.** Much of the debugging was *"what's actually in the database?"*, answered indirectly by running the app or loading the dashboard. A Supabase MCP would let the agent query directly — confirm event counts, verify that migrations ran, check `price_class` / `price_confidence` distributions after re-ingest, or diagnose the earlier "Play Frisco 0 / Total 1000" count bug straight from SQL.

**The security caveat (the part interviewers reward).** Giving an AI agent direct DB access is powerful and risky, and knowing the mitigations is the signal:
- **Read-only mode** by default — inspect, don't mutate/drop.
- **Project scoping** — one project, not the whole org.
- **Least privilege** — don't wire it up with god-mode credentials (same principle as the service-role key).
- **Prompt-injection / blast radius** — if the agent reads a row whose *contents* contain instructions (e.g., a malicious event description), a naïve setup could act on them. Read-only + scoping limit the damage. This is a documented class of risk for database MCPs.

**Summary.** *"Yes — Supabase ships an official MCP server, so an AI client can connect to the database to inspect schema, run queries, and manage migrations. I'd treat it as development tooling, not part of the product — the app still uses the SDK. And I'd run it read-only and scoped to a single project specifically because of prompt-injection and blast-radius concerns: handing an agent write access to a production database is a security decision, not just a convenience one."*

---

## C. Securing an AI endpoint — the cost-DoS vector

**The concept.** A normal unauthenticated endpoint that returns data is low-stakes. An endpoint that calls a **paid LLM** on every request is not: leave it open and anyone can script requests to run up your model bill — a **cost-based denial-of-service**. Open Eventz had exactly this — `/api/infer-age` (a testing wrapper over the inference function) was unauthenticated and called Claude. Fixed by gating it behind the same shared-secret bearer token as the ingest endpoint (401 without it). Two lessons: (1) **AI endpoints carry a cost attack surface** ordinary CRUD routes don't — audit every route that *spends money* or *writes data* for auth; (2) **security-through-obscurity isn't security** — the vulnerability lived on the deployed app regardless of source visibility; publishing the code just lowers the bar to discover it.

**Summary.** *"Before open-sourcing the code I audited every endpoint for auth and found a testing route that called the paid LLM unauthenticated — an open cost-DoS vector — and gated it. The insight is that AI endpoints have a cost attack surface normal endpoints don't, and the risk was on the live app, not the source: publishing code doesn't create the vulnerability, it removes the obscurity that was accidentally hiding it."*

---

## D. Row-Level Security & least privilege

**The concept.** Supabase exposes tables through an auto-generated API reachable with the **anon key** — which is **public by design** (it ships in the browser bundle). So the gate that actually protects the data isn't repo privacy, it's **Row-Level Security (RLS)**: per-table policies deciding what the public key can do. With RLS off, anyone with the anon key can read/write/delete. The fix is **least privilege per table**: `events` = public read-only; `like_counts` / `supervision_policies` / `ingest_runs` = fully locked (only the server's service-role key, which *bypasses* RLS, touches them). Follow-on lesson: the RLS **sweep must cover every exposed table** — the first migration fixed the three tables a linter flagged but missed `ingest_runs`, which a later advisory caught. Re-run the advisor after any migration that adds a table.

**Summary.** *"The Supabase anon key is public by design, so repository privacy is irrelevant to data security — Row-Level Security is. I enabled RLS on every exposed table with least-privilege policies: the public key can only read the events table, everything else is locked to the server's service role. And I learned to re-run the security advisor after every schema change, because my first pass missed a table a later migration added."*

---

## E. A green local hook ≠ a green pipeline

**The concept.** Local **git hooks** (pre-commit/pre-push) and **cloud CI** run the same commands in **different environments**. Open Eventz's unit-test CI job was red on every push while the local hooks passed — because `jest.config.ts` (a TypeScript config) needs `ts-node` to parse, `ts-node` wasn't a declared dependency, and the local `node_modules` happened to have it while CI's clean `npm ci` didn't. Classic "works on my machine." The fix removed the hidden dependency (config → plain `.js`), but the discipline is the point: **verify against a clean environment and check the actual CI run — a passing local hook is not proof the pipeline is green.**

**It happened twice — and the second time I fixed the process, not just the bug.** A large UI reskin later passed the *same* pre-push hook (typecheck + unit, both green) but turned **CI red on E2E** — because the Playwright smoke suite was **CI-only**, so the one layer that tests the UI I'd just rewritten wasn't in my local gate. Three assertions were coupled to changed presentation (a card label, a chip colour, a `button.rounded-full` selector). The fix wasn't just repairing them: I **moved E2E into the pre-push hook but guarded it to skip gracefully when no browser is installed** — E2E had been dropped from the hook originally *because* a missing local browser caused false failures, so a naïve "always run E2E" would just re-introduce that. Run-when-possible / skip-gracefully lets you tighten the gate without re-breaking it.

**Summary.** *"A green local hook isn't a green pipeline — and I learned it twice. First, a TypeScript config needed a dependency my machine had but CI's clean install didn't. Second, a UI reskin passed my pre-push hook because it only ran typecheck and unit tests — the E2E suite that would've caught it was CI-only, so it went red after I'd already merged. What I'm proud of is the second fix: instead of just repairing the tests, I moved E2E into the pre-push hook but guarded it to skip gracefully when no browser is installed — so the local gate now covers the layer most likely to break, without the false failures that made me leave it out before. Your local gate has to include the layer that tests what you actually change."*

---

## F. Doc↔test parity — keeping the test plan honest

**The concept.** A test plan that says *"scenario X is covered by test file Y"* is only trustworthy if something enforces it — otherwise the doc quietly rots as tests are renamed or deleted. Open Eventz added a tiny **CI `doc-parity` job**: it parses the consolidated scenario doc, extracts every test file each scenario claims, and **fails the build if any named test no longer exists**. That turns "we think this is covered" into "CI proves the named test still exists" — engineering-quality rigor a PM rarely brings.

**Summary.** *"I consolidated the functional test scenarios into one plan and added a CI check that fails the build if a scenario names a test file that no longer exists — so the PM test plan can't silently drift from the actual suite. A test plan that claims coverage is only credible if something enforces the claim."*

---

## G. Reading `npm audit` with judgment

**The concept.** `npm audit` reports every known vulnerability in the dependency tree — but the raw count ("7 high severity") conflates **exploitable** risk with **transitive-dependency noise**. Open Eventz's high-severity flags were all in the framework's bundled `sharp`/libvips (image processing) and `postcss` — and the app uses **no `next/image`**, so the `sharp` code path never runs (unreachable), while `postcss` runs only at build time. The *practical* runtime risk was ≈ none even though the number looked alarming. Judgment: **triage by reachability and runtime-vs-build-time, not by the count** — and separate *actual* risk from *optics* (a public repo's audit score a reviewer might glance at).

**Summary.** *"`npm audit` flagged several high-severity issues, but they were all transitive to the framework — an image library the app never invokes because it does no image optimization, and a build-time CSS tool — so the exploitable runtime risk was effectively zero. I treat an audit as input to triage by reachability, not a number to react to, while acknowledging the public-repo optics separately."*

---

## H. Prepping a repo to go public — the sensitivity scan

**The concept.** Making a repo public is effectively **irreversible**: the entire **git history** becomes world-readable forever, not just the current files. So before flipping visibility you scan for two classes of exposure — **secrets** (API keys, tokens, private keys) and **PII** (personal/contact data) — and you scan **history, not just the working tree**, because a secret committed once and deleted later still sits in an old commit. For Open Eventz: grepped tracked files + the full `git log` history for secret patterns (JWTs, `sk-…`, private keys) and PII (emails, phone numbers) across both repos, and **extracted text from the binary files** (`.docx`/`.xlsx`) since a text grep skips them. Result: clean — with one honest residual: a regex can find emails/phones but **can't reliably flag a plain person's name**, so free-text notes in a binary file still warrant a human eyeball. Secondary insight: the **app repo is the higher-risk one** (it holds the real keys locally) — so `.env*` being git-ignored *plus* a clean history is what makes it safe to publish, not obscurity.

**Summary.** *"Before making the repos public I ran a sensitivity scan — secrets and PII — across tracked files **and full git history**, because history is permanent even after you delete a file. I also extracted text from the binary docs, since a grep skips those, and I was explicit that a pattern scan can't catch a plain name in a notes cell — that needs a human check. The riskiest repo was the app one because it holds the real keys locally, so I confirmed the env file was git-ignored and the history was clean before publishing."*

---

## I. The 5-layer AI PM stack — as shipped evidence

**The concept.** A useful frame for AI product work is a **5-layer stack**: **Model** (capability/cost/latency), **Context** (prompts, RAG, what the model sees), **Orchestration** (how the model call fits inside a larger workflow), **Governance** (evals, guardrails, observability), and **Human** (judgment that can't be delegated). It lets you locate any AI-product problem in the right layer instead of "prompt and hope." Open Eventz maps cleanly, with a real artifact per layer: a documented **model** selection (Sonnet-over-Haiku), owned + versioned prompts and a calibration set (**Context**), an ingest pipeline with the LLM call mid-workflow plus fallback (**Orchestration**), a six-layer risk model + confidence tiers + two-tier testing + observability dashboards (**Governance**), and a full documented judgment trail (**Human**). The interview value isn't reciting the layers — it's showing **evidence at every one**, which is rare.

**Summary.** *"I think about AI products as a five-layer stack — model, context, orchestration, governance, human — so a problem lands in the right layer instead of getting solved by prompt-tweaking. In Open Eventz every layer has a shipped artifact: a documented model-selection call, versioned prompts plus a calibration set as owned context, the LLM call embedded in an ingest pipeline with fallback, a governance layer of confidence tiers and two-tier testing and dashboards, and a full human judgment trail. The point isn't the vocabulary — it's having real evidence at every layer."*

---

## J. A reskin is a product decision — presentation vs. trust-sensitive data

**The concept.** A "visual-only" reskin sounds like it can't change behaviour — but a presentation choice *becomes* a product decision the moment it touches trust-sensitive data. During Open Eventz's "Weekend Paper" reskin, the child-supervision "can I drop my kid off?" indicator moved from **colour-coded alarm** (red = no / green = yes) to **one calm, text-driven callout** ("instruction, not alarm — no red-on-pink"). That isn't styling: a wrong "you can drop off" answer must never be *dressed in reassuring colour*, so the meaning has to live in the words, not the hue. The discipline that makes it safe is keeping the resolving logic (`getSupervisionBadge`) and its unit tests **independent of presentation** — a theme swap changes how it looks and never what it decides (proven by the fact the reskin didn't touch that suite). Bonus practice: design feedback was reviewed on a **before/after mockup for sign-off before any code**, so the call was made deliberately, not discovered after shipping.

**Summary.** *"People assume a reskin is cosmetic, but a presentation choice becomes a product decision the second it touches high-stakes data. In my kids-events app, a 'visual-only' refresh was where I decided to stop colour-coding the drop-off-safety indicator — green/red can imply a safety verdict, and the model behind it can be wrong — so I moved the meaning into calm text and kept the logic and its tests independent of the styling. A theme swap can change the look but never the behaviour. And I reviewed the change on a mockup before writing code, rather than eyeballing it after."*

---

## K. Scope a feature by risk, not demand — the accounts decision

**The concept.** "Add user accounts" reads like one feature; it's actually a **step-change in risk**. On a managed auth provider (Supabase) the login itself is a day or two — but you inherit account lifecycle, session security, and the real cost: becoming a **custodian of personal data**, with privacy-law exposure, breach-notification duties, and — for a **kids** product — potential **COPPA** liability the moment any feature stores data *about a specific child*. So the decision isn't "accounts: yes/no," it's **"what's the thinnest identity that unlocks the one feature I actually want?"** For a cross-device saved list, that's **OAuth-only, adults-only, no child data, row-level security locked per user** — ~80% of the value at ~20% of the risk; full password auth + child profiles is where cost *and* liability spike. It's also a conscious crossing of the **portfolio→real-product line** (the project was deliberately scoped as a demo precisely to avoid this obligation surface).

**Summary.** *"When someone asks for 'user accounts,' I scope it by risk, not demand. The login is trivial on a managed auth provider — the real cost is becoming a data custodian, with privacy law and, for a kids app, COPPA if you ever store a child's data. So I ask what the smallest identity is that unlocks the feature: for cross-device saves, OAuth-only, adults-only, no child records, row-level security per user gets most of the value at a fraction of the liability. Recognising that 'accounts' crosses the demo-to-real-product line — and choosing the lightest identity that clears it — is the judgment, not wiring up a login."*

---

## L. Guard the output, not just the process — the post-ingest data-quality gate

**The concept.** Unit and end-to-end tests run on **mocked** data and prove one thing: the *code* matches my intent. They are structurally blind to bad *data* — a source that changes shape, an empty field, a wholesale time shift. A pipeline that reports "success" while writing garbage is **worse** than one that fails, because nothing tells you. So there's a distinct layer whose job is different from testing: after every ingest it asserts invariants against the **real database** and turns the pipeline **red** on violation — age variety, **no adult-titled event stored kid-visible**, the toddler filter actually *narrows*, per-source non-empty, freshness, plausible start times, plus a **live-source canary** that confirms the upstream source still exposes the field we depend on. Two incidents proved the layer earns its place: the client-side-render break (#17), where every event fell to an "all ages" fallback while all tests stayed green, and the timezone shift (#18). The design rule: **point the check at the thing the user actually sees (the output), because every intermediate "success" signal can lie** — and the check belongs *in front of* the user-visible write, not merely reporting after it (detection is not prevention).

**Summary.** *"Logic tests can't catch bad data — they run on mocked inputs and only prove the code does what I intended. But a data product fails when the data is wrong, even if the code is perfect. So I added a layer that runs against the real database after each refresh and asserts what has to be true: ages have variety, no adult event is stored kid-visible, the age filter actually narrows, the sources aren't empty or stale — and it turns the pipeline red if not. A green checkmark should mean the thing you care about is true, not just that the code ran. And I put the guard on the output the user sees, because every intermediate 'success' can lie."*

---

## M. LLM-primary classification with a governance pre-filter — and fail-closed

**The concept.** Deciding "is this event for kids?" started as a **keyword deny-list**. It was brittle in both directions: every new source needed the list re-tuned, and it broke on phrasing it hadn't seen. I inverted the architecture to **LLM-primary** — the model reads the unstructured event text, judges kid-relevance, and **stores its reasoning** (auditable, not a black box). A small keyword pre-filter survives, but only for **governance**: an obvious hard-block layer (an explicit adults-only override), not the primary decision. Two properties make it safe and scalable: it **generalises across sources** — the same classifier hid a *"Pop & Pour"* 21+ wine night on a brand-new calendar with no new rules, where a keyword list would have needed a rewrite — and it is **fail-closed**: an LLM error or a low-confidence result sets `kid_relevant = false` rather than guessing, because a wrong *"this is for kids"* is the expensive direction. The bounding rule pairs with #17: **use an LLM for irreducible ambiguity, deterministic code for structured data that already exists** — never an LLM to reconstruct data a source is already handing its own front-end.

**Summary.** *"My kid-relevance classifier used to be a keyword deny-list — brittle, and it needed re-tuning for every new source. I made it LLM-primary instead: the model reads the messy event text, decides, and stores its reasoning, so it's auditable. I kept a tiny keyword layer, but only as a governance hard-block, not the main decision. Two things make it work at scale: it generalises — the same model correctly hid a 21-plus wine event on a source it had never seen, no new rules — and it's fail-closed, defaulting to 'not for kids' when it errs or isn't confident, because a false 'kid-friendly' is the costly mistake. The judgment I care about is knowing when the LLM is the right tool: irreducible ambiguity, yes; structured data that already exists, no."*

---

## N. Measuring a discovery product — a north star, not vanity metrics

**The concept.** A discovery product's success isn't downloads or pageviews — it's whether a parent actually *found something to do*. So the north star is **weekly active discoverers** (people who engage the funnel), and the funnel is instrumented end-to-end: view → filter/engage → intent → **conversion**, where conversion is defined as *adding an event to a calendar* or *getting directions* — the real "I'll show up" signals, not a like. It's **channel-segmented from day one** (organic/SEO vs. direct vs. referral) so acquisition is a *dimension of the funnel*, not a separate report. And it was wired **before launch** (GA4 → BigQuery → two dashboards: a functional one for the funnel, a technical one for pipeline health + per-inference LLM cost), because you can't retrofit measurement onto traffic you've already lost. The discipline: pick the one metric that means the product did its job, define the steps that lead to it, and instrument them before you need them.

**Summary.** *"For a discovery product I don't measure downloads — I measure whether someone found something to do. My north star is weekly active discoverers, and I instrumented the whole funnel: view, filter, intent, then conversion, which I defined as adding an event to a calendar or getting directions — the real 'I'll show up' signals, not a like. I segmented it by channel from the start, so acquisition is a slice of the funnel rather than a separate dashboard, and I wired it before launch because you can't retrofit measurement onto traffic you've already lost. The judgment is choosing the single metric that means the product worked, then instrumenting the steps that lead to it."*

---

## O. Evals for an LLM feature — a calibration set, and the honest frontier

**The concept.** An LLM feature isn't done when the prompt "seems to work" — you need a way to *measure* it that survives prompt and model changes. I treat the prompt as an **owned, versioned artifact** and gate it with a **calibration set** of hand-labeled ground-truth examples, run in two tiers: a **deterministic tier in CI** (free, fixture-based, catches regressions on every push) and a **real-LLM tier** run on demand when the prompt or model changes. That turns "did I break the classifier?" into a number, not a vibe. The part interviewers reward is naming its **limits**: a calibration set scores accuracy against labels, not the richer things modern eval asks — rubric / LLM-as-judge scoring, regression suites across prompt versions, or A/B-ing a model change against a *product* metric rather than a model one. Knowing exactly where my eval coverage ends is the difference between "prompt and hope" and "I know what I'm not yet measuring."

**Summary.** *"I don't consider an LLM feature done because the prompt looks right — I gate it with a calibration set of hand-labeled examples, run deterministically in CI on every push and against the real model when the prompt or model changes. That makes 'did I break it?' a number, and it versions with the prompt as an owned artifact. What I'm honest about is the ceiling: it scores accuracy against labels, not rubric or LLM-as-judge evals, and it doesn't yet A/B a model change against a product metric. Knowing exactly where my eval coverage stops is the point — that's what separates measuring from hoping."*

---

## P. Publicly shared vs. production-ready — the gap I crossed early, and the checklist still open

> **Corrected 2026-08-19.** This section previously stated that the app has no real users, and that this was a deliberate scope decision. That was accurate when written and is no longer: the app has since been shared publicly, and `06-app/BUILD-LOG.md`'s 2026-08-15 post-mortem records real visitors on the site during the wrong-times window (#18). The current position is below — it names a real tension rather than claiming a clean one, which makes it the stronger answer.

**The concept.** There is no user *base* here — no marketing, no growth motion, a handful of real visitors from a public post, all unquantified. But the app is **genuinely live and publicly reachable**, which means I crossed the line into "real people can be affected" **before** I had finished the demo-to-production checklist I'd written for myself. That's the honest and more interesting version, because it has a receipt: during the timezone incident (#18) the site served wrong event times while real visitors could see them. I didn't discover the cost of that gap in the abstract — I discovered it by having it happen.

**The checklist — still the substance of the answer.** What separates "shared publicly" from "production-ready" here is written down: rate-limiting + abuse detection on engagement; an account-light path for cross-device sync (scoped by risk — see Concept K); scrape monitoring and alerting; a **user-facing "this is wrong" correction mechanism** (conspicuously absent — during #18 the only way a wrong time got caught was me looking); and the supervision-verification bar, moving that data from "verified where possible" to "verified before anyone relies on it," because a wrong drop-off answer is a child-safety issue, not a data-quality one.

**What the sequence actually taught me.** Publishing is not a state you enter when the checklist is done; it's a state you enter the moment a link is shareable, whether or not you're ready. So the correct order is to build the *irreversible-harm* controls first (correctness at the write path — #21) and let the rest follow. That is precisely why the fail-closed guard exists and why it prefers publishing nothing to publishing a wrong time: it's the one obligation I couldn't defer once the site was reachable.

**Summary.** *"It's live and publicly shared, but there's no user base — a handful of real visitors, no marketing, and I won't pretend otherwise. What I'd rather talk about is the gap that created. I shared it before I'd finished my own demo-to-production checklist, and it cost me: during a timezone bug, real visitors could see wrong event times, and the only reason it got caught is that I looked — there's still no user-facing 'this is wrong' correction path, which is top of the list. The checklist is written down: abuse controls, account-light sync scoped by risk, scrape alerting, a correction mechanism, and moving the child-supervision data to a 'verified before anyone relies on it' bar. The lesson I took is that you don't get to enter 'production' when your checklist is done — you enter it the moment the link is shareable. So you build the irreversible-harm controls first. That's exactly why my ingest now refuses to publish rather than publish a wrong time."*

---

## Q. A guard you haven't seen fail is not a guard — testing the tests

**The concept.** Once you start writing guardrails, the guardrail becomes a new place bugs can hide — and it hides them *worse*, because a check you believe in produces false confidence. Two incidents on this project made that concrete, and they fail in opposite directions:

- **A guard that could never fire.** To prevent the timezone bug's whole *class* from returning, I wrote a test asserting that every `new Date(...)` in the ingest module is on an explicit allowlist with a stated reason, so source times must go through `parseCentralWallTime`. The **first version used a regex that matched string literals** — it missed all three real call sites and **would have passed while the bug was live.** I only found out by rewriting it as an allowlist and then *deliberately reintroducing the original bug* to watch it fail at the right line. A test that has never been observed failing has not been shown to test anything.
- **A guard that cried wolf.** The new start-time check (nothing a family attends starts before 7 AM Central) went red on its first run against the live database — on **11 perfectly legitimate rows**: 9 `LIBRARY CLOSED` days and two all-day civic events, all stored at exactly `00:00`, because sources use midnight to mean "all day, no meaningful time." My rule had conflated that with a shifted clock. Fixed by excluding exact midnight and flagging 12:01–7:00 AM only — safe precisely *because* of how the real bug behaves: it moves an entire source at once, so dozens of events land in that window and the check still fires loudly.

**The two rules that come out of it.** (1) **Verify a guard by making it fail on purpose** — reintroduce the bug, watch the assertion break, and check it breaks at the right place. (2) **A data-quality rule isn't finished when its unit tests pass against the fixture that inspired it; it's finished after it has run against real data and you've looked at what it flagged.** A check written from the shape of one incident encodes that incident's assumptions — mine assumed "early morning ⇒ broken" and had never met "midnight ⇒ all-day." And the consequence is asymmetric: **a guardrail that cries wolf gets disabled, which is worse than never having written it.**

**Summary.** *"Once I started adding guardrails I found the guardrails themselves were a place bugs could hide — and worse, because a check you trust gives you false confidence. Two examples. I wrote a test to stop a timezone bug class from coming back, and the first version's regex missed all three real call sites — it would have passed while the bug was live. I only learned that by deliberately reintroducing the bug to watch the test fail. And a data-quality check I wrote went red on eleven legitimate rows the first time it hit production data, because sources use midnight to mean 'all day' and I'd encoded 'early morning means broken.' So: verify a guard by making it fail on purpose, and don't consider a data rule finished until it's run against real data and you've looked at what it flagged — because a check that cries wolf gets turned off, which is worse than not having it."*


## R. Scoring a product against a governance framework — including where it doesn't fit

**The concept.** A standard way to reason about AI-product risk is three **failure categories** — **content** (the output is wrong), **behavioural** (the user's intent is wrong: misuse, prompt injection, out-of-scope use), and **economic** (usage drives cost faster than value) — addressed by five **instruments**: **evals at scale**, **guardrails** (input classifier, output classifier, refusal layer, rate/cost limits), **observability**, **fallbacks**, and an **audit trail**. I scored Open Eventz against all of it, marking each as exists / partial / missing, and — the part that matters — **recorded the gaps I was accepting rather than solving**.

**The insight that made the exercise worth doing: the framework does not map one-to-one, and *why* is the interesting answer.** Open Eventz is a **scraped data pipeline with an LLM classifier in the middle**, not a user-facing generative feature. The model is called from one file, at ingest time, and **no user free-text ever reaches it**. That means the entire behavioural/misuse category — the toxic-input, prompt-injection, "your brand launders someone else's content" surface — is **deleted structurally, not mitigated**. There is nothing to defend because there is no path. Recognising that is worth more than bolting on an input classifier to satisfy a checklist, and it explains why almost all of this product's real risk concentrates in **content correctness** instead.

The same reframe changes the economics. Cost scales with the **number of new events, not the number of users** — classification is nightly, batched, and cached, so a re-run of an unchanged source costs zero model calls. A traffic spike costs bandwidth, not tokens. That is a genuinely different risk profile from a per-request generative feature, and it is why the missing cost cap is a *bounded* risk rather than an urgent one.

**Where honest scoring beat flattering scoring.** Two entries I deliberately wrote against myself: the per-run cost figure on my dashboard is `calls × unit price` — an **estimate, not metered spend**, which makes it a dashboard number rather than a control; and the real-model evaluation runs **manually with no trigger**, which is the textbook definition of a scaling liability, because the moment the loop speeds up you stop running it. Both are still open. Writing them down as accepted gaps is the difference between a governance review and a self-congratulation exercise.

**Summary.** *"I scored my product against the standard governance model — content, behavioural and economic failure categories, and the five instruments. The useful part was not the parts that fit, it was the part that didn't. My app is a data pipeline with an LLM classifier in the middle, not a chat surface: the model is called once at ingest and no user text ever reaches it. So the whole misuse and prompt-injection category is deleted structurally rather than mitigated — there is no input path to defend. That is a better answer than adding an input classifier to tick a box, and it told me where the real risk actually lives, which is output correctness. I also wrote down the gaps I am accepting: my cost figure is an estimate rather than metered spend, so it is a dashboard number not a control, and my real-model eval only runs when I remember to run it. Being able to say which parts of a framework do not apply to your architecture — and which of your own numbers you do not fully trust — is the part that shows you actually did the review."*

---

*Last updated: 2026-08-19.*
