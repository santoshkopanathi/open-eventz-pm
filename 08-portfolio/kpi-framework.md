# Open Eventz — KPI Framework

*Defines success metrics for Open Eventz v1. All five KPIs map to a specific layer of the product — discoverability, conversion, data reliability, differentiation, and retention — and together form a funnel that shows where value is delivered and where it breaks down.*

---

## North Star Metric

**Weekly Active Discoverers (WAD)**

*The number of unique visitors per week who complete at least one intent action — Add to Calendar, Share, or Get Directions — after applying at least one filter.*

### Why this metric

The north star has to capture the moment the product actually delivered on its core promise: a parent found a real, relevant free activity and committed to it in some way. Three design choices are built into this definition:

- **"Weekly"** — not daily (too volatile for a product driven by weekly event cycles) and not monthly (too slow a signal to iterate against). Library storytimes and city events run on weekly rhythms; the metric mirrors that.
- **"At least one filter applied"** — this is the differentiating gate. A visitor who scrolls a default list and bounces could have been served by any events website. A visitor who filtered by age range, verified free, and this weekend before acting is the one who found the product genuinely useful. The filter requirement separates intent from accident.
- **"Intent action"** — calendar add, share, or directions are behavioral signals that a visit converted to real-world intent. They are measurable without requiring the user to report back whether they actually attended the event.

### What it is not

WAD deliberately excludes: raw page views (vanity), session count (a frustrated user who opens three events and bounces counts the same as an engaged one), and like/attending taps (engagement signal, not a discovery signal — someone can like an event they saw last month).

### Healthy benchmark (hypothesis)

20-50 WAD within the first month of launch with no paid acquisition, growing week-over-week. This is a hypothesis to validate and revisit once usage data is available.

---

## Supporting KPIs

Five metrics that each cover a distinct layer of the product. A drop at any one of them points at a specific, addressable problem rather than a vague sense that "engagement is low."

---

### KPI 1 — Filter engagement rate

**Definition:** Percentage of sessions in which the visitor applies at least one filter before viewing an event.

**Why it matters:** The filter system (age range, free/paid, date range, recurring vs. one-time, source) is the core differentiating feature of Open Eventz — it's what makes this product different from a plain list. If users aren't engaging with filters, either the filters aren't discoverable, aren't useful, or the event list is short enough that they don't feel necessary. Low filter engagement is an early signal to revisit filter UX or event volume.

**How to measure:** Track filter_applied events (one per filter interaction per session) in the analytics layer. Filter engagement rate = sessions with at least one filter_applied ÷ total sessions.

**Healthy benchmark hypothesis:** 55%+ of sessions include at least one filter interaction within 60 days of launch.

**What to do if low:** A/B test filter placement (top bar vs. sidebar vs. bottom sheet on mobile), reduce the number of filters to the two or three highest-value ones, or add a "Quick picks" shortcut row (e.g. "Free this weekend" as a one-tap preset).

---

### KPI 2 — Intent action rate

**Definition:** Percentage of event detail views that result in at least one intent action (Add to Calendar, Share, or Get Directions).

**Why it matters:** This is the conversion metric — it measures whether the detail view is doing its job of converting interest into commitment. A visitor who opens an event but takes no action may have found it irrelevant, may have been missing key information (unclear price, missing supervision note, no description), or may have found the UX friction too high. Low intent action rate points at a detail view problem, not a discovery problem.

**How to measure:** intent_action events (calendar_add, share_tap, directions_tap) divided by detail_view events.

**Healthy benchmark hypothesis:** 25%+ of detail views result in at least one intent action.

**What to do if low:** Check which fields are most often empty on low-performing events (missing description? No supervision note? Unclear price?) and prioritize filling those data gaps before optimizing the UX.

---

### KPI 3 — Data freshness and feed reliability

**Definition:** Percentage of page loads on which all three sources return at least one upcoming event successfully.

**Why it matters:** The product's core promise is "real, currently-accurate events." If feeds are stale or erroring, the promise breaks before a single filter is applied. This KPI measures whether the technical architecture is delivering on what the PRD committed to. It also tracks the specific risk called out in the PRD: the Play Frisco scrape is more fragile than the two official library feeds and could fail silently without monitoring.

**How to measure:** Log a source_load_success or source_load_failure event per source per page load. Feed reliability = successful loads ÷ total load attempts, tracked per source separately so a Play Frisco failure is distinguishable from a library feed failure.

**Healthy benchmark hypothesis:** 99%+ for both library sources (official, documented feeds); 90%+ for Play Frisco (scrape-based, inherently more fragile).

**What to do if low:** Library feeds failing → check for CORS issues or feed URL changes; proxy fallback should activate automatically. Play Frisco failing → inspect scrape selectors against current HTML structure; add a monitoring alert for zero-result scrape runs.

---

### KPI 4 — Supervision data coverage

**Definition:** Percentage of events displayed that have a Tier 1 or Tier 2 supervision policy entry (vs. "check with venue").

**Why it matters:** The supervision flag is Open Eventz's single most defensible differentiator — the feature no competitor provides and that cannot easily be copied without the same source-by-source intake work documented in PM OS Section 7. But it only has value if it's populated. This KPI tracks progress of the Supervision Policy Intake work (04-data/supervision-intake.xlsx) and holds the team accountable to completing it. A product that shows "check with venue" on 90% of events has the feature in name only.

**How to measure:** Count of events with supervision_confidence_tier = Tier 1 or Tier 2 ÷ total events displayed. Tracked weekly as intake work progresses.

**Healthy benchmark hypothesis:** 40%+ of events have a verified supervision entry within 30 days of launch — achievable once Frisco Library Tier 2 data is live and Plano Library intake is completed.

**What to do if low:** Prioritize completing the supervision intake spreadsheet for the two remaining in-scope sources (Plano Library and Play Frisco). Consider direct outreach to each venue's program coordinator rather than relying solely on published policy documents.

---

### KPI 5 — Return visit rate

**Definition:** Percentage of visitors who return to the app in a second distinct week within their first 30 days.

**Why it matters:** One visit can be explained by curiosity or a referral link. A return visit — especially in a different week — is evidence that the visitor found enough value to seek the product out again unprompted. For Open Eventz specifically, weekly return visits map directly to the intended use case: a parent checking "what's happening this week" as a recurring habit, not a one-time search. This is the retention signal that distinguishes a useful tool from a novelty.

**How to measure:** Via a persistent anonymous session identifier (cookie or localStorage key generated on first visit). Return visit rate = visitors with at least 2 sessions in distinct calendar weeks within 30 days ÷ all visitors with at least 1 session.

**Healthy benchmark hypothesis:** 30%+ week-2 return rate among visitors who completed at least one intent action in week 1. Visitors who acted are far more likely to return than passive browsers — segmenting by intent action makes the benchmark meaningful.

**What to do if low:** The most likely cause is that visitors found events for this week but had no reason to return next week — which points at a notification/reminder gap (Dave's unmet need from PM OS Section 2). Short-term mitigation without building notifications: add a "Save this search" or "Remind me next week" prompt at the bottom of a filtered results page.

---

## How these five KPIs map to the product

| KPI | Layer | What failure tells you |
|---|---|---|
| Filter engagement rate | Feature discoverability | Filters aren't found or aren't useful |
| Intent action rate | Conversion / detail view quality | Events aren't compelling or detail view is missing key info |
| Feed reliability | Data strategy / technical | Architecture isn't delivering on the freshness promise |
| Supervision coverage | Differentiation depth | The standout feature exists in name only |
| Return visit rate | Retention | Product is useful once but not habit-forming |

Together they form a funnel: visitors arrive → engage with filters → view events → take intent actions → return next week. A drop at any stage points at a specific, addressable problem.

---

## What's intentionally not tracked

- **Raw page views / unique visitors** — volume without engagement signals nothing useful about product-market fit.
- **Bounce rate** — misleading for a content app where a visitor might find what they need in 20 seconds and leave satisfied.
- **Like/attending count** — engagement signal only, not a discovery or retention signal. Tracked in the app for social proof but not in the KPI framework.
- **Time on page** — ambiguous; a long session could mean deep engagement or confusion.

---

## Instrumentation plan

All five KPIs can be tracked with a lightweight analytics setup:

- **Google Analytics 4** (free tier) handles session-level metrics, return visit rate, and custom events for filter_applied, detail_view, and intent_action.
- **A server-side log** on the like-count endpoint gives feed reliability data as a side effect of normal operation.
- **A weekly manual check** of the supervision intake spreadsheet covers supervision coverage — no automation needed at this scale.

Total instrumentation cost: $0 (GA4 free tier) + one afternoon of event tagging in the app code.

---

*Feeds into: Open Eventz PM OS Section 1 (Product Charter — success definition).*
*Last updated: June 2026.*
