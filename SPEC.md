# Project Specification — Energy Sector Stock Gap Prediction

**Course:** Fintech Workshop (3055) — Reichman University, Semester 2, 2026
**Sector:** Energy
**Threshold:** 1% next-day return (confirmed with professor)
**Team size:** 4 students
**Final submission:** Final workshop of the semester, with a **10-minute presentation**

---

## 1. Overview

### 1.1 Problem statement
Build a binary classifier that predicts whether a given S&P 500 Energy stock will rise by more than a chosen threshold the next trading day, using 4 years of daily OHLCV data plus engineered features and external Energy-sector signals.

### 1.2 Why this matters
Stock-move prediction is a canonical ML problem with real consequences (efficient markets, look-ahead bias, class imbalance). The course frames this as the capstone exercise of the data science cycle: business understanding, data collection, EDA, preprocessing, feature engineering, modeling, evaluation.

### 1.3 Differentiation
All eleven groups solve the same question on different sectors. Our project is judged on:

- Quality of analysis and EDA
- The external Energy data we choose to integrate
- Soundness of evaluation methodology
- The story told in the paper and presentation

---

## 2. Requirements

### 2.1 Functional requirements
- Predict, for each (ticker, date) pair, whether next-day return > 1% threshold.
- Prediction is made at **4 PM market close** each day, using only data available by that time.
- Train two models: **LightGBM** (primary) and **Logistic Regression** (baseline).
- Use **roll-forward (expanding window) evaluation**, not random split. Train on days 1–50 to predict day 51, then train on days 1–51 to predict day 52, and so on.
- Report average evaluation metrics across all roll-forward steps.
- Identify and justify top features for the Energy sector.

### 2.2 Non-functional requirements
- Reproducibility: the notebook must run end-to-end on a clean machine.
- AI usage must be documented per the course AI policy.
- Code lives in a single Jupyter notebook covering all stages.

### 2.3 Constraints (non-negotiable rules)
- **No random `train_test_split`** — time series only, use roll-forward expanding-window.
- **No look-ahead bias** — every feature uses only data available **by 4 PM on the prediction date**. When joining external data, verify date alignment manually.
- 4 years of data from the supplied Polygon.io dataset, **current through March 6, 2026**.
- VIX and S&P 500 returns come from yfinance as specified.

---

## 3. Data

### 3.1 Base dataset
- **File:** `Data/sp500_Energy_dataset.csv`
- **Source:** Polygon.io (daily OHLCV, purchased) + yfinance (VIX, S&P 500)
- **Coverage:** 4 years of daily data, all S&P 500 Energy tickers, **through March 6, 2026**
- **Columns:** `date, ticker, sector, open, high, low, close, volume, daily_return_pct, 1_d_return, vix_end_of_day, sp500_return_today`

### 3.2 Engineered features (candidate list)
Derived from the base data — implementation in Stage 3.

- Moving averages (5, 10, 20, 50, **150** day — 150-day MA is a common Energy strategy)
- Distance from moving average
- Differences between MAs across time horizons (e.g. 20-day vs 50-day spread)
- Rolling volatility (5, 10, 20 day)
- RSI across multiple time periods (not just 14-day)
- Return windows (1, 3, 5, 10 day returns)
- Volume changes and volume ratios (today vs N-day average)
- Lagged returns and lagged volume
- VIX-relative measures

### 3.3 External Energy-sector data (planned)
**External data is critical** — the professor explicitly noted that the base data alone is insufficient to beat the market due to efficiency. Strong external features are the main differentiator across groups.

Each source to be tested for availability, coverage, and frequency before Stage 3.

- **Natural gas prices** — `NG=F` via yfinance
- **Oil prices** — both WTI (`CL=F`) and Brent (`BZ=F`). Professor flagged that oil pricing is **location-specific**; consider whether WTI or Brent is more relevant per ticker (US producers vs international exposure).
- **US Dollar Index (DXY)** — `DX-Y.NYB`. Oil is priced in USD; strong dollar pressures oil.
- **OPEC announcements** — event-based, requires sourcing a historical event list.
- EIA weekly petroleum inventory reports.
- Baker Hughes rig counts.
- **News sentiment (stretch goal)** — possible but expensive: ~220 trading days/year × 4 years ≈ 880 days of bullish/bearish news per ticker. Only attempt if a clean historical sentiment source is found.

### 3.4 Target variable
Binary label: `1` if next-day return > 1%, else `0`. **Threshold confirmed with professor at 1%.**

Prediction is made at **4 PM market close**, so the label for day _t_ is determined by the close-to-close return from day _t_ to day _t+1_.

---

## 4. Methodology — Four Stage Workflow

Professor confirmed four stages (not five). The presentation is part of Stage 4.

### Stage 1: Data Collection & Preparation (due May 21)
- Define target, describe data, verify integrity (ticker count, date range through March 6 2026, missing values)
- List candidate engineered features and external data sources
- Construct binary label at 1% threshold
- Report positive-class %

### Stage 2: Exploratory Data Analysis (Stage 2–3 combined: 3–4 weeks)
- Shape, types, basic statistics
- Target distribution analysis (class imbalance check)
- Univariate distributions
- Correlation heatmap
- Feature-vs-target analysis
- Identify suspicious features, missing values, outliers

### Stage 3: Data Engineering & Feature Creation
- Implement engineered features from the candidate list
- Integrate external Energy data (oil, gas, DXY, OPEC, EIA, rigs)
- Clean data and handle missing values + outliers
- Iterate with Stage 2 as needed

### Stage 4: Modeling, Evaluation & Presentation
- Train LightGBM (primary) and Logistic Regression (baseline)
- Roll-forward expanding-window evaluation (train on 1–N, predict N+1, expand)
- Choose evaluation metric: **accuracy or returns-based** — professor left this open. Discuss in group based on class balance.
- Identify top features and assess whether they make sense for Energy
- Answer: did we beat the market?
- Compile final paper + build 10-minute presentation deck

---

## 5. Deliverables

- [ ] **Jupyter Notebook (.ipynb)** — full workflow, all four stages. Any Jupyter-compatible environment (VS Code, Cursor, Colab, classic Jupyter) is acceptable.
- [ ] **Final Working Paper (PDF)** — compiled incrementally, ~1–2 pages per stage
- [ ] **Presentation (PowerPoint or equivalent)** — 10 minutes, story-focused, presented in the final session

**On code review:** Professor said code review will be **minimal** unless results are exceptional or look problematic. The grade rewards thinking, methodology, and the story — not code aesthetics.

---

## 6. Repository Structure

```
energy-gap-prediction/
├── SPEC.md                          # this file
├── CLAUDE.md                        # AI context file
├── README.md                        # quick-start for collaborators
├── requirements.txt                 # Python dependencies
├── .gitignore
├── Data/
│   └── sp500_Energy_dataset.csv     # base dataset
├── Instructions/
│   ├── Final Project Guidelines.pdf
│   └── Stage 1.pdf
├── meeting prep/
│   └── kickoff_meeting_prep.md
├── notebooks/
│   └── energy_prediction.ipynb      # main project notebook
├── paper/
│   └── final_paper.md               # written sections, compiled later to PDF
├── presentation/
│   └── final_deck.pptx              # presentation, built late
└── tasks/
    └── STAGE_N.md                   # concrete work breakdown per stage
```

---

## 7. Team & Workstreams

Four members, four parallel workstreams per stage. Roles rotate or hold steady by agreement.

| Workstream | Stage 1 focus | Stage 2/3 focus | Stage 4 focus |
|---|---|---|---|
| Data verification | Ticker counts, date range, integrity | Outlier detection | Sanity checks pre-modeling |
| Threshold + label | Threshold choice, label construction | Class imbalance handling | Threshold sensitivity |
| Engineered features | Candidate list + rationale | Implementation + EDA | Feature importance |
| External data | Source identification, test downloads | Integration + alignment | Sector storytelling |

One member additionally owns the written paper section each stage; this rotates.

---

## 8. Acceptance Criteria

### Stage 1 (due May 21)
- [ ] Ticker count, date range, trading day count reported
- [ ] Engineered feature candidates listed with one-line rationale each
- [ ] External data sources identified with source + frequency + rationale
- [ ] Threshold chosen and justified
- [ ] Binary label constructed in the notebook
- [ ] Positive observation % reported
- [ ] 1–2 page written section drafted

### Stage 2 / 3
- [ ] Target distribution plot showing class balance
- [ ] Correlation heatmap
- [ ] All candidate features implemented and validated
- [ ] At least 3 external Energy datasets integrated
- [ ] Missing values and outliers handled with documented logic

### Stage 4
- [ ] Roll-forward expanding-window evaluation implemented
- [ ] Both LightGBM and Logistic Regression trained
- [ ] Metrics reported across roll-forward steps (accuracy or returns-based)
- [ ] Top features identified and interpreted for Energy
- [ ] Beat-the-market analysis included
- [ ] Compiled PDF paper finished
- [ ] 10-minute PowerPoint deck ready
- [ ] Notebook runs end-to-end on a clean environment
- [ ] AI usage documented

---

## 9. Verification

- Run notebook end-to-end before each submission
- Cross-check reported numbers in the paper against notebook outputs
- Independently verify ticker counts, date ranges, target distributions — do not trust AI assumptions
- Watch for: class imbalance, suspicious correlations, look-ahead leakage, off-by-one date errors in next-day return calculation

---

## 10. Open Questions

- [ ] Per-ticker prediction vs pooled-across-Energy prediction?
- [ ] Evaluation metric: accuracy or returns-based? (professor left open)
- [ ] How to align weekly EIA data with daily stock data (forward-fill, last-known-value)?
- [ ] How to encode event-based features (OPEC announcements) — binary flag, window indicator?
- [ ] For oil prices, which benchmark per ticker — WTI for US producers, Brent for internationally-exposed names?
- [ ] News sentiment as a stretch goal — feasible within timeline, or skip?
