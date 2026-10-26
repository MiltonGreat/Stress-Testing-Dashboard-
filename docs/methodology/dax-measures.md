# DAX Measures

Every measure in the model, with its DAX, purpose, filter-context behaviour,
and known caveats.

This document reflects the model **as built**, not as originally planned. Where
the implementation diverged from the plan (e.g., missing columns, rebuilt
measures, renamed fields), the divergence is noted in the relevant measure's
"Caveat" or "History" section.

---

## Measures by table

### fact_loan_portfolio

#### `Total Loans`
```dax
Total Loans = COUNTROWS(fact_loan_portfolio)

Default Rate = DIVIDE(SUM(fact_loan_portfolio[default]), [Total Loans])

UPB by FICO = SUM(fact_loan_portfolio[ORG_UPB])

UPB Share =
DIVIDE(
    [UPB by FICO],
    CALCULATE([UPB by FICO], ALL(fact_loan_portfolio))
)

Expected Loss =
SUMX(
    fact_loan_portfolio,
    fact_loan_portfolio[pd_estimate]
        * fact_loan_portfolio[lgd_final]
        * fact_loan_portfolio[ORG_UPB]
)

Expected Loss Rate =
DIVIDE([Expected Loss], SUM(fact_loan_portfolio[ORG_UPB]))

Average PD = AVERAGE(fact_loan_portfolio[pd_estimate])

Total ECL = SUM(fact_scenario_analysis[ECL])

Weighted ECL =
SUMX(
    fact_scenario_analysis,
    fact_scenario_analysis[ECL] * fact_scenario_analysis[Weight]
)


---

## What Changed From the First Version

A short note on the revisions, so you can see why this version is better:

| First version | Revised version | Why |
|---|---|---|
| Named `Average PD` as a planned measure | Documents it as **built** — and adds it to the inventory | It exists in the model now; documenting planned-but-not-built would be misleading |
| No mention of `Total Assets` | Documents it as **removed**, with the full DAX and reason | A reader who finds a `Total Assets` measure in an old screenshot needs to know why it's gone |
| `Weighted ECL` described as a column | Documents it as a **rebuilt measure**, with history | The model contains the measure, not the column — the history matters |
| `Expected Loss` note said "uses `ORG_UPB` as EAD proxy" | Adds **history** — the original `ead_final` error, and why the proxy was chosen | The divergence from the brief is part of the measure's story |
| Worked examples had three cases | Adds a **fourth** (Weighted ECL card vs. table) | The card-vs-table cross-check is a real verification step the reader can run |
| No "patterns" section | Adds a **DAX patterns** reference table | Gives a reader who wants to learn DAX a map of which patterns appear where |

---

## Recall Check *(answers included)*

**Q:** *Why does the revised `dax-measures.md` document measures that were **removed** (like `Total Assets`), rather than just listing what currently exists?*

**A:** Because **a reader may encounter evidence of the removed measure** — in an old screenshot, in the commit history, in a conversation about the model — and needs to know why it isn't there anymore. Documenting the removal (with the DAX and the reason) turns an unexplained gap into a documented decision. **"Why is this missing?" is as important a question as "what does this do?"** — and a data dictionary that only answers the second leaves the first unanswered.

**Q:** *Why does the "Complexity" column in the measure inventory distinguish "simple" and "medium" rather than marking them all "simple"?*

**A:** Because **the complexity of a measure is not its DAX length — it's the cognitive load required to interpret it.** `SUM(column)` is simple: read it, understand it, done. `CALCULATE(measure, ALL(table))` requires understanding filter context — a reader who doesn't know what `ALL` does will misread the measure's behaviour. Flagging this helps a reviewer know which measures to check more carefully, and helps a future maintainer know which measures are riskier to change. **The label is a reading aid, not a complexity grade.**

**Q:** *Why does the revised document include a "DAX patterns used" section at the end, when each measure already has its DAX?*

**A:** Because **the patterns are transferable; the measures are not.** A reader studying the repository isn't going to copy `UPB Share` — but they might learn the "calculate + all" pattern, which they'll use dozens of times in their own work. Presenting the patterns separately from the measures makes the repository useful as a **learning artifact**, not just a **portfolio artifact**. For a portfolio piece aimed at demonstrating competence, showing *which patterns you know* is as valuable as showing *what you built*.

---

Fill in the change log dates and your name, verify the measure names against your actual Data pane (if any differ — e.g., if you renamed `UPB by FICO` to something else — adjust the document accordingly), and the file is ready.
