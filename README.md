# WellCo Churn Prediction — Home Assignment

Predict member churn risk for WellCo and deliver a ranked list of members for targeted outreach, along with a principled recommendation for the optimal outreach size *n*.

---

## Results at a glance

| | |
|---|---|
| **Model** | LightGBM binary classifier |
| **OOF AUC-ROC** | 0.644 |
| **OOF PR-AUC** | 0.306 |
| **Recommended n** | 4,369 members (43.7% of test population) |
| **Expected churns prevented** | ~79 (vs no outreach) |
| **Test members scored** | 10,000 (IDs 20,001–30,000) |

---

## Approach

### Timeline
- **Observation window**: Jul 1–15, 2025 — behavioural data collected (app sessions, web visits, claims)
- **Outreach event**: Jul 15, 2025 — binary treatment applied to a subset of train members
- **Churn window**: Jul 15–29, 2025 — churn label defined here

### Four questions answered

| Question | Short answer | Notebook |
|---|---|---|
| **Feature selection** | Membership tenure, health content engagement (NLP + WellCo domain), app usage consistency and variance, ICD clinical flags | `02_features.ipynb` |
| **Model evaluation** | AUC-ROC (primary — task is ranking, not classification) + PR-AUC (secondary — accounts for 20% class imbalance) | `03_model.ipynb` |
| **Outreach in modelling** | Included as a training feature so the model learns its protective effect; set to 0 at scoring time so test scores represent pre-outreach churn risk | `03_model.ipynb` |
| **Selecting n** | Expected-value framework: outreach member *i* only if their predicted churn reduction ΔP exceeds the cost ratio *c = cost / retention_value*. Since *c* is unknown, a sensitivity table is provided across 0.5%–10%; **n = 4,369 uses an assumed c = 2%** (see note below) | `04_outreach_sizing.ipynb` |

### Note on the cost assumption
The assignment states that outreach cost is *"constant but unknown."* We did not invent a specific cost figure. Instead we parameterised the problem by the **cost ratio** `c = cost_per_outreach / value_of_retaining_one_member` and produced a full sensitivity table. The headline recommendation of **n = 4,369** uses **c = 2%** as a stated baseline — a reasonable assumption for a digital-first programme where outreach is typically an automated message or short call. WellCo should substitute their actual numbers into the sensitivity table to get the right *n* for their context.

---

## Features (35 total)

| Group | Features |
|---|---|
| Membership | `tenure_days` |
| App usage (9) | `session_count`, `active_days_app`, `last_session_recency_days`, `session_trend_3d`, `session_trend_7d`, `session_std_daily`, `max_sessions_day`, `session_cv`, `weekend_session_ratio` |
| WellCo web (5) | `wellco_visit_count`, `wellco_active_days`, `wellco_last_visit_recency_days`, `wellco_visit_trend`, `n_wellco_categories` |
| NLP web (15) | 7 keyword category counts (diabetes, hypertension, nutrition, exercise, sleep, stress, heart), `n_health_categories`, `health_content_ratio`, 5 LSA components (TF-IDF + TruncatedSVD, fitted on train only) |
| Claims (6) | `has_E11.9`, `has_I10`, `has_Z71.3`, `n_clinical_conditions`, `total_claims_deduped`, `has_noise_icd` |

---

## Repository structure

```
Shay_Assignment/
├── 01_eda.ipynb                    # Exploratory data analysis
├── 02_features.ipynb               # Feature engineering → train/test parquet
├── 03_model.ipynb                  # LightGBM training, CV, evaluation
├── 04_outreach_sizing.ipynb        # Optimal n — expected-value framework
├── predictions.csv                 # All 10k test members ranked by churn risk
├── WellCo_Churn_Presentation.pptx  # Executive presentation (5 slides)
├── train_features.parquet          # Engineered train features
├── test_features.parquet           # Engineered test features
├── requirements.txt                # Python dependencies
└── claude_code_util/               # Claude Code workflow documentation
```

> Raw data files (`assignment_instructions/`) are **not** included in this repository.

---

## Setup

### Prerequisites
- Python 3.9+
- Data files placed at:
  - `assignment_instructions/train/` — `churn_labels.csv`, `app_usage.csv`, `web_visits.csv`, `claims.csv`
  - `assignment_instructions/test/` — `test_members.csv`, `test_app_usage.csv`, `test_web_visits.csv`, `test_claims.csv`

### Install dependencies

```bash
pip install -r Shay_Assignment/requirements.txt
```

### Run the notebooks (in order)

```bash
jupyter notebook
```

Open and run each notebook in `Shay_Assignment/` in sequence:

```
01_eda.ipynb → 02_features.ipynb → 03_model.ipynb → 04_outreach_sizing.ipynb
```

Each notebook saves intermediate artefacts (`train_features.parquet`, `test_features.parquet`, `predictions.csv`) that the next notebook loads.

---

## Output

`Shay_Assignment/predictions.csv` — all 10,000 test members ranked by churn risk:

| Column | Description |
|---|---|
| `member_id` | Member identifier |
| `churn_score` | Predicted churn probability with `outreach = 0` (pre-outreach risk) |
| `delta_p` | Expected churn reduction if this member receives outreach |
| `rank` | Priority rank by churn score (1 = highest risk) |
| `rank_by_effect` | Priority rank by ΔP (1 = most responsive to outreach) |

**Recommended outreach list**: members with `rank ≤ 4,369` (assuming cost ratio c = 2% — see note above).
