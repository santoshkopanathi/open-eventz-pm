# Open Eventz — Product OS

*This is a living document, not a one-time deliverable. It gets updated and appended to as the project progresses rather than rewritten from scratch. Treat the Changelog at the bottom as the audit trail of how this doc — and the thinking behind it — evolved.*

---

## 0. Company Vision

**Vision:** To help people live fuller, more connected lives by making it effortless to discover, access, and create the opportunities that matter to them.

**Belief:** Most barriers between people and a better life aren't a lack of opportunity — they're a lack of visibility, access, and tools to make things happen. We exist to close that gap, one community at a time.

**What this means for how we build.** The vision is deliberately broad on two axes. First, "opportunities" is not defined as events, activities, or any specific category — it could mean community programs, resources, people, or things that don't exist yet and need to be brought into existence. Second, "discover, access, and create" explicitly holds space for three different business models: an aggregator of what already exists (Open Eventz today), a platform that enables communities to self-organize, and eventually a creator of programming and experiences that wouldn't have happened without us. Open Eventz is the first proof point of this vision — the place where the gap between people and opportunity is most visible and most immediately solvable — not the ceiling of what this company becomes.

**How the vision connects to Open Eventz.** Open Eventz attacks the visibility and access part of the belief statement first, in one specific community (Plano-Frisco) with one specific audience (parents with kids). That specificity is intentional at the pilot stage — it's not a constraint on the vision, it's a way of proving the model before expanding it.

---

## 1. Product Charter

**Problem statement.** Parents trying to fill summer days with free or low-cost activities for kids have to manually check a dozen-plus disconnected sources — city parks departments, library calendars, school district pages, museum sites, Facebook groups — with no single place that filters specifically for *free*, *kid-appropriate*, and *near me*.

**Target user.** Parents/caregivers of school-age kids (roughly 3–14) during the summer break, in suburban/metro areas with a fragmented but real supply of free programming (libraries, parks & rec, museums, nonprofits).

**Why now / insight.** The supply of free activities is larger than it looks — it's just badly distributed across city, county, school district, and nonprofit websites that don't talk to each other. The opportunity isn't "find more events," it's "stitch together sources that already exist but nobody has bothered to unify around the *free* filter specifically."

**Success definition (early hypothesis, not yet metrics).** A parent in a pilot metro can find a real, currently-accurate, genuinely free activity for this week without visiting more than one site. Revisit this once we have a working MVP — this is a placeholder, not a committed KPI.

---

## 2. User Personas

**Persona 1 — Dave.** Working dad, 40, two boys (8 and 6), career-focused and time-constrained. Wants to know about nearby activities he can take the boys to on Friday evenings and weekends. His core problem: he often only learns about free kid-friendly events after they've happened, and feels bad about missing them. He wants advance notification, not just a browsable list — and since his sons have different preferences, he wants to introduce them to a range of things rather than assuming one activity fits both. *Product signal: proactive notifications/reminders, not just search.*

**Persona 2 — Selina.** Stay-at-home mom, 38, a 4-year-old daughter and a 14-year-old son. Values giving her kids diverse, socially-enriching experiences. For her younger daughter, she wants a predictable, recurring weekly schedule of activities — explicitly *not* random, and explicitly not something requiring an early-morning routine, since she's not trying to recreate a school commitment during break. Attending also serves her own need to get out and meet other parents. For her older son, the need is structurally different: he doesn't want to be brought along anymore, but won't seek out free events on his own either — what he actually needs is a notification he can act on independently, not a parent-chaperone flow. *Product signal: recurring/predictable-schedule filtering for younger kids; a self-directed, notification-only mode for older kids/teens who don't need a chaperone.*

**Persona 3 — Roger & Lindsay.** Gig-worker parents, 38, two daughters (10 and 2), budget-conscious. Want occasional special free events and are flexible on timing to make them work. Their real operational challenge is supervision mismatch within one outing: their 10-year-old can largely manage herself at an event, while the 2-year-old needs constant hands-on attention — so they're looking for venues/events that work for both attention levels simultaneously, not just events that are nearby and free. *Note: an earlier draft of this persona centered on needing free, daily, school-like childcare. That need was deliberately removed — it describes a licensed-childcare problem, not an events-discovery problem, and isn't something this product can responsibly serve. The persona now reflects only the in-scope need.* *Product signal: surfacing age-appropriateness/supervision-level per event so mixed-age households can evaluate one listing for both kids at once.*

**Cross-persona theme:** none of the three personas are simple "find an event near me" cases. Each implies a different filter — proactive notification (Dave), recurring-schedule visibility plus a self-directed-teen mode (Selina), and multi-age supervision clarity (Roger & Lindsay) — which is why "supervision policy" and "recurring vs. one-time" became first-class schema fields rather than afterthoughts (see Sections 6–7).

---

## 3. Market Research — Plano-Frisco Pilot

*Methodology note: figures for Plano and Frisco themselves come from Census Bureau / American Community Survey data via city and demographic-data sources. Behavioral and trend findings below are harder to source hyperlocally — there's no published "Plano-Frisco parent behavior survey" — so they lean on national surveys (Pew Research, NCES, family-travel and parenting-industry research) applied to a market that's notably higher-income and more family-dense than the US average. Treat the national figures as informed context, not literal local measurements, until validated against this metro directly.*

### 3.1 Total Addressable Market

| | Frisco | Plano | Combined |
|---|---|---|---|
| 2026 population | ~245,000 | ~295,000 | ~540,000 |
| Total households | ~87,650 | ~112,400 | ~200,000 |
| School-age population (5-17) | ~53,700 (21.9%) | ~49,800 (under-15 share, 16.9%) | ~100,000-110,000 |
| Households with children under 18 | ~41,800 (47.7% of households, 2020 ACS) | Comparable share, ~47% | order of magnitude ~90,000-95,000 |
| Median household income | ~$150,000 | ~$112,000 | — |

The core addressable population — households with at least one school-age child in Frisco and Plano — is on the order of **90,000-95,000 households**, roughly **100,000-110,000 children**. That's the TAM in the literal sense. SAM (realistic adoption) and SOM (pilot-phase capture) can't be responsibly sized from public data alone — both depend on adoption-rate assumptions that don't exist yet. Rather than invent a conversion percentage, this is logged as an open item to close with lightweight primary research once there's a real pilot user base (see Open Questions).

### 3.2 General Trends — Evenings & Weekends

A Pew Research survey of U.S. working parents (March 2026) found 75% have at least some flexibility to attend their kids' activities during work hours, but only 46% have a meaningful amount of it — and that flexibility is sharply income-stratified: 54% of upper-income parents have a lot of flexibility versus 40% of lower-income parents. Downstream, 55% of lower-income parents say they've missed their kids' activities at least sometimes in the past year because of work, versus 40% of upper-income parents. In practice, evenings and weekends aren't just a preference for a large share of this market — for lower- and middle-income households especially, they're the only realistic window.

On what families actually do in that window, NCES parent-reported data on elementary-age kids shows: 53% attended a community/religious/ethnic-group event in the past month, 43% visited a library, 33% went to a play/concert/show, 28% visited an art gallery or museum, and 27% visited a zoo or aquarium. These line up closely with the categories already in the Plano-Frisco source inventory (Section 6) — this is mainstream behavior, not a niche.

### 3.3 Where They Gather Information

Pew's November 2025 study on parenting communities found 34% of parents visit online parenting spaces at least monthly (42% of mothers, 45% of parents age 18-29, 47% of parents with a child under 5). Tellingly, while 63% of those who use these spaces feel more informed, 38% feel more overwhelmed by everything they're trying to track — independent evidence that the "too many scattered sources" problem is a real, felt experience, not just our own framing of it. Beyond online communities, the channels already cataloged in Section 4 (city/library sites, Macaroni Kid-style newsletters, word of mouth, physical flyers) all apply. A smaller, dated UK survey (Hoop, 2018) found word-of-mouth from other parents was the single biggest influence on activity choice, and that complicated booking or hard-to-find, outdated information actively prevented follow-through — directionally consistent with everything else here, even if the sample itself is old and non-local.

### 3.4 Willingness to Pay vs. Free, and How They Compare

73% of families cite affordability as their top concern heading into 2026 (Family Travel Association survey), and "researching free attractions to balance against paid experiences" is now an explicit, named planning strategy — affordability as active behavior, not just background anxiety. At the same time, real money still flows to paid structured activities (a 2023 LendingTree survey found parents spend an average of $731/child/year on extracurriculars), so paid isn't being abandoned. Qualitative parent feedback surfaces a structural reason free events have an edge independent of affordability: deposit/cancellation friction. Parents are visibly wary of pre-paid spots they could lose to a sick kid or a schedule change, and in a short-commute suburban area like this one, last-minute decision-making is the norm — which favors anything with zero financial commitment attached. There's no published data directly comparing fill rates between free and paid family events, so that exact comparison isn't claimed here with false precision — but directionally, free events likely face a follow-through problem (no-shows are costless) where paid events face a sign-up problem (deposit anxiety suppresses registration in the first place). That points at reminder/follow-through features, not payment-friction removal, as the more relevant lever for this product.

### 3.5 Economic Situation / Distribution

Both cities sit well above the national median household income (~$80,000 nationally), but aren't identical. Frisco's median is around $150,000, poverty rate ~3.5%, with 51% of households earning over $150,000. Plano's median is lower, around $112,000, poverty rate 6.8%, with a wider spread: 37% earn over $150,000 but 10% earn under $25,000 (vs. 6% in Frisco). Most of the target population has real disposable income and isn't financially forced toward free options — but a genuine minority, larger in Plano, is meaningfully budget-constrained (this is Roger & Lindsay's segment). And the affordability-conscious behavior in Section 3.4 isn't confined to that minority — it shows up broadly across income levels as a 2026 cultural pattern, not purely a necessity.

### 3.6 Event Type Preferences

Enrichment categories (library, museum, community/cultural events) stay consistently high-participation across income and demographic groups — and more so among educated, affluent populations like Plano and Frisco's. Separately, outdoor/unstructured play is having a real resurgence: 2026 trend reporting describes rising demand for simple, non-electronic, low-setup outdoor play, alongside a smaller but growing countertrend toward calmer, more contemplative outdoor activities (nature exploration, quiet sensory play) as an alternative to loud, competitive group play. Preferences are bifurcating, not converging — a useful app should filter for mood, not just location and price.

### 3.7 Entertainment Opportunities Sought

Three overlapping appetites: enrichment-and-learning (library/museum/cultural, prioritized by these education-focused populations), social-and-community connection (for kids *and* for parents — Selina's persona names both explicitly), and occasional premium/immersive experiences reserved for special occasions like birthdays (family entertainment centers). Open Eventz's lane is clearly the first two; it isn't competing with the paid-celebration category at all.

### 3.8 Latest Entertainment Trends

The most relevant 2026 trend is the active cultural pushback against screen time and over-scheduling — a "no phone summer" / digital-detox movement, with one parenting-trend report naming "raising screen-smart kids who seek real-world adventure" as the year's defining theme. It's framed as something parents are choosing and enjoying, not enduring. That's a favorable timing signal: Open Eventz's whole premise (free, real-world, community-rooted activities) rides the same cultural current rather than working against parents' default instincts.

### 3.9 Seasonality

Summer is the headline season but shorter than the calendar suggests: Frisco ISD's 2025-2026 school year runs August 13, 2025 to May 22, 2026, with the next year starting August 12, 2026 — roughly a 12-week summer window, not three full calendar months. Within it, demand is front-loaded: paid camp spots reportedly fill fast once spring registration opens, while free/drop-in events stay genuinely discoverable throughout — a real positioning angle ("the paid stuff got locked in months ago; here's what's still free and open this week"). The underlying fragmentation problem also isn't summer-exclusive: the same school calendar creates a one-week spring break, a ~2-week winter break, and a Thanksgiving week, each recreating the same "what's free this week" problem at smaller scale. The data architecture doesn't need to change for this — worth keeping on the roadmap as a non-summer expansion option.

---

## 4. Market & Competitive Landscape

The real competition for Open Eventz isn't just other apps — it's every existing way a parent currently finds this information, including channels that aren't apps at all.

| Channel | Good at | Not good / helpful at | Structural challenge |
|---|---|---|---|
| Macaroni Kid | Personal, locally-written voice; free; consistent weekly cadence | No discoverable native app right now; weekly, not real-time; not really searchable or filterable | Quality depends entirely on how engaged the local franchise owner is — wildly inconsistent metro to metro |
| ParentPass | Real native app on iOS and Android; push notifications; built-in parent community/chat | Coverage visibly thins the further you get from its home base | Built outward from one community (Tarrant County) — depth doesn't travel evenly across a multi-county metro |
| City websites (parks & rec, city calendars) | Authoritative and official; accurate when current | Each city is its own separate domain and platform; seasonal program catalogs are often flipbook-style PDFs; no "free" filter; no cross-city search | Every city government runs its own site and process independently — nothing ties neighboring cities together |
| Library websites | Generally accurate; programs are usually genuinely free; decent internal event search | No shared platform across library systems even in the same metro; a library's own policy documents can disagree with each other | Each library system independently picked its own software and writes its own policy docs — no cross-library standard |
| Bulletin boards / flyers at venues | Often the freshest, most hyperlocal source; surfaces things with zero online footprint at all | Not searchable, not remote, easy to miss, impossible to plan around in advance, no reminder mechanism | The information only exists in the physical world — this is the literal "last mile" of event discovery |

**Macaroni Kid.** Reads like a real local parent wrote it, because one did — each metro's edition runs through an independent local franchisee, free to read, weekly cadence; the Frisco-area edition is actively publishing 2026 content. But there's no evidence of a currently maintained, store-listed mobile app — the product is a website plus email newsletter, meaning no push notifications and no real free/age/distance filtering. Quality and update frequency vary franchise to franchise with no standard.

**ParentPass.** A real, live app on both the App Store and Google Play, with push notifications and a community/chat feature. But it markets itself as built for North Texas broadly while its own FAQ states explicitly that Tarrant County (Fort Worth) users are its top priority, and the overwhelming majority of its public reviews name Fort Worth specifically. Tarrant County and Collin County (Plano/Frisco) are ~30-40 miles apart with entirely separate city governments, library systems, and school districts — coverage strength radiates outward from wherever a community-built app actually started, and thins the further out you go. **This is logged as a validated competitive wedge for the pilot metro specifically (see Decision Log).**

**Individual city websites.** Authoritative but fragmented even between two adjacent cities: Frisco splits across two domains (friscotexas.gov and townoffrisco.com), Plano's seasonal catalog is a flipbook-style PDF, and neither has a "free" filter or links to the other. No shared standard exists across municipal governments.

**Library websites.** Library events are close to "verified free" by default, but Frisco's library runs on BiblioCommons while Plano's runs on Communico — two unrelated platforms in adjacent cities. It's not just a platform problem either: Frisco's own city code and its own Summer FAQ disagree on the unattended-minor age threshold (9 vs. 10) — even a single library's own documentation isn't always internally consistent.

**Bulletin boards and flyers.** Often the single freshest, most hyperlocal source, sometimes surfacing things with zero online footprint at all — but unsearchable, unplannable, and easy to miss simply by not walking past the right wall that week. This is the most extreme version of the fragmentation problem every other channel has, and exactly the category competitors can't touch with a scraper.

**What this adds up to:** every channel is good at exactly one thing and structurally incapable of the rest — voice without a real product (Macaroni Kid), a real app with a geographic blind spot (ParentPass), authority without cross-source consistency (city/library sites), freshness with zero reach (flyers). None of them stitch together, none agree on what "free" means or who needs to be there to supervise, and none are accountable to anyone for getting better at this over time.

---

## 5. Data Sourcing Strategy

- **There is no nationwide "free kids events" API.** This is the single most important constraint shaping the whole approach.
- **Eventbrite deprecated its public event-search API in February 2020** specifically to prevent third parties from building competing aggregators. What remains is event-by-ID, venue-scoped, and organization-scoped access — useful once you know *who* to watch, useless for open-ended discovery.
- **PredictHQ** is a legitimate, broad event-aggregation API (hundreds of sources, global coverage), but it's an enterprise B2B data product, priced via sales conversation, and not curated for "free" as a category — we'd still be filtering noise out of it.
- **Conclusion:** the reliable path is direct, source-by-source integration with the actual publishers of free programming, not reliance on any single aggregator. The data problem is fundamentally *per-metro*, not *one big API integration* — the system should be designed so adding a new metro is a repeatable, cheap process (a checklist + schema), not a bespoke engineering project each time.

---

## 6. Pilot Metro: Plano–Frisco, TX — Source Inventory

| Source | Type | Platform / Format | Free signal | Likely integration method | Notes |
|---|---|---|---|---|---|
| Frisco Public Library | Library programs & events | BiblioCommons (friscolibrary.bibliocommons.com) | Library programs are generally free to attend | Check for iCal/RSS export; fall back to structured scrape | Running a "Mayor's Summer Reading Challenge" all summer with prizes |
| Plano Public Library | Library programs & events | Communico (plano.libnet.info) | Generally free | Check for feed export; fall back to scrape | 5 branches; library card is free for Plano, Frisco, Allen, McKinney, Garland, Richardson, The Colony, Wylie residents — covers a wider radius than just Plano |
| Play Frisco (City of Frisco Parks & Rec) | Camps, classes, community events | friscotexas.gov + townoffrisco.com event calendar | Mixed — most camps paid, need-based "Play Fund" scholarship exists, some community events free | Scrape city calendar page | Camps fill up fast post-registration-open; surface the Play Fund explicitly |
| Plano Parks & Recreation | Camps, classes, community events | plano.gov (CivicPlus) + seasonal flipbook-style PDF catalog | Mixed — camps mostly paid, some community events free | Scrape city calendar; PDF catalog parsing lower priority | Flipbook PDF catalog is a real integration headache |
| Frisco ISD | School district summer programs | friscoisd.org (not yet directly verified) | Believed free (district-run summer school) | Manual check each season; no feed expected | Currently sourced secondhand — needs direct verification |
| Frisco Discovery Center museums | Museum free days | Individual museum sites | Occasional free-admission days | Manual tracking | Each museum sets its own policy |
| Collin County — SummitCentral.org | County-wide childcare/camp directory | summitcentral.org | Mixed | Manual review | Surfaced via Town of Frisco's own site as their own fallback resource |
| YMCA of Metropolitan Dallas (Frisco locations) | Day camps | ymcadallas.org | Camps paid, but free summer meals at qualifying at-risk sites | Manual | The "free" hook is the meal program, not the camp |
| PlanoMoms.com | Hyperlocal weekly roundup (blog) | planomoms.com | Mixed, manually curated, tags free events | **Not a scrape/republish target** — their curated editorial work | Competitive reference and possible partnership contact |
| Eventbrite (Plano/Frisco "free kids" pages) | Organizer-submitted events | eventbrite.com/d/tx--plano/free-kids/ etc. | Self-tagged free | No public search API — would require tracking specific organizer/venue IDs | Manual discovery channel for finding new organizers, not an automatable feed at MVP stage |

---

## 6.1 Technical Integration Details — Challenges & Approach by Source

*This section records why each source is integrated the way it is, so future contributors understand the reasoning rather than just the implementation.*

| Source | Final Integration Method | Why This Method |
|---|---|---|
| **Frisco Library** | HTML scrape — BiblioCommons paginated audience feeds + per-event page fetch | No iCal/RSS that includes audience data. Three separate audience feeds (0–5, 6–12, Teens) are scraped page-by-page (up to 14 pages × 20 events). Each event page is additionally fetched to get description and the "Suitable for:" field for authoritative age data. |
| **Plano Libraries** | RSS feed (`plano.libnet.info/rss/events`) | Communico platform provides a working RSS feed. No age data is available in any format — the API's `ages` filter returns an empty feed when any value is passed. All Plano events are stored with null age range as a known, accepted limitation. |
| **Play Frisco** | HTML scrape — city calendar pages fetched month-by-month + per-event detail page fetch | Started with iCal (`catID=81`) and RSS. Both failed: the iCal only contained events starting January 2027+, and the RSS was capped at 8 items. The city's own calendar page (`calendar.aspx?CID=85,81`) is server-rendered and paginated by month, so fetching the current and next two months yields the full near-term event set. EIDs are extracted from each page and detail pages are fetched per event for title, date, location, and description. |

### Frisco Library — Challenges in Detail

| Challenge | What We Tried | What We Landed On |
|---|---|---|
| **Audience "bleed"** — the same event appears in multiple audience feeds (e.g. story time in both 0–5 and 6–12 feeds) | Keep youngest audience, keep oldest audience, merge ranges | Page-scrape "Suitable for:" on each event page — authoritative per-event audience data overrides feed-based assignment |
| **Age label accuracy** — feed-based audience was wrong for multi-audience events | Apply feed label directly | Compute age_min/age_max from all audiences detected on the page; only show a label when a single clear audience is present |
| **Adults+Teens events showing under Kids filter** | N/A — discovered after initial scrape | Distinguish "Adults + young children" (→ 0–17) from "Adults + Teens only" (→ 13–17). Adults-only events are marked age_min=18 and excluded globally at the API layer. |
| **Events only showing through first week** — BiblioCommons returns 20 events/page | Initially fetching only page 1 | Added pagination loop: fetch pages until no next-page link is found or all events on a page exceed the 2-month window |

### Plano Libraries — Challenges in Detail

| Challenge | What We Tried | What We Landed On |
|---|---|---|
| **No age data** | Tried `ages` filter in RSS URL (`ages=children`, `ages=teen`) | Returns empty feed — API doesn't support audience filtering via RSS. Accepted limitation: all Plano events pass all age filters. Compensated with branch sub-filter as the contextual refinement tool when Plano is selected. |
| **Adult events showing in age-filtered views** | Could not filter without age data | Global adult exclusion (`age_min < 18`) only catches events explicitly tagged during ingest. Plano events with null age_min/max always pass through — this is a stated, documented limitation. |

### Play Frisco — Challenges in Detail

| Challenge | What We Tried | What We Landed On |
|---|---|---|
| **iCal catID was wrong** | Initial integration used `catID=14` (Main Calendar = government meetings) | Corrected to `catID=81` (Parks & Recreation) |
| **iCal only covered events from January 2027 onward** | Used RSS as fallback | RSS capped at 8 items through mid-July. Replaced both with direct month-by-month scrape of city calendar page — same source the public sees, full coverage |
| **Board meetings, advisory sessions appearing in events** | None initially | Added `PARKS_REC_EXCLUDE_KEYWORDS` list (board, council, commission, meeting, advisory, etc.) applied at ingest time to title |
| **All-day multi-day events (e.g. Soccer Celebration) showing wrong date** | Parsed iCal all-day dates as midnight UTC → showed one day early in CDT | Fixed: parse all-day dates as `T12:00:00Z` (noon UTC) to stay in the correct calendar day regardless of timezone |
| **Multi-day events not appearing when they started before today** | Primary query only fetched `start_datetime >= today` | Added supplemental query: fetch events where `start_datetime < today AND end_datetime >= today`, merge with primary results |

---

## 7. Supervision Policy — Sourcing Framework

One of the sharpest, most defensible pain points identified: nobody currently tells parents whether they need to stay with their kid or can drop them off, and it genuinely varies by venue and even by specific program within a venue. Frisco Public Library's own city code says one age threshold (9) while its own Summer FAQ says another (10) — a real, found example of how messy even "official" sourcing is here.

**Confidence tiers, highest to lowest:**
1. **Tier 1 — Event-specific statement.** The registration page/flyer for *this* event explicitly states drop-off or parent-must-stay. Highest confidence.
2. **Tier 2 — Venue general policy.** No event-specific statement exists; using the venue's own published policy as default. Re-check periodically, and pick one designated source per venue when documents conflict (prefer the more current/specific one).
3. **Tier 3 — Unverified.** No reliable source found. Leave the threshold blank, show "check with venue" in-app. **Never replace this with a guessed number** — the stakes (a parent leaving a kid unsupervised based on wrong info) are too high for false confidence.

A working intake template (separate file: `supervision-policy-intake-template.xlsx`) operationalizes this with columns for source, program type, age threshold, confidence tier, source document, last-verified date, and conflict flag — pre-filled with the Frisco library worked example and seeded with the remaining 7 sources from Section 6, currently Tier 3/unverified pending outreach.

---

## 8. Decision Log

*Format: date — decision — why. This is the part of the doc meant to show reasoning, not just outcomes.*

- **2026-06-16** — Chose Plano–Frisco, TX as the pilot metro. Rationale: home-turf knowledge advantage for validating data quality by hand before trusting it at scale.
- **2026-06-16** — Decided *against* building the data backbone on the Eventbrite API. Rationale: the public search endpoint has been gone since Feb 2020; remaining access is too narrow to serve as primary discovery.
- **2026-06-16** — Decided the geographic unit for *data sourcing* will be city/branch, while the geographic unit for *user-facing search* will be zip code, bridged via geocoding + distance filtering — two different problems, not to be conflated in the data model.
- **2026-06-16** — Ruled out PredictHQ for MVP. Rationale: strong breadth, but enterprise pricing and not curated for "free" — revisit only if a revenue case justifies the cost later.
- **2026-06-16** — Decided PlanoMoms.com and similar hyperlocal blogs are *not* scrape targets (their curated work is their IP), but are logged as potential partnership/distribution contacts.
- **2026-06-17** — Product named **Open Eventz**; adopted as the working name across all docs going forward.
- **2026-06-17** — Logged ParentPass's Tarrant-County anchor as a validated competitive wedge specifically for this pilot metro — not just "competitors are imperfect" but a real, evidenced geographic blind spot in an existing, well-built competitor.
- **2026-06-17** — Chose two distinct headline pain points for two different jobs: "not comprehensive for Frisco/Plano" as the lead *acquisition* hook (why someone switches), and "tells you whether you need to stay or can drop off" as the lead *retention* hook (why no one else can easily copy this) — treated as complementary, not competing for the same slot.
- **2026-06-17** — Adopted the three-tier confidence framework in Section 7 for supervision-policy data, with an explicit default to "unverified" rather than inferring a threshold — given the real legal/safety stakes of getting this field wrong.
- **2026-06-17** — Declined to fabricate a SAM/SOM figure for the TAM in Section 3.1. Rationale: would require adoption-rate assumptions with no current basis; logged as an open item for lightweight primary research once a pilot user base exists, rather than presenting an invented number with false precision.
- **2026-06-17** — Updated personas to add behavioral nuance for older children (Selina's 14-year-old, Roger & Lindsay's 10-year-old) rather than splitting out a separate "independent teen" persona for now — kept as an open option if that use case proves significant enough on its own.
- **2026-06-17** — Removed the daycare-need framing from Roger & Lindsay's persona. Rationale: full-day, school-like supervised childcare is a licensing/liability-heavy problem this product structurally can't serve; the persona now reflects only the in-scope need (occasional flexible free events).
- **2026-06-18** — Locked MVP scope to 3 sources, added live-feed sourcing for both libraries, and ruled VisitFrisco.com to backlog (unstable Next.js build-ID-dependent URLs, wrong content focus). Logged in full in the MVP PRD's Source Decision Log.
- **2026-06-18** — Added Add-to-Calendar (.ics export), Share (native share sheet, link-based), event images/thumbnails, price display, dynamic map view, Google Maps navigation deep-link, and an anonymous Like/Attending engagement feature to MVP scope. Logged Option B (direct SMS/email send) as a deferred future enhancement.
- **2026-06-18** — Explicitly decided to keep the project scoped as a portfolio/demo artifact for now, not a public or friends-and-family live launch. Rationale: confirmed earlier (initial elicitation) that the primary goal is showcasing PM thinking, not real user traction. Before this decision, walked through what would additionally need to change for a real public launch — rate-limiting and IP-based duplicate-detection for the like feature, optional account-light sync for cross-device like persistence, scrape monitoring/alerting, a user feedback/correction mechanism for wrong data, resolving the Play Frisco scraping ToS question definitively rather than leaving it open, and moving supervision-policy data from "Tier 3 backlog" to "verified before launch" given the real safety stakes involved. None of this is built or required now, but it's captured here as a ready-made answer to "how would you take this from demo to production" if that comes up in a portfolio conversation.
- **2026-06-23** — Revised launch intent: product is now targeting real public launch in addition to serving as a portfolio artifact. Build quality decisions should reflect production-readiness, not just demo-readiness. Pre-launch checklist (from 2026-06-18 entry above) is now active scope rather than a hypothetical exercise.
- **2026-06-23** — Frisco Library BiblioCommons RSS endpoint identified: `https://friscolibrary.bibliocommons.com/events/search.rss`. Confirmed the events page is live at `https://friscolibrary.bibliocommons.com/v2/events` with 280+ active events. Audience filter IDs confirmed: Children 0–5 = `5d93b3bfed969d4f000b6181`, Children 6–12 = `5d93b3c7bfbf9a4400c52504`, Teens = `5d8a3b0171f1994500ba504b`. RSS endpoint returns empty body via web fetch (likely valid XML the fetcher can't display); developer must verify via browser or curl at build time.
- **2026-06-23** — Play Frisco (friscotexas.gov) ToS check completed. No explicit prohibition on reading the calendar found anywhere on the site. robots.txt blocks `/RSS.aspx` and `/currenteventsview.aspx` specifically, but does NOT block `/calendar.aspx` pages or `/iCalendar.aspx`. A public iCalendar subscription feed exists at `https://www.friscotexas.gov/iCalendar.aspx` — this is a cleaner integration path than HTML scraping and is not robot-blocked. Calendar list URL confirmed: `https://www.friscotexas.gov/calendar.aspx?CID=81&view=list`. Decision: preferred integration method updated from HTML scrape to iCalendar feed; scraping `/calendar.aspx` remains available as fallback. Courtesy outreach to City of Frisco recommended before launch but no ToS barrier found.
- **2026-06-23** — Frisco Library supervision policy confirmed from official source. Frisco Public Library 2026 Service Policy (published May 2026, URL: `friscolibrary.com/wp-content/uploads/sites/78/2026/05/Frisco-Public-Library-Service-Policy.pdf`) states in Section 8.5: "Children aged nine (9) or younger must be accompanied by an adult and supervised at all times." Appendix A14.3 (Group Visits) repeats the same threshold. Conclusion: **drop-off age threshold is 10** — children 10 and older may attend without a parent/guardian. This resolves the documented conflict between city code (age 9) and Summer FAQ (age 10) — both are consistent with this policy once read correctly (9-and-under need supervision = 10-and-older do not). **Upgraded to Tier 2 confirmed** with authoritative 2026 document. Play Frisco events are variable community events with no standing supervision policy — Tier 3 / "check with event organizer" is appropriate and may remain so permanently.
- **2026-06-23** — Plano Public Library supervision policy confirmed via direct phone call (June 2026). Finding: **no specific age threshold exists**. The library's stated position is that it is the parent's discretion — the library expects children to behave responsibly but does not set a minimum age for attending without a parent or guardian. This is a meaningfully different policy from Frisco Library and is worth surfacing in the UI. Display approach: rather than showing "check with venue," show a distinct message like "No age requirement — parent's discretion" for Plano Library events. **Confidence tier: Tier 2** (venue general policy, confirmed verbally by staff). Source: direct call to Plano Public Library main branch, June 2026. Recommend re-verifying annually or if policy changes are reported.
- **2026-06-18** — Established company vision (Section 0). Key decision: kept "discover, access, and create" deliberately broad so the vision isn't constrained to aggregation or even to opportunities that already exist — leaving the door open for the company to eventually create programming and experiences, not just surface them. Explicitly chose not to define "opportunities" narrowly (e.g. as events or activities) so the vision can hold across future products well beyond Open Eventz. Open Eventz is framed as the first proof point of the vision, not the boundary of it.

### Build-phase decisions (v1.0 → public)

*PM-level decisions from the build. Engineering-level detail and the running build journal live in `06-app/BUILD-LOG.md` — that is the continuously-maintained living record; this Product OS is the strategy foundation.*

- **2026-07-16** — Shipped **v1.0 MVP** (live on Vercel + Supabase), then **v1.1**: city-first navigation (Frisco/Plano tabs with per-city filter state) and Play Frisco **LLM age inference**. Chose **Claude Sonnet over Haiku** — the cost delta is trivial at this volume, so accuracy on ambiguous confidence-tier calls was the deciding factor.
- **2026-07-16** — Adopted the **confirmed-vs-inferred trust model** as a product principle: inferred signals wear a `✦` marker plus a plain-language disclosure ("estimated from event description"), and low-confidence inferences show *nothing* rather than a guess — the same "never guess when you don't know" rule already used for supervision policy (Section 7). Simplified the card badge system: only signals a filter can't communicate appear on a card (structured age ranges moved to the detail view).
- **2026-07-18** — **v1.2 price inference — Definition A.** Since Play Frisco has no structured fee field, all Play Frisco price is an LLM read, so both free and paid carry the `✦` (libraries stay plain "Free"). Free-by-default with a six-layer risk model whose single failure mode to avoid is paid→free; genuinely torn ⇒ *unknown*. A structured CivicPlus `Cost:` field, when present, is authoritative and overrides the LLM (plain Free/Paid, no `✦`).
- **2026-07-18** — **Analytics.** GA4 instrumentation of the full funnel + a measurement framework, feeding two dashboards: **Functional** (GA4 → BigQuery — weekly active discoverers, funnel, referral, top events) and **Technical** (Supabase — pipeline health, inference visibility, LLM cost). Acquisition modeled as a GA4 channel segmentation; a Google Search Console panel deferred to the SEO phase.
- **2026-07-23** — **SEO foundation shipped:** per-event server-rendered indexable pages (`/events/[id]`), schema.org Event JSON-LD, city landing pages (`/frisco`, `/plano`), a dynamic sitemap/robots, and Google Search Console. Structured-data price policy: assert "free" for **confirmed *and* inferred-free** alike — safe because the event page visibly shows the same "Free ✦" badge, so the markup matches the page (reversible via one condition).
- **2026-07-23** — **Security hardening before going public.** Enabled Supabase Row Level Security on all PostgREST-exposed tables (events = read-only for the anon key; like_counts + supervision_policies locked; writes go through the service role). The anon key is public by design, so RLS — not repository privacy — is what protects the data.
- **2026-07-23** — **Process & quality.** Consolidated the functional test scenarios into one plan with a CI `doc-parity` check that fails the build if a scenario names a test that no longer exists; CI now captures coverage + report artifacts. Documentation is updated per feature.
- **2026-07-23** — **Publishing.** Split into two repos — the app (`Imbillionaire/open-eventz`, Vercel-deployed) and PM artifacts (`Imbillionaire/open-eventz-pm`, this repo) — both made public as a portfolio.

---

## 9. Open Questions

- Verify Frisco ISD's actual summer program page directly (friscoisd.org) rather than relying on a secondhand mention.
- Decide whether museum free-days tracking is manual-only for MVP, or worth a recurring check-in cadence.
- Play Frisco ToS check complete — no prohibition found; iCalendar feed preferred over scraping. Courtesy outreach to City of Frisco recommended before launch. Plano Public Library and other backlog sources still need ToS checks if added.
- Decide whether to reach out to PlanoMoms.com (or similar) as a partnership conversation rather than treating them purely as competitive reference.
- Decide whether Selina's 14-year-old eventually warrants a standalone "self-directed teen" persona rather than staying folded into a parent-centric persona.
- Size SAM/SOM via lightweight primary research (e.g., a short survey to the personas' real-world networks) once there's a pilot user base.
- ✅ Supervision policy confirmed for all three v1 sources: Frisco Library (Tier 2, age 10 threshold), Plano Library (Tier 2, no age threshold — parent's discretion), Play Frisco (Tier 3 indefinitely — event-specific, no standing policy).
- Validate the national-level findings in Market Research Sections 3.2–3.4 and 3.6–3.8 against Plano-Frisco specifically, rather than relying on national context indefinitely.
- ✅ **Launch intent resolved (2026-07-23):** shipped live and being made public as a portfolio (both repos). The pre-launch checklist from the 2026-06-18 decision is largely satisfied — RLS hardening done, supervision verified for the 3 v1 sources, scraping ToS clear.
- **Google Search Console (opened 2026-07-23):** monitor indexing and Event rich-result eligibility (search visibility ramps over weeks); add a GSC acquisition panel to the Functional dashboard once there is search data.
- **Production error tracking (open):** wire Sentry (`@sentry/nextjs`) for persistent exception tracking — pending a Sentry project DSN (stored as a Vercel secret).
- Frisco ISD summer-program page still unverified and museum free-days still manual — both remain backlog (only the 3 v1 sources are live).

## 10. Next Steps

*The kickoff next-steps below (draft the MVP PRD, choose the data-entry mechanism, define the activity schema, integrate the 3 sources) are all **done** — v1.0 → v1.2 shipped and live. Current next steps:*

- **Make both repos public** — after a final eyeball of `04-data/supervision-intake.xlsx` for any real staff name.
- **Google Search Console:** watch indexing + rich-result eligibility; add the GSC acquisition panel to the Functional dashboard once there is search data.
- **Sentry:** wire production error tracking once a DSN exists.
- **Supervision intake:** continue for backlog sources as the catalogue grows; re-verify the 3 confirmed sources annually.
- **Multi-metro expansion:** revisit the repeatable per-metro checklist (Section 5) if the pilot validates — the data architecture was built for it.

*(Original kickoff steps, now complete: draft the MVP PRD ✅; decide the data-entry mechanism → scraper-driven ✅; define the activity schema ✅; supervision-policy intake for the 3 v1 sources ✅.)*

---

## Changelog

- **2026-06-16** — Doc created. Seeded Product Charter, Market & Competitive Landscape, Data Sourcing Strategy, and the Plano–Frisco pilot Source Inventory based on initial research.
- **2026-06-30** — Added Section 6.1: Technical Integration Details. Documents the final integration method, challenges encountered, and reasoning for each of the three v1 sources (Frisco Library, Plano Libraries, Play Frisco). Written as institutional memory so future contributors understand the why, not just the what.
- **2026-06-17** — Major merge: added User Personas (Section 2), the full Market Research Report (Section 3 — TAM, demographics, behavioral trends, channels, willingness-to-pay, seasonality), expanded Market & Competitive Landscape into the full Channel Analysis (Section 4, replacing the original short stub), and added the Supervision Policy Sourcing Framework (Section 7). Renamed the product to Open Eventz throughout. Added six new Decision Log entries and four new Open Questions reflecting this round of work.
- **2026-07-16** — Build begins to ship: logged v1.0 MVP going live and v1.1 (city-first navigation, Play Frisco LLM age inference, the confirmed-vs-inferred trust model, badge simplification) in the Decision Log.
- **2026-07-18** — Logged v1.2 (price inference — Definition A + structured Cost-field override; GA4 analytics + measurement framework; the Functional and Technical dashboards).
- **2026-07-23** — Brought this Product OS current after a stretch where the running record lived in `06-app/BUILD-LOG.md`: added the **Build-phase decisions** block to the Decision Log (v1.0 → SEO → security → publishing), updated Open Questions and Next Steps to shipped state, and noted the split into two public repos. Named the BUILD-LOG explicitly as the continuously-maintained living record and this doc as the strategy foundation.
