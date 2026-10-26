# Dashboard Design

The reasoning behind the dashboard's structure: page purposes, visual choices,
layout decisions, and design principles.

---

## Page-level structure

The dashboard has three analytical pages and one reference page:

| Order | Tab | Purpose |
|---|---|---|
| 1 | Concentration | Where is default risk concentrated? |
| 2 | Drivers | Why are defaults happening? |
| 3 | Stress & IFRS 9 | How does the portfolio hold up under stress? |
| 4 | Limitations | What are the data caveats? |

**Tab order follows the reader's question flow:** start with where, then why,
then under stress, then the caveats. Limitations last because it's reference
material, not narrative.

**Tab names are short and parallel** — scannable in a tab bar, not descriptive
sentences.

---

## Design principles

These principles guided every visual and layout choice:

### 1. Every visual answers a specific question
The [Question Map](../questions/question-map.md) records which visual answers
which question. If a visual didn't answer a question, it wasn't built.

### 2. Rate ≠ count
A segment can be small in count but large in dollars. Every rate view is
paired with a dollar view where relevant (e.g., FICO default rate + UPB share).

### 3. Charts for patterns, tables for values
Bar charts and matrices show patterns; the scenario detail table shows exact
values for cross-checking. Readers get both reading modes.

### 4. Caveats live next to the numbers they qualify
Page 2's EAD proxy caveat is on Page 2, not in a distant Limitations page.
Page 3's "indicative only" caveat is on Page 3. **A caveat far from its number
is effectively hidden.**

### 5. Blank ≠ zero
Blank cells in matrices mean "no loans in that combination," not "zero default
rate." This is explained in a note below each matrix.

### 6. Sample-based numbers are always labelled
Every page footer states "Based on 100,000-loan sample." A reader can't reach
a chart without passing a footer.

### 7. US proxy is stated, not hidden
Every page footer states "US proxy for Canadian portfolio." A Canadian reader
cannot mistake these numbers for Canadian portfolio data.

### 8. Deletions are honest, not hidden
Capital adequacy charts were removed because the data can't support them.
This is documented in the Limitations page, not quietly dropped.

---

## Page 1 — Concentration

### Purpose
Answer "where is risk concentrated?" in under 10 seconds.

### Layout

┌──────────────────────────────────────────────────────────┐
│ TITLE: Where is default risk concentrated? │
│ Subtitle │
├────────────────────────────────────────────┬─────────────┤
│ Default Rate by FICO │ UPB Share by FICO│ Cards: │
│ │ │ Total Loans│
├────────────────────────┼────────────────────┤ Total UPB │
│ Default Rate by LTV │ Default Rate by │ Overall DR │
│ │ Age Band │ │
├────────────────────────────────────────────┴─────────────┤
│ MATRIX: Property Type × Occupancy │
│ Note: blank = no loans │
├──────────────────────────────────────────────────────────┤
│ FOOTER: caveats + See Limitations page │
└──────────────────────────────────────────────────────────┘

### Why this layout
- **Cards top-right:** the eye lands on the title, then the headline numbers.
  Cards give the baseline before the reader dives into charts.
- **Charts in 2×2:** gives each chart enough space for readability while
  keeping the page scannable in one screen.
- **Matrix at the bottom:** it's the structural detail view — readers who
  want it scroll down; readers who don't have already seen the headlines.
- **Footer at the bottom:** caveats are the last thing the eye sees before
  leaving the page.

---

## Page 2 — Drivers

### Purpose
Answer "why are defaults happening?" — explain the patterns Page 1 found.

### Layout
┌──────────────────────────────────────────────────────────┐
│ TITLE: Why are loans defaulting? │
│ Subtitle │
├──────────────────────────────────────────────────────────┤
│ FICO × LTV heatmap │ DTI vs LTV scatter │
├──────────────────────────────────────────────────────────┤
│ Expected Loss ($) │ Expected Loss Rate (%) │
├──────────────────────────────────────────────────────────┤
│ PD vs. Realized Default │
├──────────────────────────────────────────────────────────┤
│ CAVEAT text box │
│ FOOTER │
└──────────────────────────────────────────────────────────┘

### Why this layout
- **Heatmap and scatter first:** these are the "diagnostic" visuals — they
  show *why* the risk is concentrated, not just *where*.
- **Expected Loss pair in the middle:** moves from "what the risk looks like"
  to "what it costs."
- **PD vs. Realized at the bottom:** the model check is the last analytical
  step — it asks *"can we trust the model's answer?"*
- **Caveat full-width:** the page has multiple caveats (EAD proxy, LGD
  circularity, sample basis) — they need space to be readable.

---

## Page 3 — Stress & IFRS 9

### Purpose
Answer "how does the portfolio hold up under stress and staging?" — put
today's concentration into forward-looking context.

### Layout
┌──────────────────────────────────────────────────────────┐
│ TITLE: How does the portfolio hold up under stress? │
│ Subtitle │
├──────────────────────────────────────────────────────────┤
│ ECL by Scenario │ Weighted ECL card │
├──────────────────────────────────────────────────────────┤
│ Scenario Detail Table │ IFRS 9 Stage Donut │
├──────────────────────────────────────────────────────────┤
│ CAVEAT text box │
│ FOOTER │
└──────────────────────────────────────────────────────────┘

### Why this layout
- **ECL chart + Weighted ECL card top:** the stress story in one glance —
  range of scenarios plus the planning number.
- **Detail table + donut below:** the receipt (per-scenario numbers) and the
  staging burden side by side.
- **Capital charts removed:** `fact_capital_coverage` has constant capital
  across scenarios. Showing them would imply capital varies by scenario,
  which the data can't support.

---

## Visual design conventions

| Element | Convention |
|---|---|
| Titles | Sentence-case, question-form for page titles, label-form for chart titles |
| Cards | Single value, no conditional formatting (no threshold judgment implied) |
| Bar charts | Horizontal for ranked categories; data labels always on |
| Matrices | Conditional formatting (white → red gradient) on rate cells |
| Scatter | Grey for performing, red for default (visual hierarchy for the signal) |
| Donut | 3 categories max — Stage 1/2/3 fits perfectly |
| Footer | Small font, grey, on every page |
| Caveats | Small font, distinct background, on every page with a proxy-based metric |

---

## Colour choices

- **No green** — green implies "good" or "safe," which a risk dashboard cannot
  assert. Rates and losses are neutral; the reader interprets.
- **Red only in the gradient** — the matrix gradient uses red as "highest"
  within each matrix, a relative comparison, not an absolute judgment.
- **Grey for the scatter background** — performing loans recede; defaults pop.
  This is the "make the signal loud, the noise quiet" principle.

---

## Drill-through and interactivity

**Built:**
- Cross-filtering between visuals on the same page (default Power BI behaviour).

**Not built:**
- Drill-through to loan level (deferred to v1.1).
- Slicers for interactive filtering (considered; not needed for a first
  presentation).

**Why no slicers in v1.0:** the dashboard is designed to be read as a fixed
presentation, not a live exploration tool. Interactive filtering adds complexity
without adding to the narrative. If a live interactive version is needed later,
slicers on `fico_band`, `ltv_band`, and `Scenario` would be the first additions.

---

## Accessibility notes

- **Font sizes:** minimum 10pt for body text, 28–32pt for card values.
- **Contrast:** dark grey on white throughout; the red gradient uses enough
  contrast to be distinguishable.
- **Colour is not the only encoding:** bar charts use position; matrices use
  numbers plus colour; the scatter uses colour but its purpose is visual
  clustering, not individual value reading.

---

## Change Log

| Date | Change | Author |
|---|---|---|
| [YYYY-MM-DD] | Initial design documentation | [your name] |
