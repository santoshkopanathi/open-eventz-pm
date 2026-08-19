# Open Eventz — Challenges & Hard Problems (Interview Prep)

*This document captures the real obstacles encountered during the Open Eventz project — both what the challenge was and how it was resolved or handled. Use these in interviews to demonstrate PM judgment, problem-solving, and honest scoping.*

---

## 1. There's No "Free Kids Events" API — and the Obvious Ones Are Dead Ends

**The challenge.** The most natural assumption when building an events aggregator is that some API exists you can tap. It doesn't. The most prominent option — Eventbrite — deprecated its public event search endpoint in February 2020 specifically to prevent third-party aggregators from competing with it. What remains is event-by-ID access, which is useless for open discovery. PredictHQ exists and is broad, but it's an enterprise B2B data product sold via sales conversation, not curated for "free" as a category, and would add cost before there's a dollar of revenue.

**What this forced.** The entire data strategy had to pivot to direct, source-by-source integrations with the actual publishers of free programming — libraries and city governments. This means the data problem is fundamentally per-metro, which has real implications for how the product scales: adding a new metro is a repeatable process (a checklist + schema), not a single API call.

**What to say in an interview.** "My first instinct was to find an events API. I spent time verifying that the obvious candidates don't work for this use case — Eventbrite killed public search in 2020, PredictHQ is enterprise-priced and unfocused on 'free.' That constraint actually clarified the product's moat: if there were an easy API, someone would have already built this. The difficulty of the data sourcing is part of what makes it defensible."

---

## 2. The Same City's Own Documents Disagreed With Each Other

**The challenge.** Frisco Library's unattended minor policy appeared in two places with two different ages: the city code said 9, the library's own Summer FAQ said 10. This wasn't an abstract discrepancy — the supervision field is a core differentiating feature, and if the app shows a wrong age threshold, a parent could leave a child unsupervised based on bad data.

**How it was resolved.** Tracked down the official 2026 Service Policy PDF directly from the library's website. It states "Children aged nine (9) or younger must be accompanied by an adult" — which means 10-and-older can attend without a parent. This aligned with the Summer FAQ (not the city code). The city code was the stale source. Having the current, authoritative policy document in hand is now the logged source.

**What this forced.** It validated the three-tier confidence system (Tier 1: event-specific statement; Tier 2: venue general policy; Tier 3: unverified). The conflict also made it clear that "Tier 2" doesn't just mean you found a policy — it means you found the *right* document and know which one to trust when they disagree.

**What to say in an interview.** "I hit a real data quality problem early: the library's own documents contradicted each other on the age threshold. My rule was: never guess at this field, because a parent making a drop-off decision based on wrong data is a real safety issue. I tracked down the primary policy document, found the authoritative answer, and logged the source and date so it can be re-verified. That's how I'd approach any trust-sensitive data field."

---

## 3. Play Frisco Had No Official Feed — But Unexpectedly Had Something Better

**The challenge.** Unlike the libraries (which have vendor-supported RSS/XML feeds), the City of Frisco Parks & Recreation had no official data feed. The only option appeared to be HTML scraping of the city event calendar — which is fragile (structure can change), legally ambiguous, and blocked in the city's robots.txt for the RSS version (`/RSS.aspx`).

**What was discovered.** The city's calendar page included a "Subscribe to iCalendar" link — a publicly accessible `.ics` feed at `/iCalendar.aspx`. This is a structured, machine-readable format the city already publishes for public subscription. It's not blocked in robots.txt, it's not HTML (so it won't break when the page redesigns), and using a subscription feed the city explicitly provides for public use is on much firmer ground than scraping HTML.

**What this forced.** Updated the PRD to make the iCalendar feed the preferred integration method and HTML scraping the fallback. The discovery came directly from looking at what the city calendar page actually offered rather than assuming scraping was the only path.

**What to say in an interview.** "My assumption going in was that Play Frisco would require HTML scraping. I did the ToS investigation anyway, and while doing that I found that the city publishes an iCalendar subscription feed that's publicly accessible. That's a much cleaner integration — structured data, already intended for machine consumption, no gray area. The lesson is: read the full page before assuming the hard path is the only path."

---

## 4. VisitFrisco.com Looked Like a Data Source Until It Wasn't

**The challenge.** VisitFrisco.com appeared to be a promising source of Frisco events with a well-organized calendar. Before building against it, investigated the actual network calls the site makes to load its calendar data.

**What was found.** The site is built on Next.js and loads event data from internal URLs that include a build ID (`esoUENnin8q1gQN52KTW3`) — a value that changes every time the site redeploys. Any scraper built against those URLs would break on the next deployment with no warning and no recoverable path. Beyond the instability, the content itself was wrong for the audience: concerts, sports, nightlife — not kid-friendly free events.

**What this forced.** Ruled out VisitFrisco.com entirely, logged the reason in the Source Decision Log, and flagged it as a backlog candidate only (meaning: would revisit only if they ever publish a stable feed). The investigation saved build time that would have been wasted on a source that would have broken immediately.

**What to say in an interview.** "I spent time investigating VisitFrisco.com before deciding not to build against it. The technical reason is that it uses Next.js internal data URLs with a build ID that changes on every deploy — there's no stable endpoint to target. Even setting aside the technical fragility, the content was wrong: concerts and sports, not free kid activities. Sometimes the right PM decision is ruling something out clearly so it stays out of the backlog indefinitely rather than getting relitigated every sprint."

---

## 5. BiblioCommons RSS: Confirmed Existence, Can't Directly Verify Format

**The challenge.** BiblioCommons (Frisco Library's platform) is documented to support RSS export for events. But locating the specific RSS endpoint URL — as opposed to the HTML event page — required investigation, and the fetcher used returned an empty body for the RSS URL (likely valid XML that the tool couldn't display as text).

**What was found.** The endpoint `https://friscolibrary.bibliocommons.com/events/search.rss` appears to exist — it returned a 200-class response without error, just no HTML body, which is consistent with XML content the fetcher filtered out. The events page itself is confirmed live with 280+ active events. Audience filter IDs needed for children's-only filtering were also identified.

**What this forced.** A clear note in the PRD that this needs a 30-second browser/curl verification at build time. Not a blocker, but it's the kind of thing that looks like a blocker if you don't distinguish "can't verify format" from "endpoint doesn't exist."

**What to say in an interview.** "One of my early lessons on this project was learning to distinguish 'I can't confirm this' from 'this doesn't work.' The RSS URL returned a valid response — the tooling I was using just couldn't display XML. I logged it as 'confirm format at build time' rather than closing it as broken. Good PM documentation is honest about what's verified versus what's inferred."

---

## 6. Supervision Data: The Product's Best Feature Is Also Its Riskiest

**The challenge.** The supervision / drop-off flag is one of the sharpest, most defensible differentiators in the product — no other app tells parents whether they need to stay or can drop off. But it's also the highest-stakes data field: if the app shows a wrong threshold and a parent leaves a child who shouldn't be left alone, that's a real safety issue the product is implicated in.

**The approach taken.** Designed a three-tier confidence system (event-specific policy → venue general policy → unverified) and applied a hard rule: Tier 3 always shows "check with venue," never a guessed number. When the Frisco Library's documents conflicted, the answer was to find the authoritative source rather than pick the more convenient answer. When Plano Library had no age threshold at all (parent's discretion), that was captured precisely as stated — not defaulted to a number.

**What Plano Library's answer revealed.** Calling the library and getting "it's parent's discretion, we just expect the child to behave" is itself a meaningful data point — it tells a parent something true and useful. The display for Plano events isn't "check with venue" (implying uncertainty), it's "No age requirement — parent's discretion" (which is accurate and actually reassuring). Getting the exact wording right matters.

**What to say in an interview.** "The supervision field is the feature I'm proudest of and most careful about. The whole value proposition only works if parents trust it — and they'll only trust it if it's accurate. My rule was: never show a threshold we haven't verified, and never show the same generic 'check with venue' when we actually have real information. I called Plano Library directly to get their policy. Their answer — 'no age requirement, parent's discretion' — is different from Frisco's hard cutoff at 10, and both are worth showing accurately."

---

## 7. Scope Discipline: Saying No to Obvious Sources

**The challenge.** During research, several seemingly useful data sources came up: PlanoMoms.com (a well-curated local parenting blog), Eventbrite organizer pages, Plano Parks & Rec, Frisco ISD, YMCA, museum free-day calendars. Each one had a real argument for inclusion.

**The discipline applied.** Each source was evaluated against: Is there a clean, stable integration method? Is the content reliably in scope (free, kid-appropriate)? Does adding it now materially improve v1 over not adding it? PlanoMoms.com was explicitly ruled out as a scrape target — their curated editorial work is their IP, and republishing it without permission is a partnership conversation, not a technical decision. Plano Parks & Rec has a PDF-based catalog, which adds parsing complexity for minimal gain over what the library feeds already cover. Each ruled-out source is logged in the Source Decision Log with the reason, so the decision doesn't get relitigated every quarter.

**What to say in an interview.** "One of the hardest things about this project was saying no to sources that had real value. PlanoMoms.com is genuinely good content — but scraping someone else's curated editorial without permission isn't the right move. I logged it as a potential partnership instead. The Source Decision Log is my artifact for preventing scope creep disguised as obvious improvements."

---

## 8. Two Different Problems Hidden Inside One Feature: Geographic Search

**The challenge.** "Find events near me" sounds like one feature. In practice it's two separate problems: the geographic unit for *data sourcing* (which is city/branch — because that's how libraries and parks departments are organized) versus the geographic unit for *user-facing search* (which ideally would be zip code or radius). Conflating the two would mean either building a geocoded radius search before you have enough sources to make it meaningful, or confusing the data model by trying to store things in terms the user cares about rather than terms the sources use.

**The decision.** Explicitly split these in the data model from day one: sources are stored and identified by city/branch, and the user-facing location experience in v1 is source-based filtering (Frisco Library, Plano Library, Play Frisco) rather than geocoded radius. True radius search is logged as a future enhancement for when there are enough sources across a metro to make it worth the geocoding complexity.

**What to say in an interview.** "I caught an assumption buried in the feature spec early: 'near me' filtering could mean two completely different things depending on which layer of the product you're talking about. I separated them in the data model explicitly so we weren't building geocoding complexity before we needed it, and so the v1 experience still made geographic sense even without it."

---

---

## 9. BiblioCommons RSS: Confirmed Dead at Build Time

**The challenge.** Challenge 5 above noted that the BiblioCommons RSS endpoint needed build-time verification. When we actually built the ingest pipeline, the endpoint returned a 404 — the feed no longer exists. This wasn't a tooling display issue; BiblioCommons has retired RSS entirely in favor of their JavaScript-rendered web app. None of the alternative API paths worked either: the internal JSON API was private, iCal redirected to the HTML page, and XHR interception in DevTools found no separate events API call at all.

**How it was resolved.** Discovered that events are fully server-rendered in the HTML as plain markup — no JavaScript needed to see the data if you fetch the page server-side. Built a structured HTML parser that splits the page on the card boundary (`<li><div class="cp-events-search-item">`), then regex-extracts each field from the card HTML. Added audience-based pagination (three feeds × up to 14 pages × 20 events per page) to cover the full two-month event window.

**What this forced.** The integration is more brittle than an RSS feed — if BiblioCommons changes their HTML structure, the parser breaks. Mitigated with per-source error isolation so a parser failure only takes down that source, not the whole app. Also forced the decision to fetch each event's detail page individually to get accurate age data (see Challenge 11).

**What to say in an interview.** "The RSS feed I'd planned to use didn't exist when I went to build against it. Rather than treat that as a blocker, I reverse-engineered the HTML the website renders server-side and built a structured parser against it. It's a more fragile integration, and I documented that honestly — the tradeoff is full data coverage now versus some maintenance risk if their HTML changes later. I made that tradeoff consciously rather than quietly."

---

## 10. Play Frisco iCal Only Covered Events Starting January 2027

**The challenge.** The city's iCalendar feed (`catID=81`) was confirmed as the official integration method and appeared clean in ToS terms. But when actually fetched during the build, every event in the feed had a start date of January 2027 or later — a five-month gap with no near-term events. The RSS feed was already in use as a fallback, but it was capped at 8 items covering only the next six weeks.

**How it was resolved.** Investigated where the near-term events actually lived. The city's own calendar page (`calendar.aspx?CID=85,81`) is server-rendered and paginated by month — fetching it for the current and next two months via `&month=M&year=Y` returns the full event set the public sees on the website. Replaced both the RSS and iCal feeds with a month-by-month HTML scrape: extract EIDs from each monthly listing page, then fetch each event's detail page for structured data (title, date, location, description) using CivicPlus's schema.org `itemprop` attributes.

**What this forced.** The integration now makes more HTTP requests (3 listing pages + 1 detail page per event), but the data coverage is accurate and complete. Both the RSS and iCal feeds were removed entirely — two sources replaced by one more reliable one.

**What to say in an interview.** "The official feed existed but only surfaced events five months out — useless for a 'what's on this week' product. I traced back to where the city's own website was getting its near-term data from, which turned out to be a server-rendered monthly calendar page. The right integration wasn't the officially documented one; it was the one that actually had the data parents needed."

---

## 11. BiblioCommons Audience "Bleed" — Same Event, Wrong Age Label

**The challenge.** BiblioCommons serves events through three separate audience feeds (Children 0–5, Children 6–12, Teens). The same event frequently appears in multiple feeds — a story time might appear in both the 0–5 and 6–12 feeds. The first approach was to use the feed's audience as the age label. This produced obviously wrong results: a "Dinosaur George" exhibit tagged as Children 0–5 from one feed would then appear again tagged as Children 6–12 from another, with conflicting labels in the UI.

**How it was resolved.** Each BiblioCommons event page includes a "Suitable for:" section listing the actual intended audiences. Since the ingest pipeline was already fetching each event's detail page for description, the same HTTP request was used to scrape the "Suitable for:" section and derive age_min/age_max from the actual page content rather than the feed it was discovered in. Deduplication across feeds then merges age ranges, but since the page scrape is authoritative, merged values typically stay consistent.

**What this forced.** The page scrape adds latency to ingestion (one HTTP request per event), but it's the only reliable source of truth for age. It also required handling four detection cases: adults + young children (0–17), adults + teens only (13–17), adults only (excluded), and kids/teens only (computed range).

**What to say in an interview.** "Vendor feed design doesn't always match product needs. BiblioCommons serves events through audience-segmented feeds, but the same event bleeds across multiple feeds with no reliable way to know which audience assignment is correct from the feed alone. I solved it by going to the authoritative source — the event page itself — which explicitly states the intended audience. One extra HTTP request per event at ingest time bought accurate age data for every event permanently."

---

## 12. Adults + Teens Events Surfacing Under Toddler and Kids Filters

**The challenge.** After deploying the "Suitable for:" page scrape, some clearly adult-targeted events (ukulele class, music production workshop) were still appearing under the Toddlers and Kids age filters. The scraper logic treated `hasTeen = true` as evidence of a kid-appropriate event — so when a page said "Suitable for: Adults, Teens," the code hit the `adults + any kids` branch and set age_min=0, age_max=17, making it appear for all ages including toddlers.

**How it was resolved.** Distinguished between "adults with young children present" (0–17 appropriate) and "adults with teens only" (13–17 appropriate). Introduced a `hasYoungKid` check separate from `hasTeen`, and added a specific branch for Adults + Teens that sets age_min=13 rather than 0. Also added a permanent API-level exclusion for adults-only events (age_min ≥ 18) so they never surface regardless of filter state.

**What this forced.** A tighter four-case decision tree for audience detection, and the habit of checking real events on the source website to validate the logic rather than just testing with synthetic cases.

**What to say in an interview.** "An edge case in the age detection logic was surfacing adult events to parents looking for toddler activities. The bug was subtle — treating 'teens' as equivalent to 'young children' when adults were also tagged. Fixing it required thinking through the full decision tree: adults + toddlers is different from adults + teens, which is different from adults only. I validated the fix by looking up the specific events that were misbehaving on the source site and confirming what their 'Suitable for:' field actually said."

---

## 13. Multi-Day Events Disappeared Mid-Run

**The challenge.** Soccer Celebration — a Parks & Rec event running June 11 through July 19 — was correctly ingested with its start and end dates, but disappeared from the app the day after it started. The events API filtered to `start_datetime >= today`, which correctly excluded past-starting events but also incorrectly excluded events that had already started and were still running.

**How it was resolved.** Added a supplemental database query that runs alongside the primary one: fetch events where `start_datetime < today AND end_datetime >= today`. Results from both queries are merged and deduplicated before being returned. This ensures ongoing multi-day events always surface at the top of the list for as long as they're running.

**What this forced.** A two-query pattern in the events API instead of one, plus a deduplication step in the merge. Also required fixing how all-day iCal dates were parsed — they were coming in as midnight UTC, which shifted the date one day earlier in Central Time. Fixed by parsing all-day dates as noon UTC instead.

**What to say in an interview.** "A date filter that seemed obvious — only show future events — had an unintended consequence for multi-day events. An event that started last week and runs through next month is both past-starting and currently relevant. I added a supplemental query for the 'ongoing' case rather than bending the primary query into something complicated, then merged and deduplicated the results. Simple queries composed cleanly are easier to reason about than one complex query trying to handle every case."

---

## 14. Date Filter Off by One Day — a Timezone Boundary Problem

**The challenge.** When a user set the date filter to "From: July 1," events from June 30 still appeared. Specifically, an event at 7:00 PM CDT on June 30 was stored in the database as `2026-07-01T00:00:00Z` (midnight UTC, because CDT is UTC−5). The date filter was sending `2026-07-01` to the API, which Supabase compared as `>= 2026-07-01T00:00:00Z` — and the June 30 event passed because it was stored at exactly that UTC timestamp. The same bug ran the other way on the end date: events on the selected to-date didn't appear because their UTC timestamps were ahead of midnight UTC.

**How it was resolved.** Converted user-selected dates (which represent local CT calendar days) to CT-aware UTC boundaries before querying. `date_from` maps to midnight CT = `T05:00:00Z` (CDT offset). `date_to` maps to end of that calendar day in CT = the next day at `T05:00:00Z`, queried with `<` rather than `<=`. Both helpers were extracted into named functions (`dateToCtMidnightUtc`, `dateToCtEndOfDayUtc`) to make the intent explicit.

**What this forced.** Re-examining every place in the API where dates are compared — the ongoing multi-day supplemental query also needed the same treatment.

**What to say in an interview.** "Timezone bugs in date filters are one of the most common sources of off-by-one errors in consumer apps, and they're invisible until a real user tries filtering on a boundary day. The root cause here was treating a user-selected calendar date as a UTC timestamp. The fix is always the same: be explicit about which timezone a date string represents and convert it before comparing against stored UTC values. I extracted the conversion into named helper functions so the intent is clear to anyone reading the code later."

---

## 15. Plano Libraries Age Data: No Path Forward, Accepted as a Known Limitation

**The challenge.** Plano Libraries runs on Communico, which does have age/audience fields on individual events. However, the RSS feed — the only accessible integration path — does not include age fields. The Communico API supports an `ages` filter parameter, but passing any value (`children`, `teen`, `all`) returns an empty feed. The API exists but isn't usable without vendor-issued authentication tokens that Plano Library does not make publicly available.

**How it was resolved.** Documented as a known, accepted limitation rather than blocking the release on a fix that doesn't exist. All Plano events are stored with null age_min/age_max, which means they pass all age filters — a parent filtering for toddlers will see Plano events even if some of those events are adult-targeted. Compensated in the UI with a branch sub-filter (choose a specific Plano library branch) as the contextual refinement tool when Plano is selected, since branch-level filtering is data we do have.

**What this forced.** A clear decision about what "acceptable" looks like when a data gap can't be closed. The age filter's null pass-through is a deliberate product decision — showing more events than strictly match is better than showing fewer, given that Plano's events are generally family-oriented by default. Documented in the PM OS as a limitation, not silently papered over.

**What to say in an interview.** "Not every data gap has a technical solution. Plano's age data exists in their system, but the only accessible integration path doesn't expose it, and the API that would expose it requires vendor auth they don't issue publicly. Rather than pretend the limitation doesn't exist or block the feature, I documented it explicitly, decided that null-inclusive filtering was the right default behavior, and added a different type of refinement — branch filtering — that works with the data we do have. Knowing when to accept a constraint and compensate elsewhere is as important as knowing how to work around one."

---

## 16. A Shipped Feature Silently Disappeared — and I Was Confidently Wrong About Why, Twice

**The challenge.** The product's most differentiated feature — a supervision flag telling parents whether they could drop their child off or needed to stay — was once shown across all three data sources. Weeks later it was only rendering for one; the other two showed nothing. No error, no failed test, no alert. It surfaced only because old screenshots proved the feature used to do more. A silent regression on the exact feature the product's trust story depends on.

**What it revealed (and the false trails).** Diagnosing *why* took three passes, and being willing to overturn the first two answers was the whole game. Theory one — a git history rewrite (a secret-scrub `filter-branch` + force-push) had erased it — was stated confidently and was wrong: that rewrite preserved every commit. Theory two was also wrong. The actual cause only held up on the third pass: the all-source version had been refactored down to a single-source inline version **before the project was ever committed to git.** A month of development had run with no version control, so the working feature died in an unversioned window with no diff and nothing to recover.

**What this forced.** Three durable changes. (1) Get code under version control *before* refactoring, not after it's "ready." (2) Point the strongest guardrail at the differentiator — the supervision logic had been inline in a component with no unit test, so it fell out silently; it now warrants an extracted, per-source-tested module with a doc-parity check. (3) Record the *change*, not just the current state — the build log had described what the feature *is*, which hid the day it started doing less. It also reframed the incident as an asset: a real production-style regression with a documented root cause and fix, which is exactly the "operating history" that AI-PM interviews probe for and most portfolios lack.

**What to say in an interview.** "One of my most differentiated features silently lost two-thirds of its coverage and I didn't notice for weeks — no test watched it. When I investigated, I was confidently wrong about the cause twice before I found it: the feature had been refactored away before the code was ever under version control, so there was no history to recover. Three lessons came out of it — commit before you refactor, put your strongest guardrail on your most important feature, and log the change rather than just the end state. And the meta-lesson: the first explanation that fits isn't the cause, it's just the first thing that fits. I got to the real answer by disproving myself, not by trusting my first read."

---

## 17. A Scraper That Returned "Everyone" — Server-Side vs. Client-Side Rendering, and Why I Didn't Reach for the LLM

**The challenge.** Parents on the Frisco tab started seeing adult events ("D&D for Adults," "Adult Volunteer Open House"), and the age filters stopped working — selecting "Toddlers (0–5)" still returned 305 of 306 events. The data confirmed it: 304 of 306 Frisco events were stored as age 0–17. The single source of Frisco age truth — BiblioCommons' "Suitable for:" field — had gone blank for our scraper, so every event fell through to the "all ages" fallback, which passes the adults-only gate (`age_min < 18`) and overlaps every kid filter.

**The false start (an honest correction).** My first pass had called the surrounding staleness "pure staleness, not a parser break" — because I checked the *listing* page markup and it was intact. That was incomplete: I hadn't checked the *detail-page* age extraction, which was broken. Two different scrape steps; I'd validated the wrong one.

**What caused it.** BiblioCommons had migrated event pages to a **client-side-rendered `/v2`** architecture. The "Suitable for:" audience is now hydrated by JavaScript *after* the page loads. Our server-side `fetch` retrieves the raw pre-hydration HTML, where the `<span itemprop="name">` audience is empty — no JSON-LD, no `__NEXT_DATA__`, nothing. The value a human sees in the browser simply wasn't in the bytes our scraper received. And the newly-automated nightly ingest had faithfully re-scraped the empty source, overwriting good data with "0–17" everywhere — which is what surfaced the break.

**What I was going to try.** Extend the LLM age-inference we already run for Play Frisco to Frisco Library — infer the age band from title + description. Robust, but it adds a per-event cost and an inference dependency to reconstruct data that, in principle, still existed.

**What we explored instead — and what unlocked it.** The product owner pointed out the obvious-in-hindsight fact: the "Suitable for:" value *is* visible in the browser. That reframed the problem from "the data is gone" to "the data moved to client rendering." So instead of approximating it, I drove a real headless browser to the event page and watched its network tab — and found BiblioCommons' own **unauthenticated JSON API**: `GET /events/events/{id}?client_scope=events` (with `Accept: application/json`) returns `definition.audience_ids`, and `/events/event_audiences` returns the id→name taxonomy (six stable audiences — the same IDs as the original audience feeds). Validated instantly: Family Story Time → `[Children (0-5)]`; D&D for Adults → `[Adults]`.

**How it helped.** Re-pointing the scraper to the API is strictly better than the LLM path: **deterministic, free, authoritative** (the real data source with stable IDs), decoupled from fragile presentational HTML — and it came with a cleaner description field and a `featured_image_id` hook usable for event images later. We avoided spending money and adding an inference dependency to rebuild data the site was already handing its own front-end.

**What this forced.** A durable habit: when a scrape suddenly returns empty, first decide *is the data gone, or did it move to the client?* — a field visible in the browser but absent from `curl` is the tell — and before reaching for an LLM to reconstruct missing data, check whether the site's own front-end fetches it from an API you can call directly. The network tab is the shortcut past HTML scraping.

**What to say in an interview.** "Our kids-events app suddenly started showing adult events and the age filters went dead. Root cause: the source library site had moved to client-side rendering, so the age field a human sees in the browser was no longer in the HTML our server-side scraper fetched — every event defaulted to 'all ages.' I was about to solve it with LLM inference, which we already use elsewhere — but the smarter move came from a simple reframe: the data's still visible in the browser, so it isn't gone, it just moved. I opened the network tab, found the site's own JSON API feeding that field, and pointed our scraper at it. Deterministic, free, and authoritative — no model needed. The lesson I carry: distinguish 'the data is gone' from 'the data is client-rendered,' and reverse-engineer the authoritative source before you approximate it. Knowing when *not* to use an LLM is part of using AI well."

---

## 18. Every Event Was 5–6 Hours Early — a Timezone Bug the Automation Activated, and Why the Guardrail Wasn't Enough

**The challenge.** I opened a finished event page on production and saw *"Friday, August 14, 2026 at 5:00 AM"* on a Family Story Time. It wasn't one event — every Frisco and Plano event was landing 5–6 hours early. The app was live and publicly shared, so real people could have planned a morning around a time that didn't exist. For a listings product whose whole job is "trust these details," a *wrong* time is worse than a missing one: it gets trusted, acted on, and betrays the trust.

**What caused it.** `new Date("August 14, 2026 10:00 AM")` resolves an **offset-less** string in the **runtime's** timezone. Three of my four sources publish exactly that shape (bare wall-clock, no offset). On my Central dev machine the result was correct — so it passed every manual check I ever did. But the nightly ingest had moved to **GitHub Actions on 2026-08-12**, and Actions runners are **UTC**. The automation that fixed my staleness problem quietly began writing every event 5–6 hours early. The timezone assumption and the automation were each individually fine; the *combination* wasn't, and nothing in the pipeline was looking at a clock. (Plano was its own trap: its feed stamps `+0000` on times that are plainly local — a 9:30 AM storytime published as `09:30:00 +0000`. The old code was *right* to distrust that bogus offset; it just handed the naive remainder to the machine's timezone instead of the venue's.)

**Why it stayed hidden — the four reasons that actually matter.**
- **It was invisible everywhere a human looked.** Every manual check ran on a machine in the venue's timezone. The only environment that produced wrong data was the one nobody watches — the cloud runner.
- **The trigger was a change to the *environment*, not the code.** The timezone assumption was latent from day one; moving ingest to Actions *activated* it. Nobody re-audits runtime assumptions when the runtime changes — the diff was "where it runs," and the review question was only "does it still run?"
- **I fixed an instance and called it a class.** Two days earlier I'd fixed a *different* timezone bug on a new source (Kaleidoscope's misconfigured `utc_*` field) and written it up as a playbook principle — *don't trust a source's UTC.* I never audited the other three sources against that shape. **A principle recorded but not applied retroactively is a note, not a control** — and this is the miss that stings, because the answer was already written down.
- **Every guardrail ran *after* the write.** My data-quality gate does turn the pipeline red — but only once production is already serving the wrong times. **Detection is not prevention.**

**The fix — two layers.** (1) *Prevention:* `parseCentralWallTime` / `centralWallTimeToUtc` resolve every source time as `America/Chicago`, DST-aware (CDT shifts 5h, CST 6h) — timezone now comes from the venue, never the machine. (2) *Detection, hardened:* `implausiblyEarlyEvents` / `startTimeChecks` in `data-quality.ts` — nothing a family attends starts before 7 AM Central; one stray outlier is tolerated (5% threshold), a whole source shifting is not, and the failing line names the source. Exact-midnight `00:00` is excluded so legitimate all-day markers (library closures, civic observances) don't false-red. The important rule is the **uniform-shift** signature: a real reschedule moves *one* event by an arbitrary amount, a clock bug moves *every* event by an identical amount — so the check catches shifts I haven't thought of (a 1-hour DST error, an off-by-one-day parse) **even when they land at plausible hours**. And CI now runs the unit suite **twice — `TZ=UTC` and `TZ=America/Chicago`** — because a suite that only runs in one timezone can't catch a bug that depends on the timezone. An explicit escape hatch (`INGEST_ALLOW_TIME_SHIFT=1`) permits the intended mass correction — the re-ingest that *fixes* a timezone bug is supposed to move everything at once.

**What this forced.** Two durable habits. First: **when you learn a lesson from one source, audit every other source for the same shape *that day*** — otherwise the write-up is just a nicer way of being surprised twice. Second: **put the strongest check on the output the user actually sees, and in front of the write, not behind it.** A gate that runs after the write can only tell you *how long* you've been wrong.

**What to say in an interview.** "My kids-events app started showing every event 5–6 hours early — a 10 AM story time read as 5 AM, on a live site. Root cause was a classic: the code parsed each source's local wall-clock time without a timezone, which resolves in the *runtime's* zone. It was correct on my Central dev machine, but I'd just moved the nightly job to a UTC cloud runner — so the automation that fixed staleness quietly started writing wrong times. What I take from it isn't 'timezones are hard.' It's three things: a bug can be invisible everywhere a human looks and only appear in the one environment nobody watches; the trigger can be an *environment* change with no code diff, so 'does it still run?' is the wrong review question; and I'd actually fixed this exact shape on another source days earlier and never audited the rest — a lesson you write down but don't apply backward is a note, not a control. I fixed it by pinning every time to the venue's timezone, and I added a guardrail that flags a *uniform* shift across a source — the fingerprint of a clock bug versus a real reschedule — moved in front of the user-visible output. Detection after the write only tells you how long you were wrong."

---

## 19. The Ingest Outgrew Its Serverless Function — and the Green Run That Wrote Nothing

**The challenge.** The nightly ingest started as a Vercel scheduled function, but a full multi-source run — scrape four sources, then LLM-classify the new events — takes far longer than a request-scoped serverless function is allowed to live (a 10-second ceiling on the plan). The function hit the wall mid-run. The worst part wasn't the crash; it was that a timed-out or partial run didn't announce itself — the schedule looked like it had fired, and the data just quietly went stale or half-updated. **A green-looking scheduled run that actually wrote nothing is worse than a hard failure, because nothing tells you.**

**What caused it, and the fix.** The workload is a *batch job*, not a web request — the wrong shape for a function metered by seconds. I moved ingest to a **GitHub Actions scheduled workflow** (`0 11 * * *` + manual `workflow_dispatch`), with **one independent job per source** (`matrix` + `fail-fast: false`) so a slow or broken source can't block the others, each with isolated logs. Two failures on the way, both instructive:
- **Exit 2 — secrets invisible.** I'd added the keys as *Environment*-scoped ("Production") secrets, which job-level `${{ secrets.* }}` can't see. Fix: declare `environment: Production` on the job so the environment's secrets are in scope. The lesson: *where* a secret is scoped is as load-bearing as its value.
- **Exit 1 — the runtime moved under me.** `@supabase/supabase-js`'s `createClient()` threw *"Node.js 20 detected without native WebSocket support."* The default runner was Node 20; a global `WebSocket` is first stable in Node 22. Fix: pin `node-version: 22`. (Same family of lesson as the timezone bug in #18 — a dependency's behaviour depends on the runtime, and the runner is not your laptop.)

**What this forced.** A scheduled job needs a way to **shout when it fails**, or a silent green run masks staleness for days. So a `notify` job opens (or comments on) a **GitHub Issue** whenever a source job or the data-quality gate fails — chosen over SMTP/Slack because it needs no new secrets (`GITHUB_TOKEN` suffices), GitHub emails the owner, and an issue has to be *closed*, not just skimmed past at 6 AM.

**What to say in an interview.** *"My nightly data ingest outgrew its serverless function — a multi-source scrape-plus-LLM-classify job can't finish inside a 10-second request timeout, so runs were silently timing out and the data went stale without any alarm. The real fix was recognising it's a batch job, not a web request, and moving it to a scheduled CI workflow with one isolated job per source. Two failures taught me more than the move: environment-scoped secrets aren't visible to job-level references, and a dependency needed a newer Node than the default runner — both are 'the runtime isn't what you assumed' bugs. And I learned that a scheduled job's most important feature is that it shouts when it fails; a green run that quietly wrote nothing is the dangerous state."*

---

## 20. The Real Scale Test Isn't the First Hard Source — It's Whether the Second Is Cheaper

**The challenge.** My first sources were archaeology — a dead public API, a reverse-engineered feed, a page that renders its data in JavaScript so a scraper sees nothing (see #17). Each was days of bespoke work. For a data-aggregation product, that raises the existential question: does every new city/source cost the *same* slog, or does the cost *fall*? If it doesn't fall, there's no product — just an endlessly growing pile of fragile one-offs. So when I added a brand-new source — a large local park's events — I treated it as a deliberate test of the expansion thesis, not just another integration.

**What made it cheaper — and why that's the whole point.** Two things, both built *before* the source:
- **A written onboarding playbook** (`SOURCE-ONBOARDING.md`, 7 principles). The park's one surprise — its REST API is `403` WAF-blocked to a bare request but returns `200` with `Accept: application/json` + a browser UA + `Referer` — didn't cost me an afternoon of confusion; it just became the next line in the playbook. Surprises turned into documented steps instead of fresh archaeology.
- **A shared LLM classifier** (`classifyEvents`, extracted from the existing Play Frisco path in the same change). I wrote **zero new rules** about what counts as a "kids event" for this source. The same classifier read a brand-new, mixed-audience calendar and correctly **hid a "Pop & Pour" wine night** — reasoning on file: *"explicitly requires guests to be 21+"* — while keeping the story times and festivals. A hand-tuned keyword list would have needed a rewrite per source; the model generalised.

Net: **103 events → 84 kid-facing** on production, and the new source was mostly *wiring*. Adding my second *kind* of source cost **less** than the first, not more — and re-running the old source afterward made **0 extra LLM calls** (the classifier extraction was regression-free).

**What to say in an interview.** *"For a data-aggregation product the moat isn't the first integration — anyone can grind through one — it's whether the tenth is cheap. So I treated adding a new source as a test of that. It was mostly wiring, for two reasons I'd built on purpose: a written onboarding playbook that turned each new source's quirk into a documented step instead of fresh research, and a shared LLM classifier so I wrote no new 'is this for kids?' rules — the same model read a brand-new calendar and correctly hid a 21-plus wine event while keeping the family ones. The rules don't generalise; the model does. That's the difference between a product that scales and a pile of one-off scrapers."*

---

# Technical Concepts & Talking Points

*Not project obstacles — foundational concepts worth being able to explain crisply. Same format: the concept, then the interview soundbite.*

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

**What to say in an interview.** *"I separated config from code following 12-factor: public identifiers and publishable keys are exposed to the client via a `NEXT_PUBLIC_` prefix, while secrets — the Supabase service-role key, the Anthropic API key, the GCP service-account credential, and the cron shared-secret — stay server-side, git-ignored locally and stored as environment variables in Vercel for production. The two Supabase keys are a clean least-privilege example: the browser gets an anon key constrained by row-level security; the server gets the privileged key that never leaves the backend."*

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

**What to say in an interview.** *"Yes — Supabase ships an official MCP server, so an AI client can connect to the database to inspect schema, run queries, and manage migrations. I'd treat it as development tooling, not part of the product — the app still uses the SDK. And I'd run it read-only and scoped to a single project specifically because of prompt-injection and blast-radius concerns: handing an agent write access to a production database is a security decision, not just a convenience one."*

---

## C. Securing an AI endpoint — the cost-DoS vector

**The concept.** A normal unauthenticated endpoint that returns data is low-stakes. An endpoint that calls a **paid LLM** on every request is not: leave it open and anyone can script requests to run up your model bill — a **cost-based denial-of-service**. Open Eventz had exactly this — `/api/infer-age` (a testing wrapper over the inference function) was unauthenticated and called Claude. Fixed by gating it behind the same shared-secret bearer token as the ingest endpoint (401 without it). Two lessons: (1) **AI endpoints carry a cost attack surface** ordinary CRUD routes don't — audit every route that *spends money* or *writes data* for auth; (2) **security-through-obscurity isn't security** — the vulnerability lived on the deployed app regardless of source visibility; publishing the code just lowers the bar to discover it.

**What to say in an interview.** *"Before open-sourcing the code I audited every endpoint for auth and found a testing route that called the paid LLM unauthenticated — an open cost-DoS vector — and gated it. The insight is that AI endpoints have a cost attack surface normal endpoints don't, and the risk was on the live app, not the source: publishing code doesn't create the vulnerability, it removes the obscurity that was accidentally hiding it."*

---

## D. Row-Level Security & least privilege

**The concept.** Supabase exposes tables through an auto-generated API reachable with the **anon key** — which is **public by design** (it ships in the browser bundle). So the gate that actually protects the data isn't repo privacy, it's **Row-Level Security (RLS)**: per-table policies deciding what the public key can do. With RLS off, anyone with the anon key can read/write/delete. The fix is **least privilege per table**: `events` = public read-only; `like_counts` / `supervision_policies` / `ingest_runs` = fully locked (only the server's service-role key, which *bypasses* RLS, touches them). Follow-on lesson: the RLS **sweep must cover every exposed table** — the first migration fixed the three tables a linter flagged but missed `ingest_runs`, which a later advisory caught. Re-run the advisor after any migration that adds a table.

**What to say in an interview.** *"The Supabase anon key is public by design, so repository privacy is irrelevant to data security — Row-Level Security is. I enabled RLS on every exposed table with least-privilege policies: the public key can only read the events table, everything else is locked to the server's service role. And I learned to re-run the security advisor after every schema change, because my first pass missed a table a later migration added."*

---

## E. A green local hook ≠ a green pipeline

**The concept.** Local **git hooks** (pre-commit/pre-push) and **cloud CI** run the same commands in **different environments**. Open Eventz's unit-test CI job was red on every push while the local hooks passed — because `jest.config.ts` (a TypeScript config) needs `ts-node` to parse, `ts-node` wasn't a declared dependency, and the local `node_modules` happened to have it while CI's clean `npm ci` didn't. Classic "works on my machine." The fix removed the hidden dependency (config → plain `.js`), but the discipline is the point: **verify against a clean environment and check the actual CI run — a passing local hook is not proof the pipeline is green.**

**It happened twice — and the second time I fixed the process, not just the bug.** A large UI reskin later passed the *same* pre-push hook (typecheck + unit, both green) but turned **CI red on E2E** — because the Playwright smoke suite was **CI-only**, so the one layer that tests the UI I'd just rewritten wasn't in my local gate. Three assertions were coupled to changed presentation (a card label, a chip colour, a `button.rounded-full` selector). The fix wasn't just repairing them: I **moved E2E into the pre-push hook but guarded it to skip gracefully when no browser is installed** — E2E had been dropped from the hook originally *because* a missing local browser caused false failures, so a naïve "always run E2E" would just re-introduce that. Run-when-possible / skip-gracefully lets you tighten the gate without re-breaking it.

**What to say in an interview.** *"A green local hook isn't a green pipeline — and I learned it twice. First, a TypeScript config needed a dependency my machine had but CI's clean install didn't. Second, a UI reskin passed my pre-push hook because it only ran typecheck and unit tests — the E2E suite that would've caught it was CI-only, so it went red after I'd already merged. What I'm proud of is the second fix: instead of just repairing the tests, I moved E2E into the pre-push hook but guarded it to skip gracefully when no browser is installed — so the local gate now covers the layer most likely to break, without the false failures that made me leave it out before. Your local gate has to include the layer that tests what you actually change."*

---

## F. Doc↔test parity — keeping the test plan honest

**The concept.** A test plan that says *"scenario X is covered by test file Y"* is only trustworthy if something enforces it — otherwise the doc quietly rots as tests are renamed or deleted. Open Eventz added a tiny **CI `doc-parity` job**: it parses the consolidated scenario doc, extracts every test file each scenario claims, and **fails the build if any named test no longer exists**. That turns "we think this is covered" into "CI proves the named test still exists" — engineering-quality rigor a PM rarely brings.

**What to say in an interview.** *"I consolidated the functional test scenarios into one plan and added a CI check that fails the build if a scenario names a test file that no longer exists — so the PM test plan can't silently drift from the actual suite. A test plan that claims coverage is only credible if something enforces the claim."*

---

## G. Reading `npm audit` with judgment

**The concept.** `npm audit` reports every known vulnerability in the dependency tree — but the raw count ("7 high severity") conflates **exploitable** risk with **transitive-dependency noise**. Open Eventz's high-severity flags were all in the framework's bundled `sharp`/libvips (image processing) and `postcss` — and the app uses **no `next/image`**, so the `sharp` code path never runs (unreachable), while `postcss` runs only at build time. The *practical* runtime risk was ≈ none even though the number looked alarming. Judgment: **triage by reachability and runtime-vs-build-time, not by the count** — and separate *actual* risk from *optics* (a public repo's audit score a reviewer might glance at).

**What to say in an interview.** *"`npm audit` flagged several high-severity issues, but they were all transitive to the framework — an image library the app never invokes because it does no image optimization, and a build-time CSS tool — so the exploitable runtime risk was effectively zero. I treat an audit as input to triage by reachability, not a number to react to, while acknowledging the public-repo optics separately."*

---

## H. Prepping a repo to go public — the sensitivity scan

**The concept.** Making a repo public is effectively **irreversible**: the entire **git history** becomes world-readable forever, not just the current files. So before flipping visibility you scan for two classes of exposure — **secrets** (API keys, tokens, private keys) and **PII** (personal/contact data) — and you scan **history, not just the working tree**, because a secret committed once and deleted later still sits in an old commit. For Open Eventz: grepped tracked files + the full `git log` history for secret patterns (JWTs, `sk-…`, private keys) and PII (emails, phone numbers) across both repos, and **extracted text from the binary files** (`.docx`/`.xlsx`) since a text grep skips them. Result: clean — with one honest residual: a regex can find emails/phones but **can't reliably flag a plain person's name**, so free-text notes in a binary file still warrant a human eyeball. Secondary insight: the **app repo is the higher-risk one** (it holds the real keys locally) — so `.env*` being git-ignored *plus* a clean history is what makes it safe to publish, not obscurity.

**What to say in an interview.** *"Before making the repos public I ran a sensitivity scan — secrets and PII — across tracked files **and full git history**, because history is permanent even after you delete a file. I also extracted text from the binary docs, since a grep skips those, and I was explicit that a pattern scan can't catch a plain name in a notes cell — that needs a human check. The riskiest repo was the app one because it holds the real keys locally, so I confirmed the env file was git-ignored and the history was clean before publishing."*

---

## I. The 5-layer AI PM stack — as shipped evidence

**The concept.** A useful frame for AI product work is a **5-layer stack**: **Model** (capability/cost/latency), **Context** (prompts, RAG, what the model sees), **Orchestration** (how the model call fits inside a larger workflow), **Governance** (evals, guardrails, observability), and **Human** (judgment that can't be delegated). It lets you locate any AI-product problem in the right layer instead of "prompt and hope." Open Eventz maps cleanly, with a real artifact per layer: a documented **model** selection (Sonnet-over-Haiku), owned + versioned prompts and a calibration set (**Context**), an ingest pipeline with the LLM call mid-workflow plus fallback (**Orchestration**), a six-layer risk model + confidence tiers + two-tier testing + observability dashboards (**Governance**), and a full documented judgment trail (**Human**). The interview value isn't reciting the layers — it's showing **evidence at every one**, which is rare.

**What to say in an interview.** *"I think about AI products as a five-layer stack — model, context, orchestration, governance, human — so a problem lands in the right layer instead of getting solved by prompt-tweaking. In Open Eventz every layer has a shipped artifact: a documented model-selection call, versioned prompts plus a calibration set as owned context, the LLM call embedded in an ingest pipeline with fallback, a governance layer of confidence tiers and two-tier testing and dashboards, and a full human judgment trail. The point isn't the vocabulary — it's having real evidence at every layer."*

---

## J. A reskin is a product decision — presentation vs. trust-sensitive data

**The concept.** A "visual-only" reskin sounds like it can't change behaviour — but a presentation choice *becomes* a product decision the moment it touches trust-sensitive data. During Open Eventz's "Weekend Paper" reskin, the child-supervision "can I drop my kid off?" indicator moved from **colour-coded alarm** (red = no / green = yes) to **one calm, text-driven callout** ("instruction, not alarm — no red-on-pink"). That isn't styling: a wrong "you can drop off" answer must never be *dressed in reassuring colour*, so the meaning has to live in the words, not the hue. The discipline that makes it safe is keeping the resolving logic (`getSupervisionBadge`) and its unit tests **independent of presentation** — a theme swap changes how it looks and never what it decides (proven by the fact the reskin didn't touch that suite). Bonus practice: design feedback was reviewed on a **before/after mockup for sign-off before any code**, so the call was made deliberately, not discovered after shipping.

**What to say in an interview.** *"People assume a reskin is cosmetic, but a presentation choice becomes a product decision the second it touches high-stakes data. In my kids-events app, a 'visual-only' refresh was where I decided to stop colour-coding the drop-off-safety indicator — green/red can imply a safety verdict, and the model behind it can be wrong — so I moved the meaning into calm text and kept the logic and its tests independent of the styling. A theme swap can change the look but never the behaviour. And I reviewed the change on a mockup before writing code, rather than eyeballing it after."*

---

## K. Scope a feature by risk, not demand — the accounts decision

**The concept.** "Add user accounts" reads like one feature; it's actually a **step-change in risk**. On a managed auth provider (Supabase) the login itself is a day or two — but you inherit account lifecycle, session security, and the real cost: becoming a **custodian of personal data**, with privacy-law exposure, breach-notification duties, and — for a **kids** product — potential **COPPA** liability the moment any feature stores data *about a specific child*. So the decision isn't "accounts: yes/no," it's **"what's the thinnest identity that unlocks the one feature I actually want?"** For a cross-device saved list, that's **OAuth-only, adults-only, no child data, row-level security locked per user** — ~80% of the value at ~20% of the risk; full password auth + child profiles is where cost *and* liability spike. It's also a conscious crossing of the **portfolio→real-product line** (the project was deliberately scoped as a demo precisely to avoid this obligation surface).

**What to say in an interview.** *"When someone asks for 'user accounts,' I scope it by risk, not demand. The login is trivial on a managed auth provider — the real cost is becoming a data custodian, with privacy law and, for a kids app, COPPA if you ever store a child's data. So I ask what the smallest identity is that unlocks the feature: for cross-device saves, OAuth-only, adults-only, no child records, row-level security per user gets most of the value at a fraction of the liability. Recognising that 'accounts' crosses the demo-to-real-product line — and choosing the lightest identity that clears it — is the judgment, not wiring up a login."*

---

## L. Guard the output, not just the process — the post-ingest data-quality gate

**The concept.** Unit and end-to-end tests run on **mocked** data and prove one thing: the *code* matches my intent. They are structurally blind to bad *data* — a source that changes shape, an empty field, a wholesale time shift. A pipeline that reports "success" while writing garbage is **worse** than one that fails, because nothing tells you. So there's a distinct layer whose job is different from testing: after every ingest it asserts invariants against the **real database** and turns the pipeline **red** on violation — age variety, **no adult-titled event stored kid-visible**, the toddler filter actually *narrows*, per-source non-empty, freshness, plausible start times, plus a **live-source canary** that confirms the upstream source still exposes the field we depend on. Two incidents proved the layer earns its place: the client-side-render break (#17), where every event fell to an "all ages" fallback while all tests stayed green, and the timezone shift (#18). The design rule: **point the check at the thing the user actually sees (the output), because every intermediate "success" signal can lie** — and the check belongs *in front of* the user-visible write, not merely reporting after it (detection is not prevention).

**What to say in an interview.** *"Logic tests can't catch bad data — they run on mocked inputs and only prove the code does what I intended. But a data product fails when the data is wrong, even if the code is perfect. So I added a layer that runs against the real database after each refresh and asserts what has to be true: ages have variety, no adult event is stored kid-visible, the age filter actually narrows, the sources aren't empty or stale — and it turns the pipeline red if not. A green checkmark should mean the thing you care about is true, not just that the code ran. And I put the guard on the output the user sees, because every intermediate 'success' can lie."*

---

## M. LLM-primary classification with a governance pre-filter — and fail-closed

**The concept.** Deciding "is this event for kids?" started as a **keyword deny-list**. It was brittle in both directions: every new source needed the list re-tuned, and it broke on phrasing it hadn't seen. I inverted the architecture to **LLM-primary** — the model reads the unstructured event text, judges kid-relevance, and **stores its reasoning** (auditable, not a black box). A small keyword pre-filter survives, but only for **governance**: an obvious hard-block layer (an explicit adults-only override), not the primary decision. Two properties make it safe and scalable: it **generalises across sources** — the same classifier hid a *"Pop & Pour"* 21+ wine night on a brand-new calendar with no new rules, where a keyword list would have needed a rewrite — and it is **fail-closed**: an LLM error or a low-confidence result sets `kid_relevant = false` rather than guessing, because a wrong *"this is for kids"* is the expensive direction. The bounding rule pairs with #17: **use an LLM for irreducible ambiguity, deterministic code for structured data that already exists** — never an LLM to reconstruct data a source is already handing its own front-end.

**What to say in an interview.** *"My kid-relevance classifier used to be a keyword deny-list — brittle, and it needed re-tuning for every new source. I made it LLM-primary instead: the model reads the messy event text, decides, and stores its reasoning, so it's auditable. I kept a tiny keyword layer, but only as a governance hard-block, not the main decision. Two things make it work at scale: it generalises — the same model correctly hid a 21-plus wine event on a source it had never seen, no new rules — and it's fail-closed, defaulting to 'not for kids' when it errs or isn't confident, because a false 'kid-friendly' is the costly mistake. The judgment I care about is knowing when the LLM is the right tool: irreducible ambiguity, yes; structured data that already exists, no."*

---

## N. Measuring a discovery product — a north star, not vanity metrics

**The concept.** A discovery product's success isn't downloads or pageviews — it's whether a parent actually *found something to do*. So the north star is **weekly active discoverers** (people who engage the funnel), and the funnel is instrumented end-to-end: view → filter/engage → intent → **conversion**, where conversion is defined as *adding an event to a calendar* or *getting directions* — the real "I'll show up" signals, not a like. It's **channel-segmented from day one** (organic/SEO vs. direct vs. referral) so acquisition is a *dimension of the funnel*, not a separate report. And it was wired **before launch** (GA4 → BigQuery → two dashboards: a functional one for the funnel, a technical one for pipeline health + per-inference LLM cost), because you can't retrofit measurement onto traffic you've already lost. The discipline: pick the one metric that means the product did its job, define the steps that lead to it, and instrument them before you need them.

**What to say in an interview.** *"For a discovery product I don't measure downloads — I measure whether someone found something to do. My north star is weekly active discoverers, and I instrumented the whole funnel: view, filter, intent, then conversion, which I defined as adding an event to a calendar or getting directions — the real 'I'll show up' signals, not a like. I segmented it by channel from the start, so acquisition is a slice of the funnel rather than a separate dashboard, and I wired it before launch because you can't retrofit measurement onto traffic you've already lost. The judgment is choosing the single metric that means the product worked, then instrumenting the steps that lead to it."*

---

## O. Evals for an LLM feature — a calibration set, and the honest frontier

**The concept.** An LLM feature isn't done when the prompt "seems to work" — you need a way to *measure* it that survives prompt and model changes. I treat the prompt as an **owned, versioned artifact** and gate it with a **calibration set** of hand-labeled ground-truth examples, run in two tiers: a **deterministic tier in CI** (free, fixture-based, catches regressions on every push) and a **real-LLM tier** run on demand when the prompt or model changes. That turns "did I break the classifier?" into a number, not a vibe. The part interviewers reward is naming its **limits**: a calibration set scores accuracy against labels, not the richer things modern eval asks — rubric / LLM-as-judge scoring, regression suites across prompt versions, or A/B-ing a model change against a *product* metric rather than a model one. Knowing exactly where my eval coverage ends is the difference between "prompt and hope" and "I know what I'm not yet measuring."

**What to say in an interview.** *"I don't consider an LLM feature done because the prompt looks right — I gate it with a calibration set of hand-labeled examples, run deterministically in CI on every push and against the real model when the prompt or model changes. That makes 'did I break it?' a number, and it versions with the prompt as an owned artifact. What I'm honest about is the ceiling: it scores accuracy against labels, not rubric or LLM-as-judge evals, and it doesn't yet A/B a model change against a product metric. Knowing exactly where my eval coverage stops is the point — that's what separates measuring from hoping."*

---

## P. Demo vs. production — scoping to avoid obligations, and the checklist to cross

**The concept.** The sharpest honest thing about this project: **it has no real users, and that's a deliberate scope decision, not an oversight.** It was built as a portfolio artifact to demonstrate judgment — and instrumented *as if* live — specifically to avoid the obligations real users create: abuse controls, a data-correction feedback loop, and (because a wrong drop-off answer is a child-safety issue) moving supervision data from "verified where possible" to "verified before anyone relies on it." The judgment isn't "I didn't get users"; it's **knowing exactly what changes to go from demo to production, and choosing not to cross that line yet.** The checklist is written down: rate-limiting + abuse detection on engagement, an account-light path for cross-device sync (scoped by risk — see Concept K), scrape monitoring/alerting, a user-facing "this is wrong" correction mechanism, and the supervision-verification bar. Naming the line — and the cost of crossing it — is the real answer to "how would you take this to production?"

**What to say in an interview.** *"I'll be straight: it has no real users, and that's on purpose. I built it to show judgment and instrumented it like a live product, but I deliberately didn't launch to real parents — that creates obligations I'd have to earn: abuse controls, a correction loop, and moving the child-supervision data to a 'verified before anyone relies on it' bar, because a wrong drop-off answer is a safety issue. What I can point to is the exact demo-to-production checklist — rate-limiting, account-light sync scoped by risk, scrape alerting, a user correction mechanism, and the supervision standard. The judgment is knowing precisely what it takes to cross that line, and choosing when."*

---

*Last updated: 2026-08-19.*
