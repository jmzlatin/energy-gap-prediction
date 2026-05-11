# Stage 3 — Work Breakdown

**Timeline:** Stages 2 + 3 are combined per professor and run 3–4 weeks (roughly May 22 – June 19, 2026). Stage 3 emphasis is the second ~half of that window, but the two are **iterative** — expect to bounce between Stage 2 EDA and Stage 3 engineering.

**Goal:** Implement the engineered features from Stage 1's candidate list, integrate external Energy-sector data, and produce a clean modeling-ready dataset. Submit notebook feature-engineering section + 1–2 page written paper section.

**Critical reminder:** External data is the **main differentiator** per the professor. Base OHLCV alone won't beat the efficient market. Treat the external-data tasks (3.5 – 3.8) as the most impactful work in this stage.

---

## Task 3.1 — Implement trend features (moving averages and distances)

**Owner:** _unassigned_
**Branch:** `stage-3-trend-features`
**Estimated time:** 3 hours

### Goal
Compute moving averages and distance-from-MA features for every ticker. **No look-ahead bias.**

### Steps
1. Create a new section in the notebook: **"Stage 3 — Engineered Features"** with subsection "Trend".
2. For each ticker (group by ticker), compute:
   - 5, 10, 20, 50, **150**-day moving averages of `close`
   - Distance from each MA: `close / MA - 1`
   - Differences between MAs: e.g. `MA_20 - MA_50`, `MA_50 - MA_150`
3. Use `df.groupby('ticker')['close'].rolling(N).mean()` and verify it doesn't leak across tickers.
4. Verify no look-ahead: the rolling computation should use only past data. Use `min_periods=N` so partial windows produce NaN rather than misleading values.
5. Spot-check one ticker manually for 2-3 rows.

### Output
- New columns: `ma_5`, `ma_10`, `ma_20`, `ma_50`, `ma_150`, `dist_ma_5`, `dist_ma_10`, etc., and `ma_diff_20_50`, `ma_diff_50_150`.

### Acceptance criteria
- [ ] All 5 MAs computed per ticker
- [ ] All 5 distance-from-MA features computed
- [ ] At least 2 MA-difference features computed
- [ ] `min_periods=N` enforced (NaN for partial windows)
- [ ] Manual spot-check of one ticker passed

### Gotchas
- **Common bug:** computing rolling across the entire DataFrame instead of per ticker. This silently mixes ticker A's history with ticker B's. Always `groupby('ticker')` first.
- The 150-day MA produces ~150 days of NaN at the start of each ticker's history. That's expected.

---

## Task 3.2 — Implement volatility features

**Owner:** _unassigned_
**Branch:** `stage-3-volatility`
**Estimated time:** 1–2 hours

### Goal
Compute rolling volatility features.

### Steps
1. In the same notebook section, add subsection "Volatility".
2. For each ticker, compute the rolling standard deviation of daily returns over 5, 10, 20 days.
3. Optionally also compute realized volatility from squared returns.
4. Compare scales: are these in % terms or decimal? Document.

### Output
- `vol_5`, `vol_10`, `vol_20` columns

### Acceptance criteria
- [ ] All 3 rolling-vol features computed per ticker
- [ ] Units (% vs decimal) documented in markdown

---

## Task 3.3 — Implement momentum features (RSI + return windows)

**Owner:** _unassigned_
**Branch:** `stage-3-momentum`
**Estimated time:** 3 hours

### Goal
Compute RSI at multiple time periods (per professor's hint) and lookback-return features.

### Steps
1. Add subsection "Momentum".
2. Implement RSI at multiple horizons: 7, 14, 21, 28 days. Don't just use the 14-day standard — the professor explicitly suggested multiple periods.
3. Compute return-over-window features: 1-day, 3-day, 5-day, 10-day, 20-day cumulative returns.
4. Use the standard RSI formula (average gain / average loss) with Wilder's smoothing.
5. Verify RSI is bounded [0, 100].

### Output
- `rsi_7`, `rsi_14`, `rsi_21`, `rsi_28` columns
- `ret_1d`, `ret_3d`, `ret_5d`, `ret_10d`, `ret_20d` columns

### Acceptance criteria
- [ ] 4 RSI columns at different periods
- [ ] 5 return-window columns
- [ ] RSI values fall within [0, 100]
- [ ] Per-ticker computation verified (no cross-ticker leakage)

---

## Task 3.4 — Implement volume features

**Owner:** _unassigned_
**Branch:** `stage-3-volume`
**Estimated time:** 1–2 hours

### Goal
Compute volume-based features. Professor explicitly mentioned **volume changes** alongside the standard ratios.

### Steps
1. Add subsection "Volume".
2. Compute:
   - Volume ratio: today's volume / 20-day average volume
   - Volume change: % change in volume vs yesterday
   - Log volume (often less skewed)
3. Per-ticker, group-aware.

### Output
- `vol_ratio_20`, `vol_change_1d`, `log_volume` columns

### Acceptance criteria
- [ ] 3 volume-based features computed
- [ ] No cross-ticker leakage

---

## Task 3.5 — Integrate oil prices (WTI + Brent, per-ticker logic)

**Owner:** _unassigned_
**Branch:** `stage-3-oil-prices`
**Estimated time:** 4–5 hours

### Goal
Pull WTI and Brent crude oil prices from yfinance, align to our daily stock data, and produce oil-derived features. **This is one of the highest-leverage tasks in the entire project.**

### Context
Per professor: oil pricing is **location-specific**. WTI for US-focused producers, Brent for internationally-exposed names. Stage 1 confirmed yfinance access; this task is the actual integration.

### Steps
1. Pull WTI (`CL=F`) and Brent (`BZ=F`) daily data over our 4-year window via yfinance.
2. Align to our stock dates by left-joining on `date`. Forward-fill at most 1–2 days for weekends/holidays where oil markets and stock markets diverge.
3. Compute oil features:
   - Daily oil return (WTI and Brent)
   - 5-day oil return
   - WTI–Brent spread (interesting standalone signal)
   - Oil volatility (rolling 20-day std)
4. Build a per-ticker mapping: which producers are US-focused (use WTI) vs internationally-exposed (use Brent). Start with a simple manual mapping for the top tickers; document the heuristic.
5. Create a unified `relevant_oil_return` column that picks WTI or Brent based on the per-ticker mapping.

### Output
- Notebook subsection "External Data — Oil Prices"
- New columns: `wti_close`, `wti_return_1d`, `wti_return_5d`, `brent_close`, `brent_return_1d`, `wti_brent_spread`, `relevant_oil_return`

### Acceptance criteria
- [ ] WTI and Brent both downloaded successfully
- [ ] Date alignment verified (no future-leaking dates)
- [ ] Oil return features computed
- [ ] WTI–Brent spread computed
- [ ] Per-ticker WTI-vs-Brent mapping documented
- [ ] `relevant_oil_return` column added

### Gotchas
- yfinance occasionally returns extra days where US oil futures trade but US stocks don't (or vice versa). Inspect the join carefully.
- **Don't forward-fill oil return**; only forward-fill the price, then compute the return. Forward-filling returns creates fake zeros.

---

## Task 3.6 — Integrate natural gas and USD Index

**Owner:** _unassigned_
**Branch:** `stage-3-gas-dxy`
**Estimated time:** 2–3 hours

### Goal
Pull natural gas (`NG=F`) and USD Index (`DX-Y.NYB`) and add as features.

### Steps
1. Pull both via yfinance over our window.
2. Compute daily return for each.
3. Compute 5-day return for each.
4. Join to the main DataFrame on `date`.
5. Sanity-check coverage and missing values.

### Output
- New columns: `natgas_close`, `natgas_return_1d`, `natgas_return_5d`, `dxy_close`, `dxy_return_1d`, `dxy_return_5d`

### Acceptance criteria
- [ ] Both datasets downloaded and aligned
- [ ] Daily and 5-day returns computed
- [ ] No missing-value issues in the merged DataFrame

---

## Task 3.7 — Integrate EIA inventory and rig count data

**Owner:** _unassigned_
**Branch:** `stage-3-eia-rigs`
**Estimated time:** 4–5 hours

### Goal
Pull EIA weekly petroleum inventory and Baker Hughes weekly rig count, align to our daily data using forward-fill (last-known-value join).

### Context
These are **weekly** datasets joined to **daily** stock data. Critical to avoid look-ahead: forward-fill only with values whose release date precedes the trading day. EIA inventory is released Wednesdays at 10:30 AM ET; rig count is released Fridays.

### Steps
1. Register for a free EIA API key at https://www.eia.gov/opendata/. Add to `.env` (gitignored).
2. Pull the relevant EIA series (e.g., U.S. Weekly Petroleum Stocks).
3. Pull Baker Hughes rig count from their public CSV release or a scraped source.
4. For each weekly value, record its **release date** (not just the data-as-of date).
5. Forward-fill into our daily DataFrame: on day _t_, use the most recent EIA/rig value with release date <= _t_.
6. Compute week-over-week change features as well as absolute level.

### Output
- New columns: `eia_inventory`, `eia_inv_change_wow`, `rig_count`, `rig_count_change_wow`

### Acceptance criteria
- [ ] EIA API key documented and in `.env`
- [ ] EIA series downloaded
- [ ] Rig count downloaded
- [ ] **Release dates** used for the forward-fill (not data-as-of dates)
- [ ] Week-over-week changes computed
- [ ] Manual spot-check: pick a trading day and verify the EIA value used is the most recently released BEFORE that day

### Gotchas
- **Highest leakage risk in the project**. If you join on "data date" instead of "release date", you're using inventory data from before it was actually published. This is a feature leak.
- EIA holiday releases shift by a day or two. Document any anomalies.

---

## Task 3.8 — Build OPEC events feature

**Owner:** _unassigned_
**Branch:** `stage-3-opec-events`
**Estimated time:** 3–4 hours

### Goal
Encode OPEC supply-decision events as features. Professor explicitly mentioned OPEC announcements as relevant.

### Steps
1. Source a historical list of OPEC and OPEC+ meeting dates and decisions over our 4-year window. Options:
   - Wikipedia's OPEC meetings page (manual extraction)
   - News API search for "OPEC announcement" by date
   - Existing public datasets
2. For each event, record:
   - Date
   - Type (regular meeting, emergency, JMMC)
   - Decision direction (production cut, production increase, hold)
3. Build features:
   - Binary flag: is today within 1/3/5 days of an OPEC event?
   - Categorical: nature of the most recent decision
   - Days since last OPEC event
4. Join to the main DataFrame.

### Output
- A clean CSV at `Data/opec_events.csv` (committed)
- New columns: `opec_event_within_1d`, `opec_event_within_3d`, `opec_event_within_5d`, `last_opec_decision`, `days_since_opec_event`

### Acceptance criteria
- [ ] OPEC events sourced and saved to CSV
- [ ] At least 3 OPEC-derived feature columns added
- [ ] Manual spot-check of one well-known event (e.g., the 2020 emergency cut)

### Gotchas
- Event dates are the **announcement date**, not the effective date of the policy. Stock prices react to announcements.
- Time zones — OPEC announces in CET. Our stock data is US trading days. For a decision announced after US market close, the "reaction" trading day is the next one.

---

## Task 3.9 — Handle missing values, outliers, and final cleanup

**Owner:** _unassigned_
**Branch:** `stage-3-cleanup`
**Estimated time:** 3 hours
**Depends on:** Tasks 3.1 – 3.8

### Goal
Produce the final modeling-ready DataFrame.

### Steps
1. After all features are joined, audit total missing values per column.
2. Decide and apply handling for each column:
   - Drop rows for features critical to the model (early NaN from rolling windows is fine — drop those rows once)
   - Forward-fill for slow-moving external data
   - Mean/median impute only as a last resort
3. Handle outliers per the Stage 2 plan (winsorize, clip, or leave alone).
4. Verify no inf/-inf values from division features (distance-from-MA can produce these).
5. Save the final DataFrame to `Data/modeling_dataset.parquet` (gitignored — too large; regenerate from notebook).
6. Print final shape and confirm `binary_label` and all features are present.

### Output
- `Data/modeling_dataset.parquet` (gitignored)
- Final feature list printed
- Missing-value report after cleanup

### Acceptance criteria
- [ ] No NaN in feature columns of the final DataFrame
- [ ] No inf/-inf values
- [ ] Outliers handled per Stage 2 plan
- [ ] Final DataFrame saved
- [ ] Shape and column list printed

---

## Task 3.10 — Draft the Stage 3 written paper section

**Owner:** _unassigned_
**Branch:** `stage-3-paper-section`
**Estimated time:** 3 hours
**Depends on:** Tasks 3.1 – 3.9

### Goal
Write the 1–2 page Stage 3 section of the final paper.

### Steps
1. Add H1 heading **"Stage 3: Data Engineering & Feature Creation"** to `paper/final_paper.md`.
2. Subsections:
   - Engineered base features (categories + count, not exhaustive listing)
   - External Energy data integrated (oil, gas, DXY, EIA, rigs, OPEC) — emphasize this section, it's the differentiator
   - Missing-value and outlier handling
   - Final dataset shape and feature count
3. Highlight any insights discovered during engineering (e.g., WTI-Brent spread looks correlated with X).
4. Document AI usage.

### Output
- Stage 3 section in `paper/final_paper.md`

### Acceptance criteria
- [ ] All feature categories covered
- [ ] External data section is detailed (the differentiator!)
- [ ] Cleanup decisions justified
- [ ] AI usage disclosed
- [ ] 1–2 pages, tight

---

## How to use this file with Claude Code

> "Read tasks/STAGE_3.md. I'm taking Task 3.5 (oil prices). Walk me through it carefully — this is high-leverage. Verify no look-ahead bias and align WTI/Brent to our stock dates without leaking future data."

Claude Code respects the rules in CLAUDE.md: no `train_test_split`, no look-ahead, 1% threshold, 4 PM prediction time, oil prices are location-specific.
