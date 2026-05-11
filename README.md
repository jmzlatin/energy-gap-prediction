# Energy Sector Stock Prediction

Final project for **Fintech Workshop** (3055), Reichman University — Semester 2, 2026.

We predict whether an S&P 500 **Energy sector** stock will rise more than a chosen threshold the next trading day, using 4 years of OHLCV data plus engineered features and external Energy-sector signals.

---

## Quick start

### 1. Clone the repo
```bash
git clone <repo-url>
cd "Final Project"
```

### 2. Set up your environment
```bash
python -m venv .venv
source .venv/bin/activate          # macOS / Linux
# .venv\Scripts\activate            # Windows
pip install -r requirements.txt
```

### 3. Open the notebook
```bash
jupyter notebook notebooks/energy_prediction.ipynb
```

---

## Repository layout

```
Final Project/
├── SPEC.md                          # full project specification — read first
├── CLAUDE.md                        # context for AI assistants
├── README.md                        # this file
├── requirements.txt                 # Python dependencies
├── .gitignore
├── Data/
│   └── sp500_Energy_dataset.csv     # base dataset (committed)
├── Instructions/
│   ├── Final Project Guidelines.pdf
│   └── Stage 1.pdf
├── meeting prep/
│   └── kickoff_meeting_prep.md
├── notebooks/
│   └── energy_prediction.ipynb      # main project notebook
├── paper/
│   └── final_paper.md               # written sections, compiled to PDF
└── presentation/
    └── final_deck.pptx              # final presentation
```

---

## How we work together

### Branches
- `main` — always stable, all submissions live here
- `stage-1-<your-name>` — your branch while working on Stage 1
- One branch per stage per person, e.g. `stage-2-shani`, `stage-3-alex`

### Workflow
1. Pull `main` before starting work: `git pull origin main`
2. Create your branch: `git checkout -b stage-1-<your-name>`
3. Commit early and often with clear messages
4. Push your branch and open a **pull request** when ready
5. One teammate reviews before merging to `main`
6. Delete the branch after merge

### Commit message convention
Keep messages descriptive and stage-tagged:

```
[Stage 1] Verify ticker counts and date range
[Stage 1] Add WTI crude data source notes
[Stage 2] Plot return distribution histograms
[paper] Draft Stage 1 written section
```

### Jupyter notebook collaboration — important
Notebooks merge **badly** in git because outputs and metadata change with every run. To minimize pain:

- **Clear outputs before committing.** In Jupyter: `Cell → All Output → Clear`, then save.
- Or install `nbstripout` once: `pip install nbstripout && nbstripout --install`. It auto-strips outputs on commit.
- **Never edit the same notebook section simultaneously.** Coordinate via Slack/WhatsApp before touching shared cells.
- For experimental work, use a personal scratch notebook in `scratch/` (gitignored).

---

## Tech stack

- Python 3.10+
- Jupyter Notebook
- pandas, numpy
- matplotlib, seaborn
- scikit-learn
- lightgbm
- yfinance (external data)

See `requirements.txt` for pinned versions.

---

## The non-negotiable rules

These come from the course guidelines and apply to every commit:

- **No `train_test_split`** — time series only, use walk-forward evaluation.
- **No look-ahead bias** — every feature uses only data available at prediction time.
- **No accuracy as primary metric** — class imbalance is expected; use precision/recall/F1/ROC-AUC.
- **Document AI usage** in the final submission per the course AI policy.

---

## Stage deadlines

| Stage | Focus | Deadline |
|---|---|---|
| Stage 1 | Business Understanding & Data Collection | **May 21, 2026** |
| Stage 2 | Exploratory Data Analysis | TBD |
| Stage 3 | Preprocessing & Feature Engineering | TBD |
| Stage 4 | Modeling & Evaluation | TBD |
| Final | Compile paper + present | Final workshop |

---

## Team

| Name | Email | Stage 1 workstream |
|---|---|---|
| Shani | shanilocker@gmail.com | TBD |
| TBD | TBD | TBD |
| TBD | TBD | TBD |
| TBD | TBD | TBD |

---

## Where to look first

- **Just joined?** Read `SPEC.md` end to end.
- **Using AI tools?** Read `CLAUDE.md` so your AI has project context.
- **Preparing for today's meeting?** Read `meeting prep/kickoff_meeting_prep.md`.
- **Stuck on Stage 1 requirements?** Read `Instructions/Stage 1.pdf`.
