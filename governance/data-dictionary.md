# Data Dictionary

Glossary of every table and column used by the dashboard, with:
- **Description** — what the column is
- **Type** — data type
- **Used in** — which visual or measure uses it
- **Notes** — any caveats or derivation

---

## fact_loan_portfolio

**Grain:** One row per loan (100,000 rows).
**Role:** Central fact table — loan-level data, all default-related measures draw from it.

| Column | Type | Description | Used in | Notes |
|---|---|---|---|---|
| `ID` | string | Loan identifier | Scatter chart Details | Unique per row |
| `ORG_UPB` | float | Original unpaid principal balance | Total Portfolio UPB card, UPB Share, Expected Loss | EAD proxy |
| `ORG_LTV` | float | Original loan-to-value ratio | LTV Band chart, Scatter Y axis | % (0–100+) |
| `DTI` | float | Debt-to-income ratio | Scatter X axis | % (0–50+) |
| `FICO_BOR` | int | Borrower FICO at origination | fico_band derivation | 300–850 range |
| `AGE` | float | Loan age in months | age_band derivation | |
| `PROP_TYPE` | string | Property type | Property × Occupancy matrix | SF, PU, CO, MH, CP |
| `OCCU_STAT` | string | Occupancy status | Property × Occupancy matrix | P, S, I |
| `default` | int (0/1) | 1 = default, 0 = performing | `Default Rate`, `Total Loans` | |
| `default_flag` | string | 'Performing' / 'Default' | Scatter Legend | Derived from `default` |
| `fico_band` | string | FICO band | Default Rate by FICO Band, UPB Share, etc. | Low / Medium / High / Very High |
| `ltv_band` | string | LTV band | Default Rate by LTV Band | Low / Medium / High |
| `age_band` | string | Loan age band | Default Rate by Age Band | 0-12m, 12-24m, etc. |
| `pd_estimate` | float | Estimated probability of default | Average PD, Expected Loss | |
| `lgd_final` | float | Estimated loss given default | Expected Loss | Constructed from PD |
| `ead_final` | float | Estimated exposure at default | **NOT USED** | Not present in source — `ORG_UPB` used as proxy |
| `run_id` | string | Pipeline run identifier | Not shown | Reproducibility |
| `notebook_version` | string | Pipeline version | Not shown | Reproducibility |

---

## fact_scenario_analysis

**Grain:** One row per scenario (4 rows: Baseline, Adverse, Severely Adverse, Tail Risk).
**Role:** Scenario ECL and stress multipliers.

| Column | Type | Description | Used in | Notes |
|---|---|---|---|---|
| `Scenario` | string | Scenario name | ECL by Scenario chart, Scenario Detail Table | |
| `PD_Multiplier` | float | PD stress multiplier | Scenario Detail Table | Multiplies baseline PD (not summed) |
| `LGD_Multiplier` | float | LGD stress multiplier | Scenario Detail Table | Multiplies baseline LGD (not summed) |
| `ECL` | float | Scenario ECL | ECL by Scenario chart, `Total ECL` measure | |
| `Weight` | float | Scenario probability weight | Scenario Detail Table, `Weighted ECL` | Sums to 100% |
| `Weighted_ECL` | float | ECL × Weight | `Weighted ECL` card, Scenario Detail Table | |

---

## fact_capital_coverage

**Grain:** One row per scenario (4 rows).
**Role:** **Partially used.** Contains capital figures that are constant across scenarios — capital charts removed from Page 3.

| Column | Type | Description | Used in | Notes |
|---|---|---|---|---|
| `Scenario` | string | Scenario name | Not used (capital charts removed) | |
| `CET1_Ratio` | float | CET1 capital ratio | **NOT USED** | Constant across scenarios |
| `Capital_Buffer` | float | Capital buffer above minimum | **NOT USED** | Constant across scenarios |
| `CET1_Capital` | float | Absolute CET1 capital | **NOT USED** | Constant across scenarios |
| `Adequacy` | float | Capital adequacy metric | **NOT USED** | Constant across scenarios |
| `Capital_Impact_Ratio` | float | Capital impact ratio | **NOT USED** | Constant across scenarios |
| `ECL` | float | Scenario ECL | **NOT USED** | Duplicate of `fact_scenario_analysis[ECL]` |
| `Total Assets` | float | Assumed total assets | **NOT USED** | Measure derived from ECL/ECL_Pct_of_Assets — likely broken; to be deleted |

---

## fact_ifrs9_stages

**Grain:** One row per IFRS 9 stage (3 rows: Stage 1, Stage 2, Stage 3).
**Role:** IFRS 9 staging distribution.

| Column | Type | Description | Used in | Notes |
|---|---|---|---|---|
| `Stage` | string | Stage 1 / Stage 2 / Stage 3 | IFRS 9 Donut | |
| `Percentage` | float | Proportion of portfolio | IFRS 9 Donut | Sums to 100% |
| `Count` | int | Number of loans | IFRS 9 Donut values | |

---

## fact_segment_analysis

**Grain:** One row per (Segment_Type, Category).
**Role:** **NOT USED** in v1.0 — segment analysis is done via `fact_loan_portfolio` and grouping.

| Column | Type | Description | Used in | Notes |
|---|---|---|---|---|
| `Segment_Type` | string | FICO / LTV / Property Type | Not used | Could be used for a segment summary page |
| `Category` | string | Segment value | Not used | |
| `Default_Rate` | float | Default rate within segment | Not used | |
| `Count` | int | Loans in segment | Not used | |

---

## fact_fairness_testing

**Grain:** One row per protected attribute or geographic proxy.
**Role:** **NOT USED** in v1.0 — fairness is documented as out-of-scope in Limitations.

| Column | Type | Description | Used in | Notes |
|---|---|---|---|---|
| `Attribute` | string | Protected attribute or geographic proxy | Not used | |
| `Cardinality` | int | Number of groups | Not used | |
| `DI_Ratio` | float | Disparate impact ratio | Not used | |
| `Pass_4_5ths` | bool | DI ≥ 0.80 | Not used | |
| `Interpretation` | string | 'UNRELIABLE' if cardinality > 50 | Not used | |

---

## summary_metrics

**Grain:** Unspecified — used for dashboard KPI display.
**Role:** **NOT USED** — individual metrics are computed via measures from `fact_loan_portfolio`.

---

## dim_scenario

**Grain:** One row per scenario (4 rows).
**Role:** Dimension table for scenario slicing.

| Column | Type | Description | Used in | Notes |
|---|---|---|---|---|
| `Scenario` | string | Scenario name | ECL by Scenario chart, scenario slicing | |

**Relationship:** `dim_scenario[Scenario]` — 1:N — `fact_scenario_analysis[Scenario]`

---

## dim_stage

**Grain:** One row per stage (3 rows).
**Role:** Dimension table for stage slicing.

| Column | Type | Description | Used in | Notes |
|---|---|---|---|---|
| `Stage` | string | Stage name | IFRS 9 Donut (legend) | Rebuilt after accidental deletion |

**Relationship:** `dim_stage[Stage]` — 1:N — `fact_ifrs9_stages[Stage]`

---

## dim_segment, dim_date

**Role:** Dimension tables, loaded but not yet used in any visual on the current dashboard.
**Notes:** Could support future filtering or time-based views if the dashboard is extended.

---

## Measures (Not Columns)

The following DAX measures are defined (not columns):

| Measure | Table | DAX | Purpose |
|---|---|---|---|
| `Total Loans` | fact_loan_portfolio | `COUNTROWS(fact_loan_portfolio)` | Denominator for rates |
| `Default Rate` | fact_loan_portfolio | `DIVIDE(SUM([default]), [Total Loans])` | Portfolio and segment default rate |
| `UPB by FICO` | fact_loan_portfolio | `SUM(fact_loan_portfolio[ORG_UPB])` | Total UPB |
| `UPB Share` | fact_loan_portfolio | `DIVIDE([UPB by FICO], CALCULATE([UPB by FICO], ALL(fact_loan_portfolio)))` | Dollar concentration |
| `Expected Loss` | fact_loan_portfolio | `SUMX(fact_loan_portfolio, [pd_estimate] * [lgd_final] * [ORG_UPB])` | PD × LGD × EAD (proxy) |
| `Expected Loss Rate` | fact_loan_portfolio | `DIVIDE([Expected Loss], SUM([ORG_UPB]))` | Loss per dollar |
| `Average PD` | fact_loan_portfolio | `AVERAGE(fact_loan_portfolio[pd_estimate])` | Mean PD per segment |
| `Total ECL` | fact_scenario_analysis | `SUM(fact_scenario_analysis[ECL])` | Total ECL per scenario |
| `Weighted ECL` | fact_scenario_analysis | `SUMX(fact_scenario_analysis, [ECL] * [Weight])` | Probability-weighted ECL |

---

## Derivation Notes

### fico_band
**Derived from:** `FICO_BOR`
**Rule:** Four bands — Low, Medium, High, Very High (thresholds defined in the source pipeline).
**Note:** Derived columns are not present in the raw CSV; the pipeline that produced `fact_loan_portfolio` added them.

### ltv_band
**Derived from:** `ORG_LTV`
**Rule:** Three bands — Low, Medium, High (thresholds defined in the source pipeline).

### age_band
**Derived from:** `AGE`
**Rule:** Bands in 12-month increments (0-12m, 12-24m, etc.).

### default_flag
**Derived from:** `default`
**Rule:** `default = 1` → "Default"; `default = 0` → "Performing".

---

## Change Log

| Date | Change | Author |
|---|---|---|
| [YYYY-MM-DD] | Initial data dictionary | [your name] |
