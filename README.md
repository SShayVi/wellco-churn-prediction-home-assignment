# WellCo Uplift Modeling — Home Assignment

Identify which members will benefit most from outreach — members who would churn **without** it but won't **with** it — and deliver a ranked outreach list using a causal uplift model.

---

## Results at a glance

| | |
|---|---|
| **Primary model** | T-learner (two LightGBM models: one per treatment group) |
| **OOF Qini coefficient** | +14.77 (T-learner) vs −25.85 (S-learner) |
| **Baseline churn model AUC** | 0.644 (LightGBM, `03_model.ipynb`) |
| **Recommended outreach n** | ~747 members (top 7.5% by uplift score) |
| **ATE** | 1.34 pp churn reduction on average |
| **Test members scored** | 10,000 (IDs 20,001–30,000) |

---

## Approach

### Timeline
- **Observation window**: Jul 1–15, 2025 — behavioural data collected (app sessions, web visits, claims)
- **Outreach event**: Jul 15, 2025 — binary treatment applied to a subset of train members
- **Churn window**: Jul 15–29, 2025 — churn label defined here

### Why uplift modeling, not churn prediction

Targeting by churn risk maximises who *might* churn — but not who *responds* to outreach. The right question is: **who will churn without outreach but won't with it?** This is the causal quantity known as the Conditional Average Treatment Effect (CATE) or individual uplift.

A member with high churn risk but no response to outreach wastes the budget. A member with moderate churn risk but strong treatment response is the right target.

### Four questions answered

| Question | Short answer | Notebook |
|---|---|---|
| **Feature selection** | Membership tenure, health content engagement (NLP + WellCo domain), app usage consistency and variance, ICD clinical flags — same 35 features used for both churn and uplift models | `02_features.ipynb` |
| **Model evaluation** | **Qini coefficient** (primary) — measures ranking quality for uplift, analogous to AUC for classification. AUUC and Uplift@k reported as secondary metrics. AUC-ROC used for the baseline churn model. | `03_uplift_model.ipynb` |
| **Outreach in modelling** | S-learner: outreach included as a training feature, score computed as predict(outreach=0) − predict(outreach=1). T-learner: outreach used only to split training data into two groups; not a predictor. T-learner selected — treatment assignment was confounded with engagement, so S-learner counterfactuals are unreliable. | `03_uplift_model.ipynb` |
| **Selecting n** | Qini marginal benefit curve: target members until marginal incremental gain approaches zero. **n ≈ 747** (top 7.5%) using OOF uplift predictions. For full cost-sensitivity analysis, see `04_outreach_sizing.ipynb`. | `03_uplift_model.ipynb` |

### Why T-learner outperforms S-learner

Treatment assignment was not fully random: app session features show standardised mean differences up to 0.50 between treated and control groups. The S-learner learns outreach as a proxy for high engagement, so flipping the outreach flag produces biased counterfactuals (OOF Qini = −25.85). The T-learner trains separate models on each group, avoiding this confound (OOF Qini = +14.77).

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
├── 01_eda.ipynb                    # Phase 1: Exploratory data analysis
├── 01_eda_v2.ipynb                 # Phase 1v2: Treatment effect heterogeneity (CATE, SMD)
├── 02_features.ipynb               # Phase 2: Feature engineering → train/test parquet
├── 03_model.ipynb                  # Phase 3: LightGBM churn model (baseline, AUC 0.644)
├── 03_uplift_model.ipynb           # Phase 3v2: S-learner + T-learner uplift model (primary)
├── 04_outreach_sizing.ipynb        # Phase 4: Outreach sizing — cost-benefit framework
├── predictions.csv                 # Churn-based ranking (baseline, kept for reference)
├── predictions_uplift.csv          # Uplift-based ranking (primary deliverable)
├── WellCo_Churn_Presentation.pptx  # Executive presentation (5 slides)
├── train_features.parquet          # Engineered train features (with churn + outreach labels)
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

Open and run in sequence:

```
01_eda.ipynb → 01_eda_v2.ipynb → 02_features.ipynb → 03_model.ipynb → 03_uplift_model.ipynb
```

`02_features.ipynb` saves `train_features.parquet` and `test_features.parquet` which all downstream notebooks load. `03_uplift_model.ipynb` saves `predictions_uplift.csv`.

---

## Output

### Primary: `Shay_Assignment/predictions_uplift.csv`

All 10,000 test members ranked by predicted uplift (T-learner):

| Column | Description |
|---|---|
| `member_id` | Member identifier |
| `uplift_score` | Predicted churn reduction from outreach — T0(x) − T1(x) |
| `churn_risk` | Predicted churn probability without outreach — T0(x) |
| `t_uplift_score` | T-learner uplift (same as `uplift_score`) |
| `s_uplift_score` | S-learner uplift (included for comparison) |
| `rank` | Priority rank by uplift score (1 = most responsive to outreach) |

**Recommended outreach list**: members with `rank ≤ 747` (top 7.5% by predicted uplift).

### Reference: `Shay_Assignment/predictions.csv`

Churn-based ranking from the Phase 3 baseline model (`churn_score`, `rank`). Retained for comparison; superseded by the uplift ranking for outreach targeting.
