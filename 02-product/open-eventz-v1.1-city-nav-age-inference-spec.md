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
  - **Source dropdown:** multi-select dropdown with "Frisco Library" and "Play Frisco" checkboxes (none selected = all Frisco). One selected shows that source's name; multiple selected show a "Sources" label with a count badge.
  - **Age dropdown:** multi-select dropdown — Toddlers (0–5), Kids (6–12), Teens. Multiple selections use OR logic (an event whose range overlaps any selected group is shown). One selected shows the age name; multiple show an "Age range" label with a count badge. Applicable to Frisco Library events; behavior for Play Frisco described in Section 4.
  - **Date range:** from/to date pickers, pre-populated on load to today through 7 days from today. User can adjust freely; clearing resets to the default 7-day window.

### Plano City tab

When active, shows:
- Events from: all Plano Libraries branches (Davis, Schimelpfenig, Parr, Haggard, Harrington)
- Sub-filters:
  - **Branch dropdown:** multi-select dropdown of the individual library branches (Davis, Haggard, Harrington, Parr, Schimelpfenig, and Virtual); none selected = all Plano Libraries. One selected shows the branch name; multiple show a "Branches" label with a count badge.
  - **Age dropdown:** multi-select dropdown — Toddlers (0–5), Kids (6–12), Teens (OR logic, same as the Frisco tab); applicable to all Plano branches (Communico structured data supports this)
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

**Structured age badges (Frisco Library and Plano Library events):** Removed from event cards. Age information for library events is available in the detail view. The age filter chips at the top already communicate this when active — the badge is redundant.

**Plano Library "Families (All Ages)" events (Communico explicit tag):**
Show a "Family" badge on the card. This is the only structured age badge that remains on cards, because family events appear across all age filter selections and a parent needs to know at the scan level that this is an all-ages event. Never derived from a numeric multi-group span — only shown when Communico explicitly tags the event "Families (All Ages)".

**Play Frisco events — inferred family (high or medium confidence):**
Show `~ Family ✦` on the card. The `~` and `✦` always accompany the text label. Visual treatment should be distinct from the confirmed "Family" badge to signal that this is estimated, not source-confirmed.

**Play Frisco events — inferred specific age group (high or medium confidence):**
Show `✦` only — bare symbol, no tilde, no age text. The specific age range is disclosed in the detail view. Desktop: hover tooltip on `✦` reads "Estimated from description" (a single simplified string used on all cards; the full scenario-specific disclosure is detail-only — see §6). Mobile: no tooltip; tapping opens the detail view where the full disclosure appears.

**Play Frisco events — low confidence or kid_relevant: false:**
Show nothing.

**No age data:**
Show nothing.

**Free/Paid, Reg., and ↻ Recurring badges are unchanged — always shown on cards.**

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

**Badge simplification does not change filtering.** Filtering remains overlap-based on the underlying age range data. An all-ages/family event appears under every age chip. A multi-group event appears under each overlapping chip. Removing age badges from cards has no effect on which events appear under which filter.

**Plano ingest note:** The explicit "Families (All Ages)" Communico tag must be stored as its own signal (`age_buckets = ['family']`) — a 0–17 numeric range is not equivalent. The Communico parser excludes Adults and Older Adults from the kid-facing range, so "Kids + Adults" resolves to 6–12 only.

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

Classification rule (important):
- If the description EXPLICITLY states an age group or age range, tag ONLY that specific age group. Do not add "family" alongside it.
- If the description does NOT explicitly state an age group but is clearly relevant for families and children, tag as "family" only. Do not infer specific age groups from activity type alone.
- "Family" covers all ages — do not break it down further unless an age is explicitly mentioned.

Confidence levels:
- "high" = age range explicitly stated OR the event clearly uses family/child/youth language
- "medium" = reasonably inferable from context but no explicit family or age signal
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
| Artist-Led Workshop: Painting Dreamscapes | true | teen | high |
| Walnut Wednesdays | true | family | high |
| History of Play 2026 | true | family | high |
| Fun Float Night | true | family | high |
| Play For All Sensory Swim | true | family | high |
| Calling All Heroes 2026 | true | family | high |
| Heritage How-To: Wands, Wizards & Cookies | true | family | high |

---

## 5. Family Events Handling

Family events from both Plano Libraries (explicitly tagged "Families (All Ages)" in Communico) and Play Frisco (inferred as "family" by the LLM) share the same filtering behavior:
- Appear in the default all-events view
- Appear under ALL age filter selections (Toddlers, Kids, Teens) — family events are relevant to every parent regardless of their child's age
- Are never excluded by age filtering

The "Family event" text label previously specified for the detail view is removed — the `~ Family ✦` badge plus the inference disclosure line covers this for Play Frisco events. The confirmed "Family" badge covers it for Plano events. No separate redundant label needed.

---

## 6. Detail View Updates

**Age display in detail view — all sources:**

1. **Frisco Library and Plano Library — specific age group:** Show structured age label ("Ages 0–5", "Ages 6–12", "Teens"). Source-confirmed, no disclosure needed.

2. **Plano Library — "Families (All Ages)":** Show "Family" badge. Source-confirmed, no disclosure.

3. **Frisco/Plano — multi-group, not family:** Show a single collapsed range (e.g. "Ages 6–17"). Do not show "Family" for non-explicit multi-group events.

4. **Play Frisco — inferred family:** Show `~ Family ✦` badge. Detail disclosure: "Family suitability estimated from event description". (If price is also inferred, this merges into one combined line — see the v1.2 spec, Layer 4.)

5. **Play Frisco — inferred specific age:** Show `~ Ages [range] ✦` badge. Detail disclosure: "Age suitability estimated from event description".

6. **Low confidence or no age data:** Show nothing.

**Drop-off badge treatment ("Can kids be dropped off?") — keep exactly as is. No structural changes to the detail view.**

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
