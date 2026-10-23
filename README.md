# Telecom Customer Churn Analysis in R

Predicting which telecom customers are likely to leave, and what drives them to do so, so the company can target retention efforts.

## Overview

This project analyses 7,043 telecom customers (21 fields each) to:

1. Explore how churn relates to customer demographics, services and billing.
2. Build and compare four predictive models.
3. Score every customer by churn risk and group them into retention tiers.

The overall churn rate is **26.5%**.

## Key findings

- **Contract type matters most.** Month-to-month customers churn at 42.7%, versus 11.3% on one-year and 2.8% on two-year contracts.
- **New customers leave.** About 47% of customers in their first 12 months churn, versus 9.5% of those with over four years' tenure.
- **Fiber optic customers churn more** (41.9%) than DSL customers (19.0%).
- **Electronic check payers churn at 45.3%**, much higher than other payment methods.
- **Support services help.** Customers without Online Security or Tech Support churn at about 42%, versus about 15% for those with them.
- **Gender has no effect** on churn.

## Model results

Evaluated on a 20% held-out test set, using a 0.35 probability cutoff for precision and recall.

| Model | AUC | Accuracy | Precision | Recall |
|---|---|---|---|---|
| Logistic Regression | 0.846 | 0.780 | 0.569 | 0.708 |
| Decision Tree | 0.812 | 0.775 | 0.558 | 0.721 |
| Random Forest | 0.833 | 0.782 | 0.577 | 0.665 |
| Gradient Boosting | 0.847 | 0.787 | 0.582 | 0.694 |

Logistic regression and gradient boosting are effectively tied on AUC, so the simpler logistic model is a sensible choice when explainability matters. In the gradient boosting model, **Contract** (33%) and **tenure** (18%) are the strongest predictors.

## Retention risk tiers

Every customer is scored with the gradient boosting model and grouped by predicted churn probability.

| Tier | Probability | Customers | Actual churn rate |
|---|---|---|---|
| Low | below 30% | 4,511 | 9.7% |
| Medium | 30% to 60% | 1,537 | 44.8% |
| High | above 60% | 995 | 74.6% |

The Medium and High tiers together contain roughly three quarters of all churners while covering about 36% of customers.

## Repository contents

| File | Description |
|---|---|
| `telecom_churn.R` | Full analysis: cleaning, exploration, modelling, evaluation and risk scoring |
| `Telcom-Customer-Churn.csv` | Input dataset (7,043 rows, 21 columns) |
| `customer_risk_scores_R.csv` | Output: every customer with churn probability and risk tier |
| `churn_results_summary.xlsx` | Summary workbook: model comparison, feature importance and risk tiers |

## How to run

1. Install R (4.4 or later recommended) and RStudio.
2. Install the required packages:

   ```r
   install.packages(c("tidyverse", "caret", "pROC", "rpart", "randomForest", "gbm"))
   ```

3. Open the project in RStudio so the working directory is the repository folder.
4. Open `telecom_churn.R` and run the whole script (Ctrl+A, then Ctrl+Enter). It takes a minute or two because of the random forest and cross-validated gradient boosting steps.

The script prints model summaries and the comparison table, draws the exploratory and ROC plots, and writes `customer_risk_scores_R.csv` to the project folder.

## Notes and limitations

- Findings are associations, not proof of cause. Any retention offer should be tested with an A/B design.
- The dataset records which services a customer has, not call or data usage, so "usage" is approximated through services and charges.
- The final risk scores come from a model trained on 80% of the data and applied to all customers, so the tier results are slightly optimistic. Out-of-fold scoring would give a stricter estimate.
- In the logistic regression output, some coefficients show `NA` because levels such as "No internet service" duplicate information already in other columns. This is harmless.

## Data

Telco Customer Churn sample dataset, a publicly available dataset commonly used for churn modelling practice.
