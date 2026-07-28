# Open Eventz — Functional Test Scenarios v1.1
## Badge simplification, Family-label rule, and dropdown / multi-select filters

*Companion to `functional-test-scenarios.md` (the v1.0 baseline). This document evolves the test cases for the areas changed in the v1.1 badge/filter work. Where a v1.0 case still holds, it is **referenced, not duplicated** (see §8). Where a v1.0 case is **superseded**, this document is authoritative.*

**Tags:** `[R]` regression (run every deploy) · `[A]` automated (Jest unit test — `npm test`) · `[M]` manual / browser

**Supersedes in the v1.0 doc:** §2.2 (age badges on cards) and §6.3 (age display in detail). The age-filter behavior in v1.0 §5 is extended here to multi-select OR, and the dropdown descriptions in v1.0 §4–§5 are realized as built here.

*Last updated: July 2026.*

---

## 1. Age badge — card rendering (supersedes v1.0 §2.2)

Cards show only "Family" (confirmed or inferred) and the bare inferred marker; **structured age ranges are removed from cards** (detail-only).

| # | Scenario | Expected result | Tag |
|---|---|---|---|
| 1.1 | Frisco/Plano event, single structured age group (0–5, 6–12, teen) | **No** age badge on card | [A] [R] |
| 1.2 | Frisco/Plano event, multi-group but not family (e.g. Kids + Teens) | **No** age badge on card | [A] [R] |
| 1.3 | Plano event explicitly tagged "Families (All Ages)" | Gold **"Family"** badge on card | [A] [R] |
| 1.4 | Play Frisco inferred family (high/medium confidence) | Indigo **`~ Family ✦`** badge on card | [A] [R] |
| 1.5 | Play Frisco inferred specific age (high/medium confidence) | Bare **`✦`** marker (no age text, no tilde) | [A] [R] |
| 1.6 | Play Frisco low confidence or `kid_relevant:false` | No badge | [A] [R] |
| 1.7 | Event with no age data | No badge | [A] [R] |
| 1.8 | Desktop hover on any `✦` marker (age or price) | Tooltip: "Estimated from description" (single simplified string, all scenarios). Mobile: no tooltip. | [M] |
| 1.9 | Mobile tap on `✦` | Opens the detail view (full disclosure shown there) | [M] |

*1.1–1.7 automated by `src/lib/age-badge.test.ts` (`cardAgeBadge`).*

---

## 2. Age badge — detail rendering (supersedes v1.0 §6.3)

| # | Scenario | Expected result | Tag |
|---|---|---|---|
| 2.1 | Frisco/Plano single structured age group | Age-label badge ("Ages 0–5" / "Ages 6–12" / "Teens") | [A] [R] |
| 2.2 | Frisco/Plano multi-group, not family | Single collapsed range ("Ages 6–17"); **not** labeled "Family" | [A] [R] |
| 2.3 | Plano "Families (All Ages)" | Gold **"Family"** badge; no disclosure line | [A] [R] |
| 2.4 | Play Frisco inferred family | `~ Family ✦` + detail disclosure "Family suitability estimated from event description" (combined with price if price is also inferred); **no** separate "Family event" label | [A] [R] |
| 2.5 | Play Frisco inferred specific age | `~ Ages [range] ✦` + detail disclosure "Age suitability estimated from event description" | [A] [R] |
| 2.6 | Low confidence or no age data | Nothing shown | [A] [R] |
| 2.7 | Legacy "👶 Suitable for: …" text line | Removed (no longer rendered) | [M] |

*2.1–2.6 automated by `src/lib/age-badge.test.ts` (`detailAgeBadge`).*

---

## 3. Family-label rule (source logic)

"Family" appears **only** from an explicit signal — never derived from a numeric multi-group span.

| # | Scenario | Expected result | Tag |
|---|---|---|---|
| 3.1 | Plano event tagged "Families (All Ages)" | Labeled "Family" (confirmed, gold) | [A] [R] |
| 3.2 | Play Frisco event inferred family | Labeled "Family" (inferred, indigo `~ Family ✦`) | [A] [R] |
| 3.3 | Frisco event tagged for multiple age groups | **Never** "Family" (Frisco has no family field) | [A] [R] |
| 3.4 | Plano "Kids + Adults" (numeric range resolves to 6–12) | **Never** "Family" from a numeric span | [A] [R] |
| 3.5 | Ingest: explicit "Families (All Ages)" tag stored as `age_buckets=['family']` | `communicoIsFamily` true only for the explicit tag (a plain 0–17 numeric range is not family) | [A] |

*Automated by `age-badge.test.ts` and `age-parsers.test.ts` (`communicoIsFamily`).*

---

## 4. Age filter — multi-select OR logic (extends v1.0 §5)

| # | Scenario | Expected result | Tag |
|---|---|---|---|
| 4.1 | Single age chip (Kids 6–12) | Events overlapping 6–12 shown | [A] [R] |
| 4.2 | Multiple age chips (Toddlers + Teens) | OR logic — events overlapping 0–5 **or** 13–17; kids-only (6–12) events **excluded** | [A] [R] |
| 4.3 | All-ages / family event, any active chip | Shown under every selected age chip | [A] [R] |
| 4.4 | Play Frisco low-confidence under active age filter | Excluded | [A] [R] |
| 4.5 | Play Frisco `kid_relevant:false` | Never shown in any view | [A] [R] |
| 4.6 | Plano "Kids + Adults" event under the Teens chip | **Not** shown (adult range excluded from kid-facing range — no bleed) | [A] [R] |

*Automated by `src/lib/age-filter.test.ts` (`passesAgeFilter`).*

---

## 5. Filter dropdowns (realizes v1.0 §4–§5)

Multi-select dropdowns with count badges; each takes the active city's accent color.

| # | Scenario | Expected result | Tag |
|---|---|---|---|
| 5.1 | Open Source dropdown (Frisco tab) | Checkboxes: "Frisco Library", "Play Frisco" | [M] |
| 5.2 | Open Branch dropdown (Plano tab) | Davis, Haggard, Harrington, Parr, Schimelpfenig, Virtual | [M] |
| 5.3 | Open Age dropdown | Toddlers (0–5), Kids (6–12), Teens — multi-select checkboxes | [M] |
| 5.4 | Exactly one option selected | Button label shows that option's name (e.g. "Haggard") | [M] |
| 5.5 | Two or more options selected | Button shows the group label + count badge (e.g. "Branches" ‑ 2) | [M] |
| 5.6 | Zero options selected | Group label, no badge = "all" (no filter applied) | [M] |
| 5.7 | Date range placement (desktop) | From/to date inputs sit on the same filter row as the dropdowns; wrap to a second line on narrow widths | [M] |
| 5.8 | Click outside an open dropdown | Dropdown closes | [M] |

---

## 6. Recurring indicator (extends v1.0 §2.1.7)

| # | Scenario | Expected result | Tag |
|---|---|---|---|
| 6.1 | Event title on 2+ future dates within a source | `is_recurring` set → `↻ Recurring` badge on card and detail | [A] [R] |
| 6.2 | Single-occurrence event | No recurring badge | [A] [R] |
| 6.3 | Same title in two different sources | Not merged (detection is source-scoped) | [A] |
| 6.4 | Event already flagged recurring at scrape time (Frisco "View all dates") | Flag preserved; label not overwritten | [A] |

*Automated by `src/lib/recurring.test.ts` (`markRecurring`).*

---

## 7. Automated unit test coverage

Pure logic runs under Jest (`npm test`) — no server or DB required.

| Suite | Covers | Cases in this doc |
|---|---|---|
| `age-parsers.test.ts` | "Suitable for" + Communico AGE GROUP parsing, URL-encoded "Families (All Ages)" tag, adult-range exclusion | 3.4, 3.5 |
| `age-badge.test.ts` | `getAgeBadge` (5 badge kinds), `cardAgeBadge`, `detailAgeBadge` | 1.1–1.7, 2.1–2.6, 3.1–3.3 |
| `age-filter.test.ts` | `passesAgeFilter` — overlap, Play Frisco buckets, multi-select OR, low-confidence / kid-relevant gating | 4.1–4.6 |
| `recurring.test.ts` | title-based recurring detection, source-scoped | 6.1–6.4 |

**68 unit tests, all passing.** UI wiring (dropdown open/close, count-badge rendering, tab persistence, tooltips) and the ingest / events-API HTTP flow remain manual/browser-verified.

---

## 8. Carried forward from the v1.0 doc (unchanged — still regression)

These v1.0 scenarios remain valid as-is and are **not** superseded by this document. Continue running them from `functional-test-scenarios.md`:

- **City-first navigation & tab switching** — v1.0 §3.1
- **Per-city filter state persistence** — v1.0 §3.2
- **Detail view core / actions / supervision badge** — v1.0 §6.1, §6.4, §6.2
- **Play Frisco LLM inference accuracy** (validated events) — v1.0 §7.1
- **Ingest pipeline** — v1.0 §1
- **Error / empty states** — v1.0 §8

---

## 9. v1.1 Regression suite (run before every deploy)

- **Automated:** `npm test` (68 tests) — covers the pure logic in §1–§4 and §6.
- **Manual / browser:** §1.8–1.9 (tooltips), §2.7, §5 (dropdowns), plus the carried-forward v1.0 `[R]` cases in §8.
