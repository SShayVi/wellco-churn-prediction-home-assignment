# WellCo Churn Prediction

## Goal
Predict member churn risk and determine optimal outreach size n for targeted outreach.

## Data
Located in ~/assignment_instructions
- full home assignment instructions: VI Data Science Home Assignment — Instructions.pdf
- train/: churn_labels.csv, web_visits.csv, app_usage.csv, claims.csv
- test/: test_members.csv, test_web_visits.csv, test_app_usage.csv, test_claims.csv
- Reference: wellco_client_brief.txt, schema_*.md files
- scheme: schema_app_usage.md, scheme_churn_labels.md, scheme_claimes.md, scheme_web_visits.md

## Timeline
- Observation window: Jul 1–15, 2025 (14 days) — behavioral data collected
- Outreach event: Jul 15, 2025 — binary treatment (outreach column in churn_labels.csv)
- Churn measurement window: Jul 15–29, 2025 (14 days) — churn label defined here

## Output Directory
All deliverables live in `Shay_Assignment/`:
- `eda.ipynb` — exploratory data analysis notebook
- `requirements.txt` — Python dependencies
- (future) feature engineering, modeling, and scoring notebooks
- (future) `predictions.csv` — top n test members for outreach
- (future) executive presentation slides

## Required Deliverables
1. ✅ Public Git repository with reproducible end-to-end solution — https://github.com/SShayVi/wellco-churn-prediction-home-assignment
2. ✅ README with setup/run instructions and approach description — `README.md` committed to repo
3. Executive presentation (3–5 slides) for non-technical stakeholders
4. CSV with top n test members for outreach: member_id, prioritization score, rank

## Four Questions the Solution Must Answer
1. **Feature selection**: which features were chosen and why (domain relevance, data quality, predictive power)
2. **Model evaluation**: which metrics were used and why
3. **Using outreach data in modelling**: outreach is a post-observation treatment that precedes churn — must explain how it's incorporated and what effect it has
4. **Selecting n**: how optimal outreach size is determined (cost + other factors)

## Environment
- Python 3.9 venv: `~/PycharmProjects/vi-ml-projects1/.venv`
- Jupyter kernel registered as `vi-ml` ("Python 3.9 (vi-ml)")
- Install deps: `pip install -r Shay_Assignment/requirements.txt`
- Data paths in notebooks use `../assignment_instructions/` (relative to `Shay_Assignment/`)

## EDA Conclusions (Phase 1)

### Dataset
- 10,000 train members; test members are IDs 20,001–30,000 (no overlap)
- Signup dates: Jan 2024 – May 2025

### Target
- Overall churn rate: **20.2%** (2,021/10,000) — moderately imbalanced
- Churn rises sharply with membership recency: 13% for Jan 2024 cohort → 36% for May 2025 cohort

### Outreach
- 39.8% of members received outreach; assignment looks roughly random across cohorts (37–51%)
- Outreach reduces churn by ~1.3 pp (19.4% vs 20.7%) — small effect; correlation = -0.016
- Must treat outreach as a post-observation treatment, not a feature for scoring (data leakage risk)

### App usage
- Near-universal coverage: 9,998/10,000 members have sessions
- Single event type: "session"; mean 9.8 sessions, max 26; all within observation window
- Strongest signal: session_count correlation with churn = **-0.079** (more sessions → less churn)

### Web visits
- 9,975/10,000 members have visits; mean 26 visits, max 140
- **90.3% of visits are non-WellCo** (portal.site, example.com, world.news, etc.) — likely noise/sabotage
- Raw visit_count has near-zero churn correlation (+0.003); WellCo-specific visits likely more predictive
- WellCo content covers: sleep, nutrition, heart, hypertension, stress, fitness, mindfulness

### Claims
- 9,980/10,000 members have claims; mean 6.5 claims per member
- 3 expected ICD codes: E11.9 (diabetes), I10 (hypertension), Z71.3 (nutrition counseling) — all mildly protective against churn
- **7 unexpected ICD codes** (H10.9, B34.9, A09, M54.5, J00, R51, K21.9): 37,464 records — treat as noise or separate feature
- **1,676 duplicate claim records** — deduplicate before feature engineering
- claim_count correlation with churn = -0.022

### Data quality
- No nulls anywhere; no out-of-window events; no train/test leakage
- Issues to handle in feature engineering: non-WellCo web noise, unexpected ICD codes, duplicate claims

### Feature engineering implications
- Use **session_count** as primary engagement signal
- Use **WellCo-only visit count** (filter `health.wellco` domain) rather than raw visit count
- Create **binary flags** for E11.9, I10, Z71.3 (deduplicated); optionally a flag for noise ICD codes
- Use **membership tenure** (days since signup as of Jul 15) — strong predictor via cohort analysis
- Outreach column: include in model training only to model its effect, then score test members **without** it (they haven't received outreach yet)

## Rules
- Never proceed to the next phase without my approval
- After each phase, summarize findings and wait for my go-ahead
- Python only: pandas, lightgbm, matplotlib, scikit-learn