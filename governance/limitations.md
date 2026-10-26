# Limitations

This document records every limitation of the IFRS 9 Stress Testing Dashboard.
Each limitation is listed with: **what it is**, **why it matters**, and
**how it should affect interpretation**.

Limitations are not appendices. They are part of the project's argument. A
dashboard without them is a chart; a dashboard with them is a decision tool.

---

## 1. Sample, not population

- **What:** All metrics are derived from a random sample of 100,000 Fannie Mae
  single-family loans, not the full population.
- **Why it matters:** Segment-level default rates carry sampling error. Small
  segments (e.g., rare property-type × occupancy combinations) may have rates
  that are unstable and not representative.
- **How to interpret:** Treat all numbers as **indicative of direction**, not
  precise. Relative comparisons across segments are more reliable than absolute
  levels. For segments with fewer than ~1,000 loans, treat rates with caution.

---

## 2. US data used as a Canadian portfolio proxy

- **What:** The dataset is Fannie Mae single-family loan performance (US). The
  intended audience is a Canadian bank portfolio.
- **Why it matters:** Property types, occupancy conventions, borrower behaviour,
  and regulatory frameworks differ between the two markets. A pattern observed
  in Fannie Mae data may not hold in a Canadian portfolio.
- **How to interpret:** Segment **rankings and patterns** (e.g., "Low FICO
  defaults more than High FICO") are indicative and likely transferable.
  **Absolute levels** and market-specific behaviours (e.g., default timing by
  province) are not. Do not present these numbers as a Canadian portfolio
  without recalculation against Canadian data.

---

## 3. EAD proxied by ORG_UPB

- **What:** True Exposure at Default (`ead_final`) is not present in the source
  data. Expected Loss uses `ORG_UPB` (Original Unpaid Principal Balance) as an
  EAD proxy.
- **Why it matters:** ORG_UPB is the **original** balance; EAD is the exposure
  **at the time of default**, after some amortisation. For seasoned loans
  (high `AGE`), ORG_UPB overstates EAD.
- **How to interpret:** Expected Loss figures are **directionally correct but
  overstated for seasoned loans**. Relative comparisons across FICO bands,
  LTV bands, and property types are still valid because the overstatement is
  roughly uniform across segments. Absolute loss forecasts are not valid
  without a true EAD calculation.

---

## 4. LGD constructed from PD (partly circular)

- **What:** `lgd_final` is constructed as a function of `pd_estimate` in the
  data pipeline, not empirically calibrated from observed recovery rates.
- **Why it matters:** PD × LGD is not a product of two independent estimates;
  it is partly a function of PD squared. This inflates the implied correlation
  between default probability and loss severity.
- **How to interpret:** Expected Loss is **directionally correct** — high-PD
  loans do carry more expected loss. But absolute Expected Loss should not be
  used for capital planning or provisioning without an independently
  calibrated LGD. Relative comparisons across segments remain valid.

---

## 5. SICR is a proxy rule, not true IFRS 9 SICR

- **What:** Stage 2 in the IFRS 9 donut is derived from a proxy rule, not the
  full IFRS 9 Significant Increase in Credit Risk assessment.
- **Why it matters:** True IFRS 9 Stage 2 classification uses lifetime PD
  comparisons, qualitative triggers, and 30-days-past-due presumptions. A proxy
  rule will misclassify some loans in both directions.
- **How to interpret:** The Stage 2 proportion is **indicative, not reportable**.
  Stage 1 and Stage 3 proportions are more reliable (Stage 1 = performing,
  Stage 3 = impaired/90+ DPD, both more objectively defined).

---

## 6. Capital adequacy not modelled

- **What:** `fact_capital_coverage` contains capital figures (`CET1_Ratio`,
  `Capital_Buffer`) that are **constant across all scenarios**. No meaningful
  scenario comparison of capital position is possible.
- **Why it matters:** A chart of CET1 Ratio by scenario would show four
  identical bars, falsely implying capital is stable under stress. The capital
  charts were **removed from Page 3** rather than shown with a caveat.
- **How to interpret:** The dashboard **does not assess capital adequacy**.
  Capital adequacy is a separate analysis requiring regulatory capital data,
  risk-weighted assets, and a stress-testing framework. Page 3's caveat states
  this explicitly.

---

## 7. Fairness testing not shown

- **What:** `fact_fairness_testing` includes Disparate Impact ratios for
  protected attributes and geographic proxies, but high-cardinality geographic
  attributes (more than 50 groups) are flagged `UNRELIABLE`.
- **Why it matters:** A DI ratio computed on a high-cardinality attribute can
  appear to pass or fail the 4/5ths rule purely from small-sample noise.
- **How to interpret:** Fairness is **not assessed on this dashboard**. It
  belongs in a separate compliance report using low-cardinality attributes
  and a larger sample. The decision to exclude it is deliberate.

---

## 8. Scenario weights are illustrative

- **What:** The scenario probabilities used in `Weighted ECL` are illustrative
  weights, not derived from a regulatory or bank-internal stress framework.
- **Why it matters:** The weighted ECL depends on the weights — different
  weights produce different planning numbers, without any change to the
  underlying ECL scenarios.
- **How to interpret:** `Weighted ECL` is a **planning number** under stated
  assumptions, not a forecast of expected loss. It is most useful as a
  comparison across weight schemes, not as an absolute number.

---

## 9. Model calibration check, not validation

- **What:** The "Model PD vs. Realized Default" chart on Page 2 compares
  `Average PD` against actual default rate per FICO band.
- **Why it matters:** A close match per band is **evidence of calibration**, but
  it is not model validation. Validation requires out-of-sample testing,
  discriminatory power analysis (Gini, KS), and stability testing over time.
- **How to interpret:** Use the chart as a **first-order check** — does the
  model over-predict or under-predict in specific bands? — not as a validation
  claim. A well-calibrated model at the segment level can still be poorly
  calibrated at the portfolio level.

---

## 10. Data lineage available, not exposed

- **What:** `run_id` and `notebook_version` columns are present in
  `fact_loan_portfolio`, recording the pipeline that produced the data.
- **Why it matters:** These enable reproducibility (you can trace which pipeline
  version produced a result) but are not shown on any dashboard page.
- **How to interpret:** For audit or governance purposes, these columns are
  available in the underlying `.pbix`. They are not shown in the dashboard
  itself to keep the pages focused on decision-relevant information.

---

## Summary Table

| # | Limitation | Severity | Where disclosed |
|---|---|---|---|
| 1 | Sample, not population | Medium | Every page footer |
| 2 | US proxy for Canadian portfolio | High | Every page footer |
| 3 | EAD proxied by ORG_UPB | Medium | Page 2 caveat, footer |
| 4 | LGD constructed from PD | High | Page 2 caveat, footer |
| 5 | SICR proxy | Medium | Page 3 donut note |
| 6 | Capital adequacy not modelled | High | Page 3 caveat, footer |
| 7 | Fairness not assessed | Medium | Limitations page only |
| 8 | Scenario weights illustrative | Medium | Page 3 caveat |
| 9 | Calibration check, not validation | Low | Page 2 chart note |
| 10 | Data lineage not exposed | Low | This document |

**Severity guide:**
- **High** — the limitation could mislead if not disclosed; must appear on-page.
- **Medium** — the limitation affects interpretation; should appear on-page or in footer.
- **Low** — the limitation affects reproducibility or auditability; documented here only.
