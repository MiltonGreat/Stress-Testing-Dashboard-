# Data Preparation

How the data was sourced, loaded, modelled, and transformed for the dashboard.

---

## 1. Source

**Dataset:** Fannie Mae Single-Family Loan Performance Data (public).
**Sample:** 100,000-loan random sample.
**Format:** CSV.
**Access:** Publicly available from Fannie Mae's website. The raw CSV is **not**
included in this repository (size + licensing); the pipeline that produced the
sample is documented below.

**Why Fannie Mae:** It is the largest publicly available source of US
single-family loan performance data, with loan-level origination and
performance fields. It is a widely used proxy for credit-risk modelling
education and portfolio analysis.

---

## 2. Loading into Power BI

**Tool:** Power BI Desktop → Get Data → Text/CSV.
**Tables loaded:**

| Source file | Table name in model | Rows |
|---|---|---|
| fact_loan_portfolio.csv | fact_loan_portfolio | 100,000 |
| fact_segment_analysis.csv | fact_segment_analysis | ~15 |
| fact_scenario_analysis.csv | fact_scenario_analysis | 4 |
| fact_capital_coverage.csv | fact_capital_coverage | 4 |
| fact_ifrs9_stages.csv | fact_ifrs9_stages | 3 |
| fact_fairness_testing.csv | fact_fairness_testing | ~5 |
| summary_metrics.csv | summary_metrics | ~10 |
| dim_scenario.csv | dim_scenario | 4 |
| dim_stage.csv | dim_stage | 3 |
| dim_segment.csv | dim_segment | ~10 |
| dim_date.csv | dim_date | ~1,000 |

**Loading options:** Data was loaded with **"Transform Data"** enabled, so
Power Query could verify column types before loading. No transformations were
applied in Power Query — the CSV was assumed clean (documented in project brief).

---

## 3. Column typing

All columns were typed in Power Query based on the source schema:

- **Numeric:** `ORG_UPB`, `ORG_LTV`, `DTI`, `FICO_BOR`, `AGE`, `pd_estimate`,
  `lgd_final`, `ECL`, `Weight`, `Weighted_ECL`, `CET1_Ratio`, `Capital_Buffer`,
  `CET1_Capital`, `Adequacy`, `Capital_Impact_Ratio`, `Percentage`, `Count`.
- **Text:** `Scenario`, `Stage`, `Segment_Type`, `Category`, `PROP_TYPE`,
  `OCCU_STAT`, `default_flag`, `fico_band`, `ltv_band`, `age_band`,
  `run_id`, `notebook_version`.
- **Integer:** `default`, `Cardinality`, `Pass_4_5ths`, `Total_Assets`.

**Note:** `default` is stored as 0/1 integer — treated as numeric for `SUM`,
not as boolean.

---

## 4. Derived columns (from the source pipeline)

The following columns are **not present in the raw CSV** — they were added by
the source pipeline that produced the dataset:

- `fico_band` — four bands (Low / Medium / High / Very High)
- `ltv_band` — three bands (Low / Medium / High)
- `age_band` — 12-month bands (0-12m, 12-24m, etc.)
- `default_flag` — 'Performing' / 'Default'
- `pd_estimate` — estimated probability of default
- `lgd_final` — loss given default (constructed from PD)
- `run_id`, `notebook_version` — pipeline identifiers

**Why they were not rebuilt in Power BI:** The brief documented them as
already-derived. Rebuilding them in DAX would duplicate work and risk
inconsistency. The dashboard trusts the source pipeline's derivation and
documents the assumptions in `governance/assumptions.md`.

---

## 5. Missing columns and their proxies

| Missing column | Proxy used | Where |
|---|---|---|
| `ead_final` | `ORG_UPB` | Expected Loss measure |

`ead_final` is not present in the source. See `governance/limitations.md` §3
for the impact.

---

## 6. Duplicate or unused tables

**Not used in v1.0:**

- `fact_segment_analysis` — segment analysis done via `fact_loan_portfolio` + grouping
- `fact_fairness_testing` — fairness documented as out-of-scope
- `summary_metrics` — individual metrics computed via DAX measures
- `dim_segment`, `dim_date` — loaded but not used in current visuals

**Why they're still in the model:** They're part of the source dataset's
structure, and future versions may use them. Keeping them preserves the
option to extend the dashboard without a re-load.

---

## 7. Data refreshes

The data is a **static sample** — not refreshed. No scheduled refresh is
configured. The dashboard reflects the sample as loaded.

**To refresh:**
1. Re-download the sample CSV from Fannie Mae.
2. Update the file path in Power Query.
3. Refresh in Power BI Desktop.

---

## 8. Reproducibility

To reproduce the dataset:

1. Download the Fannie Mae Single-Family Loan Performance sample.
2. Run the source pipeline (referenced by `run_id` and `notebook_version`
   in the data) — this pipeline produces the derived columns.
3. Load the output CSV into Power BI using the same table names.

**Note:** The source pipeline is not included in this repository. Only the
final `.pbix` and the governance documentation are.

---

## Change Log

| Date | Change | Author |
|---|---|---|
| [YYYY-MM-DD] | Initial documentation | [your name] |
