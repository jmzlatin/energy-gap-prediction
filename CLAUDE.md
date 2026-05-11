# CLAUDE.md

Context for AI assistants (Claude, ChatGPT, etc.) working on this project.

## Project at a glance

This is the final project for **Fintech Workshop** at Reichman University. It is a group data science project (4 students) building a binary classifier that predicts whether an S&P 500 **Energy sector** stock will rise more than a chosen threshold the next trading day.

The shared question across all groups in the class is: **"Will the stock rise significantly tomorrow?"** We're solving it for the Energy sector.

Read `SPEC.md` for the full specification. Read `meeting prep/kickoff_meeting_prep.md` for the immediate Stage 1 context.

## Where things live

- `Data/sp500_Energy_dataset.csv` — base dataset (4 years OHLCV + VIX + S&P)
- `Instructions/Final Project Guidelines.pdf` — full project rules
- `Instructions/Stage 1.pdf` — Stage 1 requirements
- `notebooks/energy_prediction.ipynb` — main project notebook (to be created)
- `paper/final_paper.md` — written sections, compiled to PDF at the end
- `presentation/final_deck.pptx` — final presentation, built late
- `meeting prep/` — meeting notes and prep docs

## Tech stack

- **Python 3** in **Jupyter Notebook**
- **pandas** for data wrangling
- **numpy** for numerics
- **matplotlib** / **seaborn** for plots
- **scikit-learn** for preprocessing + Logistic Regression baseline
- **lightgbm** for the primary model
- **yfinance** for VIX, S&P 500, and external commodity data
- Possibly **fredapi** or direct EIA API for external Energy data

## The five-stage workflow

The project follows the data science cycle. Each stage produces both notebook code and a written paper section.

1. **Stage 1 — Business Understanding & Data Collection** (deadline May 21, 2026)
2. **Stage 2 — Exploratory Data Analysis**
3. **Stage 3 — Preprocessing & Feature Engineering** (iterative with Stage 2)
4. **Stage 4 — Modeling & Evaluation**
5. **Stage 5 — Compile final paper + present**

## Non-negotiable rules — do not violate these

- **No `train_test_split`.** Data is time series. Use **walk-forward evaluation**: train on expanding windows, test on the next unseen period.
- **No look-ahead bias.** Every feature must use only data available at the moment of prediction. If you compute a rolling mean, it must NOT include the current day's close when predicting that day.
- **No accuracy as primary metric.** Class imbalance is expected. Prefer precision, recall, F1, or ROC-AUC and justify the choice.
- **AI usage must be documented** in the final submission per the course AI policy.

## Energy sector context

When working on features, EDA, or the writeup, remember what actually moves Energy stocks:

- **Oil prices** — WTI and Brent. Tightly correlated with most Energy-sector equities.
- **Natural gas prices** — relevant for utilities-adjacent and gas-heavy producers.
- **USD index (DXY)** — oil is priced in USD; strong dollar tends to pressure oil.
- **OPEC decisions** — supply shocks.
- **US shale / rig counts** (Baker Hughes weekly).
- **EIA weekly petroleum inventories** — supply signal.
- **Geopolitical events** — Middle East tensions, Russia, sanctions.

When suggesting features or interpreting model output, ground the explanation in these mechanics.

## Coding conventions

- One main notebook, sections clearly separated by markdown headers (Stage 1, Stage 2, …).
- Imports at the top in a single cell.
- Constants (threshold, walk-forward window sizes, file paths) at the top, not buried.
- Save engineered DataFrames to disk only if expensive to recompute.
- Use descriptive variable names: `next_day_return`, `binary_label`, not `y` or `tgt`.
- Comment any non-obvious financial reasoning ("RSI uses 14-day window — standard convention").

## What "done" looks like for each stage

Each stage submission includes:

1. Updated notebook with the stage's code and outputs
2. A 1–2 page written section added to `paper/final_paper.md`
3. All checkpoints from the Acceptance Criteria in `SPEC.md` for that stage

## Important domain reminders

- Energy stocks are **more volatile** than market average. The default 1% threshold may be too low — analyze the return distribution before committing.
- A 0.1% move is **noise**, not signal. Don't lower the threshold to inflate positive-class size.
- The market is **efficient**. Beating it with public data is genuinely hard. Modest results are fine; honest interpretation matters more than impressive metrics.
- Class imbalance is the **default expectation**, not a surprise. Plan for it from Stage 1.

## When in doubt

- Check `SPEC.md` for the formal definition.
- Check `Instructions/` for the original course documents.
- Verify numbers independently — do not trust assumptions in code or AI output.
- Ask: does this feature use future information? If yes, it's a leak.
