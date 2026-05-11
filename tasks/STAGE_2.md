# Stage 2 — Work Breakdown

**Timeline:** Stages 2 + 3 are combined per professor and run 3–4 weeks (roughly May 22 – June 19, 2026). Stage 2 emphasis is the first ~half of that window. The two stages are **iterative**: preprocessing reveals new EDA questions, new features open new patterns.

**Goal:** Produce a comprehensive exploratory data analysis of the base dataset plus the binary label constructed in Stage 1. Submit notebook EDA section + 1–2 page written paper section.

This file follows the same structure as `STAGE_1.md`. Each task is self-contained and Claude-Code-ready. Pick a task, set yourself as Owner, branch off `main`, do the work, open a PR.

---

## Task 2.1 — Basic statistics and data quality audit

**Owner:** _unassigned_
**Branch:** `stage-2-basic-stats`
**Estimated time:** 2–3 hours

### Goal
Produce summary statistics for the dataset and identify data quality issues that need to be addressed before feature engineering.

### Steps
1. In the notebook, create a top-level section **"Stage 2 — Exploratory Data Analysis"**.
2. Compute and display:
   - `df.describe()` for all numeric columns
   - `df.describe(include='object')` for categorical columns
   - Per-ticker summary statistics (mean close, mean volume, mean daily return, return volatility)
3. Check for:
   - Duplicate (ticker, date) pairs
   - Gaps in the trading-day sequence per ticker
   - Tickers with missing dates that other tickers have
   - Zero or negative volume rows
   - Unrealistic OHLC relationships (high < low, close outside [low, high])
4. Print a clean summary table of data quality findings.

### Output
- Summary statistics tables in the notebook
- Data quality findings printed and explained in markdown

### Acceptance criteria
- [ ] `df.describe()` outputs printed
- [ ] Per-ticker summary statistics computed
- [ ] Duplicate-row check run
- [ ] Trading-day gap check run
- [ ] OHLC sanity checks run
- [ ] Markdown summary of findings written

---

## Task 2.2 — Target distribution and class balance

**Owner:** _unassigned_
**Branch:** `stage-2-target-distribution`
**Estimated time:** 2 hours

### Goal
Analyze the distribution of the `binary_label` column. Class balance directly informs the evaluation metric decision (accuracy vs returns-based).

### Steps
1. Compute and print overall positive-class %.
2. Plot a bar chart of class balance (0 vs 1).
3. Break down class balance by:
   - Year
   - Ticker (top 10 most positive, top 10 most negative)
   - Day of week
   - Month
4. Compute and plot the rolling 60-day positive-class % over time — is the rate stable, or does it shift?
5. Write a markdown summary that ends with a clear recommendation: given this class balance, is **accuracy** an acceptable primary metric, or should we use a **returns-based** metric? (This is the input to the Stage 4 metric decision.)

### Output
- Class-balance bar chart
- Multiple breakdowns (year, ticker, day of week, month)
- Rolling positive-class % time series
- Markdown recommendation on evaluation metric

### Acceptance criteria
- [ ] Overall positive-class % printed
- [ ] Class balance by year, ticker, day-of-week, month
- [ ] Rolling 60-day positive-class % plotted
- [ ] Written recommendation on metric choice

### Gotchas
- If positive class is <20%, accuracy will be misleading (a model predicting all zeros looks 80%+ accurate). Flag this clearly.
- Watch for **regime shifts** — if early years had very different positive rates than recent years, the model may extrapolate poorly.

---

## Task 2.3 — Univariate distributions of base features

**Owner:** _unassigned_
**Branch:** `stage-2-univariate`
**Estimated time:** 2–3 hours

### Goal
Plot the distribution of every base feature to identify skew, outliers, multi-modality, and anything suspicious that needs cleaning.

### Steps
1. For each numeric column (`open`, `high`, `low`, `close`, `volume`, `daily_return_pct`, `vix_end_of_day`, `sp500_return_today`):
   - Plot a histogram (or KDE for highly skewed columns)
   - Print skewness and kurtosis
2. Volume is typically log-skewed — plot `log(volume + 1)` alongside.
3. Flag any column with extreme outliers (z-score > 5 or beyond 99.9th percentile).
4. Add a markdown cell summarizing which features look clean vs which need transformation in Stage 3.

### Output
- One histogram per numeric base feature
- Skewness/kurtosis table
- Markdown summary of features needing transformation

### Acceptance criteria
- [ ] Histograms for all numeric columns
- [ ] Log-transformed volume plot
- [ ] Skewness/kurtosis values printed
- [ ] Outlier flags noted
- [ ] Summary markdown written

---

## Task 2.4 — Correlation analysis

**Owner:** _unassigned_
**Branch:** `stage-2-correlations`
**Estimated time:** 2 hours

### Goal
Identify which base features correlate with each other (multicollinearity risk) and which correlate with the target (predictive signal).

### Steps
1. Compute the Pearson correlation matrix across all numeric columns.
2. Plot a correlation heatmap with `seaborn.heatmap`. Use a diverging colormap and annotate values.
3. Print the top-10 strongest feature-vs-feature correlations (excluding self).
4. Print the correlation of every feature with `binary_label`.
5. Add a markdown cell flagging:
   - Highly correlated feature pairs (|r| > 0.9) — these are redundant
   - Features with non-trivial correlation to the target (|r| > 0.05 is interesting in this domain)

### Output
- Correlation heatmap
- Top-correlations table
- Target correlation table
- Markdown summary

### Acceptance criteria
- [ ] Correlation heatmap rendered
- [ ] Top-10 cross-feature correlations listed
- [ ] Per-feature correlation with target listed
- [ ] Multicollinearity risks flagged in markdown

### Gotchas
- OHLC columns will be ~perfectly correlated. That's expected — note it and move on.
- Linear correlation is weak for non-linear relationships. Don't dismiss a feature just because its Pearson correlation is near zero.

---

## Task 2.5 — Feature-vs-target visual analysis

**Owner:** _unassigned_
**Branch:** `stage-2-feature-target`
**Estimated time:** 3 hours

### Goal
For each candidate feature category, visualize how the feature distributes differently when `binary_label = 0` vs `binary_label = 1`. This is qualitative evidence of predictive signal.

### Steps
1. For each base feature (and a few engineered candidates if quick to compute — e.g., 5-day return, 20-day rolling vol):
   - Plot overlaid KDE or histogram for label=0 vs label=1
   - Compute and print the mean for each group, plus a t-statistic or Welch's t-test
2. For VIX specifically: are high-VIX days more or less likely to produce a >1% move?
3. Add a markdown cell summarizing which features look promising and which look uninformative.

### Output
- Conditional-on-target plots for each base feature
- Mean comparison table
- Markdown summary

### Acceptance criteria
- [ ] At least 6 features plotted conditional on label
- [ ] Mean comparison table printed
- [ ] VIX-vs-label analysis included
- [ ] Markdown summary of promising vs unpromising features

---

## Task 2.6 — Missing values, outliers, and time alignment

**Owner:** _unassigned_
**Branch:** `stage-2-missing-outliers`
**Estimated time:** 2 hours

### Goal
Document every data quality issue that needs to be addressed in Stage 3 preprocessing.

### Steps
1. Compute missing-value % per column.
2. For each missing value, determine if it's:
   - A real gap (e.g., trading holiday)
   - A ticker-level issue (one ticker is missing data for a stretch)
   - A column-level issue (one column has many missing values)
3. Identify outliers using IQR or z-score methods. Document but don't remove yet.
4. Confirm that all tickers have continuous trading days (after accounting for market holidays).
5. Write a markdown plan for how Stage 3 will handle each issue.

### Output
- Missing-value summary
- Outlier list per column
- Stage 3 preprocessing plan in markdown

### Acceptance criteria
- [ ] Missing-value % per column documented
- [ ] Each missing value categorized
- [ ] Outliers identified per column
- [ ] Stage 3 cleaning plan written

---

## Task 2.7 — Draft the Stage 2 written paper section

**Owner:** _unassigned_
**Branch:** `stage-2-paper-section`
**Estimated time:** 3 hours
**Depends on:** Tasks 2.1 – 2.6

### Goal
Write the 1–2 page Stage 2 section of the final paper.

### Steps
1. Add an H1 heading **"Stage 2: Exploratory Data Analysis"** to `paper/final_paper.md`.
2. Subsections:
   - Data quality summary (Task 2.1, 2.6)
   - Target distribution and implications for evaluation metric (Task 2.2)
   - Univariate findings (Task 2.3)
   - Correlation findings (Task 2.4)
   - Feature-vs-target signal (Task 2.5)
3. End with a one-paragraph **EDA → Stage 3 handoff**: what features look most promising, what preprocessing is needed, what external data is most urgent to integrate.
4. Document AI usage.

### Output
- Stage 2 section written in `paper/final_paper.md`

### Acceptance criteria
- [ ] All six EDA themes covered
- [ ] Numbers in paper match notebook output
- [ ] Stage 3 handoff paragraph written
- [ ] AI usage disclosed
- [ ] 1–2 pages, tight

---

## How to use this file with Claude Code

From inside the repo, run `claude` and try:

> "Read tasks/STAGE_2.md. I'm taking Task 2.4. Walk me through it and add the code into notebooks/energy_prediction.ipynb."

Claude Code will load SPEC.md, CLAUDE.md, and this file as context, respect the rules (no train_test_split, no look-ahead, 1% threshold, 4 PM prediction time), and produce code matching the acceptance criteria.
