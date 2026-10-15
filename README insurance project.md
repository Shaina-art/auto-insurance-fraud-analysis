# Auto Insurance Claims: Fraud Risk Analysis

**Tools:** Python (pandas, scikit-learn, seaborn), SQL (SQLite), Looker Studio

**Links:**
- Kaggle notebook: `https://www.kaggle.com/code/shainapereira/auto-insurance-project`
- Interactive dashboard: `https://datastudio.google.com/reporting/70930b79-65a0-4124-8bc4-0921bcdf9618`

![Dashboard page 1](C:\Users\shain\Downloads\insurance project\dashboard1.png)
![Dashboard page 2](C:\Users\shain\Downloads\insurance project\dashboard2.png)

## Problem
Which segments of auto insurance claims are riskiest, and how can fraud be flagged earlier so investigators review the right claims first?

## Data
Public Kaggle dataset "Insurance Claims and Fraud Analysis" (1,000 claims, 40 columns covering policy, customer, incident and claim amounts). Target column: `fraud_reported` (Y/N). 247 of 1,000 claims (24.7%) are flagged as fraud.

## Method
1. **Cleaning (pandas)**
   - Dropped 1 empty column (`_c39`) and 3 ID-like columns (`policy_number`, `incident_location`, `insured_zip`).
   - Found hidden `?` placeholders in `collision_type` (178 rows), `property_damage` (360) and `police_report_available` (343), and labelled them "Unknown" instead of guessing.
   - Relabelled 91 blank `authorities_contacted` values as "None" (no authority was contacted).
   - Set 1 invalid negative `umbrella_limit` to 0.
   - Kept negative `capital_loss` values (a loss is stored as a negative number) and added a positive `capital_loss_amt` column for charts.
   - Renamed hyphenated columns, converted dates, mapped `fraud_reported` to 1/0 and created age bands. No duplicates were found.
2. **SQL analysis (SQLite):** 10 queries, including a window function (`RANK`), a CTE, a subquery and a comparison of each state against the overall fraud rate.
3. **EDA:** fraud rate by severity, collision type and age band, claim amounts for fraud vs non-fraud, and a correlation heatmap.
4. **Modelling:** class-weighted random forest with 5-fold cross-validation, compared with a simple baseline rule. Reported precision and recall because only 24.7% of claims are fraud.
5. **Dashboard (Looker Studio):** two pages (Overview, Fraud indicators).

## Key findings
1. **Severity is the strongest signal.** Major Damage claims are fraudulent 60.5% of the time, against 6.7-12.9% for the other severity levels. They are 276 of 1,000 claims but hold 167 of the 247 fraud cases (67.6%).
2. **Fraud claims are larger.** The average fraud claim is $60,302 against $50,289 for non-fraud, about 20% higher.
3. **Collisions are riskier than non-collision claims.** Rear collisions have the highest rate (31.2%); claims with an unknown collision type are only 9.0%.
4. **Authority contact matters.** Claims where no authority was contacted are rarely fraud (5.6-7.4%). Claims where "Other" or "Fire" was contacted with no police report reach about 34%.
5. **State and age are weak signals.** Fraud rate by state ranges only from 22.8% to 25.9%. The higher-rate age bands (56+ and 25 and under) have small samples (64 and 40 claims).
6. **Approximate claim-to-premium ratio** (claimed amount divided by annual premium, not a true loss ratio): about 50x for collision claims and about 4x for theft and parked-car claims.
7. **Unusual hobby pattern.** Chess (82.6%, 46 claims) and cross-fit (74.3%, 35 claims) stand far above all other hobbies (9-30%). This looks like an artifact of a public, probably synthetic dataset, so I do not recommend using hobby as a screening rule.

## Model results
| Approach | Precision | Recall | Claims flagged (of 1,000) |
|---|---|---|---|
| Baseline rule: flag Major Damage | 60.5% | 67.6% | 276 |
| Random forest, threshold 0.35 (out-of-fold) | 60.5% | 85.4% | 349 |

- 5-fold cross-validated ROC-AUC: 0.84.
- At the same precision as the simple rule, the model catches about 44 more fraud cases by flagging about 73 more claims.
- The threshold was chosen on the same data, so these figures are cross-validated estimates and may be slightly optimistic.

## Recommendations
- Send Major Damage claims for manual review first (28% of claims, 68% of fraud).
- Use the model score to rank the remaining claims and extend review down to a 0.35-0.40 threshold, depending on how many claims investigators can handle.
- Give extra priority to claims with no police report where "Other" or "Fire" was contacted.
- Do not screen by hobby, sex or age. These are weak or unreliable proxies and could create unfair outcomes.

## Limitations
- Only 1,000 rows from a public and probably synthetic dataset, so patterns may not hold on real claims.
- Small groups (some occupations, hobbies and age bands) give noisy rates.
- The claim-to-premium ratio uses claimed amounts, not paid losses.
- The model threshold and findings were not tested on separate, unseen data.

## Repository contents
- `Auto_Insurance_Fraud_Analysis.ipynb`: full notebook (cleaning, SQL, EDA, model)
- `claims_clean.csv`: cleaned dataset used for the dashboard
- `images/`: dashboard screenshots
