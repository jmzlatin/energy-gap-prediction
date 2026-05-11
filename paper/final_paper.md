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

# Stage 2: Exploratory Data Analysis

*Stages 2 and 3 are combined per professor instruction and run roughly May 22 – June 19, 2026; Stage 2 emphasis is the first half of that window.*

## 1. Data quality

The base panel is clean on every silent-failure check. There are zero duplicate `(ticker, date)` keys, zero OHLC inversions (`high < low`, `close` outside `[low, high]`, `open` outside `[low, high]`), zero non-positive volume rows, and zero in-window trading-day gaps for any of the 22 tickers (Task 2.1). Trading-day spacing is consistent with the NYSE calendar — gap distribution between consecutive panel dates is **769 / 13 / 177 / 28** for 1/2/3/4-day gaps respectively, no gaps exceeding 4 calendar days (Task 2.6). Both partial-coverage tickers are contiguous from their first appearance: CTRA from 2022-05-06 (961 sessions) and EXE from 2025-05-08 (208 sessions). Outlier counts under the 1.5×IQR rule are non-trivial — 4.56% panel-wide on `daily_return_pct`, dropping to ~4% per-ticker on average — but every flagged row is a real market event (TPL high-price prints, the 2025-04-03 APA −16.48% tariff selloff, the 2025-04-08 VIX 52.33 peak). **No row-level cleaning is required before Stage 3 feature engineering.**

## 2. Target distribution and metric implications

On `df_model`, the positive-class rate is **29.25%**, putting the constant-zero predictor at **70.75% accuracy** — a baseline any candidate model must beat by a meaningful margin to be useful. The rate varies materially by year (**37.53% in 2022**, **23.79% in 2024** trough, **37.32% in early 2026**; range 13.74 pp) and is non-stationary on a 60-day rolling window (**17.30% to 43.53%**, std 5.73 pp). Cross-ticker spread is **15.20 pp**: TPL leads at 36.88%, CVX trails at 21.68%; the bottom is dominated by the diversified majors and the highest-positive ticker is the royalty-trust play TPL. Day-of-week is essentially flat (~3 pp range, 27.7% Wed to 30.9% Thu) and is included only as a control. Month-of-year shows an **8.70 pp spread** with December a clear trough (24.41%) and March a peak (33.11%) — a calendar-month feature, not just a control, will go into Stage 3. Given the 70.75% accuracy floor and the documented regime drift, we recommend that Stage 4 lead with a **returns-based** primary metric (simulated long-only P&L under the roll-forward expanding-window walk-forward), with **F1, precision, recall, and ROC-AUC** as secondary diagnostics. Raw accuracy is rejected as a primary metric because the marginal improvements above 70.75% are hard to communicate honestly.

## 3. Univariate findings

OHLC distributions are near-identical, right-skewed (skew ≈ 1.8–1.9, excess kurtosis ≈ 6.5) with a long tail of high-priced TPL observations; the tail rows are real prints, not data errors — Stage 3 will use ticker-relative price features only, never raw cross-ticker levels. Volume is heavy-right-skewed in raw form (skew 2.47, excess kurtosis 13.29); a `log(volume + 1)` transform pulls these to **−1.01** and **1.70** respectively — much closer to symmetric with a mild left tail driven by occasional low-volume sessions, and that is the form Stage 3 will use. `daily_return_pct` is centered near zero (std 2.17%) with a mildly negative skew (−0.23) and excess kurtosis ~4.9 — the typical heavy-tailed shape of daily equity returns; tail rows are real market events (TPL +24.26% on 2024-06-10; APA −16.48% on 2025-04-03 tariff selloff). VIX is right-skewed (1.42 / 3.13) with values clustered in the 14–20 range and one peak at **52.33 on 2025-04-08** (the same tariff-selloff window). `sp500_return_today` is close to symmetric (skew 0.22, std 1.09%, excess kurtosis ~8.2). No column requires removal; volume requires log-transform; returns and VIX are usable as-is with the understanding that Stage 3 will derive their changes and rolling moments rather than relying on the levels alone.

## 4. Correlation analysis

Pearson correlations against `binary_label` are uniformly small, with one exception worth committing to: **`vix_end_of_day` correlates at r = +0.130** — the strongest linear signal in the base panel. Higher-VIX days are more likely to be followed by a >1% Energy move, which is consistent with volatility regime (more variance in the cross-section produces more observations clearing any fixed threshold). The second-largest signal is **`sp500_return_today` at r = −0.048**, a small market-wide mean-reversion hint. All other base features fall below `|r| < 0.02` to the binary target; against the continuous `next_day_return` the correlations are weaker still (`vix_end_of_day` r = +0.035, all others `|r| < 0.03`), implying that the VIX signal lives at the threshold-crossing margin — VIX widens the *spread* of next-day returns rather than shifting their *center*. The only meaningful cross-feature collinearity outside the trivial OHLC block is `volume` vs `log_volume` at r = **+0.81** (Stage 3 keeps only the log form). No independent feature pair exceeds the `|r| > 0.9` redundancy threshold.

## 5. Feature-vs-target signal

Effect sizes (Cohen's *d* on label=1 vs label=0 conditional distributions) rank consistently with the correlation analysis:

| Feature              | Cohen's *d* | Welch *t* | Interpretation |
|----------------------|-----------:|----------:|----------------|
| `vix_end_of_day`     | **+0.28** | 17.7 | volatility regime, strongest single feature |
| `vol_20d` (preview)  | **+0.25** | 15.8 | realized vol, parallel signal to VIX |
| `sp500_return_today` | **−0.10** | −6.7 | small market-wide mean reversion |
| `ret_5d` (preview)   | −0.05 | −3.3 | faint 5-day mean reversion |
| `high`, `close`, `log_volume`, `daily_return_pct` | < ±0.025 | flat | no usable separation |

Drilling into VIX by decile confirms the regime story: positive-class rate climbs from **21.8% at the lowest VIX decile (11.86–13.33)** to **42.54% at the highest (26.27–52.33)** — nearly double — with a noisy-but-monotone trajectory in between. The two preview-engineered features (`vol_20d` = 20-day rolling std of `daily_return_pct`, and `ret_5d` = 5-day cumulative return, both computed per ticker without leakage) suggest that **volatility-regime features are the dominant signal lever in the base panel**, with multi-day mean-reversion adding a faint second-tier contribution.

## 6. EDA → Stage 3 handoff

Stage 3 begins with the following plan: (a) the highest-priority engineered features are VIX level and VIX 5-day change, rolling realized volatility (5- and 20-day stds of `daily_return_pct`), and an S&P-relative return (Energy return minus same-day S&P return) — these are the categories where Stage 2 surfaced real signal; (b) trend/momentum features (5/20/50/150-day MAs, distance from MA, MA spreads, RSI multi-period, multi-day returns) go in as a second tier and will be evaluated against the volatility tier rather than assumed independent; (c) volume features enter only as `log(volume+1)` or as ratios against rolling means; (d) all rolling features are computed per ticker with `min_periods == window` so EXE's short history nulls cleanly rather than leaking. The most urgent external integration is **WTI/Brent** — the location-specific oil benchmark map (WTI for US producers, Brent for internationally-exposed names) is the single largest lever for breaking past the near-zero base-feature signal the correlation analysis revealed.

---

## AI usage disclosure

Per the course AI-usage policy, we used **Claude Code (Anthropic)** as a programming and writing assistant during Stages 1 and 2. **Stage 1:** the assistant scaffolded the project structure (`CLAUDE.md`, `SPEC.md`, per-stage task breakdowns under `tasks/`), drafted the notebook cells that verify data quality (Task 1.1), built the binary label and surfaced the 2026-03-06 vendor placeholder issue (Task 1.2), assembled the candidate-feature catalogue (Task 1.3), audited the external data sources and wrote the test cells (Task 1.4), and drafted the Stage 1 paper section. **Stage 2:** the assistant drafted the data-quality audit cells (Task 2.1), the class-balance breakdowns by year / ticker / day-of-week / month and the rolling-rate analysis (Task 2.2), the univariate histograms and IQR / z-score outlier tables (Tasks 2.3, 2.6), the Pearson correlation matrix and heatmap (Task 2.4), the conditional distribution plots and Welch's t-tests (Task 2.5), and this paper section. All code was executed and every numerical claim in this paper was verified against the executed notebook output. All methodological decisions — the 1% threshold, the evaluation-metric recommendation, the prioritization of volatility-regime features for Stage 3, the choice of LightGBM as primary model, the per-feature rationale — were made by the team; the assistant provided drafts and implementation. Final review and submission are the responsibility of the human authors.
