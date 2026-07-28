# SEO Scoping — Open Eventz

*Status: **BUILT (2026-07-23)** — this scoping doc is superseded by the as-built design in `06-app/SEO-DESIGN.md` (and the BUILD-LOG "SEO Foundation" entry). Two changes vs. the scoping below: (1) **all six workstreams shipped in one pass** — per-event pages, Event JSON-LD, metadata, sitemap/robots, **city landing pages**, plus a **consent banner** (open decisions #2 and #4 both resolved to "yes, now"); (2) the **structured-data price policy changed** — see the correction inline in the "structured-data honesty" note below.*

## Why SEO matters for this product

Open Eventz is a local, time-sensitive events aggregator for parents in Frisco/Plano. That is almost a best-case SEO profile:

- **High-intent local queries** — "kids events frisco this weekend", "plano library storytime", "toddler activities near me".
- **Google Event rich results** — Google has a dedicated Events experience in Search. Well-marked events can surface directly with date/location, *above* the organic results. For an events app this is the single highest-leverage channel.
- **Fresh, structured content** — a constantly-updated catalogue of dated events is exactly what search engines reward for local/temporal queries.

SEO is the **acquisition** engine: it feeds the top of the funnel the analytics work measures. The two are complementary, not the same task.

## The one architectural decision that gates everything

**Per-event indexable URLs.** Today the app is a single page with a detail *panel* — there is no URL per event. Search engines rank *pages*, and Event rich results need a canonical page per event. So the foundational decision is:

> Introduce real routes like `/event/[id]` (server-rendered, one indexable page per event), while keeping the current list/panel UX for in-app browsing.

This decision also touches analytics (a per-event page view is a cleaner `detail_view` signal) and should be made deliberately before the rest of the SEO work — most items below depend on it.

## Workstreams (prioritized)

| # | Workstream | What it is | Depends on |
|---|---|---|---|
| 1 | **Per-event pages** | `/event/[id]` server-rendered routes, indexable, canonical | — (the gating decision) |
| 2 | **Event structured data (JSON-LD)** | `Event` schema.org markup per event (name, startDate, location, offers/price, isAccessibleForFree) → Google Event rich results. **Highest leverage.** | #1 |
| 3 | **Metadata** | Per-page `<title>`, meta description, OpenGraph/Twitter cards (for shares) | #1 |
| 4 | **Sitemap + robots** | Dynamic `sitemap.xml` generated from the live event catalogue; `robots.txt` | #1 |
| 5 | **City landing pages** | Indexable `/frisco`, `/plano` (and possibly category) pages targeting local queries | #1 |
| 6 | **Canonicalization** | Canonical URLs; sensible handling of recurring-event duplicates so they don't compete | #1, #2 |
| 7 | **Core Web Vitals** | Performance/LCP/CLS hygiene (Next.js already helps; verify) | — |

**Note on structured-data honesty:** ~~JSON-LD `offers`/`isAccessibleForFree` must reflect what we actually know... Only mark price in structured data when it is confirmed.~~ **SUPERSEDED (2026-07-23):** the PM decided to assert the free signal for **both confirmed AND inferred "Free ✦"** events (`isAccessibleForFree: true` + `$0` offer whenever `is_free === true`). This stays within Google's policy because the event page **visibly** renders the same "Free ✦" badge — so the markup matches the visible page. Paid ⇒ `isAccessibleForFree: false` (no offer, no numeric price stored); unknown ⇒ omit price. Reversible via one condition (`price_confidence === 'confirmed'`) with no re-ingest. See `06-app/SEO-DESIGN.md` §4.

## How SEO ties back to measurement (the "acquisition step")

"Acquisition" splits into two data sources — see the analytics spec's new *Acquisition & channel segmentation* note:

- **Channel mix of sessions** → GA4, automatic once analytics is wired. The funnel becomes segmentable by channel (organic / direct / referral / social). No new custom events needed.
- **Search funnel above the session** (impressions → clicks → sessions) → **Google Search Console**, a separate integration that only produces meaningful data *after* SEO ships.

Recommended: model acquisition as a **channel segmentation** of the existing funnel now; add a **GSC search-funnel panel** to the dashboard later, once there's search data to show.

## Non-goals / later

- Paid acquisition (SEM), backlink outreach, off-site SEO — out of scope for the portfolio stage.
- Multi-city expansion SEO — revisit if the catalogue grows beyond Frisco/Plano.

## Open decisions (need a call before build)

1. **Per-event URLs** (`/event/[id]`) — yes/no. Everything else depends on this.
2. **City landing pages** — include `/frisco` + `/plano` in v1, or defer?
3. **Structured-data price policy** — confirm: only emit price in JSON-LD when *confirmed* (never for inferred `Free ✦`).
4. **Consent/cookies** — GA4 + any SEO analytics set cookies; decide on a consent banner (portfolio stage may defer).

## Suggested sequence

1. Decide #1 (per-event URLs).
2. Ship pages (#1) → structured data (#2) → metadata + sitemap (#3, #4).
3. Verify indexing in Google Search Console; connect GSC.
4. Add the GSC acquisition panel to the analytics dashboard.
