# Stage 1: Business Understanding & Data Collection

*Fintech Workshop — Reichman University. Group project: Energy sector, 4 students.*
*Stage 1 deadline: May 21, 2026. Data window: 2022-03-29 → 2026-03-06.*

## 1. Goal

The shared workshop question is *"Will this S&P 500 stock rise significantly tomorrow?"* Our group operationalizes this for the **Energy sector** with a daily binary classification task: at the 4 PM market close on day *t*, predict whether the close-to-close return from *t* to *t+1* will exceed **1%**. The 1% threshold was confirmed with the professor and is held fixed. Energy equities sit in a near-efficient market driven by exogenous commodity, currency, and policy shocks, so honest interpretation of modest results matters more than headline accuracy. Evaluation uses roll-forward expanding-window validation — never `train_test_split` — and every feature is constrained to use only information available by 4 PM on the prediction date.

## 2. Data source

The base dataset is `Data/sp500_Energy_dataset.csv`, an instructor-provided 4-year panel of S&P 500 Energy sector equities sourced from a standard market-data vendor. Columns: `date`, `ticker`, `sector`, OHLCV (`open`, `high`, `low`, `close`, `volume`), `daily_return_pct` (today's close-to-close return as a float percent), `1_d_return` (next-day return as a `%`-suffixed string), `vix_end_of_day`, and `sp500_return_today`. Dates are formatted `DD/MM/YYYY` and parsed with `dayfirst=True`. External Energy data sources (commodities, USD, EIA inventory, OPEC events — see §5) are pulled live during feature engineering rather than redistributed in this repo.

## 3. Data overview

The panel contains **20,929 rows × 12 columns** spanning **22 unique tickers** over **988 trading days** from **2022-03-29 to 2026-03-06** — the end date matches the professor's stated expectation exactly. There are **zero missing cells** across all 12 columns. Ticker coverage is mildly unbalanced: 20 tickers carry the full 988 rows, **CTRA** carries 961 (joined shortly after the start), and **EXE** carries only 208 rows (recent addition to the index). Tickers reporting per date range from 20 to 22 with a mode of 21, so Stage 4's roll-forward loop must accept uneven panel widths and EXE's feature pipeline must tolerate short rolling-window histories.

## 4. Candidate engineered features

We have catalogued **20 candidate features**, grouped into six buckets, with implementation deferred to Stage 3:

- **Trend (6).** Distance of close from the 5/20/50/150-day moving averages, plus the 20–50 and 50–150 day MA spreads. The 150-day MA and the 20–50 MA spread were explicitly flagged by the professor as common Energy strategies.
- **Momentum (7).** Returns over 1/3/5/10 trading days, plus RSI computed across three windows (7/14/28 days). Multi-period RSI was flagged by the professor.
- **Volatility (3).** 5- and 20-day rolling standard deviations of daily returns, plus their ratio as a vol-expansion/contraction indicator.
- **Volume (2).** Today's volume relative to its 20-day mean, and the 5-day-vs-prior-5-day mean ratio.
- **Market context (4).** VIX level (already in base data) and its 5-day change; same-day S&P return (already in base data) and 5-day cumulative S&P return.
- **Energy-specific external (7).** WTI 1-day and 5-day returns, Brent–WTI spread, Natural Gas 5-day return, USD-index 5-day change, EIA crude-stocks change, and a `days_since_last_OPEC_event` counter.

All rolling features are computed per ticker via `groupby('ticker').rolling(...)` to prevent cross-ticker leakage; the first ~150 rows per ticker will null on the longest-window features and Stage 3 will decide between dropping or imputing.

## 5. External Energy data plan

The professor noted that base OHLCV is insufficient given how exogenously Energy equities are driven — external data is the main differentiator across groups. Our Task 1.4 audit confirmed download feasibility for the following:

- **WTI (`CL=F`) and Brent (`BZ=F`) crude** via yfinance, daily, full window. Oil pricing is location-specific: WTI tracks US producers, Brent tracks internationally-exposed names; per-ticker selection is a Stage 3 decision.
- **Natural gas (`NG=F`)** via yfinance, daily, full window. Relevant for gas-heavy tickers (EQT, OKE, KMI, WMB).
- **USD Index (`DX-Y.NYB`)** via yfinance, daily, full window. Crude is priced in USD; a stronger dollar pressures crude and Energy equities.
- **EIA weekly U.S. crude stocks ex-SPR (`WCESTUS1`)** via direct XLS download — no API key required for the historical view — 206 weekly observations in window. Inventory builds are bearish, draws bullish.
- **OPEC+ events** — manually curated list of 12 major Ministerial/JMMC announcements in window. Feature is `days_since_last_announcement_date`; dates will be verified against `opec.org` before use.

**Deferred:** Baker Hughes rig count (host reachable, requires XLSX scraping; Stage 3). **Skipped:** news sentiment — at our scale (~21k ticker-date pairs) no free historical news API meets the coverage requirement, and the engineering cost of a custom corpus + FinBERT pipeline is unjustified before the baseline shows headroom.

**Look-ahead traps to respect in Stage 3.** (a) EIA files date rows by *as-of week ending*, not release date — joining naively leaks ~5 days of future info; use `release_date ≈ as_of_date + ~5 days`. (b) Commodity futures trade on a different calendar; weekend/holiday bars must be dropped, not forward-filled into next Monday's equity row. (c) OPEC features must use announcement date, not meeting date — these differ for surprise events (e.g. the 2023-04-02 voluntary cuts).

## 6. Threshold choice and justification

The 1% threshold is set by the workshop in dialogue with the professor and is fixed; Stage 1's job is to characterize the distribution this threshold sits in. Across all 20,929 (ticker, date) observations, next-day returns have mean +0.13%, median +0.11%, standard deviation 6.10%. The threshold-sensitivity table is:

| Threshold | Share of observations |
|---|---|
| Return > 0%    | 52.42% |
| **Return > 1%**  | **29.27% (positive class)** |
| Return > 1.5%  | 20.32% |
| Return > 2%    | 14.14% |

A 1% threshold yields a tractable ~30% positive class — imbalanced but not pathologically so. The 0% threshold reduces to sign-of-near-zero-mean prediction; the 2% threshold pushes positives below 15% into rare-event territory. The 1% rate also varies materially by year — 37.5% in 2022 (commodity supercycle), troughing at **23.8% in 2024**, recovering to 29.5% in 2025 — a non-stationarity that the roll-forward expanding-window evaluation in Stage 4 is specifically designed to surface.

## 7. Binary label construction

The provided `1_d_return` column is a `%`-suffixed string (e.g. `"0.59%"`) representing next-day return rounded to two decimals. We strip the `%`, divide by 100, and store the result as `next_day_return` (float). We verified the semantics by comparing each value against the same ticker's `daily_return_pct[t+1]` after a per-ticker shift; the maximum absolute difference across 20,907 comparable rows is **0.00005** — exactly the rounding gap — confirming `1_d_return[t]` is the close-to-close return from *t* to *t+1* for the same ticker. No shift correction is required. The label is then `binary_label = (next_day_return > 0.01).astype(int)`, with `THRESHOLD = 0.01` defined as a top-of-notebook constant. The 22 rows dated **2026-03-06** carry implausible values (min -92.35%, max +672.15%) — vendor placeholders for a day with no observable *t+1* in our panel. We exclude them via `df_model` (20,907 rows) that drops the last row per ticker; all Stage 4 training uses `df_model`.

## 8. Positive observation percentage

The final positive-class rate is **29.27% on the full panel (20,929 rows) and 29.25% on `df_model` (20,907 rows after excluding the 22 polluted last-day rows)**.

---

## AI usage disclosure

Per the course AI-usage policy, we used **Claude Code (Anthropic)** as a programming and writing assistant during Stage 1. Specifically: it helped scaffold the project structure (`CLAUDE.md`, `SPEC.md`, per-stage task breakdowns under `tasks/`), draft the notebook cells that verify data quality (Task 1.1), build the binary label and surface the 2026-03-06 vendor placeholder issue (Task 1.2), assemble the candidate-feature catalogue (Task 1.3), audit the external data sources and write the test cells (Task 1.4), and draft this paper section. All code was executed and numerical results were verified by the human team against the notebook output. All methodological decisions — the 1% threshold confirmation, the choice of LightGBM as primary model, the deferral of news sentiment, the per-feature rationale — were made by the team; the assistant provided drafts and implementation. Final review and submission are the responsibility of the human authors.
