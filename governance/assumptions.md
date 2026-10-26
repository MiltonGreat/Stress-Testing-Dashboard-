# Assumptions

Every assumption made in the dashboard, with its location, the impact if it is
wrong, and how it is disclosed to the reader.

Assumptions are where dashboards go wrong **silently**. Documenting them turns
hidden guesses into disclosed decisions.

---

## Assumption Inventory

| # | Assumption | Where used | Impact if wrong | Disclosed in |
|---|---|---|---|---|
| 1 | `ead_final ≈ ORG_UPB` | Expected Loss (Page 2) | Overstates EL for seasoned loans | Page 2 caveat, footer |
| 2 | `lgd_final = f(pd_estimate)` | Expected Loss (Page 2) | PD × LGD partly circular; overstates correlation | Page 2 caveat, footer |
| 3 | Fannie Mae data as Canadian proxy | All pages | Segment rankings may not transfer | Every page footer |
| 4 | 100,000 loans representative of full portfolio | All pages | Segment rates carry sampling error | Every page footer |
| 5 | Scenario weights are illustrative | Weighted ECL (Page 3) | Weighted ECL is a planning number, not a forecast | Page 3 caveat |
| 6 | Stage 2 = proxy SICR | IFRS 9 donut (Page 3) | Stage 2 may be over- or under-stated | Donut note, Limitations page |
| 7 | Capital figures are constant across scenarios | Page 3 (charts removed) | No scenario comparison of capital is possible | Page 3 caveat |
| 8 | Average PD is a valid per-band summary | PD vs. Realized (Page 2) | Averages can mask within-band variation | Page 2 chart note |
| 9 | FICO/LTV/age bands are adequate segmentations | Pages 1–2 | Coarser segmentation may hide within-band variation | Limitations page |

---

## Detailed Assumption Notes

### Assumption 1 — `ead_final ≈ ORG_UPB`

**Where the assumption is made:** In the `Expected Loss` measure, which uses
`fact_loan_portfolio[ORG_UPB]` in place of the missing `ead_final` column.

**Why this assumption was necessary:** `ead_final` is not present in the source
data (the pipeline that built the model did not export it). Without an EAD
figure, Expected Loss cannot be computed per the definition PD × LGD × EAD.

**Why ORG_UPB is a reasonable proxy:**
- For newly originated loans, ORG_UPB ≈ current balance ≈ EAD.
- For seasoned loans, ORG_UPB overstates EAD because principal has amortised.

**Impact if wrong:**
- **Direction:** Expected Loss is overstated, especially for high-age loans.
- **Magnitude:** For a 10-year-old loan with 50% amortisation, ORG_UPB is
  roughly 2× true EAD — so Expected Loss is roughly 2× overstated for that loan.
- **Effect on segment comparisons:** Roughly uniform across segments (since
  amortisation rates are similar across FICO/LTV bands), so relative
  comparisons remain valid.

**How it is disclosed:** Page 2 caveat and every page footer.

---

### Assumption 2 — `lgd_final = f(pd_estimate)`

**Where the assumption is made:** In the source data pipeline (not in Power BI),
`lgd_final` is constructed as a function of `pd_estimate` rather than empirically
calibrated from observed recovery rates.

**Why this assumption was necessary:** The dataset does not include recovery
amounts or workout timelines, so LGD cannot be computed empirically. The pipeline
author chose a PD-linked construction — a defensible simplification for
illustrative purposes, but not regulatory-grade.

**Why this matters:**
- PD × LGD is **not** a product of two independent estimates.
- The implied correlation between PD and LGD is artificially high.
- Expected Loss is therefore **directionally correct** (high-PD loans do carry
  more expected loss) but **partly circular**.

**Impact if wrong:**
- **Direction:** Expected Loss is overstated because PD × LGD > PD × LGD_true.
- **Magnitude:** Depends on the function used. Roughly, if LGD ≈ k × PD, then
  PD × LGD ≈ k × PD², which inflates the effect of PD on Expected Loss.
- **Effect on segment comparisons:** Segments with the highest PD are most
  overstated. The Low-FICO × High-LTV cell will have the most inflated EL.

**How it is disclosed:** Page 2 caveat and every page footer.

---

### Assumption 3 — Fannie Mae data as a Canadian proxy

**Where the assumption is made:** Everywhere — the entire dashboard is built
on US data but presented to a Canadian bank audience.

**Why this assumption was necessary:** A Canadian bank portfolio dataset was
not available for this project. Fannie Mae single-family loan performance is
a widely used, publicly available proxy.

**Why this matters:**
- **Property types** (SF, PU, CO, MH, CP) are US-conventional; Canadian
  portfolios may use different classifications.
- **Occupancy status** (P, S, I) conventions differ slightly by market.
- **Default behaviour** by province/state, loan age, and vintage may differ.
- **Regulatory framework** (IFRS 9 in Canada, different reporting for Fannie
  Mae-adjacent institutions in the US) is not reflected in the data.

**Impact if wrong:**
- **Relative rankings** (Low FICO > High FICO; high LTV > low LTV) are likely
  to transfer — they're driven by fundamental credit behaviour.
- **Absolute levels** (the default rate for a specific segment) will not
  transfer without recalculation.
- **Structural segments** (property type × occupancy) may rank differently in
  a Canadian portfolio.

**How it is disclosed:** Every page footer.

---

### Assumption 4 — Sample representativeness

**Where the assumption is made:** All aggregate metrics.

**Why this matters:** The 100,000-loan sample is treated as representative
of the full portfolio. Segment-level rates will vary around the population
rate due to sampling error. For segments with fewer than ~1,000 loans, this
variance is material.

**Impact if wrong:**
- **Rate magnitudes:** Small segments may show rates that are unstable and
  not representative.
- **Relative rankings:** Rankings between segments with similar rates may
  flip on resampling.

**How it is disclosed:** Every page footer ("Based on 100,000-loan sample").

---

### Assumption 5 — Scenario weights are illustrative

**Where the assumption is made:** In the `Weighted ECL` measure, which multiplies
each scenario's ECL by its `Weight`.

**Why this matters:** The `Weight` column values in `fact_scenario_analysis` are
illustrative — they represent the pipeline author's assumed scenario
probabilities, not a formal stress-testing framework.

**Impact if wrong:**
- **Weighted ECL magnitude** depends entirely on the weights.
- **Weighted ECL is a planning number, not a forecast.** Different weights
  produce different planning numbers without any change to the underlying
  scenarios.

**How it is disclosed:** Page 3 caveat.

---

### Assumption 6 — Stage 2 = proxy SICR

**Where the assumption is made:** In the IFRS 9 donut, which shows the
percentage of loans in each stage.

**Why this matters:** Stage 2 in the dataset is derived from a proxy rule
(not true IFRS 9 SICR). Some loans in Stage 2 may not meet true SICR criteria;
some loans not in Stage 2 may meet them.

**Impact if wrong:**
- **Stage 2 proportion** may be over- or under-stated.
- **Stage 1 and Stage 3 proportions** are more reliable — they are based on
  performing/impaired status, not a proxy rule.

**How it is disclosed:** Donut note and Limitations page.

---

### Assumption 7 — Capital figures are constant

**Where the assumption is made:** `fact_capital_coverage` contains
`CET1_Ratio` and `Capital_Buffer` values that are identical across all four
scenarios.

**Why this matters:** No scenario comparison of capital position is possible
from this data. Charts were removed from Page 3 to avoid misleading the reader
into thinking capital varies by scenario.

**Impact if wrong:**
- **The chart is absent** — so the impact is a **missing visual**, not a
  misleading one. Readers looking for capital adequacy information will not
  find it, which is honest.

**How it is disclosed:** Page 3 caveat and Limitations page.

---

### Assumption 8 — Average PD is a valid per-band summary

**Where the assumption is made:** In the PD vs. Realized chart, which uses
`Average PD` (mean of `pd_estimate` per band).

**Why this matters:** An average PD per band can mask within-band variation.
A band with PDs of 1%, 1%, 1%, 10% has the same mean (3.25%) as a band with
PDs of 3%, 3%, 3%, 4%. The calibration check may be insensitive to outliers.

**Impact if wrong:**
- **Calibration conclusions** may be less reliable for bands with high
  within-band PD dispersion.
- **The chart is a first-order check**, not a rigorous calibration analysis.

**How it is disclosed:** Page 2 chart note.

---

### Assumption 9 — Band segmentation is adequate

**Where the assumption is made:** In the choice of FICO bands (Low / Medium /
High / Very High), LTV bands (Low / Medium / High), and age bands.

**Why this matters:** Coarser banding may hide within-band variation. A
"Medium" FICO band with FICO 600–650 may have two very different risk profiles
inside it — one at 600 and one at 650.

**Impact if wrong:**
- **Segment-level rates** are averages across heterogeneous populations.
- **Fine-grained decisions** (e.g., pricing a specific FICO) cannot be made
  from band-level rates.

**How it is disclosed:** Limitations page.

---

## Summary by Page

| Page | Assumptions active |
|---|---|
| Page 1 — Concentration | 3, 4, 9 |
| Page 2 — Drivers | 1, 2, 3, 4, 8, 9 |
| Page 3 — Stress & IFRS 9 | 3, 4, 5, 6, 7 |

---

## Change Log

| Date | Change | Author |
|---|---|---|
| [YYYY-MM-DD] | Initial assumptions documented | [your name] |
