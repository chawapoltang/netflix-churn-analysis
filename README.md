# Netflix Customer Churn Analysis

> Analysis of a 5,000-user sample to understand **who cancels, why, and what it costs**, then turns the findings into practical retention recommendations.

## Table of Contents
- [About This Project](#about-this-project)
- [Data and Scope](#data-and-scope)
- [Limitations](#limitations)
- [Analysis and Findings](#analysis-and-findings)
- [Churn Prediction](#churn-prediction)
- [Key Findings](#key-findings)
- [Recommendations](#recommendations)
- [Tools](#tools)

---

## About This Project

Subscription streaming lives or dies on retention. This project analyzes a 5,000-user sample of the Netflix Customer Churn dataset to understand who cancels, when, and what it costs, then turns the findings into practical retention recommendations.

**Headline results**

| Metric | Value |
| --- | --- |
| Users analyzed | 5,000 |
| Churned users | 2,515 (50.3%) |
| Lost monthly revenue | $33,010 (48.2% of potential MRR) |
| Avg. watch hours (retained vs. churned) | 17.45 vs. 5.92 |

---

## Data and Scope

Sample of 5,000 users from the Netflix Customer Churn dataset.

**Columns:** `customer_id`, `age`, `gender`, `subscription_type`, `watch_hours`, `last_login_days`, `region`, `device`, `monthly_fee`, `churned`, `payment_method`, `number_of_profiles`, `avg_watch_time_per_day`, `favorite_genre`

**Definitions**
- Subscription type: Basic = $8.99, Standard = $13.99, Premium = $17.99
- Churn: `1` = cancelled subscription, `0` = active

---

## Limitations

- This dataset contains synthetic data simulating customer behavior for a Netflix-like video streaming service.
- It includes 5,000 records with no time-series component.

---

## Analysis and Findings

### Revenue Loss Overview

| churned | subscription_type | N customers | SUM of monthly_fee |
| --- | --- | --- | --- |
| 0 | Premium | 953 | 17,144 |
| 0 | Standard | 898 | 12,563 |
| 0 | Basic | 634 | 5,700 |
| **0 Total** | | **2,485** | **35,407** |
| 1 | Premium | 740 | 13,313 |
| 1 | Standard | 748 | 10,465 |
| 1 | Basic | 1,027 | 9,233 |
| **1 Total** | | **2,515 (50.3%)** | **33,010 (48.2%)** |

**Total monthly revenue loss: $33,010 (48.2%)**  
Premium = highest revenue loss | Basic = highest number of customers lost

### Churn Overview

| churned | N | SUM of watch_hours | Average watch hours |
| --- | --- | --- | --- |
| 0 | 2,485 | 43,362.23 | 17.45 |
| 1 | 2,515 | 14,885.02 | 5.92 |

Retained users average 17.5 hours of viewing, while churned users average only 5.9 hours.

### Churn Rate by Watch Time Group
Users who watch less than 1 hour have an approximately 88% churn rate.

### Deep Dive: Users with watch_hours < 1

- In the group that watched less than 1 hour, **Basic package users churned 100%**.
  <img width="606" height="391" alt="Churn Rate by Watch Time Group" src="https://github.com/user-attachments/assets/11b13ca1-b2a1-4d7f-8ed1-d4e8a0bf1afa" />

- In the same group, users with **only 1 profile churned 100%**.
  <img width="606" height="391" alt="Number of Profiles in Low Watch Group ( 1hr)" src="https://github.com/user-attachments/assets/6a3ffd5a-1f14-46f9-a42d-f51849ceee25" />

- Age distribution remains spread out (no clear pattern).
  <img width="858" height="447" alt="Age distribution " src="https://github.com/user-attachments/assets/dda4f2ff-d91c-4b2f-b462-b14cdae48381" />

- Device shows similar churn rates across types.
  <img width="858" height="447" alt="device" src="https://github.com/user-attachments/assets/e98d7698-3401-4cb5-a08b-1da7750461be" />

- Favorite genres among this high-risk group include Action, Documentary, and Comedy.
<img width="858" height="447" alt="genre" src="https://github.com/user-attachments/assets/86f9410e-26ab-409e-88c8-88e1ba12ce4d" />

---

## Churn Prediction

Logistic regression was used to predict churn with three engagement features:  
`last_login_days` + `watch_hours` + `avg_watch_time_per_day`

### Model Coefficients

| Variable | Coefficient | p-value | Significant at 0.1%? | Notes |
| --- | --- | --- | --- | --- |
| Intercept | 0.178 | 0.302 | No | Not significant |
| last_login_days | 0.082 | < 2e-16 | Yes | Extremely strong |
| watch_hours | -0.175 | < 2e-16 | Yes | Extremely strong |
| avg_watch_time_per_day | -2.428 | 5.85e-09 | Yes | Very strong |

All three engagement variables are significant at the 0.1% level (p < 0.001).

### Test Set Performance (80/20 split)

**Confusion Matrix**

| Actual \ Predicted | 0 | 1 |
|--------------------|---|---|
| **0**              | 423 | 76 |
| **1**              | 58  | 443 |

| Metric | Value |
| --- | --- |
| Accuracy | 0.866 |
| Precision | 0.854 |
| Recall | 0.884 |
| F1 Score | 0.869 |

### Odds Ratios

| Variable | Odds Ratio | Interpretation |
| --- | --- | --- |
| last_login_days | 1.085 | Each additional day without login increases churn odds by ~8.5% |
| watch_hours | 0.839 | Each additional hour watched reduces churn odds to 83.9% of the previous level (~16% reduction) |
| avg_watch_time_per_day | 0.088 | Each additional hour of average daily watching strongly reduces churn odds (to 8.8% of the previous level) |

---

## Key Findings

1. `last_login_days`: longer inactivity → higher churn risk  
2. `watch_hours`: lower total viewing → higher churn risk  
3. `avg_watch_time_per_day`: lower daily average → higher churn risk  
4. High-risk segment for retention focus: **Basic + low watch hours** (especially those with 1 profile)

---

## Recommendations

### Proposed Experiment: A/B Test for High-Risk Basic Users

**Target group:** Basic subscribers with `watch_hours < 1` (not limited to 1 profile)  
**Duration:** 4 weeks or until sufficient sample size is reached  
**Randomization:** Users who click the cancel button

**Groups**
- A (Control, 33%): Existing cancel page  
- B (33%): Offer to pause membership for 1 month  
- C (33%): Offer 50% discount for 2 months  

**Metrics**
1. **Primary**: Churn rate within 30 days after visiting the cancel page  
2. **Secondary**:
   - Offer acceptance rate (B and C)
   - `watch_hours` and `avg_watch_time_per_day` at 14 and 30 days after the offer
   - `last_login_days` after receiving the offer
   - Retention rate after the pause / promotion period ends (for B and C)

---

## Tools

- R (dplyr, ggplot2, glm)
- Looker Studio (optional dashboard)
