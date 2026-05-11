# Project Specification — Energy Sector Stock Gap Prediction

**Course:** Fintech Workshop (3055) — Reichman University, Semester 2, 2026
**Sector:** Energy
**Team size:** 4 students
**Final submission:** Final workshop of the semester

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
- Predict, for each (ticker, date) pair, whether next-day return > threshold.
- Train two models: **LightGBM** (primary) and **Logistic Regression** (baseline).
- Use **walk-forward evaluation**, not random split.
- Report average evaluation metrics across all walk-forward steps.
- Identify and justify top features for the Energy sector.

### 2.2 Non-functional requirements
- Reproducibility: the notebook must run end-to-end on a clean machine.
- AI usage must be documented per the course AI policy.
- Code lives in a single Jupyter notebook covering all stages.

### 2.3 Constraints (non-negotiable rules)
- **No random `train_test_split`** — time series only, use walk-forward.
- **No look-ahead bias** — every feature uses only data available at prediction time.
- 4 years of data, no more, no less, from the supplied Polygon.io dataset.
- VIX and S&P 500 returns come from yfinance as specified.

---

## 3. Data

### 3.1 Base dataset
- **File:** `Data/sp500_Energy_dataset.csv`
- **Source:** Polygon.io (daily OHLCV) + yfinance (VIX, S&P 500)
- **Coverage:** ~4 years of daily data, all S&P 500 Energy tickers
- **Columns:** `date, ticker, sector, open, high, low, close, volume, daily_return_pct, 1_d_return, vix_end_of_day, sp500_return_today`

### 3.2 Engineered features (candidate list)
Derived from the base data — implementation in Stage 3.

- Moving averages (5, 10, 20, 50 day)
- Distance from moving average
- Rolling volatility (5, 10, 20 day)
- RSI (14-day standard)
- Return windows (1, 3, 5, 10 day returns)
- Volume ratios (today vs N-day average)
- Lagged returns and lagged volume
- VIX-relative measures

### 3.3 External Energy-sector data (planned)
Each source to be tested for availability, coverage, and frequency before Stage 3.

- WTI crude oil (`CL=F` via yfinance)
- Brent crude oil (`BZ=F` via yfinance)
- Natural gas (`NG=F` via yfinance)
- US Dollar Index — DXY (`DX-Y.NYB`)
- EIA weekly petroleum inventory reports
- Baker Hughes rig counts
- OPEC announcements (event-based)

### 3.4 Target variable
Binary label: `1` if next-day return exceeds threshold, else `0`.

- **Default threshold:** 1%
- **Energy-specific consideration:** Energy stocks are more volatile than market average; a higher threshold (1.5% or 2%) may be justified. Decision to be made after analyzing return distribution.

---

## 4. Methodology — Five Stage Workflow

### Stage 1: Business Understanding & Data Collection
- Define target, describe data, verify integrity (ticker count, date range, missing values)
- List candidate engineered features and external data sources
- Choose threshold and construct binary label
- Report positive-class %

### Stage 2: Exploratory Data Analysis
- Shape, types, basic statistics
- Target distribution analysis (class imbalance check)
- Univariate distributions
- Correlation heatmap
- Feature-vs-target analysis
- Identify suspicious features, missing values, outliers

### Stage 3: Preprocessing & Feature Engineering
- Implement engineered features from the candidate list
- Integrate external Energy data
- Clean data and handle missing values + outliers
- Iterate with Stage 2 as needed

### Stage 4: Modeling & Evaluation
- Train LightGBM (primary) and Logistic Regression (baseline)
- Walk-forward evaluation
- Report metrics chosen to fit the problem (likely precision, recall, F1, ROC-AUC — not accuracy due to class imbalance)
- Identify top features and assess whether they make sense for Energy
- Answer: did we beat the market?

### Stage 5: Compile and Present
- Merge written sections into final PDF paper
- Build PowerPoint deck
- Present in final class

---

## 5. Deliverables

- [ ] **Jupyter Notebook (.ipynb)** — full workflow, all five stages
- [ ] **Final Working Paper (PDF)** — compiled incrementally, ~1–2 pages per stage
- [ ] **Presentation (PowerPoint)** — story-focused, presented in the final session

---

## 6. Repository Structure

```
Final Project/
├── SPEC.md                          # this file
├── CLAUDE.md                        # AI context file
├── README.md                        # quick-start for collaborators
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
└── presentation/
    └── final_deck.pptx              # presentation, built late
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
- [ ] Walk-forward evaluation implemented
- [ ] Both LightGBM and Logistic Regression trained
- [ ] Metrics reported across walk-forward steps
- [ ] Top features identified and interpreted for Energy
- [ ] Beat-the-market analysis included

### Final
- [ ] Compiled PDF paper
- [ ] PowerPoint with story narrative
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
- [ ] Final threshold value: 1%, 1.5%, or 2%?
- [ ] How to align weekly EIA data with daily stock data (forward-fill, last-known-value)?
- [ ] How to encode event-based features (OPEC announcements) — binary flag, window indicator?
