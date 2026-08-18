# Open Eventz — AI PM Portfolio Project

> A live, deployed kids' event discovery app for Frisco and Plano, TX —
> the only one that tells parents whether they need to stay or can drop their child off.

**Live app:** https://openeventz.com
**App code:** https://github.com/santoshkopanathi/open-eventz

[![CI](https://github.com/santoshkopanathi/open-eventz/actions/workflows/ci.yml/badge.svg)](https://github.com/santoshkopanathi/open-eventz/actions/workflows/ci.yml)

---

## Why this project exists

Parents trying to find free activities for kids have to check a dozen
disconnected sources — city calendars, library sites, parks departments —
with no single place that filters for free, age-appropriate, and nearby.

No existing app tells you whether you need to stay with your child or can
drop them off. Open Eventz does.

Open Eventz solves the discovery problem by aggregating three live data
sources into one filtered feed, with AI-powered inference filling gaps where
sources lack structured data.

This is a portfolio project targeting Principal AI PM roles. The goal was
to build something real — live data, deployed product, honest documentation
of what was planned vs. what actually happened.

---

## AI PM judgment calls — the short version

These are the decisions I'm most proud of and would want to talk through
in an interview:

- **Never guess when you don't know.** Applied consistently — supervision
  policy, age inference, price inference, LLM tie-breaks. When the system
  is uncertain, it says so rather than fabricating confidence.

- **Honest disclosure scales with context.** Card hover shows
  "Estimated from description." Detail view shows the full scenario-specific
  disclosure. Mobile shows nothing on the card — tapping opens the detail.
  One principle, three implementations.

- **Free-by-default is a population-level assumption, not a lazy default.**
  City parks events are free by institutional nature. The six-layer
  framework protects against the cases where that assumption breaks.

- **Store raw, derive display.** LLM outputs stored separately from what's
  shown to users. Policy changes require a one-function redeploy, not a
  re-ingest. Same pattern for age and price.

- **Calibration is ongoing, not a one-time check.** Ground truth set
  established through human review, two-tier testing (parser CI + LLM
  manual script), growing calibration set as new failures emerge.

---

## Built on the AI PM operating system

Open Eventz is a working proof of the **5-layer AI PM stack** — not as a concept,
but as shipped evidence. Each layer has a real artifact behind it:

- **Model** — Claude Sonnet chosen over Haiku: the cost delta is trivial at this
  volume, so accuracy on ambiguous confidence-tier calls was the deciding factor.
  The selection rationale is in the BUILD-LOG.
- **Context** — Owned prompt design, deterministic classification rules, and a
  human-labeled calibration set. The model behaves because the context is engineered
  and versioned — not because a prompt got lucky.
- **Orchestration** — A three-source ingest pipeline (scrape → infer → store →
  serve) with the LLM call embedded mid-workflow, plus fallback logic and graceful
  degradation at every failure point.
- **Governance** — A six-layer price-inference risk model, confidence tiers, honest
  disclosure, two-tier testing (deterministic CI + manual LLM calibration), doc↔test
  parity enforcement, and two live observability dashboards.
- **Human** — Every non-obvious decision is documented with its reasoning: why
  free-by-default, why Definition A, why the badge system was simplified, why the LLM
  tie-break resolves to *unknown* rather than *paid*. The judgment trail is the artifact.

The "operating system" here is a way of working, not a single file. Its **living
record is the BUILD-LOG** (`06-app/BUILD-LOG.md`), appended at every phase — a running
talk → decide → build → observe → iterate loop, not a one-time spec. `02-product/pm-os.md`
is the **strategy foundation** (vision, market, source strategy, decision log), kept
current through the build.

---

## The product

**Three live data sources:**
- Frisco Public Library (BiblioCommons HTML scraping)
- Plano Libraries network (Communico RSS — reverse-engineered base64 JWT)
- Play Frisco / Frisco Parks & Rec (CivicPlus city calendar scraping)

**Supervision guidance — the standout differentiator:**
- Tells parents whether they need to stay or can drop the child off, per
  venue/event — something no other kids-events app does
- A three-tier confidence model (event-specific policy → venue general policy
  → unverified) that **never shows a guessed age threshold** — a wrong
  "drop-off OK" is a real safety issue, so unverified always reads "check with venue"
- Frisco Library: drop-off OK at 10+ (from the official 2026 service policy);
  Plano: no age requirement, parent's discretion; unverified sources: "check with venue"

**AI-powered inference (Claude Sonnet):**
- Age suitability inference for Play Frisco events (no structured age field)
- Free/paid classification for Play Frisco events (no structured price field)
- Confidence tiers — high/medium/low — with honest disclosure when data
  is estimated rather than confirmed
- Graceful degradation — low confidence shows nothing rather than a bad guess

**Scan-level decision-making — before you tap anything:**
- Every event card surfaces the signals a parent needs to judge relevance at a
  glance — family suitability, free vs. paid, registration required, and whether
  it recurs
- The card badge set is deliberately minimal: only signals a filter *can't*
  already communicate appear on a card (structured age ranges, for instance, are
  filter-driven, so they live in the detail view) — everything else is noise removed
- Competitors we analyzed bury these signals in the detail view; Open Eventz puts
  the decision-relevant ones on the card itself

**Discoverability (SEO):**
- Per-event, server-rendered, individually indexable pages (`/events/[id]`)
- schema.org Event structured data → eligible for Google Event rich results
- City landing pages (`/frisco`, `/plano`) + a dynamic sitemap, submitted to
  Google Search Console

**Measurement (analytics):**
- GA4 instrumentation of the full acquisition-to-conversion funnel
- Two dashboards — functional (GA4 → BigQuery) and technical (Supabase
  pipeline health, inference visibility, LLM cost)

**Key PM decisions worth reading:**
- Why free-by-default with six-layer risk mitigation beats unknown-by-default
- Why all Play Frisco price classifications carry ✦ (Definition A)
- Why the product never guesses when it doesn't know — applied consistently
  across supervision policy, age inference, and price inference
- Why the badge system was simplified — filters do the work, badges
  shouldn't repeat what filters already communicate
- Why the SEO structured-data "free" signal is asserted for confirmed *and*
  inferred-free events (the page visibly shows the same badge, so the markup
  matches the page)

---

## Key artifacts — where to start

If you read nothing else, read these four:

**1. BUILD-LOG** (`06-app/BUILD-LOG.md` — in the app repo)
The honest record of what was planned vs. what was actually built.
Includes the BiblioCommons audience_id silent failure, the Communico JWT
reverse-engineering, the Vercel timeout constraint, and the keyword parser
false positive that led to LLM price inference. This is where the real
product thinking lives.

**2. v1.1 Feature Spec** (`02-product/open-eventz-v1.1-city-nav-age-inference-spec.md`)
City-first navigation, LLM age inference with calibrated test cases,
badge simplification decisions, and the trust disclosure framework.

**3. Analytics, Price Inference & Measurement Spec** (`02-product/open-eventz-v1.2-feature-spec-analytics-price-inference.md`)
Analytics dashboard design with JTBD-grounded metrics, LLM price
classification with six-layer risk mitigation, and the measurement /
calibration framework — how to keep an AI feature honest over time (ground
truth, two-tier testing, a growing calibration set). Includes the decision
to move from keyword heuristics to LLM inference and why.

**4. SEO Design** (`06-app/SEO-DESIGN.md` — in the app repo)
The full concept → functional → technical write-up of the SEO foundation:
per-event indexable pages, schema.org Event structured data, city landing
pages, dynamic sitemap/robots, and the Google Search Console setup — plus
the one judgment call (asserting "free" for inferred-free events, and why
it stays within Google's policy).

---

## What this repo contains

| Folder | Contents |
|---|---|
| `01-company/` | Company vision — deliberately broad to hold space beyond aggregation |
| `02-product/` | PRD, PM OS, feature specs (v1.1, v1.2), SEO scoping, test-scenario pointer, KPI framework, case study |
| `03-research/` | Personas, market research, competitive analysis |
| `04-data/` | Supervision policy intake — three-tier confidence framework |
| `05-design/` | Interactive prototypes — v1 (live app) and v1.1 (city navigation, inference) |
| `08-portfolio/` | Interview prep — challenges, judgment-call talking points, technical concepts |

*(App source code lives in the separate app repo linked above, not in this repo.)*

---

## Engineering & delivery

Open Eventz is built spec-first on a real CI/CD pipeline:

- **Nothing merges red.** CI runs type-checking, the full unit suite, and
  Playwright E2E smoke tests on every push. Local pre-commit and pre-push
  hooks run type-check + unit before code leaves the machine (E2E is CI-only —
  it depends on a browser binary that isn't guaranteed on a dev machine).
- **Documentation is updated per feature — and one part is CI-enforced.** Every
  feature updates the engineering BUILD-LOG, its spec in `02-product/`, and the
  consolidated test plan (`06-app/TEST-SCENARIOS.md`). That's disciplined practice,
  not automation — but a CI `doc-parity` job makes one slice non-optional: it fails
  the build if the test plan claims a test file that no longer exists, so the plan
  can't silently drift from the real suite.
- **LLM calibration is a targeted checkpoint, not a gate on every change.**
  It runs only when a prompt, model version, or inference logic changes —
  because it costs money per run and is non-deterministic. Everything else
  is covered by the always-on automated suite.
- **Auto-deploy.** Merging to `master` triggers a Vercel production build;
  rollback is one click.

---

## Observability & logging

- **Ingest pipeline — persisted.** Every ingest run writes a row to an
  `ingest_runs` table: timing, per-source counts, LLM calls and estimated
  cost, status, and errors. The Technical dashboard reads this directly.
- **Product usage — persisted.** GA4 instruments the full funnel, exported
  daily to BigQuery. The Functional dashboard reads that for WAD, funnel
  conversion, referral, and top events.
- **CI / test results — captured per run.** Every CI run uploads a coverage
  report and Playwright HTML report as artifacts, writes a per-job summary,
  and surfaces the status badge at the top of this file.
- **Application errors — the current gap → next step.** Production exceptions
  currently live only in Vercel's ephemeral function logs. The planned fix
  is Sentry (`@sentry/nextjs`) — deferred only because it needs a project
  DSN stored as a Vercel secret.

---

## Test Coverage

| Layer | What it covers | Tool | Count | Runs in CI |
|---|---|---|---|---|
| Type checking | Types compile cleanly | tsc --noEmit | — | ✅ every push |
| Unit tests | Badge logic, price inference, filters, JSON-LD, SEO, calendar | Jest | 224 tests / 13 suites | ✅ every push |
| E2E smoke tests | UI renders and behaves correctly, /api mocked | Playwright | 9 tests | ✅ every push |
| Doc↔test parity | Every automated scenario names a test file that still exists | Node script | — | ✅ every push |
| LLM calibration | Claude price + age accuracy vs. ground truth | Manual script | 8 calibration events | ❌ manual — run on prompt/model changes |
| Manual scenarios | Live ingest, GA4 realtime, dashboards, mobile tooltips | Human | — | ❌ manual |

**Coverage:** ~96% line / 94% statement coverage on the unit suite. Results
are captured, not ephemeral — every CI run uploads the full coverage report
and the Playwright report as artifacts.

**Key design decision:** LLM calibration is deliberately outside CI — it
costs money per run and is non-deterministic. Instead it runs as a checkpoint
(`npm run calibrate:price`) whenever the system prompt changes, the model
version changes, or drift is suspected. Ground truth is a human-labeled set
of Play Frisco events covering free, paid, and edge cases.

**Full scenario plan:** `06-app/TEST-SCENARIOS.md` — the consolidated,
scenario-by-scenario functional test plan (tagged automated / regression /
manual). A CI `doc-parity` job keeps it honest: it fails the build if a
scenario claims a test that no longer exists.

*Last verified: all 224 unit tests and 9 E2E tests passing as of 2026-07-23.*

---

## Tech stack

Next.js 16 · TypeScript · Supabase (PostgreSQL) · Tailwind CSS ·
Google Maps API · GA4 + BigQuery · Vercel · Claude Sonnet (Anthropic) for
LLM inference

---

## Status

**v1.2 shipped & live** — price inference, analytics instrumentation, and
both dashboards (functional + technical) deployed to production. **SEO
foundation live** — per-event indexable pages, Event JSON-LD, city landing
pages, sitemap, and Google Search Console (search indexing still ramping).
