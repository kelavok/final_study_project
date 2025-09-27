 Final Study Project — A/B Test Analysis (Subscription Monetization)

## Overview
This project analyzes the impact of a product change on subscription monetization using an **A/B test** with two control groups and one test group. The notebook walks through data loading, cleaning, exploratory analysis, statistical testing, and interpretable conclusions.

The full analysis lives in Jupyter Notebook: `notebooks/final_project_var3.ipynb`.

---

## Goals
- Validate data integrity and group split (AA checks).
- Explore user and transaction behavior by country and group.
- Measure **Conversion to paying user**, **ARPU**, and **ARPPU**.
- Test statistical significance of differences between groups.
- Provide a decision and recommendations for rollout or retest.

---

## Dataset

### Files
Six CSV files are used (semicolon `;` as delimiter):
- `Проект_3_users_control_1.csv`
- `Проект_3_transactions_control_1.csv`
- `Проект_3_users_control_2.csv`
- `Проект_3_transactions_control_2.csv`
- `Проект_3_users_test.csv`
- `Проект_3_transactions_test.csv`

### Users schema (sample from notebook)
Columns include: `uid`, `age`, `attraction_coeff`, `coins`, `country`, `visit_days`, `gender`,
`age_filter_start`, `age_filter_end`, `views_count`, `was_premium`, `is_premium`.

### Transactions schema (sample from notebook)
Columns include: `uid`, `country`, `joined_at`, `paid_at`, `revenue`, `payment_id`, `from_page`, `product_type`.
Derived fields used in analysis: monthly bucket `trans_month`, time delta `period_join_paid`.

### Timeframe
Test window established from the notebook: **2017-10-14 to 2017-11-15**.

### Groups
- `control_1`
- `control_2`
- `test`

---

## Methodology

1. **Data import and sanity checks**
   - Basic inspection (`info`, `head`, nulls, dtypes), duplicates removed as needed.
   - Countries filtered where at least one group has zero transactions (to avoid degenerate comparisons).
   - Noted anomaly: mass payments on the 11th; suspicious transactions removed while retaining valid users with normal transactions.

2. **Feature engineering**
   - Time-based features (`trans_month`, `period_join_paid`).
   - Product filtering for focused views, especially **Premium No Trial** (`product_type == "premium_no_trial"`).

3. **Outliers**
   - Explored top-1% revenue share. Group-level share ranged roughly **4–8%**, with test group on the higher side.
   - Robust analyses repeated with **6 outliers removed** per PNT slice where indicated.

4. **Exploratory analysis**
   - Country-level pivots for users and revenue by `group`.
   - Distribution checks for revenue, and scatterplots of revenue concentration by group and country.

5. **Statistical testing**
   - **AA checks** to ensure comparable control groups.
   - **Z-test for proportions**: conversion to paying user.
   - **ANOVA / Welch t-tests** where assumptions were reasonable.
   - **Kruskal–Wallis** for non-normal or heteroscedastic distributions (primary for ARPU/ARPPU).

---

## Summary Tables (from notebook)

### Group-level KPIs (all products, grouped)
| Group      | Total Users | Paying Users | Total Revenue |   ARPU  |  ARPPU  | Conversion Rate (%) |
|------------|------------:|-------------:|--------------:|--------:|--------:|--------------------:|
| control_1  |        4089 |          178 |      2,427,633|  593.70 | 13638.39|                4.35 |
| control_2  |        4009 |          172 |      1,770,379|  441.60 | 10292.90|                4.29 |
| test       |        4040 |          139 |      2,240,485|  554.58 | 16118.60|                3.44 |

> Notes: values above are taken from the rendered notebook outputs. Minor rounding may occur.

### Before/After filtering (PNT-focused slices where applicable)
| Group      | ARPU_before | ARPPU_before | Revenue_before | ARPU_after | ARPPU_after | ΔARPU (abs) | ΔARPU (%) | ΔARPPU (abs) | ΔARPPU (%) |
|------------|------------:|-------------:|---------------:|-----------:|------------:|------------:|----------:|-------------:|-----------:|
| control_1  |      103.83 |       8844.88|       424,554.0|      61.34 |     5572.38 |     -42.49  |   -40.92  |     -3272.50 |    -37.00  |
| control_2  |      115.05 |       8702.89|       461,253.0|      78.95 |     6204.31 |     -36.10  |   -31.38  |     -2498.58 |    -28.71  |
| test       |      105.50 |      11839.39|       426,218.0|      77.41 |     8935.46 |     -28.09  |   -26.63  |     -2903.93 |    -24.53  |

---

## Statistical Results (high-level)

- **Conversion (Z-test):** no statistically significant uplift of the `test` group vs controls in the notebook runs. The observed direction points toward **lower conversion** in `test`, but not at conventional significance levels within this short window.
- **ARPU (Kruskal–Wallis):** p-values well above 0.05 on filtered and unfiltered slices — **no significant differences**.
- **ARPPU:** higher values appear for the `test` group in some cuts (e.g., PNT), but results are **not conclusive** and should not be interpreted without conversion context.
- **Country effects:** strong between-country variation; comparisons were constrained to countries with data in all groups.

---

## Conclusions
- No evidence of **ARPU uplift** for the `test` group.
- **Conversion to paying** tends to be **lower** in `test` during the analyzed period.
- **ARPPU differences**, when present, are not reliable without accounting for conversion rates.
- Given the short timeframe and variability by country, results should be treated as **inconclusive** for rollout decisions.

### Recommendations
1. **Extend the test** to at least **2 months** to gather more stable evidence.
2. **Control for pricing consistency** across identical products and geographies. Investigate why identical products show different prices in the same country.
3. **Clarify `payment_id` semantics** and ensure deduplication and attribution are correct.
4. **Predefine outlier handling** and keep the same rules across groups.
5. Track **per-country** cohorts; consider stratified analysis or geo-level randomization.

---

## How to Run

```bash
git clone https://github.com/kelavok/final_study_project.git
cd final_study_project

python -m venv .venv
# Windows: .venv\Scripts\activate
# macOS/Linux:
source .venv/bin/activate

pip install -r requirements.txt
jupyter notebook notebooks/final_project_var3.ipynb