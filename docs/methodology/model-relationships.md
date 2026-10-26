# Model Relationships

The star-schema structure of the data model, with grain definitions and
relationship rationale.

---

## Star schema diagram
┌─────────────────┐
│ dim_scenario │
│ Scenario │
└────────┬────────┘
│ 1:N
▼
┌──────────────────┐ ┌─────────────────────┐ ┌──────────────────┐
│ dim_stage │ │ fact_scenario_ │ │ fact_capital_ │
│ Stage │ │ analysis │ │ coverage │
└────────┬─────────┘ │ Scenario │ │ Scenario │
│ 1:N │ PD_Multiplier │ │ CET1_Ratio │
│ │ LGD_Multiplier │ │ ... │
▼ │ ECL │ └──────────────────┘
┌──────────────────┐ │ Weight │
│ fact_ifrs9_ │ │ Weighted_ECL │
│ stages │ └─────────────────────┘
│ Stage │
│ Percentage │ ┌─────────────────────┐
│ Count │ │ fact_loan_portfolio │
└──────────────────┘ │ default │
│ ORG_UPB │
┌──────────────────┐ │ FICO_BOR │
│ dim_segment │ │ ... │
│ Segment_Type │ └─────────────────────┘
└──────────────────┘ 100,000 rows

┌──────────────────┐
│ dim_date │
└──────────────────┘


---

## Grain definitions

| Table | Grain | Notes |
|---|---|---|
| `fact_loan_portfolio` | One row per loan | Central fact table |
| `fact_scenario_analysis` | One row per scenario | 4 rows |
| `fact_capital_coverage` | One row per scenario | 4 rows |
| `fact_ifrs9_stages` | One row per stage | 3 rows |
| `fact_segment_analysis` | One row per (Segment_Type, Category) | Pre-aggregated |
| `fact_fairness_testing` | One row per attribute | Pre-aggregated |
| `summary_metrics` | Unspecified | Unused |
| `dim_scenario` | One row per scenario | 4 rows |
| `dim_stage` | One row per stage | 3 rows |
| `dim_segment` | One row per segment | Unused |
| `dim_date` | One row per date | Unused |

---

## Active relationships

### 1. `dim_scenario` → `fact_scenario_analysis`

| Property | Value |
|---|---|
| From (one side) | `dim_scenario[Scenario]` |
| To (many side) | `fact_scenario_analysis[Scenario]` |
| Cardinality | 1:N |
| Cross-filter direction | Single |
| Active | Yes |

**Why:** Allows slicing `fact_scenario_analysis` by `dim_scenario` attributes.
Currently both tables have the same 4 scenario names — the relationship is
future-proofing (if a scenario attribute table grows, slicing will still work).

### 2. `dim_stage` → `fact_ifrs9_stages`

| Property | Value |
|---|---|
| From (one side) | `dim_stage[Stage]` |
| To (many side) | `fact_ifrs9_stages[Stage]` |
| Cardinality | 1:N |
| Cross-filter direction | Single |
| Active | Yes |

**Why:** Allows slicing `fact_ifrs9_stages` by `dim_stage` attributes. Same
future-proofing rationale as above.

---

## Relationships NOT built (and why)

### `dim_scenario` → `fact_capital_coverage`

**Why not built:** `dim_scenario[Scenario]` already has an active relationship
to `fact_scenario_analysis`. Adding a second active relationship to
`fact_capital_coverage` would create ambiguity in the filter context — Power BI
would not know which path to use when both tables are referenced in one visual.

**What we do instead:** Group by `fact_capital_coverage[Scenario]` directly.
The capital charts were ultimately removed (constant capital figures), so this
relationship is now moot.

### `dim_segment` → `fact_loan_portfolio`

**Why not built:** `dim_segment` was loaded but not used in v1.0. If it were
used, the natural relationship would be `dim_segment[Category]` to a matching
column in `fact_loan_portfolio` — but no such column exists yet (segment
categories are derived from FICO/LTV bands, not a separate key).

### `dim_date` → `fact_loan_portfolio`

**Why not built:** No date column in `fact_loan_portfolio` that maps cleanly
to `dim_date`. The sample is not time-series-analysed in v1.0.

---

## Relationship conventions

1. **Single direction only.** All relationships are "one to many, single
   direction" — the dimension filters the fact, never the reverse. This avoids
   ambiguous filter paths and performance issues.
2. **No inactive relationships.** If a relationship is needed, it is active
   and documented here.
3. **No many-to-many relationships.** None of the tables require them at the
   current grain.
4. **Keys match exactly.** Relationship keys are character-for-character
   matched (verified when building — mismatched spellings are the most common
   cause of failed relationships).

---

## Model size and performance

| Table | Rows | Notes |
|---|---|---|
| `fact_loan_portfolio` | 100,000 | Largest table |
| `fact_segment_analysis` | ~15 | Tiny |
| `fact_scenario_analysis` | 4 | Tiny |
| `fact_capital_coverage` | 4 | Tiny |
| `fact_ifrs9_stages` | 3 | Tiny |
| `fact_fairness_testing` | ~5 | Tiny |
| `summary_metrics` | ~10 | Tiny |
| `dim_scenario` | 4 | Tiny |
| `dim_stage` | 3 | Tiny |
| `dim_segment` | ~10 | Tiny |
| `dim_date` | ~1,000 | Small |

**Performance:** 100,000 rows is small for Power BI. All measures complete in
<1 second interactively. The scatter chart with `LoanID` in Details may sample
100,000 points — this is a rendering decision, not a model issue.

---

## Change Log

| Date | Change | Author |
|---|---|---|
| [YYYY-MM-DD] | Initial model documentation | [your name] |
