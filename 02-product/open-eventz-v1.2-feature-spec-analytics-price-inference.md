# Feature Spec: Analytics Dashboard + Play Frisco Price Inference

## Context

Two independent features scoped for the next build phase. Neither requires changes to the core event list or detail view UI. Both strengthen the product's PM story — one makes success measurable, the other closes a data quality gap that affects user trust.

Read the full spec before writing any code.

---

## Part 1 — Analytics Dashboard

### Problem

Open Eventz has no instrumentation. There is no way to tell whether parents are finding events, committing to them, or coming back. Without measurement, product decisions are based on assumption rather than evidence.

### What to build

Two internal dashboard views — Functional and Technical — accessible at a protected internal route (e.g. `/dashboard`). No auth required for the portfolio stage. Sample data is not acceptable — dashboards must read from real Supabase and GA4 data.

---

### Functional Dashboard

#### North Star

**Weekly Active Discoverers (WAD)**
Unique visitors per week who completed at least one conversion action — Add to Calendar or Attending tap — counted once per visitor per week regardless of how many conversion actions they take.

Display: large number, week-over-week trend, target indicator.

---

#### Conversion Funnel

Four steps in sequence. Each step shows count and conversion rate from the prior step.

| Step | Definition | Event to track |
|---|---|---|
| Sessions | Any visit to the app | `session_start` (GA4 automatic) |
| Engaged | Clicked ≥1 event card OR applied ≥1 filter | `event_card_click`, `filter_applied` |
| Expressed Intent | Viewed event details OR tapped Get Directions | `detail_view`, `directions_tap` |
| Converted | Added to calendar OR tapped Attending (once per session) | `calendar_add`, `attending_tap` |

**Sub-metrics shown as breakdowns under each step — not as separate funnel steps:**

Under Expressed Intent:
- View event details — count + % of intent sessions
- Get directions — count + % of intent sessions

Under Converted:
- Add to calendar — count + % of converted sessions
- Attending tap — count + % of converted sessions

**Important:** A session that converts without passing through an explicit intent action (e.g. a parent adds to calendar directly from the list view) still counts as converted. The funnel measures the most advanced step reached, not a strict linear path.

---

#### Supporting KPIs

| KPI | Definition | Target |
|---|---|---|
| Engagement rate | % of sessions that reached Engaged | Establish baseline first 30 days |
| Intent rate | % of engaged sessions that reached Expressed Intent | Establish baseline first 30 days |
| Conversion rate | % of intent sessions that Converted | Establish baseline first 30 days |
| Return visit rate | % of week-1 converters who return in week 2 | 30%+ (GA4 proxy — cookie-based, documented limitation) |

Note: Supervision coverage moves to the Technical dashboard as an operational metric. It measures data completeness, not user behavior.

---

#### Acquisition & channel segmentation

Acquisition (how visitors arrive) is not a single pre-session funnel step — it comes from two sources:

- **Channel mix of sessions** — organic search / direct / referral / social. GA4 captures this **automatically** from `session_start` (source/medium); no custom events needed. The funnel above is therefore **segmentable by channel** ("of organic sessions, what % convert?"). The measurement functions and their fixtures carry a `channel` dimension from day one so this segmentation is native, not a retrofit.
- **Search funnel above the session** — impressions → clicks → sessions. This lives in **Google Search Console**, not GA4, and only produces meaningful data *after* SEO ships. It is added later as a separate **GSC acquisition panel** on the dashboard (see `open-eventz-seo-scoping.md`).

For now: build the funnel channel-segmentable (GA4); defer the GSC search-funnel panel to the SEO phase.

---

#### Referral

**Share taps** — tracked standalone outside the funnel. A parent can share at any point — before or after converting. Reported as total weekly count and as % of converted sessions that also shared.

---

#### Top Events Table

Columns: Event name, Source, Attending count, Calendar adds, Directions taps, Shares.
Sorted by attending count descending. Show top 10.

---

#### Additional breakdown tiles (dashboard)

Beyond the funnel, the Functional tab renders three breakdowns:
- **Engaged sub-metrics** — card-click vs filter-applied counts among engaged sessions.
- **Filter Usage** — how often each filter type (age / date / source / branch) and city is applied, from `filter_applied`'s `fields`/`city` params.
- **Conversion Actions** — Google Calendar / Apple Calendar (= ICS download) / Attending, from `calendar_add`'s `method` param. (Reconciled to the app's *two* real calendar buttons — Apple **is** the ICS download — not a 3-way split.)

---

### Technical Dashboard

#### Ingest Pipeline

- Last ingest timestamp
- Last ingest status (OK / WARN / ERR)
- Last ingest duration
- Total events in DB

**Status semantics** (derived at ingest time, stored on the `ingest_runs` row):
- **OK** — run completed, data written, zero errors.
- **WARN** — run completed and data **was** written (`total_upserted > 0`), but some non-fatal errors occurred (e.g. a few event detail pages returned HTTP errors, or one source partially failed). No operator action normally required; the specific errors are stored in `ingest_runs.errors` (jsonb).
- **ERR** — **nothing** was written (`total_upserted = 0`) — a genuine failure needing investigation.

#### Per-Source Event Counts

- Frisco Library — total events
- Plano Libraries — total events
- Play Frisco — total events
- Free events — total across all sources
- Paid events — total across all sources

#### Ingest History

14-day bar chart showing OK / WARN / ERR status per day. Color-coded: green = OK, amber = WARN, red = ERR.

#### Play Frisco — Inferred Age Visibility

Shows how much of the Play Frisco catalogue is actually reachable through the age filter. Four buckets:

| What shows in UI | Description |
|---|---|
| `~ Family ✦` shown | Inferred family — appears under all age filter selections |
| `✦` only shown | Inferred specific age — bare indicator on card, age in detail view |
| Nothing shown | Low confidence — event appears in default list but invisible under any age filter |
| Hidden entirely | Adult / not kid-relevant — excluded from all views |

Display: count + % of total Play Frisco events for each bucket. Total must sum to 100%.

#### Play Frisco — Inferred Free vs Paid

The price analogue of age visibility — splits the Play Frisco catalogue's price by how it was determined. Five buckets summing to 100%:

| Bucket | Meaning |
|---|---|
| `Free ✦` (inferred) | Free-by-default — no price stated on the source; the **"free by assumption" exposure** (the main risk surface) |
| `Free` (Cost field) | Source-confirmed free from the structured `Cost:` field — no `✦` |
| `Paid ✦` (inferred) | LLM inferred paid from the description (no Cost field) |
| `Paid` (Cost field) | Source-confirmed paid from the structured `Cost:` field — no `✦` |
| Unknown (no badge) | Ambiguous / Layer 2–3 suppressed — no price shown |

The **`Free ✦` (inferred)** count is the operational exposure metric: how many events we show as free purely on assumption. In practice most *paid* Play Frisco events carry a `Cost:` field, so `Paid ✦` tends to be near zero.

#### LLM Inference

- Total inference calls (lifetime)
- Cost last ingest run
- Cost cumulative
- Model in use + rationale: "claude-sonnet-4-6 chosen over Haiku — cost difference <$0.03 at current scale; accuracy the deciding factor"

**Cost is an estimate:** `cost = llm_calls × PER_INFERENCE_COST_USD` (currently `$0.006/call`, ~1.5k tokens/call). **Caching behavior to expect:** inference runs only for **new** Play Frisco events (those with `kid_relevant IS NULL`). A re-ingest where every event is already cached makes **0 LLM calls → $0.00** for that run — so "cost last run" can legitimately be $0.00 while "cumulative" is non-zero.

#### Ingest Log

Last 7 runs. Per row: timestamp, status, per-source events fetched (Frisco / Plano / Play Frisco), new events upserted, LLM calls made, cost, duration.

---

### Technical dashboard — implementation notes

- **Data source:** `/dashboard` is a server-rendered route reading **live Supabase** via the service-role client (server-side only; the table `ingest_runs` has RLS enabled so it's never exposed to the browser). Metrics are pure functions in `src/lib/technical-metrics.ts`.
- **Counts use exact `COUNT` queries, not `.select()`.** A plain PostgREST `select()` caps at **1000 rows**, which silently under-counts (and can drop a whole source). Totals and the free/paid/unknown split use `{ count: 'exact', head: true }` per predicate; only the small Play Frisco set is fetched in full for the visibility buckets.
- **"Event counts" (DB) vs "Ingest log" (per run) differ, by design.** The log shows events **scraped in that run** (pre-dedup); the counts show **total rows in the DB**. Library events are **not purged**, so they accumulate across runs (DB > fetched); Play Frisco **is** purged to the current batch each run, so its two numbers match. *(Follow-up candidate: purge stale/past library rows.)*
- **`ingest_runs` schema:** one row per run — `ran_at`, `duration_ms`, `status`, per-source `*_fetched`, `total_upserted`, `llm_calls`, `llm_cost_usd`, `errors` (jsonb). Written best-effort at the end of `POST /api/ingest`; a failure to log never fails the ingest.

### Instrumentation requirements

All functional metrics require custom GA4 events. The following must be fired at the correct moments in the app:

| Event name | Fired when |
|---|---|
| `filter_applied` | Any filter dropdown selection changes |
| `event_card_click` | Any event card tapped in list view |
| `detail_view` | Event detail panel opens |
| `directions_tap` | Get Directions tapped |
| `calendar_add` | Add to Google Calendar OR Add to Apple Calendar OR ICS download |
| `attending_tap` | Attending button tapped (toggle on only — not toggle off) |
| `share_tap` | Share button tapped |

Technical metrics (ingest status, LLM calls, cost, event counts) are read directly from Supabase — no GA4 events needed.

---

### Implementation order

1. Fire GA4 custom events in the app (required before any functional metric is meaningful)
2. Build Technical dashboard (reads from Supabase — no GA4 dependency)
3. Build Functional dashboard (reads from GA4 after events are firing)
4. Validate data for one week before presenting metrics publicly

### Implementation status (updated 2026-07-22 — SHIPPED & DEPLOYED)

- ✅ **Step 1 — GA4 instrumentation:** base tag via `next/script` (`NEXT_PUBLIC_GA_MEASUREMENT_ID`); `src/lib/analytics.ts` `trackEvent()`; all 7 events fired (each with `source` + `event_id`; `calendar_add` carries `method`; `filter_applied` carries `city`+`fields`; `attending_tap` toggle-on only; `detail_view` from the page's single instance). Verified firing into `dataLayer` in prod.
- ✅ **Measurement framework:** `src/lib/measurement.ts` (fixture-tested) — WAD, funnel (most-advanced-step, channel-segmentable, with **Engaged** sub-metrics), KPIs, return rate, referral, top events, plus `conversionActionBreakdown` (Google / Apple-ICS / Attending) and `filterUsage`.
- ✅ **Step 2 — Technical dashboard:** reads Supabase; `ingest_runs` table (migration `004`) + ingest instrumentation; `src/lib/technical-metrics.ts` (per-source + free/paid/unknown, inferred-age visibility, **inferred free vs paid**, ingest pipeline/history/log, LLM cost) + tests.
- ✅ **Step 3 — Functional dashboard:** BigQuery **Daily** export (`open-eventz.analytics_546304403`) read via `src/lib/bigquery.ts` (service-account key `GCP_SA_KEY_B64`); maps events → `AnalyticsRow[]` → the measurement framework. Renders WAD, funnel, KPIs, referral, top events. Deployed; shows real (currently sparse) data — populates as traffic + the daily export accrue.
- ✅ **Unified tabbed dashboard:** `/dashboard` = one page, **Functional | Technical** tabs (from the `open-eventz-dashboards_12.html` prototype's look, our metrics). Server component fetches Supabase + BigQuery; client `DashboardTabs` renders. `force-dynamic` → refreshes on each browser reload (Technical = last ingest; Functional = ~daily export). **No** North Star target, **no** per-source-status table / stale-removed count (deliberately dropped); **kept** our Definition A price model.
- ⏳ **Step 4 — validation:** ~1 week of real events (in progress post-deploy).

**Refresh cadence:** page re-queries live on every browser load (no auto-poll). Technical data updates when an ingest runs (currently manual — Vercel's function timeout precludes a cron for the ~5-min ingest). Functional data updates ~once/day via GA4's Daily BigQuery export (streaming would need billing).

---

## Part 2 — Play Frisco Price Inference

### Problem

The existing Play Frisco price detection uses a keyword parser (`parsePriceFromText`) that matches against a list of paid-event signals. This approach has a fundamental flaw: it matches substrings, not intent. The word "fee" matches inside "feeling" ("whether you're feeling competitive"). "Coffee" could match a cost-related pattern. "Feedback" contains "fee." Every keyword added to patch one false positive risks introducing another — it is a losing maintenance game.

This led to real mislabeling: Play Frisco events were being marked as paid based on incidental word matches in event descriptions that had nothing to do with price.

The solution follows naturally from the existing architecture: Open Eventz already makes a Claude API call for every Play Frisco event to infer age. Folding a free/paid/unknown price classification into that same call costs essentially nothing in additional API cost — and an LLM reads intent rather than substrings. It understands that "buy tickets" means paid, "$95/week" means paid, "no cost" means free, and "whether you're feeling competitive" means nothing about price at all.

**In one line: replace a fragile keyword heuristic that was mislabeling events with a far more robust LLM classification, at near-zero added cost because the inference call already exists.**

This decision is documented in the BUILD-LOG under v1.1 challenges: "substring keyword false-positives" and the lesson "LLM > keyword heuristics for extracting structured facts from messy free-text."

---

### What to build

Extend the existing Play Frisco LLM inference call to also classify price alongside age. No separate API call — price classification rides the same inference that already runs for age. **UI changes are required** (this supersedes an earlier "no UI changes" note): the price badge gains the `✦` estimate marker on Play Frisco (Layer 4 / Definition A) and the detail view shows one combined age+price disclosure line.

---

### Price model — `price_class` (raw) + derived display

The LLM's raw classification is stored as **`price_class`** (`free` / `paid` / `unknown`) plus **`price_confidence`** (`confirmed` / `inferred`); the display fields are derived from it.

| `price_class` | `is_free` (derived) | Play Frisco badge | Library badge |
|---|---|---|---|
| `free` | `true` | `Free ✦` (estimate) | `Free` (confirmed) |
| `paid` | `false` | `Paid ✦` (estimate) | `Paid` (confirmed) |
| `unknown` | `null` | no badge | — |

**Definition A:** every Play Frisco price is an LLM read of the description (no structured fee field exists), so both free and paid carry the `✦`. **Critical rule: `unknown` is never shown as `free`** — absence of a paid signal doesn't guarantee free; if price can't be determined, show nothing.

Library events (Frisco Library and Plano Libraries) remain hardcoded `is_free = true`, shown as a plain confirmed `Free`. This change only affects Play Frisco events.

---

### LLM classification rule

The price classification is added to the existing system prompt as a second task alongside age inference. The LLM must choose one of three values:

- **"free"** — no paid signal detected. This is the default for Play Frisco community events where no price is mentioned. City parks and recreation events are free by institutional nature in the overwhelming majority of cases — the absence of a paid signal is itself a meaningful signal.
- **"paid"** — explicit paid signal detected: a stated price, fee, "buy tickets", "register for $X", "tickets required", "purchased ticket", or any indication of a cost to attend.
- **"unknown"** — ambiguous signals that make free an unreliable assumption, even without an explicit price (e.g. "tickets at the gate", "members only", "reservation required", "donations welcome"), **OR** a genuinely-torn call where there are weak/conflicting hints and no clear read either way. Not clearly free, not clearly paid → don't guess.

Keep two cases distinct: **no price mention at all → "free"** (the community-event default); **weak/conflicting hints you can't resolve → "unknown"**.

**The LLM must not output a dollar amount — classification only.**

---

### Risk mitigation — six layers

Free-by-default shifts all risk into a single failure mode: a paid event slipping through as free. The six layers below are designed to minimize that risk systematically. Each layer is independent — any one of them catching an event is sufficient to pull it out of the free default.

---

**Layer 1 — When genuinely torn, return "unknown" (don't guess)**

The LLM will occasionally encounter a borderline description — a vague "tickets" reference, unclear whether required or just informational. Left to its own devices it might go either way.

One instruction is added to the system prompt: *"When genuinely torn between paid and free with no clear signal either way, return 'unknown' — do not guess."*

The reason is asymmetric risk, resolved honestly. A wrong-free means a parent shows up expecting a free experience and gets charged — a real trust failure. A wrong-paid deters a parent from an event that's actually free — the exact discovery failure this product exists to prevent. **"Unknown" (no badge) avoids *both*:** it never asserts free (so no unexpected charge) and never asserts paid (so no wrongly-deterred free event). The parent simply sees no price and checks the source.

A useful side effect: because torn calls resolve to unknown, **`Paid ✦` now appears only when the description actually indicates a cost** — never from a coin-flip. That makes the paid badge a trustworthy signal.

> **Revision note (Definition A):** an earlier version tie-broke torn cases to *paid* to guard against wrong-free. That still guarded wrong-free but introduced wrong-paid (deterring free events). Since "unknown" guards wrong-free *and* avoids wrong-paid, torn → unknown strictly dominates. On a free-dominated parks & rec calendar, this is the better trade.

**Cost: one prompt line. Risk removed: borderline cases never become a wrong Free *or* a wrong Paid.**

---

**Layer 2 — Never default-free for structurally-paid event types**

Not all Play Frisco events are equally likely to be free. Drop-in community events — storytimes, park activities, weekly family programs — are almost always free. But certain event types cluster heavily toward paid: camps, classes, clinics, workshops, lessons, leagues, academies, tournaments. These are structured, instructor-led programs that cost money to run and almost always cost money to attend.

The rule: if an event title or description contains a structurally-paid keyword **and** no explicit price signal is detected, resolve to "unknown" — not free.

Structurally-paid keywords: camp, class, clinic, workshop, lessons, league, academy, tournament, "per child", "per person", "materials fee", "deposit".

**Important nuance:** if the event contains these keywords but also has an explicit free statement ("free summer camp", "no cost to attend"), Layer 2 does not fire. An explicit free statement always overrides the keyword suspicion. Layer 2 only activates in the absence of any price signal.

The calibration data illustrates this perfectly: Events A and C (History of Play, Walnut Wednesdays) are drop-in community activities — free-by-default applies correctly. Event D (Wands, Wizards & Cookies) is a cookie decorating class — even without "purchased ticket" in the description, Layer 2 would have caught "class" in the title and resolved to unknown.

**Cost: keyword list in price.ts. Risk removed: the highest-risk event types never get the free default.**

---

**Layer 3 — Registration-required is a paid tell**

The pipeline already captures whether an event requires registration (`registration_required` field). That field exists to show the Reg. badge on cards — but it's also a useful price proxy.

Genuinely free drop-in events almost never require registration. You just show up. Events that require registration are almost always structured programs. And structured programs almost always cost money.

The rule: if there is no price signal in the description **and** `registration_required` is true, resolve to "unknown" — not free.

This works as a second safety net that catches events Layer 2 might miss. Layer 2 looks at event type keywords. But what if a paid event has a generic title with none of those keywords? If it requires registration, Layer 3 catches it anyway.

Event D from the calibration set has registration required — even without "purchased ticket" language, Layer 3 would have resolved it to unknown independently of Layer 2.

Layers 2 and 3 work together. Either one firing is sufficient to pull an event out of the free default. Both don't need to fire.

**Cost: one condition in the ingest mapping. Risk removed: paid events with generic titles that Layer 2 misses.**

---

**Layer 4 — Mark inferred price with the `✦` estimate marker (Definition A)**

Every Play Frisco price classification — free **and** paid — is an LLM read of the event description, because CivicPlus exposes no structured price field. There is no "confirmed" price on Play Frisco in any meaningful sense. So under **Definition A**, *all* Play Frisco price badges carry the `✦` estimate marker:

- Play Frisco free → **`Free ✦`**
- Play Frisco paid → **`Paid ✦`**
- Play Frisco unknown → **no badge**
- Library free → plain **`Free`** (institutionally confirmed, no `✦`)

The `✦` is the visual contract: this classification is an estimate, not a confirmed fact. This is the same trust principle used elsewhere in the product — supervision policy ("check with venue"), age inference (`~` + `✦`) — be honest about what you know vs. what you're inferring.

**Note on `price_confidence`:** the raw `confirmed` vs `inferred` distinction is still *stored* (Layer 5) — it powers the dashboard's "free by assumption" exposure metric — but under Definition A it no longer decides whether a `✦` shows. The **source** decides that: Play Frisco → estimate (`✦`), library → confirmed (no `✦`).

**The disclosure text — one combined line.** The full, scenario-specific disclosure appears **only in the detail view**, and if both age and price are inferred it is a **single combined line**, never two. On list cards the `✦` shows a single simplified hover tooltip instead:

| Context | What shows |
|---|---|
| List — desktop | `✦` on the badge; hover tooltip = **"Estimated from description"** (same text for every scenario) |
| List — mobile | `✦` only, no tooltip; tapping the card opens the detail view |
| Detail — desktop & mobile | The full scenario-specific string from the table below (age + price merged into one line) |

**The eight detail-view disclosure strings:**

| Scenario | Disclosure string |
|---|---|
| Age range inferred | `Age suitability estimated from event description` |
| Family inferred | `Family suitability estimated from event description` |
| Free inferred | `'Free' admission status estimated from event description` |
| Paid inferred | `'Paid' admission status estimated from event description` |
| Age + Free inferred | `Age suitability and 'Free' admission status estimated from event description` |
| Age + Paid inferred | `Age suitability and 'Paid' admission status estimated from event description` |
| Family + Free inferred | `Family suitability and 'Free' admission status estimated from event description` |
| Family + Paid inferred | `Family suitability and 'Paid' admission status estimated from event description` |

`'Free'` and `'Paid'` are single-quoted to signal they are classification labels, not descriptions. Implementation: `age_inferred` / `price_inferred` are **derived at render time** from the badge logic (no stored flags, no migration); the combined string lives in `src/lib/inference-disclosure.ts`.

**Cost: a small `getPriceBadge()` helper. Risk removed: wrong inferred-free is visually hedged, not silently asserted.**

---

**Layer 5 — Store the raw price classification separately from what's displayed**

The LLM's price output ("free", "paid", "unknown") must not be immediately collapsed into display fields (`is_free`, `price_text`) at ingest time. If that conversion happens and the original conclusion is discarded, three problems follow:

**Problem 1 — The policy becomes a one-way door.** If free-by-default turns out too risky and needs to be reversed, every LLM call must be re-run against the entire Play Frisco catalogue. That costs time, money, and introduces risk to a production database — purely because the original signal wasn't kept.

**Problem 2 — Exposure becomes invisible.** There is no way to answer "how many events are currently showing Free purely because of the default?" That number is the risk. Without the raw classification stored, the question is unanswerable.

**Problem 3 — Layer 4 becomes impossible.** To show `Free ✦` vs `Free`, the display layer needs to know which kind of free it's dealing with — confirmed or inferred. If the distinction was collapsed at ingest time, that information is gone and cannot be reconstructed.

The fix mirrors the age inference pattern already built:

- Age inference stores raw: `kid_relevant`, `age_buckets`, `age_confidence`, `age_reasoning`
- Age inference derives display: `getAgeBadge()` applies the policy at render time

Price should do the same:
- Store raw: `price_class` column ("free" / "paid" / "unknown") + `price_reasoning` field
- Derive display: `getPriceBadge()` applies the free-by-default policy at read time

Flipping the policy later is then a one-function change and a redeploy — no re-ingest, no LLM spend. The Technical dashboard can show "N events showing Free by assumption" — exposure visible and auditable at all times.

**Cost: one additional column + one pure function. Risk removed: policy is reversible; exposure is always visible.**

---

**Layer 6 — A growing calibration set that keeps paid-detection honest over time**

The entire free-by-default policy rests on one claim: the LLM reliably detects paid signals. Layer 6 is what actually measures that claim rather than just trusting it.

**Why this matters more under free-by-default.** Under unknown-by-default, a model failure means a free event shows no badge — annoying but recoverable. Under free-by-default, a model failure means a paid event shows "Free" — a trust failure that reaches a real parent. The stakes are higher, so the evidence that the model is working needs to be proportionally stronger.

**Two-tier testing approach:**

*Tier 1 — Parser tests (automatic, runs in CI on every commit, costs nothing)*
Covers Layers 1, 2, and 3 — all pure logic. Keyword matching, structurally-paid type detection, registration-required gating are all deterministic and testable for free. These run on every commit as standard Jest tests.

*Tier 2 — LLM calibration script (manual, run deliberately)*
A script (`npm run calibrate:price`) that runs the real model against the ground truth set and prints a pass/fail table. Run it when the system prompt changes, when models are swapped, or when drift is suspected. Not on every commit — that costs money — but as a deliberate checkpoint before shipping prompt changes.

**Ground truth calibration set — starting point (4 events, human-labeled):**

| Event | Description signal | Expected | Reasoning |
|---|---|---|---|
| History of Play 2026 | No price mention; activity description only; "feeling competitive" contains "fee" as substring but means nothing about price | free | Free-by-default; no paid signal; Layer 2/3 do not fire |
| Fun Float Night | "Event only attendees - $7 youth / $9 adult" under explicit EVENT COST header | paid | Explicit price stated; confirmed paid |
| Walnut Wednesdays | No price mention; drop-in weekly family program | free | Free-by-default; no paid signal; no structurally-paid keywords |
| Heritage How-To: Wands, Wizards & Cookies | "Each participant must have a purchased ticket. No refunds unless class is cancelled." | paid | Explicit paid signal ("purchased ticket"); also triggers Layer 2 ("class") and Layer 3 (registration required) |

**The "growing" part is what makes it durable.** Every time a wrong label is caught in production — especially a wrong free — that event's description and the correct answer are added to the fixture file. Over time the set encodes every real failure mode actually encountered. A later "harmless" prompt tweak that would have broken the "feeling competitive" case fails loudly on the calibration run instead of silently shipping.

Both tiers read from the same fixture file (`src/lib/__fixtures__/price-calibration.ts`) so ground truth lives in exactly one place.

**Cost: a fixture file + a calibration script. Risk removed: paid-detection recall is measured, not assumed.**

---

### Structured `Cost:` field — authoritative override

CivicPlus exposes a structured `Cost:` field on some Play Frisco events (~11% in a live sample), via `itemprop="price"`, with values like `Free`, `$35`, or `Paid`. When present it is **authoritative** — a source-confirmed price the LLM never sees — and it **overrides the entire description pipeline** below:

- `Cost:` contains "free" → **confirmed free**
- `Cost:` contains "$"/a number, or "paid"/"fee" → **confirmed paid**
- `Cost:` absent or unrecognized → fall through to the decision tree

Because it's *confirmed*, a Cost-field price renders as a **plain `Free`/`Paid` with NO `✦`** (**Option A** — the `✦` marks only LLM-inferred prices). This is what fixes events like "Learn to Fish" (Cost: Free) that the description pipeline would otherwise suppress to `unknown` via Layers 2/3. Paid-keyword note (**Option 2**): the fallback parser adds only unambiguous phrases (`purchase a ticket`, `tickets on sale`) — not the broader `ticket available`, which could false-positive on a free ticketed event (the LLM handles that phrasing).

### Decision tree — full logic (resolves `price_class`)

```
0. Structured Cost: field present?
   → YES: confirmed free/paid from its value (overrides everything below)

1. Explicit paid signal in description?
   → YES: paid

2. Explicit free statement ("free admission", "no cost", "free to attend")?
   → YES: free (confidence = confirmed)

3. Structurally-paid keyword in title/description (camp, class, clinic, workshop, …)?
   → YES + no price signal: unknown          (Layer 2)

4. registration_required = true?
   → YES + no price signal: unknown          (Layer 3)

5. Genuinely torn — weak/conflicting hints, no clear read?
   → unknown                                 (Layer 1: don't guess)

6. No price mention at all (and none of the above):
   → free (confidence = inferred)
```

Then the badge is decided by **source** (Definition A): Play Frisco `free`/`paid` → `Free ✦`/`Paid ✦`; Play Frisco `unknown` → no badge; library → plain `Free`.

---

### Price badge — decision table & rationale

The price badge has exactly four outcomes: **`Free ✦`**, **`Paid ✦`**, plain **`Free`** (library only), or **no badge**.

| # | Source | What the description contains | Resolved `price_class` | Badge | Why |
|---|---|---|---|---|---|
| 1 | Library | — (n/a) | — (`is_free=true`) | **`Free`** (no ✦) | Free by institutional default — a confirmed fact, not an LLM guess. No ✦, no disclosure. |
| 2 | Play Frisco | Explicit **paid** signal (price, fee, "buy tickets", "purchased ticket", "$7") | `paid` | **`Paid ✦`** | A cost is stated. Def A: all Play Frisco price is LLM-read from the description → ✦. |
| 3 | Play Frisco | Explicit **free** statement ("free admission", "no cost") | `free` (confirmed) | **`Free ✦`** | Text says free, but there's no structured price field — still an estimate read → ✦. |
| 4 | Play Frisco | **No price signal**, plain drop-in (no paid keyword, no registration) | `free` (inferred) | **`Free ✦`** | Free-by-default: parks & rec events are free unless stated; no risk signals. |
| 5 | Play Frisco | No signal **+** structurally-paid keyword (camp/class/clinic/workshop/…) | `unknown` (Layer 2) | **no badge** | Structured programs almost always cost money → refuse the free default; show nothing rather than a wrong "Free". |
| 6 | Play Frisco | No signal **+** `registration_required` | `unknown` (Layer 3) | **no badge** | Registration correlates with paid programs → same refusal. |
| 7 | Play Frisco | **Ambiguous** language ("members only", "reservation required", "at the gate", "donations welcome") | `unknown` | **no badge** | Not clearly free or paid → no guess in either direction. |
| 8 | Play Frisco | **Genuinely torn**, weak/conflicting hints, no clear read | `unknown` (Layer 1) | **no badge** | Don't guess. Showing nothing avoids both a wrong "Free" *and* a wrong "Paid". `Paid ✦` is reserved for an actual stated cost. |

**The logic / thought process behind it:**

- **Why free-by-default at all.** CivicPlus has no structured price field, so price can only come from the description text. On a city Parks & Recreation calendar the overwhelming majority of events are free, and paid ones almost always *say so* (they have to collect money). So "no paid signal" is itself a meaningful signal — defaulting silence to free is correct far more often than not.
- **The single failure mode.** Free-by-default concentrates all risk into one place: a paid event slipping through as **free** (a parent shows up and gets charged). Every layer exists to shrink that one risk.
- **Why "no badge" exists (rows 5–8).** When we can't be confident, we show **nothing** rather than guess — because a wrong "Free" is the worst outcome and a wrong "Paid" wrongly deters a free event (the exact discovery failure the product fights). "Unknown → no badge" is the honest answer that avoids *both*.
- **Why torn → unknown, not paid (row 8).** An earlier design tie-broke torn calls to *paid* to guard wrong-free. But "unknown" guards wrong-free *and* avoids wrong-paid, so it strictly dominates on a free-dominated calendar. Bonus: `Paid ✦` now means "the text actually indicated a cost," never a coin-flip.
- **Why the ✦ on everything Play Frisco (Definition A).** There's no confirmed price on Play Frisco — free *and* paid are both description reads — so both wear the ✦. The `confirmed`/`inferred` nuance still lives in the data (for the dashboard's "free by assumption" exposure metric) but no longer drives the badge; the **source** does.

---

### Mapping to stored fields

| Scenario | `price_class` | `price_confidence` | `is_free` (derived) | Badge |
|---|---|---|---|---|
| Play Frisco, explicit free statement | free | confirmed | true | `Free ✦` |
| Play Frisco, free-by-default (no signal) | free | inferred | true | `Free ✦` |
| Play Frisco, explicit paid signal | paid | confirmed | false | `Paid ✦` |
| Play Frisco, Layer 2/3 override (keyword / registration, no signal) | unknown | — | null | (no badge) |
| Play Frisco, ambiguous or genuinely torn | unknown | — | null | (no badge) |
| Library (any) | null | null | true | `Free` (plain) |

`is_free` remains a stored, derived column so the events API can filter on it; `price_class` is the source of truth. `age_inferred` / `price_inferred` are **not** stored — they are derived at render time from the badge logic.

---

### What does not change

- Library events — hardcoded free, plain `Free`, not affected.
- Age inference logic — price classification is additive; validate after first run that age accuracy has not drifted.

---

### Rollout note

New Play Frisco events ingested after this change is deployed will receive automatic price classification. Existing Play Frisco events (~20 at time of writing) retain their current price state until inference is cleared and re-run. Re-ingest is the same process used for age re-classification — one combined operation.

---

### Validation checklist before shipping

- [ ] LLM returns a valid `price` ("free" / "paid" / "unknown") AND `price_confidence` ("confirmed" / "inferred") for every event in the calibration set (`npm run calibrate:price`)
- [ ] No event shows a **confirmed** `Free` (solid badge) without an explicit free signal in the text; free-by-default (no signal) shows **`Free ✦`** (inferred), never a solid `Free`
- [ ] A structurally-paid keyword (Layer 2) or `registration_required` (Layer 3) with no explicit signal resolves to **unknown** (no badge) — not free
- [ ] Fallback (no-LLM) path resolves through the same layers — paid / free-inferred / unknown — and never asserts a *confirmed* free without an explicit free statement
- [ ] Events with `price_class = unknown` (`is_free = null`) show no price badge on card or in detail view
- [ ] Library events unaffected — still hardcoded `is_free = true`, shown as a confirmed `Free`
- [ ] Age inference accuracy unchanged after prompt extension (spot-check 5 events)

---

*Last updated: July 2026. Feeds into: BUILD-LOG (implementation notes), functional test scenarios (new test cases for price inference), case study (AI-driven data quality decision).*
