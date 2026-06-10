# WellCo Churn Prediction — Home Assignment

Predict member churn risk for WellCo and determine the optimal outreach list size **n** for targeted intervention.

---

## Approach

### Observation window
Behavioral data (app sessions, web visits, claims) is collected over **Jul 1–15, 2025** (14 days).  
Outreach is applied on Jul 15 (binary treatment).  
Churn is measured over **Jul 15–29, 2025**.

### Modelling strategy
1. **EDA** — understand data distributions, coverage, quality issues, and univariate churn signals
2. **Feature engineering** — build member-level features from raw event tables
3. **Model** — LightGBM binary classifier; outreach included as a training feature to model its effect, but **excluded at scoring time** (test members have not yet received outreach)
4. **Outreach sizing** — select optimal *n* using expected-value / cost-benefit analysis on model scores

### Four questions answered
| Question | Location |
|---|---|
| Feature selection | `Shay_Assignment/02_features.ipynb` |
| Model evaluation | `Shay_Assignment/03_model.ipynb` |
| Using outreach in modelling | `Shay_Assignment/03_model.ipynb` |
| Selecting n | `Shay_Assignment/04_outreach_sizing.ipynb` |

---

## Repository structure

```
Shay_Assignment/
├── 01_eda.ipynb              # Exploratory data analysis
├── 02_features.ipynb         # Feature engineering
├── 03_model.ipynb            # Model training & evaluation
├── 04_outreach_sizing.ipynb  # Optimal n selection
├── predictions.csv           # Top n test members (member_id, score, rank)
└── requirements.txt          # Python dependencies
```

> Raw data files are **not** included in this repository.

---

## Setup

### Prerequisites
- Python 3.9+
- The four training/test CSV files placed under `assignment_instructions/train/` and `assignment_instructions/test/`

### Install dependencies

```bash
pip install -r Shay_Assignment/requirements.txt
```

### Run the notebooks (in order)

```bash
jupyter notebook
```

Open and run each notebook in `Shay_Assignment/` in sequence:  
`01_eda.ipynb` → `02_features.ipynb` → `03_model.ipynb` → `04_outreach_sizing.ipynb`

Each notebook is self-contained and saves intermediate artefacts (processed dataframes, model file) that the next notebook picks up.

---

## Output

`Shay_Assignment/predictions.csv` — top *n* test members recommended for outreach:

| Column | Description |
|---|---|
| `member_id` | Member identifier |
| `churn_score` | Model probability of churn (0–1) |
| `rank` | Priority rank (1 = highest risk) |
