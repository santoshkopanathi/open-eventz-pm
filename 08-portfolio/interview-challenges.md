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

**What to say in an interview.** *"My local pre-push hooks were passing, so I assumed CI was green — it wasn't, for a while. A TypeScript config needed a package present in my local install but not in CI's clean one. I removed the dependency to fix it, but the real lesson is that a green local hook isn't a green pipeline; you verify against a clean environment and actually look at the CI run."*

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

*Last updated: 2026-07-26.*
