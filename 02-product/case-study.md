# Open Eventz — Product Case Study

**Live product:** [open-eventz.vercel.app](https://open-eventz.vercel.app)
**Role:** Product Manager (solo — strategy, data sourcing, architecture decisions, build oversight)
**Stack:** Next.js 14, TypeScript, Supabase (PostgreSQL), Tailwind CSS, Google Maps API, Vercel
**Timeline:** Concept to live product in under 3 weeks

---

## The Problem

Parents in Frisco and Plano, TX have no single place to find free activities for their kids. The supply of free programming is larger than it looks — Frisco Public Library, the Plano Libraries network, and Play Frisco (Frisco Parks & Recreation) collectively run dozens of events per week. But each publishes on its own platform, in its own format, with no shared standard for what "free" means, who needs to supervise a child, or how to add an event to a calendar.

The result: parents either check three websites manually, rely on word of mouth, or miss events entirely. This isn't a supply problem. It's a visibility and trust problem.

**Market context:** The Plano-Frisco metro has approximately 90,000–95,000 households with school-age children (US Census / ACS data). 73% of US families cite affordability as a top concern in 2026 (Family Travel Association). The demand for free, trustworthy, locally-specific activity discovery is real, measurable, and underserved.

---

## The Insight

Three things that existing solutions miss — and that Open Eventz addresses directly:

**1. Geographic blind spots in existing apps.** ParentPass, the closest competitor, built its community in Tarrant County (Fort Worth). Its own FAQ states Tarrant County users are its top priority. Collin County (Plano/Frisco) is 30–40 miles away with entirely separate city governments and library systems — coverage thins precisely where Open Eventz starts.

**2. The "free" filter doesn't exist anywhere.** Eventbrite's "free" tag is self-reported by organizers and unreliable. City and library websites have no "free" filter at all. Open Eventz verifies free status at the source level: library programs are free by institutional default; Play Frisco events are verified per event with explicit price display where costs exist.

**3. Nobody tells parents whether they need to stay.** Drop-off vs. parent-must-stay is a real, practical decision that varies by venue and program — and no competitor surfaces it. Open Eventz introduces a structured supervision policy framework with three confidence tiers: event-specific statement → venue general policy → unverified. "Check with venue" is the honest default rather than a guessed threshold. Frisco Library's policy (children 10+ may attend unattended, per Service Policy §8.5) is verified, stored in a dedicated database table, and surfaced directly on every event detail view.

---

## Key Product Decisions — and the Tradeoffs

**Decision 1: Three sources in v1, not nine.**
The source inventory catalogued 9+ potential sources across Plano and Frisco. Scoping to three was deliberate — both library systems and Play Frisco cover the highest-frequency, highest-trust free programming in the metro. The remaining six sources were ruled out with documented rationale (VisitFrisco.com's unstable build-ID-dependent URLs, Plano Parks & Rec's PDF-only catalog, etc.).

*Tradeoff accepted:* Narrower coverage in v1 in exchange for higher data reliability and a faster ship.

**Decision 2: No Eventbrite API.**
Eventbrite deprecated its public event-search endpoint in February 2020 specifically to block third-party aggregators. The remaining access is event-by-ID only — useful for known events, useless for discovery. This ruling forced a more defensible, source-by-source integration strategy.

**Decision 3: Supervision data is never guessed.**
When no verified policy exists for a source, the UI shows "check with venue" — never an inferred threshold. This decision was made after discovering that Frisco Library's own city code and its own Summer FAQ disagree on the minimum unattended age (9 vs. 10). If a venue's own documents disagree, inferring a number from context would introduce false confidence where the stakes are real safety decisions. The supervision policy is stored in a dedicated database table, versioned, and sourced to the exact policy document.

**Decision 4: Database-first architecture over live feed fetching.**
The PRD originally planned to fetch live from vendor feeds on every page load. During build, this was replaced with a daily ingest pipeline that writes to Supabase, with the UI reading only from the database. This was a deliberate architectural shift, not a workaround — it means source failures are isolated to the pipeline rather than surfacing as broken user experiences, page loads are fast (a single DB query, not three live HTTP fetches), and the last good data always shows even if a vendor has a bad day.

---

## What the Data Reality Actually Looked Like

The PRD assumed three sources with official, vendor-supported data feeds. The actual data acquisition story was more complex — and more interesting.

**Frisco Library (BiblioCommons):** BiblioCommons has quietly retired their RSS feed without announcement. Seven different integration approaches were attempted and failed — RSS URLs returned 404, iCal exports redirected to the HTML page, DevTools network inspection found no separate API call because events are fully server-rendered. The final solution was HTML scraping: parsing the rendered page using the CSS class structure (`<li><div class="cp-events-search-item">`). Along the way, two bugs were discovered and fixed: BiblioCommons formats times as `10:00am` without a space, which JavaScript's `Date()` constructor can't parse; and the same event appears across all three audience feeds, causing duplicate inserts without deduplication logic.

**Plano Libraries (Communico):** Communico's documented XML export API requires vendor-issued authentication tokens they don't issue publicly. A working feed was found by a different route: the library's own website exposes an RSS icon that, when clicked, reveals a feed URL with base64-encoded filter parameters embedded in a JWT token. Decoding that token showed a default of 1 day of events; modifying the token to encode 365 days and re-encoding in base64 produced a working feed returning 500+ events across all five Plano branches.

**Play Frisco (CivicPlus):** City calendar scraping as planned, with graceful fallback to a source-unavailable banner if the HTML structure changes.

The broader pattern — official API paths blocked or retired, but the website itself always talks to something — is a recurring reality in civic data work. The ingest pipeline is built with per-source error isolation specifically because of this: if BiblioCommons changes their HTML structure next month, only the Frisco Library ingest fails, not the whole app.

---

## What Was Validated Before Building

Two engineering learnings from the build are worth naming explicitly because they illustrate a "validate first" discipline:

**BiblioCommons audience filter was silently ignored.** The original ingest fetched the events feed three times — once per audience segment (Children 0–5, Children 6–12, Teens) — and assigned age ranges at the feed level. A quick empirical check (fetching all four URLs in parallel and comparing results) revealed all four returned identical events. The `audience_id` filter parameter is ignored server-side; it requires a browser session cookie. The bug was masked downstream — a per-event page scrape was overwriting the wrongly-assigned age ranges for near-term events, while far-future events silently received incorrect age data. Fix: drop the three-feed loop, fetch once from the unfiltered endpoint, treat the per-event page scrape as the sole source of age truth.

**Plano's 1-month detail page window was a premature optimization.** To reduce HTTP requests, the ingest was initially configured to fetch individual event detail pages only for events within one month, using title parsing as a fallback for the rest. Testing 10 consecutive live Plano events showed 100% had structured AGE GROUP data on their detail pages — Communico treats it as a required field. The optimization was solving a problem that didn't exist, while creating a real one: events beyond one month silently got no age data and became invisible under any age filter. Fix: remove the window entirely, always fetch the detail page.

The lesson in both cases is the same: validate assumptions about third-party API behavior empirically before building logic on top of them. A two-minute test would have caught both issues immediately.

---

## What Was Built

**Live at:** [open-eventz.vercel.app](https://open-eventz.vercel.app)

A responsive React/Next.js web app with a daily data pipeline that keeps event data current without manual intervention.

**Data pipeline (ingest):**
- Three scrapers run in sequence via a secured `/api/ingest` endpoint
- Events upsert into Supabase using composite IDs (`{source}-{original-id}`) to prevent duplicates without extra deduplication queries
- Stale Play Frisco events are cleaned up automatically — events not in today's batch are deleted
- Security: endpoint protected by a bearer token; service role key never leaves the server (principle of least privilege — public client uses anon key, admin client uses service role key, separation enforced at the codebase level)
- Ingest currently runs locally due to Vercel's 10-second serverless function timeout — a known constraint with documented upgrade paths (split endpoints, GitHub Actions cron, Railway/Render for long-lived jobs)

**User-facing features:**
- Unified event list, date-sorted, sourced from Supabase on each filter interaction
- City-first navigation: Frisco tab (Frisco Library + Play Frisco) and Plano tab (all five branches) — solves information overload from mixed-source default view
- Age filters: Toddlers (0–5), Kids (6–12), Teens — using overlap matching so a Kids filter correctly returns "All Ages" events
- Event detail: description, time, location, free/paid status, supervision badge (drop-off yes/no based on verified policy), registration requirement
- Add to Calendar: Google Calendar, Apple Calendar, downloadable ICS
- Map view: color-coded venue pins by source, with embedded Google Maps directions (enter starting address or use current location)
- Attending tracker: anonymous per-event engagement, shared count in Supabase, personal selection in localStorage

**Schema design decisions:**
- `events` table indexed on `source`, `start_datetime`, `is_free` — the three most common filter combinations
- `supervision_policies` table separate from events — different update cadence (yearly vs. daily)
- `like_counts` table separate from events — prevents like updates from locking the main events table
- `raw_json` column on events — stores original parsed data for debugging when source HTML changes

---

## How Success Is Measured

**North star metric: Weekly Active Discoverers (WAD)**
Unique visitors per week who complete at least one intent action (Add to Calendar, Share, or Get Directions) after applying at least one filter. The filter gate distinguishes visitors who used the product as designed from those who landed and bounced.

| KPI | What it measures | Benchmark hypothesis |
|---|---|---|
| Filter engagement rate | % sessions with ≥1 filter applied | 55%+ within 60 days |
| Intent action rate | % detail views → calendar/share/directions | 25%+ |
| Feed reliability | % ingest runs completing without errors | 99%+ libraries; 90%+ Play Frisco |
| Supervision coverage | % events with Tier 1 or 2 verified policy | 40%+ within 30 days |
| Return visit rate | % week-1 intent actors returning in week 2 | 30%+ |

---

## What's Next

**v1.1 (in progress):**
- City-first navigation tab structure (replacing source dropdown)
- Age indicator badge on event cards in list view
- Play Frisco LLM age inference — Claude API call at ingest time classifies each event as toddler/kids/teen/family/adult using event title and description; confidence tiers determine UI treatment; low-confidence inferences show nothing rather than a wrong badge; inferred age displayed with a `~` prefix and "estimated from event description" disclosure tooltip

**v2:**
- Push notifications — the single feature most likely to drive return visits without requiring accounts
- Plano Parks & Recreation as a fourth source
- Automated ingest via GitHub Actions cron to remove the local-run dependency

**Strategic horizon:**
The data architecture is per-metro repeatable. Adding a new city is a documented checklist — identify sources, verify feeds or ToS, map to the normalized schema — not a bespoke engineering project. The company vision is deliberately broader than event aggregation: helping people discover, access, and create the opportunities that matter to them. Open Eventz is the first proof point, not the ceiling.

---

## What This Project Demonstrates

**AI-native product development.** The full product lifecycle — market research, competitive analysis, PRD, data schema, KPI framework, prototype, and production build — was executed using AI tools (Claude Chat, Cowork, Claude Code) as a genuine integrated workflow. This is not a project built with AI assistance; it is a project that demonstrates what a PM can ship when AI is the primary execution layer.

**Structured PM methodology applied in real time.** Decision log, source inventory, supervision confidence tiers, and KPI definitions were documented before and during build — not retrofitted after. When the build diverged from the plan (architecture, vendor APIs, ingest scheduling), the rationale was logged explicitly. The artifact trail shows the difference between planning and reality, which is the actual work.

**Data strategy under real-world constraints.** No clean API exists for this use case. Both major library vendor APIs — BiblioCommons and Communico — are either retired or gated behind non-public authentication. The sourcing strategy navigated that with empirical validation, creative reverse-engineering, and a resilient architecture that isolates source failures from user experience.

**Validate before building.** Two explicit engineering learnings from the build document cases where assumptions were tested empirically before being relied upon — and where failing to validate earlier caused silent data quality issues. This discipline is documented in the BUILD-LOG and directly references the PM practice of treating technical assumptions as hypotheses, not facts.

**Trust as a product feature.** The supervision policy framework, the `~` disclosure on LLM-inferred age data, and the "check with venue" default for unverified policies all reflect the same principle: be honest about what you know versus what you're inferring, because a wrong confident answer is worse than an honest uncertain one. This is a distinctly AI PM judgment call — knowing when not to surface a data point.

---

*Full documentation — PM OS, MVP PRD with implementation notes, KPI framework, market research, competitive analysis, and BUILD-LOG — available on request.*
