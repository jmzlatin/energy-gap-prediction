# Stage 4 — Work Breakdown

**Timeline:** Final stretch of the semester. Starts after Stage 3 closes (~June 19, 2026), ends with the final workshop presentation.

**Goal:** Train both models (LightGBM primary, Logistic Regression baseline) using roll-forward expanding-window evaluation, report metrics, interpret features, compile the final paper, and deliver a **10-minute presentation**.

**Critical reminders:**
- **No `train_test_split`** — use roll-forward expanding-window only.
- **No look-ahead bias** — every feature at time _t_ uses only data available by 4 PM on day _t_.
- **Evaluation metric is an open decision** — accuracy or returns-based, decided based on Stage 2 class-balance findings.

---

## Task 4.1 — Implement roll-forward expanding-window evaluation framework

**Owner:** _unassigned_
**Branch:** `stage-4-rollforward-framework`
**Estimated time:** 4–5 hours

### Goal
Build the reusable evaluation framework. Every model in this stage uses it. Get this right and the rest is easy; get it wrong and every result is invalid.

### Context
Professor specified: train on days 1–50, predict day 51. Train on days 1–51, predict day 52. Etc. This is **expanding window**, not sliding window.

### Steps
1. Create a new notebook section **"Stage 4 — Modeling & Evaluation"** with subsection "Roll-Forward Framework".
2. Define `INITIAL_TRAIN_WINDOW = 250` (1 year minimum; adjust based on data) — this is the smallest training set size before we start predicting.
3. Sort the DataFrame by `date` (and `ticker` as secondary). Build a unique sorted list of trading days.
4. Write a function `roll_forward_evaluate(model_class, df, features, target, initial_window, **model_kwargs)` that:
   - For each day _t_ starting at `initial_window`:
     - Train on all rows with `date < t`
     - Predict on rows with `date == t`
     - Store (date, ticker, y_true, y_pred, y_pred_proba)
   - Returns a DataFrame of all predictions
5. Implement an **option to retrain every day** (most accurate, slowest) or **retrain weekly** (much faster, comparable accuracy). Default to weekly for the long runs.
6. Time a small run to estimate full-run wall clock.

### Output
- `roll_forward_evaluate()` function in a clean notebook cell
- A small smoke-test run (e.g., 30 days) verifying the function works

### Acceptance criteria
- [ ] Function written and documented
- [ ] Expanding window (not sliding) confirmed by inspection
- [ ] Output DataFrame has columns `date, ticker, y_true, y_pred, y_pred_proba`
- [ ] Retrain frequency configurable
- [ ] Smoke test runs without error
- [ ] Wall-clock estimate for full run documented

### Gotchas
- **Tempting bug:** training on `date <= t` instead of `date < t` — this leaks the target day into training.
- If the model uses standardization (e.g., for Logistic Regression), the scaler must be **fit on the training window only**, then applied to the prediction day. Don't fit on the whole DataFrame.

---

## Task 4.2 — Train Logistic Regression baseline

**Owner:** _unassigned_
**Branch:** `stage-4-logreg-baseline`
**Estimated time:** 3 hours
**Depends on:** Task 4.1

### Goal
Run the baseline. Even if it performs poorly, we need it to interpret the LightGBM results in context.

### Steps
1. Choose a clean feature set. Avoid features with high multicollinearity flagged in Stage 2.
2. Standardize features inside the roll-forward loop (fit on training, transform prediction day).
3. Train with `LogisticRegression(max_iter=1000, class_weight='balanced')` to handle class imbalance.
4. Run the full roll-forward evaluation.
5. Compute and report all candidate metrics: accuracy, precision, recall, F1, ROC-AUC, plus a returns-based metric (cumulative P&L of taking long positions when `y_pred = 1`).
6. Save the predictions DataFrame as `Data/logreg_predictions.parquet` (gitignored).

### Output
- Logistic Regression roll-forward results
- All candidate metrics computed

### Acceptance criteria
- [ ] Full roll-forward run completed
- [ ] All metrics printed
- [ ] Predictions saved
- [ ] Standardization done inside the loop (no leakage)

---

## Task 4.3 — Train LightGBM primary model

**Owner:** _unassigned_
**Branch:** `stage-4-lightgbm`
**Estimated time:** 5–6 hours
**Depends on:** Task 4.1

### Goal
Run the primary model with sensible hyperparameters and produce the full prediction set.

### Steps
1. Define a reasonable starting hyperparameter set: `num_leaves=31, learning_rate=0.05, n_estimators=300, min_child_samples=50, is_unbalance=True`.
2. LightGBM handles categorical and missing data natively — confirm none of our features need special handling.
3. Run the full roll-forward evaluation (use weekly retrain).
4. Compute all candidate metrics.
5. Save predictions to `Data/lightgbm_predictions.parquet` (gitignored).
6. **Do not over-tune.** A modest hyperparameter set with honest results is more impressive than overtuned numbers.

### Output
- LightGBM roll-forward results
- All candidate metrics computed

### Acceptance criteria
- [ ] Full roll-forward run completed
- [ ] All metrics printed
- [ ] Predictions saved
- [ ] Hyperparameters documented in a clear table

### Gotchas
- LightGBM trained per-loop is slow. Time the loop carefully — full run may be several hours.
- If the prediction probabilities are clustered (e.g., always around 0.3), the model isn't learning. Investigate before reporting.

---

## Task 4.4 — Choose evaluation metric and report results

**Owner:** _unassigned_
**Branch:** `stage-4-metric-decision`
**Estimated time:** 2 hours
**Depends on:** Tasks 4.2, 4.3

### Goal
Make the metric decision the professor left open: **accuracy or returns-based**. Justify the choice with the actual class balance and produce the final results table.

### Steps
1. Re-read Task 2.2's recommendation on metric choice given class balance.
2. Decide as a group:
   - If positive class is roughly balanced (~40–60%), accuracy is defensible.
   - Otherwise, returns-based (simulated P&L) is more meaningful.
   - We can also report both.
3. Produce a side-by-side results table: Logistic Regression vs LightGBM vs a naive baseline (e.g., "predict majority class" and "always predict 1").
4. Compute the **returns-based metric**: simulate going long $1 of each ticker when `y_pred = 1`, hold one day, sum P&L over time. Compare to buying-and-holding the Energy sector.
5. Write a markdown cell with the metric justification.

### Output
- Final results table (markdown + saved CSV)
- Metric justification

### Acceptance criteria
- [ ] Metric chosen with written justification
- [ ] Side-by-side table: LogReg, LightGBM, naive baseline
- [ ] Returns-based metric computed regardless of primary choice
- [ ] Beat-the-market analysis included

---

## Task 4.5 — Feature importance and sector interpretation

**Owner:** _unassigned_
**Branch:** `stage-4-feature-importance`
**Estimated time:** 3 hours
**Depends on:** Task 4.3

### Goal
Identify the top features driving LightGBM predictions and interpret whether they make sense for Energy.

### Steps
1. Extract LightGBM feature importances (gain-based). Average across the roll-forward retrains.
2. Plot a horizontal bar chart of top-20 features.
3. Compute SHAP values on a sample of recent predictions (optional but powerful for the paper).
4. Write a markdown interpretation: do the top features match what we'd expect for Energy?
   - Are oil-derived features in the top? (Expected — good sign.)
   - Are simple price-based features dominating? (Suggests external data isn't pulling weight — investigate.)
   - Any surprising features?
5. Discuss which features helped vs which were noise.

### Output
- Feature importance plot
- Optional SHAP plot
- Written interpretation tied to Energy sector mechanics

### Acceptance criteria
- [ ] Top-20 feature importance plot
- [ ] Written interpretation grounded in Energy sector knowledge
- [ ] Comparison: which external-data features ranked highly?

---

## Task 4.6 — Beat-the-market and honest interpretation

**Owner:** _unassigned_
**Branch:** `stage-4-market-comparison`
**Estimated time:** 2 hours
**Depends on:** Task 4.4

### Goal
Answer the project's headline question honestly: did we beat the market?

### Steps
1. Define "the market" as: buying and holding the Energy sector ETF (XLE) over our test window, or the equal-weighted Energy basket.
2. Compute the cumulative return of our strategy (long when `y_pred = 1`, flat otherwise) vs the market.
3. Compute Sharpe ratio for both.
4. Plot cumulative-return curves over time.
5. Write a one-paragraph honest interpretation: did we beat it, by how much, on what risk-adjusted basis, and what's the most likely explanation (genuine alpha, lucky timing, overfitting to test window noise)?

### Output
- Cumulative-return comparison plot
- Sharpe ratio comparison
- Honest written interpretation

### Acceptance criteria
- [ ] Strategy vs market cumulative return plotted
- [ ] Sharpe ratios computed
- [ ] Honest interpretation written (no overclaiming)

---

## Task 4.7 — Compile final paper

**Owner:** _unassigned_
**Branch:** `stage-4-final-paper`
**Estimated time:** 4 hours
**Depends on:** Tasks 4.4 – 4.6

### Goal
Stitch the four stage sections into a coherent final paper and export to PDF.

### Steps
1. Add H1 heading **"Stage 4: Modeling & Evaluation"** to `paper/final_paper.md`, covering:
   - Modeling approach (roll-forward, both models)
   - Metric decision and justification
   - Results table (LogReg vs LightGBM vs naive)
   - Feature importance and interpretation
   - Beat-the-market analysis
   - Honest summary of findings
2. Add an **Executive Summary / Abstract** at the top of `paper/final_paper.md` (1 paragraph).
3. Add a **Conclusion** section at the bottom (1 paragraph: what we learned, what we'd do differently).
4. Add an **AI Usage Disclosure** appendix (required by course AI policy).
5. Export to PDF using Pandoc, Typora, or Word.

### Output
- `paper/final_paper.pdf` (compiled from final_paper.md)

### Acceptance criteria
- [ ] All four stage sections present
- [ ] Executive summary added
- [ ] Conclusion added
- [ ] AI usage disclosed
- [ ] Exported to PDF
- [ ] PDF reviewed by all team members

---

## Task 4.8 — Build the 10-minute presentation deck

**Owner:** _unassigned_
**Branch:** `stage-4-presentation`
**Estimated time:** 5 hours

### Goal
Produce the slide deck for the final 10-minute presentation. Focus on **story**, not code.

### Suggested slide outline (10 minutes = ~10 slides)
1. **Title** — Energy Sector Stock Prediction, team names
2. **The question** — will an Energy stock rise more than 1% tomorrow?
3. **Why Energy is interesting** — oil mechanics, sector volatility, what we'd expect to drive prices
4. **Data** — base dataset + external sources, with emphasis on the external sources
5. **Method** — roll-forward expanding window, both models, why we chose this approach
6. **Results** — main metric, side-by-side comparison
7. **What the model learned** — top features, do they make sense?
8. **Beat-the-market?** — honest answer, with the chart
9. **What surprised us / what we'd do differently**
10. **AI usage** — tools used, what worked, what didn't

### Steps
1. Build the deck in `presentation/final_deck.pptx`.
2. Pull plots from the notebook (clean exports, high resolution).
3. Rehearse to hit 10 minutes — typically 1 minute per slide.
4. Assign sections to teammates for delivery.

### Output
- `presentation/final_deck.pptx`

### Acceptance criteria
- [ ] ~10 slides covering the outline
- [ ] Plots are clean, large, and labeled
- [ ] Story flows: problem → method → results → interpretation
- [ ] Each section has an assigned presenter
- [ ] Rehearsed at 10 minutes ± 30 seconds

### Gotchas
- Don't put code on slides. Code is for the notebook, not the audience.
- Don't include every metric. Pick one or two and make them clear.
- Don't oversell: "modest results, honestly interpreted" plays better than "we cracked the market."

---

## Task 4.9 — Final integration and dry run

**Owner:** _whole team_
**Branch:** `stage-4-integration`
**Estimated time:** 3 hours

### Goal
Final pre-submission check. Everything works end-to-end.

### Steps
1. Run the entire notebook top to bottom on a clean kernel. Confirm no errors.
2. Cross-check every number in the paper against the notebook output.
3. Cross-check every chart in the deck against the notebook output.
4. Strip notebook outputs (or let `nbstripout` do it) before final commit.
5. Tag the commit: `git tag final-submission && git push --tags`.
6. Rehearse the presentation as a team.

### Acceptance criteria
- [ ] Notebook runs top to bottom on a clean kernel
- [ ] Paper numbers verified against notebook
- [ ] Deck charts match notebook outputs
- [ ] AI usage documented in all three deliverables
- [ ] Final tag pushed
- [ ] Presentation rehearsed to 10 minutes

---

## Task 4.10 — Submit and present

**Owner:** _one designated submitter + the whole team_
**Deadline:** Final workshop

### Steps
1. Submit notebook + PDF paper + PPTX per course instructions.
2. Present in the final class.

### Acceptance criteria
- [ ] All three deliverables submitted
- [ ] Presentation delivered
- [ ] Submission confirmation received

---

## How to use this file with Claude Code

> "Read tasks/STAGE_4.md. I'm taking Task 4.1. Build the roll-forward evaluation framework carefully — this is foundational. Verify the date filter is `date < t` not `date <= t`."

Claude Code respects: no `train_test_split`, no look-ahead, 4 PM prediction time, 1% threshold, evaluation metric is an open group decision.
