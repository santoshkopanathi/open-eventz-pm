# Functional Test Scenarios — v1.2 (Price Inference + Analytics)

Supersedes the price/analytics portions of earlier versioned docs. Tags: **[A]** automated (Jest/Playwright), **[R]** regression (run every change), **[M]** manual (can't be automated — live scrape, LLM accuracy, GA4 realtime, dashboard-with-real-data).

Run the automated layers with: `npm run typecheck && npm test && npm run test:e2e`. LLM price accuracy: `npm run calibrate:price`.

---

## 1. Price classification — Play Frisco (Definition A)

| # | Scenario | Expected | Tag |
|---|---|---|---|
| 1.1 | Structured `Cost: Free` field present | Confirmed free → plain **`Free`** badge, **no `✦`**; price omitted from the disclosure | [A] [R] |
| 1.2 | Structured `Cost: $35` / `Cost: Paid` field | Confirmed paid → plain **`Paid`**, no `✦` | [A] [R] |
| 1.3 | No Cost field; description says "FREE ADMISSION" | Inferred **`Free ✦`** (LLM/parser confirmed free, but Play Frisco → still `✦`) | [A] [R] |
| 1.4 | No Cost field; no price words; plain drop-in | Free-by-default → **`Free ✦`** | [A] [R] |
| 1.5 | No Cost field; no price words; "workshop"/"class" or registration required | Layer 2/3 → **unknown → no badge** | [A] [R] |
| 1.6 | Explicit paid signal ("BUY TICKETS", "$7", "purchase a ticket") | **`Paid ✦`** | [A] [R] |
| 1.7 | Member-gated pricing ("Free for members … $7/$9") | **`Paid ✦`** (a stated price for the public wins over "free for members") | [A] [R] |
| 1.8 | Ambiguous ("members only", "reservation required") or genuinely torn | **unknown → no badge** | [A] [R] |
| 1.9 | Library event (Frisco/Plano) | Institutional **`Free`**, no `✦` | [A] [R] |
| 1.10 | LLM accuracy on the calibration set | `npm run calibrate:price` — all ground-truth events pass | [M] |
| 1.11 | Live re-ingest picks up Cost fields | After migration 003 + re-ingest, "Learn to Fish" shows confirmed `Free` (no `✦`) | [M] |

*1.1–1.9 covered by `price.test.ts` (`getPriceBadge`, `interpretCostField`, calibration set) + `e2e/smoke.spec.ts`.*

## 2. Inference disclosure (detail view)

| # | Scenario | Expected | Tag |
|---|---|---|---|
| 2.1 | Age inferred only (family) | One line: "Family suitability estimated from event description" | [A] [R] |
| 2.2 | Age inferred only (specific) | "Age suitability estimated from event description" | [A] [R] |
| 2.3 | Price inferred only (free/paid) | "'Free'/'Paid' admission status estimated from event description" | [A] [R] |
| 2.4 | Both age + price inferred | **ONE** combined line (e.g. "Family suitability and 'Paid' admission status estimated…"); never two lines | [A] [R] |
| 2.5 | Confirmed price (Cost field) + inferred age | Disclosure mentions **age only**; price omitted (it's confirmed) | [A] [R] |
| 2.6 | Card `✦` hover (desktop) | Tooltip "Estimated from description" (single string). Mobile: no tooltip; tap opens detail | [M] |

*2.1–2.5 covered by `inference-disclosure.test.ts` + `e2e/smoke.spec.ts`.*

## 3. Analytics — GA4 instrumentation

| # | Scenario | Expected | Tag |
|---|---|---|---|
| 3.1 | Base tag loads | `gtag` present, config = the Measurement ID (verify in GA4 DebugView / `dataLayer`) | [M] |
| 3.2 | `filter_applied` | Fires on any filter dropdown change | [M] |
| 3.3 | `event_card_click` | Fires on card tap (with `source`, `event_id`) | [M] |
| 3.4 | `detail_view` | Fires when detail opens (once, not doubled by mobile+desktop instances) | [M] |
| 3.5 | `directions_tap` | Fires on Get Directions | [M] |
| 3.6 | `calendar_add` | Fires on Google Calendar **and** Apple/ICS | [M] |
| 3.7 | `attending_tap` | Fires on Attending **toggle-on only** (not un-attend) | [M] |
| 3.8 | `share_tap` | Fires on Share | [M] |

*Verified in-browser via `dataLayer` during build; ongoing check is GA4 Realtime/DebugView.*

## 4. Measurement framework (metric definitions)

| # | Scenario | Expected | Tag |
|---|---|---|---|
| 4.1 | WAD | Unique visitors/week with ≥1 conversion action, once per visitor/week | [A] [R] |
| 4.2 | Funnel | Cumulative "most advanced step"; convert-without-intent still counts | [A] [R] |
| 4.3 | Channel segmentation | Funnel scopes to a single acquisition channel | [A] [R] |
| 4.4 | Return-visit rate | % of week-1 converters active in week 2 | [A] [R] |
| 4.5 | Referral / top events | Weekly shares + % of converted sessions shared; per-event tallies | [A] [R] |

*All covered by `measurement.test.ts`.*

## 5. Technical dashboard (`/dashboard`)

| # | Scenario | Expected | Tag |
|---|---|---|---|
| 5.1 | Per-source counts + free/paid/unknown | Real Supabase counts | [A] [R] (`technical-metrics.test.ts`) |
| 5.2 | Inferred-age visibility | 4 buckets summing to Play Frisco total | [A] [R] |
| 5.3 | Ingest pipeline / history / log / LLM cost | Populated after migration 004 + an ingest run | [M] |
| 5.4 | No `ingest_runs` table yet | Graceful "No runs recorded yet" (verified) | [M] |

## 6. Functional dashboard (GA4)

| # | Scenario | Expected | Tag |
|---|---|---|---|
| 6.1 | — | Pending BigQuery export + read key (#3); build after a week of data | [M] |

---

*Rollout reminders (manual): run migrations `003` (price) + `004` (ingest_runs) in Supabase; clear inference + re-ingest to apply Cost-field prices; set the GA4 Data/BigQuery read credential for the Functional dashboard.*
