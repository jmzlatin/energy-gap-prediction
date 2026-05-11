# Stage 1 — Work Breakdown

**Deadline:** May 21, 2026
**Goal:** Submit a Jupyter notebook with data collection + verification + binary label, plus a 1–2 page written section covering the eight Stage 1 items.

This file is the source of truth for **what to do**. SPEC.md describes **what we're building**, CLAUDE.md describes **the rules**, this file says **execute these tasks**.

Each task below is self-contained and Claude-Code-ready. Pick a task, set yourself as Owner, branch off `main`, do the work, open a PR.

---

## Task 1.1 — Verify the base dataset

**Owner:** _unassigned_
**Branch:** `stage-1-data-verify`
**Estimated time:** 1–2 hours
**Deadline:** May 13

### Goal
Confirm that `Data/sp500_Energy_dataset.csv` is complete and clean. Produce the data-overview numbers required by Stage 1.

### Context
The CSV has columns: `date, ticker, sector, open, high, low, close, volume, daily_return_pct, 1_d_return, vix_end_of_day, sp500_return_today`. The `1_d_return` column appears to be a string with `%` suffix — handle this when parsing. Date format is `DD/MM/YYYY`.

### Steps
1. In `notebooks/energy_prediction.ipynb`, create a top-level section header **"Stage 1 — Data Collection & Verification"**.
2. Load the CSV into a pandas DataFrame with `pd.read_csv()`. Parse `date` with `dayfirst=True`.
3. Print the DataFrame shape and `df.info()`.
4. Compute and print:
   - Number of unique tickers: `df['ticker'].nunique()`
   - Occurrences per ticker: `df['ticker'].value_counts()` (and flag any ticker with significantly fewer rows than the others)
   - Date range: `df['date'].min()` and `df['date'].max()` — **expected to end on or near March 6, 2026** per the professor.
   - Total unique trading days: `df['date'].nunique()`
   - Missing values per column: `df.isnull().sum()`
5. Sanity-check: plot the number of tickers present per date. Drops indicate tickers entering/leaving the index.
6. Add a short markdown cell summarizing findings in plain English. Flag any deviation from the expected end date.

### Output
- New notebook section "Stage 1 — Data Collection & Verification" with cells producing the numbers above
- One paragraph of findings in markdown

### Acceptance criteria
- [ ] Notebook runs top to bottom without error
- [ ] Ticker count printed
- [ ] Per-ticker occurrence counts printed
- [ ] Date range and total trading days printed
- [ ] Missing-value summary printed
- [ ] Tickers-per-date plot included
- [ ] Markdown summary written

### Gotchas
- `1_d_return` is string-formatted with `%`. Don't parse it as numeric for this task; we deal with that in Task 1.2.
- Date format is European (DD/MM/YYYY). `dayfirst=True` is required or pandas will silently misparse.

---

## Task 1.2 — Construct the binary label at 1% threshold

**Owner:** _unassigned_
**Branch:** `stage-1-binary-label`
**Estimated time:** 2–3 hours
**Deadline:** May 14
**Depends on:** Task 1.1

### Goal
Construct the binary target column using the **confirmed 1% threshold**, and produce the return-distribution analysis needed to justify the threshold in the written paper.

### Context
**The 1% threshold is confirmed with the professor.** This task is not about deciding the threshold — it's about implementing it correctly and providing the supporting distribution analysis for the paper.

### Steps
1. Clean the `1_d_return` column: strip `%`, convert to float, divide by 100 if needed. Verify by spot-checking a few rows against `close` of consecutive days.
2. Plot a histogram of next-day returns across all (ticker, date) pairs. Use 100 bins, mark a vertical line at +1%.
3. Compute and print:
   - % of observations where return > 0%
   - % where return > 1% ← this is our positive-class %
   - % where return > 1.5%
   - % where return > 2%
4. Compute the same percentages broken down by year (look for regime shifts — useful color for the paper).
5. Define `THRESHOLD = 0.01` as a constant near the top of the notebook (easy to change later if we ever want sensitivity analysis).
6. Create the binary label column: `df['binary_label'] = (df['next_day_return'] > THRESHOLD).astype(int)`.
7. Report final positive-class %.

### Output
- Cleaned `next_day_return` numeric column
- Return distribution histogram with 1% line
- `THRESHOLD = 0.01` constant defined
- `binary_label` column constructed
- Positive-class % reported

### Acceptance criteria
- [ ] `1_d_return` cleaned to numeric
- [ ] Histogram plotted with 1% threshold marker
- [ ] Threshold % table printed for 0%, 1%, 1.5%, 2%
- [ ] Per-year breakdown printed
- [ ] `THRESHOLD = 0.01` constant in notebook
- [ ] `binary_label` column exists in the DataFrame
- [ ] Positive observation % reported (single number)

### Gotchas
- Verify that `1_d_return` actually means "next day's return" and not "today's return". Spot-check 2-3 rows by comparing to consecutive `close` values. If it's today's return, you need to shift by -1 grouped by ticker.
- Don't introduce look-ahead bias: when shifting, group by ticker so you don't pull data from a different stock.
- Prediction is at 4 PM market close, so the label for day _t_ is the close-to-close return from day _t_ to day _t+1_. Sanity-check this aligns with `1_d_return`.

---

## Task 1.3 — List candidate engineered features with rationale

**Owner:** _unassigned_
**Branch:** `stage-1-feature-list`
**Estimated time:** 1–2 hours
**Deadline:** May 15

### Goal
Produce the candidate feature list required by Stage 1. **Do not implement features yet** — that's Stage 3. We're just listing what we plan to test.

### Steps
1. In the notebook, add a markdown section **"Stage 1 — Candidate Engineered Features"**.
2. List 10–15 candidate features. For each, include:
   - Feature name
   - How it's computed
   - One-line rationale: why might this predict next-day return?
3. Group features into buckets: Trend (MAs, distance from MA), Volatility (rolling std), Momentum (RSI, return windows), Volume (volume ratios), Market context (VIX-relative, S&P-relative).

### Reference list to start from
- 5/10/20/50/**150**-day moving averages of close (professor mentioned 150-day MA as a common Energy strategy)
- Distance from each MA (close / MA - 1)
- **Differences between MAs across time horizons** (e.g. 20-day vs 50-day spread) — professor flagged this
- Rolling volatility (std of daily returns) — 5/10/20 day
- **RSI across multiple time periods** (not just the 14-day standard) — professor flagged this
- Returns over 1, 3, 5, 10 days
- Volume **changes** and volume ratios (today / 20-day average)
- Lagged returns (yesterday, 2 days ago)
- VIX level
- VIX 5-day change
- S&P return today
- S&P 5-day return

### Output
- Markdown cell in the notebook with the structured feature list

### Acceptance criteria
- [ ] At least 10 features listed
- [ ] Each has computation method + rationale
- [ ] Features grouped into categories
- [ ] No actual feature implementation code yet (that's Stage 3)

---

## Task 1.4 — Identify and test external Energy data sources

**Owner:** _unassigned_
**Branch:** `stage-1-external-data`
**Estimated time:** 3–4 hours
**Deadline:** May 16

### Goal
Identify external Energy-sector data sources, **test that each can actually be downloaded**, and document availability + frequency + alignment with our stock data window.

### Why this matters now
The biggest Stage 3 risk is discovering that your chosen external dataset is paywalled, has different date coverage, or has a frequency you can't easily align. Catching that now saves a week later.

### Steps
1. For each source in the list below, write a test cell that downloads the data and checks: does the date range cover our stock window? What's the frequency (daily / weekly / event-based)? Are there missing periods?
2. Build a summary markdown table documenting each source.

### Why this matters even more after the professor meeting
The professor explicitly said **the base data alone is insufficient** — external data is the main differentiator across groups. Treat this as a critical task, not optional.

### Sources to test
| Source | Ticker / API | Frequency | Notes |
|---|---|---|---|
| WTI crude | `CL=F` via yfinance | daily | for US-focused producers (location-specific) |
| Brent crude | `BZ=F` via yfinance | daily | for internationally-exposed producers (location-specific) |
| Natural gas | `NG=F` via yfinance | daily | relevant for gas-heavy producers |
| USD Index (DXY) | `DX-Y.NYB` via yfinance | daily | strong USD pressures oil |
| OPEC announcements | manual or scraped list | event-based | supply shocks, prof explicitly mentioned |
| EIA petroleum inventory | EIA API (free key) | weekly | supply signal |
| Baker Hughes rig count | Public CSV / scrape | weekly | drilling activity |
| **News sentiment** (stretch) | NewsAPI / FinBERT / etc. | daily | ~880 trading days × tickers — expensive. Only attempt if a clean historical source is identified. |

### On oil benchmarks — location-specific
Professor flagged that oil pricing depends on where the producer operates. For Stage 3, consider per-ticker mapping (WTI for US producers, Brent for international exposure). For Stage 1, just confirm both sources download and document the distinction.

### Output
- Notebook section **"Stage 1 — External Data Source Audit"** with one test cell per source
- Summary markdown table

### Acceptance criteria
- [ ] At least 5 external sources tested (oil, gas, DXY, plus at least 2 of OPEC/EIA/rigs)
- [ ] Each source downloads successfully without authentication issues (or auth method documented)
- [ ] Date coverage for each source confirmed to overlap with our stock window (through March 6, 2026)
- [ ] Summary table in markdown
- [ ] News sentiment feasibility briefly assessed (even if conclusion is "skip for now")

### Gotchas
- Some yfinance tickers return empty DataFrames silently. Always print `.shape` after download.
- EIA requires a free API key — register at https://www.eia.gov/opendata/. Document the key in `.env` (gitignored), never commit it.
- Weekly data must be forward-filled or last-known-value joined to daily — this is a Stage 3 decision, not a Stage 1 problem. Just note the frequency.

---

## Task 1.5 — Draft the Stage 1 written paper section

**Owner:** _unassigned_
**Branch:** `stage-1-paper-section`
**Estimated time:** 3–4 hours
**Deadline:** May 19
**Depends on:** Tasks 1.1 – 1.4

### Goal
Write the 1–2 page Stage 1 section of the final paper, covering the eight items required by `Instructions/Stage 1.pdf`.

### Steps
1. Create `paper/final_paper.md` if it doesn't exist.
2. Add an **H1 heading "Stage 1: Business Understanding & Data Collection"**.
3. Write subsections covering each required item:
   - Goal (1 paragraph)
   - Data source (1 paragraph)
   - Data overview (ticker count, date range, trading days — pull numbers from Task 1.1)
   - Candidate engineered features (reference Task 1.3's list, summarize categories)
   - External Energy data plan (reference Task 1.4, justify each source's predictive logic for Energy)
   - Threshold choice and justification (from Task 1.2)
   - Binary label construction (one paragraph)
   - Positive observation % (single sentence with the number)
4. Add a brief **AI usage disclosure** at the end of the section per the course AI policy: which tools were used, for what.
5. Keep it ~1.5 pages. Tight, not padded.

### Output
- `paper/final_paper.md` with the Stage 1 section written

### Acceptance criteria
- [ ] All 8 Stage 1 items covered
- [ ] Numbers match what's in the notebook (cross-check)
- [ ] AI usage disclosed
- [ ] 1–2 pages when rendered (not 4)
- [ ] No placeholder TODOs left

---

## Task 1.6 — Group review and integration

**Owner:** _whole team_
**Branch:** none (PR reviews on `main`)
**Estimated time:** 1–2 hours
**Deadline:** May 20

### Goal
Merge all Stage 1 branches into `main`, sanity-check end-to-end, ensure paper numbers match notebook outputs.

### Steps
1. Review and merge all open Stage 1 PRs.
2. Run the notebook top to bottom on `main` to confirm it executes cleanly.
3. Cross-check every number in `paper/final_paper.md` against the notebook output.
4. Strip notebook outputs before final commit (or let `nbstripout` handle it).
5. Tag the commit: `git tag stage-1-submission && git push --tags`.

### Acceptance criteria
- [ ] All Stage 1 PRs merged
- [ ] Notebook runs end-to-end without error
- [ ] Paper numbers match notebook output
- [ ] Stage 1 tag pushed

---

## Task 1.7 — Submit Stage 1

**Owner:** _one designated submitter_
**Deadline:** May 21

### Steps
1. Export the notebook to a clean version: `jupyter nbconvert --to notebook --execute notebooks/energy_prediction.ipynb`.
2. Export `paper/final_paper.md` to PDF (Pandoc, or paste into Word and save as PDF).
3. Submit per course instructions (Moodle / email — confirm with Daniel/Liam).

### Acceptance criteria
- [ ] Notebook submitted (executed, with outputs)
- [ ] Paper submitted as PDF
- [ ] Submission confirmation received

---

## How to use this file with Claude Code

From inside the repo, run `claude` and try:

> "Read tasks/STAGE_1.md. I'm taking Task 1.1. Walk me through it step by step and write the code into notebooks/energy_prediction.ipynb."

Claude Code will read SPEC.md, CLAUDE.md, and this file automatically as context. It will respect the rules in CLAUDE.md (no train_test_split, no look-ahead, etc.) and produce code that matches the acceptance criteria here.
