# Question Map

Every visual on the dashboard exists to answer a specific question. This file
records which visual answers which question — the **design rationale** of the
dashboard.

If a visual doesn't appear in this map, it should not be on the dashboard.
If a question appears in this map without a visual, it is either deferred
(with a reason) or answered by a caveat.

---

## Page 1 — Concentration

**Umbrella question:** *Where is our default risk concentrated, and how big is each pocket?*

| # | Question | Visual | Notes |
|---|---|---|---|
| 1 | What is our overall default rate and total loan count? | Cards: Total Loans, Overall Default Rate | Baseline for all segment comparisons |
| 2 | Which FICO bands carry the highest default rates? | Bar: Default Rate by FICO Band | Borrower credit quality concentration |
| 3 | Which LTV bands carry the highest default rates? | Bar: Default Rate by LTV Band | Collateral risk concentration |
| 4 | Which loan-age bands carry the highest default rates? | Bar: Default Rate by Loan Age Band | Seasoning concentration |
| 5 | Does dollar concentration match count concentration? | Bar: UPB Share by FICO Band | Read with Q2 — high-rate + high-dollar bands are the priority |
| 6 | How do property type and occupancy split default risk? | Matrix: Default Rate by Property Type × Occupancy | Structural concentration; blanks = no loans |

**What Page 1 does NOT answer (and why):**
- **Capital adequacy** — no capital data on this page (see Page 3).
- **Fairness** — out of scope for a risk-manager audience; see Limitations.

---

## Page 2 — Drivers

**Umbrella question:** *Why are defaults happening in those pockets?*

| # | Question | Visual | Notes |
|---|---|---|---|
| 7 | When FICO and LTV combine, does default rate compound? | Matrix: Default Rate by FICO × LTV | Double-trigger test |
| 8 | How do DTI and LTV jointly shape default risk? | Scatter: DTI vs LTV — Defaults Highlighted | Joint distribution; where defaults cluster |
| 9 | Which loans carry the highest Expected Loss? | Bar: Expected Loss by FICO Band ($) | Dollar-weighted loss |
| 10 | Which loans are riskiest per dollar? | Bar: Expected Loss Rate by FICO Band (%) | Loss per dollar, size-adjusted |
| 11 | Is the model's PD calibrated to realized defaults? | Line+Column: Model PD vs. Realized Default by FICO Band | Model calibration check |
| 12 | Are high-PD loans also high-LGD loans? | **NOT VISUALISED** — addressed by caveat | LGD = f(PD) — see Limitations |
| 13 | Does PD estimate align with realized default? | (Covered by Q11) | Same visual |

**What Page 2 does NOT answer (and why):**
- **High-PD vs. high-LGD independence** — not visualised because the LGD-from-PD
  construction makes any scatter circular. Documented as a caveat instead.

---

## Page 3 — Stress & IFRS 9

**Umbrella question:** *How does today's concentration behave under stress and staging?*

| # | Question | Visual | Notes |
|---|---|---|---|
| 14 | How does ECL change from Baseline → Tail Risk? | Bar: ECL by Scenario | Stress-testing story |
| 15 | What is the probability-weighted ECL? | Card: Weighted ECL, Table: Scenario Detail | Planning number + receipt |
| 16 | What percentage of the portfolio sits in each IFRS 9 stage? | Donut: IFRS 9 Stage Distribution | Staging burden |
| 17 | Is ECL as % of assets stable across scenarios? | **NOT BUILT** — capital figures are constant | See Limitations |
| 18 | Do any fairness indicators flag concern? | **NOT BUILT** — high-cardinality unreliable | See Limitations |

**What Page 3 does NOT answer (and why):**
- **Capital adequacy by scenario** — capital figures in `fact_capital_coverage`
  are constant across scenarios. Charts removed deliberately.
- **Fairness assessment** — belongs in a separate compliance report using
  low-cardinality attributes.

---

## Cross-Cutting Questions

These apply to every page and are answered by caveats, footers, or the
Limitations page — not by a specific visual.

| # | Question | Where answered |
|---|---|---|
| 19 | Is the audience seeing sample-based numbers or population numbers? | Every page footer ("Based on 100,000-loan sample") |
| 20 | Is the US-as-Canada proxy visible to the viewer? | Every page footer |
| 21 | Are the known model limitations disclosed where they affect interpretation? | Page 2 caveat, Page 3 caveat, Limitations page |
| 22 | Can the risk manager drill from a segment down to individual loans? | **NOT BUILT** — deferred to v1.1 |

---

## Deferred Questions

Questions identified but deliberately deferred to a future version, with reasons:

| # | Question | Reason for deferral |
|---|---|---|
| 12 | Are high-PD loans also high-LGD loans? | LGD = f(PD) → circular; best handled by caveat |
| 17 | Is ECL as % of assets stable across scenarios? | `fact_capital_coverage` constant → no variation to show |
| 18 | Do fairness indicators flag concern? | High-cardinality geographic proxies unreliable → separate report |
| 22 | Drill-through to loan level? | Out of scope for v1.0; adds a detail page |
| — | Fairness report | Separate compliance deliverable, not a risk-manager dashboard |

---

## Visuals by Question — Reverse Index

For quick lookup, which visual answers which question:

| Visual | Questions answered |
|---|---|
| Card: Total Loans | Q1 |
| Card: Total Portfolio UPB | Q1 (reference for Q5) |
| Card: Overall Default Rate | Q1 |
| Bar: Default Rate by FICO Band | Q2 |
| Bar: UPB Share by FICO Band | Q5 |
| Bar: Default Rate by LTV Band | Q3 |
| Bar: Default Rate by Age Band | Q4 |
| Matrix: Property Type × Occupancy | Q6 |
| Scatter: DTI vs LTV | Q8 |
| Matrix: FICO × LTV | Q7 |
| Bar: Expected Loss ($) | Q9 |
| Bar: Expected Loss Rate (%) | Q10 |
| Line+Column: PD vs. Realized | Q11 |
| Bar: ECL by Scenario | Q14 |
| Card: Weighted ECL | Q15 |
| Table: Scenario Detail | Q15 |
| Donut: IFRS 9 Stage Distribution | Q16 |

---

## Change Log

| Date | Change | Author |
|---|---|---|
| [YYYY-MM-DD] | Initial question map | [your name] |
