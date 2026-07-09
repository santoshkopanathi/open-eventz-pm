# Feature Spec: City-First Navigation + Age Indicators + Play Frisco LLM Inference

## Context

Open Eventz currently shows events from three sources in a single unified list, filtered by a source dropdown at the top. This spec replaces that with a city-first navigation model and adds age indicators at the event card level. It also introduces LLM-based age inference for Play Frisco events where no structured age data exists.

Read the full spec before writing any code. Implement in the order listed — each section builds on the previous.

---

## 1. City-First Tab Navigation

### What to build

Replace the current source filter dropdown ("Frisco Library ▼") with two persistent city tabs at the top of the page, directly below the header:

- **Frisco City** (default active tab)
- **Plano City**

These are navigation tabs, not filter chips. One is always active. Switching tabs reloads the event list for that city only.

### Frisco City tab

When active, shows:
- Events from: Frisco Public Library + Play Frisco
- Sub-filters (shown below the city tabs, above the event list):
  - **Source sub-filter:** "Frisco Library" | "Play Frisco" | "All Frisco" (default)
  - **Age filter:** Toddlers (0–5) | Kids (6–12) | Teens — applicable to Frisco Library events; behavior for Play Frisco described in Section 4
  - **Date range:** from/to date pickers, pre-populated on load to today through 7 days from today. User can adjust freely; clearing resets to the default 7-day window.

### Plano City tab

When active, shows:
- Events from: all Plano Libraries branches (Davis, Schimelpfenig, Parr, Haggard, Harrington)
- Sub-filters:
  - **Branch sub-filter:** individual library branches as options + "All Plano Libraries" (default)
  - **Age filter:** Toddlers (0–5) | Kids (6–12) | Teens — applicable to all Plano branches (Communico structured data supports this)
  - **Date range:** from/to date pickers, pre-populated on load to today through 7 days from today (same default as Frisco tab)

### Tab visual treatment

- Active tab: distinct accent color per city, white text on the active label, no border on inactive tabs
- **Frisco and Plano tabs use distinct accent colors that carry through from the tab underline into the filter bar, so users have an immediate visual signal of which city context they're in when switching tabs.**
- Tabs sit in a bar directly below the header, full width
- Sub-filters appear below the tab bar in a secondary filter row
- Clear filters only clears sub-filters within the active tab — switching tabs resets to default state for that tab

### What to remove

Remove the existing source dropdown filter ("Frisco Library ▼"). The city tabs replace it entirely.

---

## 2. Age Indicator on Event Cards (List View)

### What to build

Add an age badge to every event card in the list view where age data exists. Currently age information only appears in the detail view ("Suitable for: Children (0–5)"). It needs to also appear on the card so parents can scan the list without opening each event.

### Badge placement

On the event card, add the age badge in the same row as the existing "Reg." and "Free" badges (right side of the card, after those badges).

### Badge content and styling

**Frisco Library events (BiblioCommons structured "Suitable for:" field):**
- Show the age group directly: "Ages 0–5", "Ages 6–12", "Teens"
- Badge style: muted gold background (#F5F0DE), dark amber text (#8A7840)
- No disclosure indicator — this is structured, sourced data
- Events tagged for multiple age groups (e.g. Family Story Time) show "Family" as the badge label

**Plano Library events (Communico AGE GROUP structured field):**
- Map Communico values to display labels: Babies/Toddlers/Preschoolers → "Ages 0–5", Kids → "Ages 6–12", Teens → "Ages 13–17"
- Events with an "All Ages" or multi-group Communico value → show "Family" badge label
- Same badge style as Frisco Library — structured data, no disclosure needed

**Important filtering behavior for family-tagged events (all sources):**
There is no "All Ages" filter chip in the UI. Instead, events internally tagged as "family" (either from structured source data or LLM inference) appear in results when ANY age chip is selected — Toddlers, Kids, or Teens. A parent filtering for Kids (6–12) will still see Family Story Time because it is relevant to that age group. This is a behind-the-scenes filtering logic decision, not a visible UI element.

**Play Frisco events (LLM-inferred — see Section 4):**
- Show: "~ Ages 5–12" or "~ Family" etc. with the tilde prefix
- Badge style: lighter indigo background (#EEEDF5), indigo text (#2D3561) — visually distinct from library badges
- Small sparkle icon (✦) after the tilde, before the age text, on hover/tap shows tooltip: "Age range estimated from event description · not confirmed by source"

**No age data available:**
- Show nothing — omit the badge entirely rather than showing "Unknown"

---

## 3. Age Filter Behavior Per Source

### Frisco Library

Age filter chips (Toddlers / Kids / Teens) filter against the BiblioCommons "Suitable for:" structured field. This is already working — no changes needed to the underlying logic. Confirm the filter correctly excludes events with no age data when a filter chip is active.

### Plano Libraries

Age filter chips filter against Communico's AGE GROUP field using overlap matching (an event tagged "All Ages" appears under every age chip; an event tagged "Kids" appears only under Kids 6–12). This is already implemented — confirm it works correctly across all five branches.

### Play Frisco (LLM-inferred)

When an age filter chip is active under the Frisco City tab:
- Play Frisco events with a high or medium confidence LLM inference that matches the selected age bucket → shown
- Play Frisco events tagged as "family" by the LLM → shown under ALL age filter selections (family events are relevant to every age group)
- Play Frisco events with low confidence inference → shown under no age filter (only appear when no age chip is selected)
- Play Frisco events flagged as not kid-relevant (kid_relevant: false) → never shown in any age filter, and excluded from the default all-events view entirely

---

## 4. Play Frisco LLM Age Inference

### Overview

Play Frisco (CivicPlus) has no structured age data. The only signal is in event titles and descriptions. Use the Claude API (model: `claude-sonnet-4-6`) to infer age relevance and age suitability at cache/scrape time — not at page load time. Store the result with the event record.

**Model rationale:** Sonnet over Haiku. First-run cost difference is ~$0.03 ($0.05 vs $0.08 for ~80 events); ongoing cost is sub-cent on either model. At this cost profile, accuracy is the deciding factor. Sonnet's confidence tier classification is meaningfully more reliable on ambiguous descriptions, and the confidence tier is load-bearing — it determines whether an age badge appears in the UI at all.

### API route

Create `/api/infer-age` as a POST endpoint accepting `{ title, description }`.

### System prompt

```
You are a children's activity classification assistant. Your job is to analyze the title and description of a community event and determine:
1. Whether the event is genuinely relevant for children (vs. general public or adult-focused)
2. If child-relevant, what age range it is most suitable for

You must respond only with a valid JSON object. No preamble, no explanation, no markdown.

Age range buckets to use:
- "toddler" = ages 0-5
- "kids" = ages 6-12
- "teen" = ages 13-17
- "family" = all ages welcome, mixed child/adult participation
- "adult" = not suitable for children as primary audience

Confidence levels:
- "high" = age range explicitly stated or strongly implied by activity type
- "medium" = reasonably inferable from context but not explicit
- "low" = best guess only, very little signal in the text

Response format (JSON only):
{
  "kid_relevant": true | false,
  "age_buckets": ["toddler" | "kids" | "teen" | "family"],
  "confidence": "high" | "medium" | "low",
  "reasoning": "one sentence explanation"
}

If kid_relevant is false, age_buckets should be an empty array.
If confidence is "low", treat kid_relevant as uncertain.
```

### When to call it

Call `/api/infer-age` once per Play Frisco event at scrape/cache time. Store `kid_relevant`, `age_buckets`, `confidence`, and `reasoning` in the cached event record. Do not call the Claude API per user page load.

### Response handling

| Result | Action |
|---|---|
| kid_relevant: true, confidence: high or medium | Show event; surface inferred age badge with ~ prefix |
| kid_relevant: true, confidence: low | Show event; omit age badge |
| kid_relevant: false | Hide event from all views |
| Parse error / API failure | Show event; omit age badge; log error silently |

### Validated test cases

Test against these 8 real Play Frisco events before shipping:

| Event | Expected kid_relevant | Expected age_buckets | Expected confidence |
|---|---|---|---|
| Second Saturday: Sensational Soccer | true | family | high |
| Artist-Led Workshop: Painting Dreamscapes | true | teen | high (age 16+ explicitly stated) |
| Walnut Wednesdays | true | toddler, kids | high |
| History of Play 2026 | true | family | high |
| Fun Float Night | true | family | medium |
| Play For All Sensory Swim | true | family | high (inclusive for sensory sensitivities — still family-relevant) |
| Calling All Heroes 2026 | true | family | medium (military/first responder focus — still family-relevant) |
| Heritage How-To: Wands, Wizards & Cookies | true | kids, family | high |

---

## 5. Family Events Handling

Play Frisco events where `age_buckets` includes "family":
- Appear in the default all-events view for the Frisco City tab
- Appear under ALL age filter selections (Toddlers, Kids, Teens) — family events are relevant to every parent regardless of their child's age
- Show a "Family event" label in the detail view alongside the inferred age badge
- Are never excluded by age filtering

---

## 6. Detail View Updates

Two small additions to the existing detail view (no structural changes):

1. **Age badge in detail view** — add the same age badge treatment from the list view (Section 2) to the detail view header area, next to "Free admission" and registration badges. Currently "Suitable for: Children (0–5)" is shown as a text line — replace or supplement it with the styled badge.

2. **Play Frisco inference disclosure** — for Play Frisco events with LLM-inferred age data, add a sub-label below the age badge: "Age range estimated from event description · not confirmed by source" in small muted text. This only appears in the detail view, not on the list card.

---

## 7. Recurring Indicator on Event Cards and Detail View

### Context

The data model includes a `recurring` boolean field populated at ingest time. Recurring vs. one-time is meaningful information for parents — a recurring weekly storytime can be built into a routine; a one-time event requires a specific commitment. This field is not currently surfaced in the UI at either the card or detail level.

### What to build

**On the event card (list view):**
Add a small recurring indicator badge alongside the existing Free/Paid and age badges:
- Recurring events only: `↻ Recurring` — neutral gray badge (same treatment as the age badge border style)
- One-time events: no badge — omit entirely
- If the recurring field is false, null, or undefined, show nothing

**On the event detail view:**
Add the recurring badge to the detail chip row (same row as Free/Paid, age, and registration badges). No additional explanation needed — the label is self-describing.

### Badge styling

Consistent with the existing neutral badge treatment:
- Background: `var(--bg)` (light off-white)
- Border: `1px solid var(--border)`
- Text: `var(--text-2)` (muted)
- Font size: 11px, same as other detail chips

### Data source per origin

- **Frisco Library (BiblioCommons):** Recurring is derivable from the event listing — recurring programs appear on multiple dates with the same title and series ID
- **Plano Libraries (Communico):** Communico's feed includes explicit recurring grouping — events in a series share a parent ID
- **Play Frisco (CivicPlus):** Recurring must be inferred from scrape — events appearing on multiple future dates with the same title are treated as recurring

---

## 8. Implementation Order

Build in this sequence to avoid regressions:

1. City-first tab navigation (Section 1) — structural change, do this first
2. Age indicator on event cards (Section 2) — additive, no logic changes
3. Verify age filter behavior per source (Section 3) — confirm existing logic works correctly under new tab structure
4. Play Frisco LLM inference (Section 4) — new API route + cache integration
5. Family event handling (Section 5) — filter logic addition
6. Detail view updates (Section 6) — final polish

---

## 9. What NOT to change

- Header design and Dusk palette — keep exactly as is
- Map view and directions functionality — keep exactly as is
- Drop-off badge treatment ("Can kids be dropped off?") — keep exactly as is
- Calendar add, share, attending functionality — keep exactly as is
- Event card design and layout — add age badge only, no other changes

---

## 10. Portfolio Note

This spec represents two distinct AI PM judgment calls worth referencing in the case study:

**City-first navigation:** Identified that source-based filtering created cognitive overload (five Plano library branches visible simultaneously), and restructured the information architecture around how parents actually think — by city, not by data source.

**LLM inference with honest disclosure:** Moved from deterministic to inferred data for Play Frisco age ranges, with explicit confidence tiers, a visual distinction between stated and inferred data (~ prefix), and graceful degradation (low confidence = show nothing). This mirrors the supervision policy "never guess" principle already in the product — applying the same trust framework to a new data quality problem.
