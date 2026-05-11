# Kickoff Meeting Prep — Stage 1

**Date:** May 11, 2026
**Stage 1 submission deadline:** May 21, 2026
**Sector:** Energy

---

## What you need to walk in understanding

If you can explain each of the items below in one sentence, you are ready for the meeting.

### The question your group is solving
"Will an Energy stock rise by more than X% tomorrow?" — a binary classification problem. Every group in the class is solving the same question on a different sector. What makes your project differentiated is the **quality of analysis, the external data you bring in, and your evaluation** — not the question itself.

### What "significantly" means and why
Default threshold is **next-day return > 1%**. The reason it is 1% and not 0% is that a 0.1% move is indistinguishable from noise — labeling it as "up" trains the model to learn from noise. The 1% threshold ensures every positive label is a real move. You are allowed to argue for a different threshold for Energy, but you have to justify it with data on Energy's volatility.

### What data you already have
A CSV with 4 years of daily OHLCV for every Energy ticker in the S&P 500, plus VIX and S&P 500 daily returns. That is your **base data**. From the base data you will later derive **engineered features** (moving averages, RSI, rolling vol, etc.). On top of that you must add **external sector data** — for Energy, that means oil prices and similar.

### Exactly what Stage 1 is asking for (the May 21 submission)
A 1–2 page written section plus notebook code covering eight items:

1. Goal
2. Data source
3. Data overview (unique tickers, occurrences, date range, total trading days)
4. Candidate engineered features
5. External sector data plan
6. Threshold choice + justification
7. The constructed binary label
8. % of positive observations

Stage 1 is **planning and verification**, not modeling. You are not building features yet — you are listing them.

---

## The three decisions your group must leave the meeting with

1. **The threshold** — 1% or something else, with a reason.
2. **Initial list of external Energy data sources** (oil prices, gas prices, USD index, EIA inventory, OPEC events, rig counts) and who is responsible for sourcing each.
3. **Workstream owners** — who owns each of the four workstreams (data verification, threshold/label, engineered features, external data) and who owns the written section.

---

## The non-negotiable rules

- **No random `train_test_split`** — it is time series, you will use walk-forward evaluation later.
- **No look-ahead bias** — every feature must use only data available at the moment of prediction.
- **AI policy** — you are encouraged to use Claude/ChatGPT as a coding partner, but you must document how you used it in the submission.

---

## The mental model for what you are really doing

You are trying to beat an efficient market using public data — which is genuinely hard, because public data is already priced in. Modest results are expected. The grade rewards good thinking, clean methodology, and interesting external data, not a model that magically prints money. Do not let your group commit to "we'll beat the S&P by 20%" — that is a red flag, not a goal.

---

## Two things worth raising in the meeting that groups commonly miss

- **Class imbalance** — if only ~25% of days have a >1% move, your model will be tempted to predict "no" every day and look 75% accurate. Decide now that accuracy is the wrong metric.
- **Stock-level vs sector-level** — you have many tickers in one sector. Decide whether you are predicting per-ticker or pooling across all Energy tickers. The Guidelines do not dictate this, but your group should be aligned before Stage 2.

---

## Stage 1 timeline (10 days)

| Dates | Workstream |
|---|---|
| May 11 | Kickoff meeting — divide roles |
| May 12–13 | Verify the base dataset |
| May 13–14 | Decide threshold + build binary label |
| May 14–15 | List candidate engineered features |
| May 14–16 | Identify external Energy data sources |
| May 17–19 | Draft the 1–2 page written section |
| May 19–20 | Group review and integration |
| May 21 | **Submit Stage 1** |
