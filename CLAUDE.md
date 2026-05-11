# CLAUDE.md

Context for AI assistants (Claude, ChatGPT, etc.) working on this project.

## Project at a glance

This is the final project for **Fintech Workshop** at Reichman University. It is a group data science project (4 students) building a binary classifier that predicts whether an S&P 500 **Energy sector** stock will rise more than **1%** the next trading day. Prediction is made daily at **4 PM market close**.

The shared question across all groups in the class is: **"Will the stock rise significantly tomorrow?"** We're solving it for the Energy sector with a 1% threshold (confirmed with professor).

The dataset is current through **March 6, 2026**.

Read `SPEC.md` for the full specification. Read `meeting prep/kickoff_meeting_prep.md` for the immediate Stage 1 context.

## Where things live

- `SPEC.md` — formal project specification (what we're building)
- `tasks/STAGE_N.md` — concrete work breakdown per stage (what to actually do). **Start here if you've been asked to work on a task.**
- `Data/sp500_Energy_dataset.csv` — base dataset (4 years OHLCV + VIX + S&P)
- `Instructions/Final Project Guidelines.pdf` — full project rules
- `Instructions/Stage 1.pdf` — Stage 1 requirements
- `notebooks/energy_prediction.ipynb` — main project notebook (to be created)
- `paper/final_paper.md` — written sections, compiled to PDF at the end
- `presentation/final_deck.pptx` — final presentation, built late
- `meeting prep/` — meeting notes and prep docs

## Tech stack

- **Python 3** in any Jupyter-compatible environment (classic Jupyter, VS Code, Cursor, Colab — professor said all are fine)
- **pandas** for data wrangling
- **numpy** for numerics
- **matplotlib** / **seaborn** for plots
- **scikit-learn** for preprocessing + Logistic Regression baseline
- **lightgbm** for the primary model
- **yfinance** for VIX, S&P 500, and external commodity data
- Possibly **fredapi** or direct EIA API for external Energy data

## The four-stage workflow

The project follows the data science cycle. Each stage produces both notebook code and a written paper section.

1. **Stage 1 — Data Collection & Preparation** (deadline May 21, 2026)
2. **Stage 2 — Exploratory Data Analysis** (Stages 2–3 combined: 3–4 weeks)
3. **Stage 3 — Data Engineering & Feature Creation** (iterative with Stage 2)
4. **Stage 4 — Modeling, Evaluation & 10-minute presentation** (includes compiling the final paper and presenting)

## Non-negotiable rules — do not violate these

- **No `train_test_split`.** Data is time series. Use **roll-forward expanding-window evaluation**: train on days 1–N, predict day N+1, then train on 1–(N+1) and predict N+2, and so on.
- **No look-ahead bias.** Every feature must use only data available **by 4 PM on the prediction date**. When joining external data (OPEC events, EIA reports, etc.), verify date alignment manually — easy to silently leak future info here.
- **AI usage must be documented** in the final submission per the course AI policy.

## Evaluation metric — open decision

The professor explicitly left this open: **accuracy or returns-based** are both acceptable. Discuss with the group based on the actual class balance after Stage 1. Accuracy can be misleading when classes are heavily imbalanced — if positive class is <20%, prefer precision/recall/F1/ROC-AUC or a returns-based metric (simulated P&L of taking long positions when the model predicts 1). Document the rationale either way.

## Energy sector context

When working on features, EDA, or the writeup, remember what actually moves Energy stocks:

- **Oil prices** — WTI (`CL=F`) and Brent (`BZ=F`). Tightly correlated with most Energy equities. The professor flagged that oil pricing is **location-specific**: use WTI for US producers, Brent for internationally-exposed names.
- **Natural gas prices** — `NG=F`. Relevant for utilities-adjacent and gas-heavy producers.
- **USD index (DXY)** — oil is priced in USD; strong dollar tends to pressure oil.
- **OPEC decisions** — supply shocks. Event-based, needs historical event list.
- **US shale / rig counts** (Baker Hughes weekly).
- **EIA weekly petroleum inventories** — supply signal.
- **Geopolitical events** — Middle East tensions, Russia, sanctions.
- **News sentiment** — stretch goal. ~880 trading days × number of tickers makes this expensive; only attempt with a clean historical sentiment source.

**External data is critical to this project.** The professor explicitly noted that base OHLCV data alone is insufficient because the market is efficient. The quality and creativity of external Energy data is the main differentiator across groups.

When suggesting features or interpreting model output, ground the explanation in these mechanics.

## Coding conventions

- One main notebook, sections clearly separated by markdown headers (Stage 1, Stage 2, …).
- Imports at the top in a single cell.
- Constants (`THRESHOLD = 0.01`, roll-forward window sizes, file paths) at the top, not buried.
- Save engineered DataFrames to disk only if expensive to recompute.
- Use descriptive variable names: `next_day_return`, `binary_label`, not `y` or `tgt`.
- Comment any non-obvious financial reasoning ("RSI uses 14-day window — standard convention").

## What "done" looks like for each stage

Each stage submission includes:

1. Updated notebook with the stage's code and outputs
2. A 1–2 page written section added to `paper/final_paper.md`
3. All checkpoints from the Acceptance Criteria in `SPEC.md` for that stage

## Important domain reminders

- The **1% threshold is confirmed** — don't relitigate it. The task is to model accurately given that target.
- The market is **efficient**. Beating it with public data is genuinely hard. Modest results are fine; honest interpretation matters more than impressive metrics. External data is the main lever.
- Class imbalance is the **default expectation**, not a surprise. Check it in Stage 1 and use it to inform the evaluation-metric decision.
- Code review will be **minimal** per the professor — focus on thinking and methodology, not code aesthetics.

## When in doubt

- Check `SPEC.md` for the formal definition.
- Check `Instructions/` for the original course documents.
- Verify numbers independently — do not trust assumptions in code or AI output.
- Ask: does this feature use future information? If yes, it's a leak.
