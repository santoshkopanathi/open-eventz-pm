# MVP PRD — Open Eventz (Plano-Frisco Pilot)

*Last updated: 2026-06-18. Changes from previous version: expanded from 2 to 3 data sources (added Play Frisco / Frisco Parks & Rec); updated all affected sections accordingly; logged VisitFrisco.com ruling. Latest update: added empty state, loading state, error state, mobile map view, and event category icon system to scope.*

---

## Overview

A free, kid-activity discovery tool that solves one specific problem first: there's no single place to see what's actually free and happening for kids in the Frisco and Plano area this week. This PRD scopes the smallest version of Open Eventz that's real (live data, not mocked), automatically current (no manual upkeep where feed-based), and demonstrable as a portfolio piece, fast.

## Goal & Success Criteria

**Goal:** Ship a working, responsive web app — live in days, not weeks — that pulls real, currently-accurate events from three v1 sources and displays them with the filters that came directly out of persona research.

**Success for this MVP specifically:** A visitor can open the app and see real upcoming events across Frisco and Plano, correctly tagged for age range, free-verification, recurring-vs-one-time, and (where known) supervision requirements — without anyone having manually entered that data for the library sources. From any event, the visitor can add it to their personal calendar in one tap, or share it with a spouse or friend via the native share sheet. It needs to look and work well enough to anchor a portfolio conversation about product thinking, data strategy, and execution, not to have real user traction yet.

## Scope

**In scope for v1 — three data sources:**

| Source | Type | Integration method | Free signal |
|---|---|---|---|
| Frisco Public Library (BiblioCommons) | Library programs & events | Public RSS calendar feed — vendor-confirmed, official | Free by default — library programs |
| Plano Public Library (Communico) | Library programs & events | Communico public XML export endpoint — documented, vendor-supported | Free by default — library programs |
| Play Frisco / Frisco Parks & Rec | Community events, camps, programs | Scrape of friscotexas.gov city calendar — no official feed available | Mixed — community events (free); camps (paid, filtered out or clearly tagged) |

**Also in scope:**
- A responsive web app (works well on phone browsers; no native iOS/Android build)
- Plain browsable/searchable event list as the headline feature
- Four secondary filters/tags: age-range filter, "verified free" badge, recurring vs. one-time tag, and supervision/drop-off flag (populated where data exists, shown as "check with venue" otherwise — never guessed)
- Source filter (so a user can view all sources combined or narrow to one)
- **Date range filter** — from/to date pickers plus quick-pick presets ("This weekend," "Next 7 days") to bound a long event list to a manageable window, rather than scrolling a flat chronological feed
- **"Added to calendar" indicator on the list view** — once an event has been added via the detail view's Add to Calendar button, its card in the list picks up a small visible badge/checkmark. Solves a real scale problem: once the list grows past a handful of events, a visitor has no way to remember what they've already committed to without re-opening every card. Same localStorage mechanism as the detail-view confirmed state — both read from the same `added_<event_id>` flag, so they always agree.
- **"Add to Calendar" button** on every event detail view — generates a standard `.ics` file on the fly from existing event data (title, date/time, location, description, source URL), compatible with Google Calendar, Apple Calendar, and Outlook. One-time export, not a live sync — if an event is cancelled or rescheduled after export, the user's calendar won't auto-update (acceptable tradeoff at this stage; displayed honestly if needed).
- **"Share" button** on every event detail view — triggers the device's native share sheet, letting the user send the event link via SMS, WhatsApp, email, or copy-link with no app-side backend required. Requires each event to have a stable, shareable URL (e.g. `openevents.app/event/<event-id>`).
- **Event image / thumbnail** displayed on event cards and detail view — pulled from each source's feed/scrape where available (Communico feed includes an optional image field; BiblioCommons RSS may include an image element; Play Frisco scrape attempts to capture featured image from event page HTML). Falls back to a clean category-based placeholder icon (book icon for library events, park icon for Play Frisco) when no image is available — UI stays consistent regardless of source.
- **Price display** on both event card and detail view — shows "Free ✓" for verified-free events, or the extracted price for paid events (e.g. Play Frisco camps). Falls back to "See event page for pricing" when the scrape can't reliably parse a clean cost value. This is a trust requirement, not just a nice-to-have — surfacing paid events without upfront pricing would feel misleading alongside free ones.
- **Dynamic map view** — a "Map" toggle/button on the event list page renders all currently-visible events (post-filter) as pins on a Google Map simultaneously. Map state mirrors the list state: applying or removing a filter updates the map in real time. Tapping a pin shows the event name and a link to its detail view. Requires Google Maps JavaScript API (or Maps Embed API for a simpler implementation).
- **Like / Attending engagement button** on every event card and detail view — a single toggleable icon (e.g. ♥ or ✓ Attending) that any visitor can tap without an account. Tapping increments the shared like count; tapping again decrements it. The user's own selection is persisted in browser localStorage so it's restored on their next visit to the same device/browser. Two-layer storage: shared like count lives on a lightweight server endpoint (one API call per like/unlike); personal selection lives in localStorage (no server call needed for restoration). Caveat: localStorage is per-device — a user who likes something on their phone won't see that selection on their laptop, and vice versa. This is expected and acceptable at v1 with no accounts.
- **Event category icon system** — each event card displays a color-coded icon tile based on the event's category, used as the thumbnail when no image is available from the source feed/scrape. Eight categories with distinct icon + background color combinations: Storytime/reading (book icon, green tile), Arts & crafts (palette icon, purple tile), STEM/maker (gamepad icon, blue tile), Outdoor/nature (trees icon, green tile), Movie/performance (film icon, coral tile), Community event (users icon, amber tile), Music/dance (music icon, pink tile), and a General event fallback (calendar icon, gray tile) for anything that doesn't match a category. Icons are sourced from the Tabler icon set — no licensing concerns, consistent visual weight across all categories.
- **Empty state** — displayed when the current filter combination returns no events. Shows a calendar-off icon, the message "No events found — no events match your current filters. Try expanding your search or adjusting the date range," and a "Clear all filters" button that resets all active filters in one tap. Triggered by two scenarios: filters set too narrowly (most common), or a first-time visitor before any data loads.
- **Loading state** — displayed while live feeds are being fetched on page load. Shows a spinner, "Loading events..." label, and a rotating Frisco/Plano trivia fact in a card below the spinner (e.g. "Frisco was named after the St. Louis–San Francisco Railway, which arrived in 1902"). Trivia rotates randomly from a bank of 8–10 real local facts — turns dead wait time into something locally charming and reinforces the "this app knows Plano-Frisco" positioning.
- **Error state** — displayed when one or more source feeds/scrapes fail. Source-specific: if only Play Frisco fails, library events still display normally — the error banner appears above the working event list, not instead of it. Banner copy: "Unfortunately, [Source Name] events are temporarily unavailable — we're working on it. [Other source] events are loading normally." Uses an amber/warning treatment rather than red/danger — this is a temporary inconvenience, not a critical failure. Never crashes the whole app if a single source is down.
- **Mobile map view** — map is accessed via a "Map" toggle at the top of the screen (consistent with desktop). Full-screen map with color-coded pins (blue for library events, green for free Play Frisco events, amber for paid Play Frisco events). A small floating event card sits persistently at the bottom of the screen showing the nearest/selected event — always visible, not triggered only by a pin tap. Tapping a pin updates the floating card to show that event's details. Tapping the floating card navigates to the full event detail view.

**Explicitly out of scope for v1** (not forgotten, just sequenced later):
- Any additional sources beyond the three above — Plano Parks & Rec, museums, Frisco ISD, YMCA, SummitCentral, Eventbrite, VisitFrisco.com — all cataloged in PM OS Section 6, all backlog (see VisitFrisco.com note below)
- Push notifications (Dave's core ask — real, just not v1)
- User accounts, login, or saved favorites
- True zip-code-radius geocoded search — with three fixed sources, "near me" in v1 means filtering by source/city, not a geocoded radius query
- Native mobile apps

**VisitFrisco.com ruling (logged here for the record):** After investigating the site's network calls directly, the events calendar loads via Next.js internal data URLs that include a build ID (`esoUENnin8q1gQN52KTW3`) that changes every time the site redeploys — no stable, intentional feed exists. The category-specific URLs investigated (concerts & live music, sports events) also confirmed the content is the wrong audience for Open Eventz. Ruling: backlog candidate only, do not build against.

## Primary Users for v1

This version most directly serves Selina's persona (recurring/predictable library programming is exactly what weekly storytimes and ongoing programs are) and partially serves Roger & Lindsay's (the age-range and supervision filters address their multi-kid-attention-level problem). Adding Play Frisco also begins to address Dave's weekend-activity need, since city events skew toward one-off weekend programming. Dave's push-notification need remains deferred — v1 is a pull (browse) experience.

## Core User Flow

1. Visitor opens the web app.
2. App fetches live event data from all three sources in the background (library feeds load automatically; Play Frisco data fetched via scrape on a scheduled refresh cadence — see Technical Approach).
3. Visitor sees a unified list of upcoming events sorted by date. Each event card shows: thumbnail/placeholder icon, title, source, date/time, age range, free-verification badge or price, recurring/one-time tag, supervision status, and a small "Added" indicator if the event is already on the visitor's calendar (see below).
4. Visitor filters by source, age range, date range, and/or recurring-vs-one-time to narrow the list.
5. Visitor toggles to **Map View** — all currently filtered events render as pins on a Google Map simultaneously. Tapping a pin shows event name and links to the detail view. Toggling back returns to the list with filters intact.
6. Visitor taps an event for full details (time, location, description, price or "Free ✓", supervision status, source link to original listing).
7. From the event detail view, visitor can:
   - **Add to Calendar** — downloads an `.ics` file that opens in their default calendar app. Once added, the button switches to a confirmed "Added ✓" state (checked against localStorage), and the same event's card in the list view picks up a small "Added" badge — so a visitor scanning a long list can tell at a glance what they've already committed to without re-opening each event.
   - **Share** — triggers the native share sheet to send the event link via SMS, email, WhatsApp, or copy-link
   - **Get Directions** — taps the Google Maps icon to deep-link into Google Maps with the venue pre-filled as destination; user enters their origin in Google Maps
   - **Like / Attending** — toggleable engagement button; increments or decrements the shared like count via a lightweight API call; personal selection cached in browser localStorage and restored on next visit

## Data Model (Event Record)

| Field | Notes |
|---|---|
| `title`, `description`, `start_time`, `end_time`, `location` | Pulled directly from each source |
| `source` | "Frisco Public Library", "Plano Public Library", or "Play Frisco / Frisco Parks & Rec" |
| `event_id` | Stable unique identifier per event — required to generate shareable URLs and `.ics` files |
| `age_range` | Parsed from feed/scrape where available; gracefully degrades to "age not specified" — never guessed |
| `image_url` | Pulled from feed/scrape where available; falls back to category-based placeholder icon (book for library sources, park for Play Frisco) — never left blank in the UI |
| `price` | "Free" for verified-free events; extracted cost string for paid events (e.g. "$150/week"); falls back to "See event page for pricing" if not parseable from scrape |
| `venue_address` | Full address string for each event location — required for Google Maps deep-link and map pin placement. Libraries have fixed branch addresses (can be hardcoded per branch); Play Frisco events may vary by location and need to be extracted from scrape |
| `like_count` | Integer stored server-side, incremented/decremented via a lightweight API endpoint on like/unlike. User's personal like selection stored in browser localStorage (key: `liked_<event_id>`) — restored on page load, no server call needed for restoration |
| `added_to_calendar` | Client-side only, no server component — boolean stored in browser localStorage (key: `added_<event_id>`) when the user taps Add to Calendar. Read by both the detail view (to show the confirmed "Added ✓" state) and the list view (to show the small list-card badge), so the two always stay in sync since they share one flag |
| `free_verified` | True by default for both library sources; for Play Frisco, set per-event based on whether cost is listed (free community events = true; paid camps = false, still surfaced but clearly tagged as paid) |
| `recurring` | Derived from feed/scrape data (Communico explicitly flags recurring; BiblioCommons and Play Frisco may need light parsing) |
| `supervision_age_threshold`, `supervision_confidence_tier`, `supervision_last_verified` | From Supervision Policy Intake (PM OS Section 7) — Frisco Library is Tier 2 (threshold 10); Plano Library and Play Frisco are Tier 3/unverified, display "check with venue" |

## Technical Approach

- **Plano Public Library:** Communico public XML export (`api.communico.co/v2/<keyword>/events/export.xml`) — documented, vendor-supported, fetched on page load.
- **Frisco Public Library:** BiblioCommons public RSS calendar feed — confirmed vendor capability; exact endpoint URL for Frisco's library still needs to be located at build time (capability confirmed, specific URL not yet retrieved).
- **Play Frisco / Frisco Parks & Rec:** Scrape of the city event calendar (friscotexas.gov/events or townoffrisco.com calendar) — no official feed exists. Unlike the library feeds, this requires a scheduled refresh (suggested: once daily) rather than live-on-load, and needs a ToS check before scraping is built. Events should clearly indicate source and link back to original listing.
- **CORS handling:** If either library feed blocks direct browser fetches, route through a small server-side proxy — keeps the "official feeds only" position intact without scraping.
- **Database:** Not required for the two library sources (fetched fresh on load). Play Frisco scrape results will need lightweight storage (a simple JSON cache or small DB table) since they can't be re-fetched on every page load.
- **Google Maps integration:** Two touchpoints — (1) dynamic map view on the list page requires the Google Maps JavaScript API (API key needed; free tier covers up to $200/month credit, sufficient for portfolio-scale usage); (2) navigation deep-link on event detail page uses a standard `https://maps.google.com/?q=<address>` URL — no API key required. Library branch addresses can be hardcoded; Play Frisco venue addresses need to be extracted from the scrape.
- **Like count backend:** A minimal server-side endpoint is required to store and serve shared like counts (one integer per event). This is the only backend need introduced by the like feature — the user's personal selection is handled entirely client-side via localStorage (`liked_<event_id>: true/false`). A simple key-value store or lightweight DB table is sufficient.

## Success Metrics

**North star metric: Weekly Active Discoverers (WAD)** — the number of unique visitors per week who complete at least one intent action (Add to Calendar, Share, or Get Directions) after applying at least one filter. The filter gate is deliberate: it separates visitors who used the product as designed from those who landed and bounced without engaging.

Five supporting KPIs cover the full product funnel:

| KPI | What it measures | Benchmark hypothesis |
|---|---|---|
| Filter engagement rate | Percentage of sessions with at least one filter applied — measures whether the differentiating feature is actually being used | 55%+ within 60 days of launch |
| Intent action rate | Percentage of event detail views that result in a calendar add, share, or directions tap — the conversion metric | 25%+ of detail views |
| Feed reliability | Percentage of page loads where all three sources return events successfully — measures data strategy execution | 99%+ libraries; 90%+ Play Frisco |
| Supervision data coverage | Percentage of events with a Tier 1 or Tier 2 verified supervision entry — tracks progress of the differentiating intake work | 40%+ within 30 days of launch |
| Return visit rate | Percentage of visitors who return in a second distinct week within 30 days — the retention signal | 30%+ among week-1 intent actors |

Intentionally not tracked: raw page views, bounce rate, time on page, and like counts — none of these reliably indicate whether the product delivered on its core promise.

**Full KPI framework:** `08-portfolio/kpi-framework.md` — includes full definitions, how-to-measure specifics, and what-to-do-if-low guidance for each metric.

---

## Risks / Open Items

- Frisco Library's exact BiblioCommons RSS endpoint URL needs to be confirmed before build.
- Play Frisco's city calendar terms of service need to be checked before the scraper is built — not yet done.
- Play Frisco's event pages may change structure without notice, breaking the scrape; build with graceful fallback (show "Play Frisco events temporarily unavailable" rather than crashing).
- CORS may block direct library feed fetches — proxy fallback plan accepted.
- Feed/scrape data quality varies by source — age-range, recurring, and cost fields may be missing or inconsistent; app degrades gracefully throughout rather than guessing.
- Plano Library and Play Frisco supervision data are Tier 3/unverified — both will show "check with venue" in v1.
- Like count abuse: without accounts, nothing prevents a single user from repeatedly liking/unliking to inflate counts. Rate-limiting by IP on the like endpoint is a lightweight mitigation worth adding at build time.

## Source Decision Log

| Source | v1 decision | Rationale |
|---|---|---|
| Frisco Public Library | ✅ In | Official RSS feed confirmed; free by default; highest-trust data |
| Plano Public Library | ✅ In | Official Communico XML feed confirmed; free by default; vendor-documented |
| Play Frisco / Frisco Parks & Rec | ✅ In | Adds city-level community events missing from libraries; scrape viable with ToS check |
| VisitFrisco.com | ❌ Backlog | Build-ID-dependent Next.js URLs — unstable, no official feed; content mix (concerts, sports, nightlife) wrong for this product |
| Plano Parks & Rec | ❌ Backlog | No official feed; PDF-based catalog adds parsing complexity; viable v2 source |
| Frisco ISD | ❌ Backlog | Currently only sourced secondhand; needs direct verification first |
| Frisco Discovery Center museums | ❌ Backlog | Occasional free-day only; low event frequency; manual tracking burden not worth v1 effort |
| YMCA / SummitCentral / Eventbrite | ❌ Backlog | Mixed paid/free; no clean feed; lower priority vs. above |

## Out-of-Scope Reminder for Future Versions

Notifications (Dave), additional sources per backlog table above, true zip-radius search, native apps, and user accounts are all real, already-cataloged future work — none of this was forgotten, it's sequenced.

**Share — Option B (direct SMS/email send):** App sends event details directly to a phone number or email address on the user's behalf. Requires Twilio (SMS) and SendGrid or equivalent (email), introduces per-message cost, phone-number verification, and spam/abuse mitigation. Deliberately deferred to post-v1 once there's a user base to justify the infrastructure investment.

---
*Feeds into: Open Eventz PM OS. Recommend adding as Section 11 (MVP PRD) once reviewed.*

---

## Implementation Notes — What Changed During Build and Why

*This section documents where the implementation diverged from the plan above, and the reasoning behind each change. Added after build completion. The original PRD sections above are preserved as written — they represent the planning artifact. This section represents the execution reality.*

---

### Architecture Pivot: Live Feed Fetching → Database-First

**What the PRD planned:** Fetch live from vendor feeds on every page load. No database required for library sources.

**What was built:** A daily ingest pipeline that writes all three sources to Supabase (PostgreSQL), with the UI reading only from the database. Page loads hit a single Supabase query, not three live HTTP fetches.

**Why it changed:** The resilience argument became clear during build. If a vendor feed is slow or down, a live-fetch architecture surfaces that failure directly to the user. A database-first architecture isolates source failures to the pipeline — the last good data always shows, and source errors appear in ingest logs rather than as broken user experiences. Page loads are also measurably faster: one indexed DB query vs. three sequential HTTP fetches with parsing.

**Current ingest schedule:** Runs manually via a secured `/api/ingest` endpoint (bearer token protected). Vercel's free-tier serverless function timeout (10 seconds) prevents automated cloud scheduling — the full ingest takes 3-5 minutes due to per-event page fetching across ~400+ events. Current workaround: ingest runs locally, writing to the shared Supabase instance. Documented upgrade paths: split endpoints per source, GitHub Actions cron, or a dedicated background job service (Railway, Render).

---

### Data Source Reality: Official APIs → Creative Acquisition

**What the PRD planned:** Communico XML export (vendor-documented) and BiblioCommons RSS feed (vendor-confirmed capability) as the primary integration methods.

**What was built:**

*Frisco Library (BiblioCommons):* RSS feed retired without announcement (404). Seven integration approaches attempted and failed before landing on HTML scraping of the server-rendered event listing. Events are fully embedded in the HTML markup — no separate API call exists. Parser splits on `<li><div class="cp-events-search-item">` and regex-extracts each field. Bugs caught and fixed during build: time format normalization (`10:00am` → `10:00 AM`) and deduplication of events appearing across multiple audience feeds.

*Plano Libraries (Communico):* Documented XML API requires vendor-issued authentication tokens not issued publicly. Found a working path by inspecting the library website's own network calls: an RSS feed URL with base64-encoded JWT filter parameters. Decoded the token, modified the date range parameter from 1 day to 365 days, re-encoded, and produced a feed returning 500+ events across all five Plano branches. Branch address and coordinates lookup table added to enable accurate map pins per branch.

*Play Frisco (CivicPlus):* City calendar scraping as planned. EID-based two-pass scrape (listing page → individual event detail pages). Stale event cleanup runs at end of each ingest to remove events no longer on the city calendar.

**Broader pattern:** Both library vendor APIs — BiblioCommons and Communico — are either retired or authentication-gated. The workaround in both cases was finding what the website itself uses rather than relying on official documentation. Ingest is built with per-source error isolation specifically because of this pattern: if any source changes its HTML or URL structure, only that source fails, not the whole app.

---

### Age Data: Feed-Level Assumptions → Per-Event Page Scraping

**What the PRD planned:** Age range parsed from feed data where available; graceful degradation to "age not specified" otherwise.

**What was built:** Per-event detail page scraping for age data on both library sources, after discovering that neither vendor's feed includes age data reliably.

*Frisco Library:* The "Suitable for:" block only exists on individual event detail pages, not in the listing feed. A per-event fetch is required for every event. Also discovered that BiblioCommons' audience filter URL parameter (`audience_id`) is silently ignored server-side — all four audience-filtered URLs returned identical results. The three-feed loop was fetching the full 252-event catalogue three times over and assigning wrong default age ranges. Fix: single unfiltered endpoint, per-event page scrape as sole source of age truth.

*Plano Libraries:* Communico's RSS feed includes no age data. The AGE GROUP block is only on individual event detail pages. Scraped 40 events to catalogue the complete Communico audience taxonomy — exactly 8 values, all observed, none ambiguous. Adults and Older Adults filtered out at API layer. Families (All Ages) mapped to appear under all age filter selections.

**Key learning:** Validate that third-party API filters are actually doing what you think they are. A quick empirical check — fetch with the filter, fetch without, compare results — would have caught the BiblioCommons audience_id issue in 2 minutes. Assumption validated after the fact rather than before cost one engineering learning.

---

### Supervision Policy: Spreadsheet Framework → Live Database Table

**What the PRD planned:** Supervision policy confidence tiers managed via companion spreadsheet, surfaced as UI text.

**What was built:** A dedicated `supervision_policies` Supabase table, pre-seeded with verified data. The supervision badge on event detail views is derived dynamically from stored `age_min`/`age_max` values against the venue's verified policy threshold — not a static label. Frisco Library: Tier 2, threshold age 10 (sourced from Service Policy §8.5, 2026). Plano Libraries: Tier 2, parent's discretion (confirmed via phone). Play Frisco: Tier 3, always shows "Check with venue."

---

### Planned: Play Frisco LLM Age Inference

*Not yet implemented — documenting the design for the next build phase.*

Play Frisco (CivicPlus) has no structured age data. Planned approach: Claude API call at ingest time, passing each event's title and description. Response classifies as toddler/kids/teen/family/adult with a confidence level. High/medium confidence inferences surface in the UI with a `~` prefix and "estimated from event description" disclosure. Low confidence inferences show nothing. Events classified as adult (kid_relevant: false) are excluded from all views. Family events appear under all age filter selections. This is the first point in the app where data transitions from deterministic to inferred — the disclosure treatment reflects that distinction explicitly.

---
Next phase: see feature-spec-city-nav-age-inference.md for city-first navigation, age indicators, and Play Frisco LLM inference.

*Original PRD authored: June 2026. Implementation Notes added: July 2026.*

