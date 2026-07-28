# Open Eventz — Functional Test Scenarios

*`[R]` = Regression test — must pass after every deployment and after the v1.1 city-first navigation feature is implemented. All other tests are new functionality specific to v1.1.*

*Organized by feature area. Hand this document to Claude Code to generate executable test code against the real implementation.*

---

## 1. Data Ingest Pipeline

### 1.1 Frisco Library (BiblioCommons HTML scrape)

| # | Scenario | Expected result | Tag |
|---|---|---|---|
| 1.1.1 | Ingest runs against live friscotexas.gov library calendar | Events table populated with Frisco Library events; no ingest errors logged | [R] |
| 1.1.2 | Event with time formatted as `10:00am` (no space) | Time parsed correctly as 10:00 AM; no date parse error | [R] |
| 1.1.3 | Same event appears in multiple audience feed URLs | Only one record created per event; no duplicate IDs in events table | [R] |
| 1.1.4 | Event detail page scraped for age data | `age_label` field populated from "Suitable for:" block on detail page | [R] |
| 1.1.5 | Event with no "Suitable for:" block on detail page | `age_label` is null; event still ingested; no ingest failure | [R] |
| 1.1.6 | BiblioCommons returns HTTP error on detail page fetch | Ingest logs error for that event; other events continue ingesting; Plano and Play Frisco unaffected | [R] |

### 1.2 Plano Libraries (Communico RSS)

| # | Scenario | Expected result | Tag |
|---|---|---|---|
| 1.2.1 | Ingest runs against Communico RSS feed | Events table populated with Plano Library events across all five branches | [R] |
| 1.2.2 | Event has AGE GROUP field in Communico feed | `age_label` populated correctly; overlap matching maps to correct bucket | [R] |
| 1.2.3 | Event tagged "Adults" or "Older Adults" in Communico | Event ingested into Supabase with `age_min=18`; filtered out at API layer (`age_min < 18`); never appears in UI | [R] |
| 1.2.4 | Event tagged "Families (All Ages)" in Communico | Event ingested with `age_min=0`, `age_max=17`; appears under all age filter selections | [R] |
| 1.2.5 | Communico RSS returns malformed XML | Ingest logs error for Plano source; Frisco Library and Play Frisco events still load in UI | [R] |
| 1.2.6 | Base64 JWT token in RSS URL encodes 365-day window | Feed returns events up to 365 days ahead; not truncated to 1 day | [R] |

### 1.3 Play Frisco (CivicPlus scrape)

| # | Scenario | Expected result | Tag |
|---|---|---|---|
| 1.3.1 | Ingest runs against friscotexas.gov city calendar | Play Frisco events populated in events table | [R] |
| 1.3.2 | Event present in previous ingest but absent from today's scrape | Event deleted from events table (stale cleanup runs) | [R] |
| 1.3.3 | CivicPlus HTML structure changes on event listing page | Ingest fails gracefully; error logged; UI shows "Play Frisco events temporarily unavailable" banner; library events unaffected | [R] |
| 1.3.4 | Paid event scraped (e.g. Youth Soccer Camp with $95/week) | `is_free=false`, `price_text='$95/week'` stored correctly | [R] |

### 1.4 Ingest security and architecture

| # | Scenario | Expected result | Tag |
|---|---|---|---|
| 1.4.1 | `/api/ingest` called without bearer token | Returns 401; no ingest runs | [R] |
| 1.4.2 | `/api/ingest` called with valid bearer token | Ingest runs; returns 200 with summary | [R] |
| 1.4.3 | Composite event ID (`{source}-{original-id}`) for event already in DB | Upsert updates existing record; no duplicate row created | [R] |

---

## 2. Event Display — List View

### 2.1 Core list behavior

| # | Scenario | Expected result | Tag |
|---|---|---|---|
| 2.1.1 | App loads with no filters applied | All upcoming events shown, sorted by date ascending | [R] |
| 2.1.2 | Events from all three sources present in DB | All three sources display in unified list | [R] |
| 2.1.3 | Event card displays title, time, location, source | All four fields visible on card without opening detail | [R] |
| 2.1.4 | Free event | "Free" green badge displayed on card | [R] |
| 2.1.5 | Paid event | "Paid" amber badge displayed; price NOT shown on card (shown in detail view) | [R] |
| 2.1.6 | Event requires registration | "Reg." badge displayed on card | [R] |
| 2.1.7 | Recurring event | "↻ Recurring" purple badge displayed on card | [R] |
| 2.1.8 | One-time event | No recurring badge shown | [R] |
| 2.1.9 | Date range pre-populated on load (v1.1) | From date = today; To date = today + 7 days; both fields populated without user action | |

### 2.2 Age badges on cards (NEW)

| # | Scenario | Expected result | Tag |
|---|---|---|---|
| 2.2.1 | Frisco Library event with structured age data | Gold age badge displayed on card (e.g. "Children (0–5)") | |
| 2.2.2 | Plano Library event with structured age data | Gold age badge displayed on card | |
| 2.2.3 | Play Frisco event with high-confidence inferred age | Blue `~ [Age] ✦` badge displayed on card | |
| 2.2.4 | Play Frisco event with medium-confidence inferred age | Blue `~ [Age] ✦` badge displayed on card | |
| 2.2.5 | Play Frisco event with low-confidence inferred age | No age badge shown on card | |
| 2.2.6 | Event with no age data and not inferred | No age badge shown on card | |
| 2.2.7 | Hover over `~ [Age] ✦` badge | Tooltip appears: "Age range estimated from event description · not confirmed by source" | |

---

## 3. City-First Navigation (NEW)

### 3.1 City tab switching

| # | Scenario | Expected result | Tag |
|---|---|---|---|
| 3.1.1 | App loads for first time | Frisco City tab active by default; Frisco events shown | |
| 3.1.2 | Click Plano City tab | List refreshes showing only Plano Library events; Frisco events not shown | |
| 3.1.3 | Click Frisco City tab | List refreshes showing only Frisco Library + Play Frisco events | |
| 3.1.4 | Frisco City tab active | Gold accent visible on tab underline and filter bar left border | |
| 3.1.5 | Plano City tab active | Distinct blue accent visible on tab underline and filter bar left border | |
| 3.1.6 | Switch from Frisco to Plano and back | Visual accent changes correctly on each switch | |

### 3.2 Per-city filter state persistence (NEW)

| # | Scenario | Expected result | Tag |
|---|---|---|---|
| 3.2.1 | Select "Kids (6–12)" on Frisco tab, switch to Plano tab | Plano tab shows its own default filter state (unfiltered); Frisco age selection not applied to Plano | |
| 3.2.2 | Select "Kids (6–12)" on Frisco tab, switch to Plano, switch back to Frisco | "Kids (6–12)" selection still active on Frisco tab | |
| 3.2.3 | Select "Haggard Library" branch on Plano tab, switch to Frisco, switch back to Plano | "Haggard Library" branch selection still active on Plano tab | |
| 3.2.4 | Select age filter on Plano, select different age on Frisco | Each city retains its own independent age selection | |

---

## 4. Source and Branch Dropdowns (NEW)

### 4.1 Frisco City source dropdown

| # | Scenario | Expected result | Tag |
|---|---|---|---|
| 4.1.1 | Open source dropdown on Frisco tab | Shows "Frisco Library" and "Play Frisco" as checkboxed options | |
| 4.1.2 | Select "Frisco Library" only | List shows only Frisco Library events | |
| 4.1.3 | Select "Play Frisco" only | List shows only Play Frisco events | |
| 4.1.4 | Select both sources | List shows all Frisco events (same as no filter) | |
| 4.1.5 | One source selected | Dropdown button label updates to show selected source name | |
| 4.1.6 | Both sources selected | Dropdown shows "Sources" label with count badge "2" | |

### 4.2 Plano City branch dropdown

| # | Scenario | Expected result | Tag |
|---|---|---|---|
| 4.2.1 | Open source dropdown on Plano tab | Shows five branch options: Harrington, Haggard, Schimelpfenig, Davis, Memorial | |
| 4.2.2 | Select "Haggard Library" | List shows only events at Haggard branch | |
| 4.2.3 | Select multiple branches | List shows events from all selected branches | |
| 4.2.4 | One branch selected | Dropdown label shows branch name | |
| 4.2.5 | Multiple branches selected | Dropdown shows "Branches" label with count badge | |

---

## 5. Age Range Dropdown (NEW + REGRESSION)

| # | Scenario | Expected result | Tag |
|---|---|---|---|
| 5.1 | Select "Toddlers (0–5)" | Shows Frisco/Plano events with age range overlapping 0–5; Play Frisco family events also shown | [R] for library filtering logic |
| 5.2 | Select "Kids (6–12)" | Shows events overlapping 6–12; Play Frisco family events also shown | [R] for library filtering logic |
| 5.3 | Select "Teens" | Shows events with age range overlapping 13–17; Play Frisco family events also shown | [R] for library filtering logic |
| 5.4 | Select multiple age chips | Shows events matching any selected age bucket (OR logic) | |
| 5.5 | Age filter active; Play Frisco event inferred as "family" | Event appears in results regardless of which age chip is selected | |
| 5.6 | Age filter active; Play Frisco event inferred as "adult" (kid_relevant:false) | Event does not appear in results | |
| 5.7 | Age filter active; Play Frisco event with low-confidence inference | Event does not appear in results | |
| 5.8 | Age filter active; Plano "All Ages" event | Event appears under all age chip selections | [R] |
| 5.9 | One age selected | Dropdown label shows age name | |
| 5.10 | Multiple ages selected | Dropdown shows "Age range" label with count badge | |

---

## 6. Event Detail View

### 6.1 Core detail behavior

| # | Scenario | Expected result | Tag |
|---|---|---|---|
| 6.1.1 | Click any event card (desktop) | Detail panel opens on right side showing title, date, time, location, description | [R] |
| 6.1.2 | Click any event card (mobile) | Full-screen detail overlay opens; back button visible | [R] |
| 6.1.3 | Tap back button on mobile detail | Overlay closes; event list visible | [R] |
| 6.1.4 | Free event detail | "Free admission" green pill shown | [R] |
| 6.1.5 | Paid event detail | Price displayed (e.g. "$95/week") amber pill shown | [R] |
| 6.1.6 | Registration required event | Yellow registration banner shown | [R] |
| 6.1.7 | Click close button (desktop) | Detail panel closes; welcome panel shown | [R] |

### 6.2 Supervision badge (Frisco Library only)

| # | Scenario | Expected result | Tag |
|---|---|---|---|
| 6.2.1 | Frisco Library event, age 0–9 | Red badge: "❌ Can kids be dropped off? No — adult must stay with child" | [R] |
| 6.2.2 | Frisco Library event, age 10+ | Blue badge: "🔵 Drop-off OK for ages 10+" | [R] |
| 6.2.3 | Frisco Library event, teens (13–17) | Green badge: "✅ Yes — teens 13+ may attend alone" | [R] |
| 6.2.4 | Plano Library event | No supervision badge shown | [R] |
| 6.2.5 | Play Frisco event | No supervision badge shown | [R] |

### 6.3 Age display in detail view (NEW)

| # | Scenario | Expected result | Tag |
|---|---|---|---|
| 6.3.1 | Frisco/Plano event with structured age | "👶 Suitable for: [age label]" shown in periwinkle text | |
| 6.3.2 | Play Frisco event with inferred age (high/medium confidence) | Blue `~ [Age] ✦` pill shown with "Estimated from event description · not confirmed by source" text immediately after | |
| 6.3.3 | Play Frisco event with low-confidence inferred age | No age shown in detail view | |

### 6.4 Actions

| # | Scenario | Expected result | Tag |
|---|---|---|---|
| 6.4.1 | Click "Add to Google Calendar" | Google Calendar link opens with event pre-filled | [R] |
| 6.4.2 | Click "Add to Apple Calendar" | ICS file downloaded | [R] |
| 6.4.3 | Click "Get directions" | Directions flow opens with venue address pre-filled | [R] |
| 6.4.4 | Click "Attending?" | Count increments; button state changes to "Attending" | [R] |
| 6.4.5 | Click "Attending" again | Count decrements; button returns to "Attending?" state | [R] |
| 6.4.6 | Reload page after marking attending | Attending state persists (localStorage) | [R] |

---

## 7. Play Frisco LLM Age Inference (NEW)

### 7.1 Inference accuracy — validated test events

| # | Event | Expected kid_relevant | Expected age_buckets | Expected confidence |
|---|---|---|---|---|
| 7.1.1 | Second Saturday: Sensational Soccer | true | family | high |
| 7.1.2 | Artist-Led Workshop: Painting Dreamscapes | true | teen | high |
| 7.1.3 | Walnut Wednesdays | true | family | high |
| 7.1.4 | History of Play 2026 | true | family | high |
| 7.1.5 | Fun Float Night | true | family | high |
| 7.1.6 | Play For All Sensory Swim | true | family | high |
| 7.1.7 | Calling All Heroes 2026 | true | family | high |
| 7.1.8 | Heritage How-To: Wands, Wizards & Cookies | true | family | high |

### 7.2 Inference behavior

| # | Scenario | Expected result |
|---|---|---|
| 7.2.1 | Adult-only event in Play Frisco feed (e.g. city board meeting) | kid_relevant:false; event excluded from all views |
| 7.2.2 | Inference returns confidence:low | age_buckets stored but not shown in UI; no age badge on card or detail |
| 7.2.3 | Claude API returns malformed JSON | Ingest logs error; event still stored without age data; no ingest failure |
| 7.2.4 | Claude API times out | Ingest logs error; event stored without age data; pipeline continues |
| 7.2.5 | Event already in DB with existing inference | Re-ingest does not re-call Claude API (cache at ingest time) |
| 7.2.6 | New Play Frisco event added to city calendar | Next ingest run calls inference only for the new event |

---

## 8. Error and Edge Case States

| # | Scenario | Expected result | Tag |
|---|---|---|---|
| 8.1 | All filters applied; no events match | Empty state shown: "No events match your filters" + "Clear filters" button | [R] |
| 8.2 | Clear filters button clicked | All filter selections reset; full event list restored | [R] |
| 8.3 | Play Frisco ingest fails; library ingest succeeds | UI shows library events normally; amber banner: "Unfortunately, Play Frisco events are temporarily unavailable" | [R] |
| 8.4 | All three ingests fail | UI shows appropriate error state; does not crash | [R] |
| 8.5 | Date range set with no events in that range | Empty state shown | [R] |
| 8.6 | Map toggle clicked | Map panel shows venue pins for all currently filtered events | [R] |

---

## Regression Test Suite Summary

Tests marked `[R]` above constitute the regression suite. Run before every deployment:

| Area | Test IDs |
|---|---|
| Ingest pipeline | 1.1.1–1.1.6, 1.2.1–1.2.6, 1.3.1–1.3.4, 1.4.1–1.4.3 |
| List view core | 2.1.1–2.1.8 |
| Date range default | 2.1.9 (after v1.1 deploy) |
| Age filtering logic | 5.1–5.3, 5.8 |
| Detail view core | 6.1.1–6.1.7 |
| Supervision badge | 6.2.1–6.2.5 |
| Calendar / actions | 6.4.1–6.4.6 |
| Error states | 8.1–8.6 |

*New functionality tests (no `[R]` tag) should be added to the regression suite once v1.1 is deployed and stable.*

---

*Last updated: July 2026. Hand to Claude Code with instruction: "Generate executable test code for each scenario against the real implementation. Start with the regression suite."*
