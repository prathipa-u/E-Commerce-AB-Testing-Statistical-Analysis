# E-Commerce Website A/B Testing — Statistical Analysis

A statistical analysis of an A/B test run by an e-commerce company to determine whether a **new webpage design** leads to a higher **conversion rate** than the **existing (old) webpage**. The project uses probability, hypothesis testing (bootstrap simulation and z-test), and logistic regression to arrive at a data-driven recommendation.

## Table of Contents

- [Project Overview](#project-overview)
- [Business Problem](#business-problem)
- [Dataset](#dataset)
- [Methodology](#methodology)
  - [1. Data Cleaning](#1-data-cleaning)
  - [2. Probability Analysis](#2-probability-analysis)
  - [3. A/B Hypothesis Testing](#3-ab-hypothesis-testing)
  - [4. Regression Analysis](#4-regression-analysis)
- [Key Results](#key-results)
- [Conclusion & Recommendation](#conclusion--recommendation)
- [Tech Stack](#tech-stack)
- [References](#references)

## Project Overview

A/B testing is one of the most widely used techniques by data analysts and data scientists to evaluate whether a proposed change improves a business metric before it is rolled out to all users. In this project, an e-commerce company tested a new landing page against its existing page, with the goal of increasing the proportion of users who convert (i.e., purchase the product).

This analysis walks through the full statistical workflow used to answer the question:

> **Should the company implement the new page, keep the old page, or run the experiment longer?**

## Business Problem

The company needs a statistically sound recommendation on one of three actions:
1. **Implement the new webpage**
2. **Keep the old webpage**
3. **Run the experiment for a longer period** before deciding

The decision is based on rigorous statistical evidence rather than the raw, observed difference in conversion rates alone, since random variation can produce misleading results if not tested formally.

## Dataset

The analysis uses two files, provided as part of the Udacity Data Analyst Nanodegree program:

**`ab_data.csv`** — the core experiment data (294,478 rows):

| Column | Description | Valid Values |
|---|---|---|
| `user_id` | Unique identifier for each user | Int64 |
| `timestamp` | Time the user visited the page | — |
| `group` | Experimental group assignment | `control`, `treatment` |
| `landing_page` | Page version shown to the user | `old_page`, `new_page` |
| `converted` | Whether the user purchased the product | `0` (no), `1` (yes) |

**`countries.csv`** — maps each `user_id` to a country (`US`, `CA`, `UK`), used later to test for regional effects on conversion.

> Note: `control` users are expected to see `old_page` and `treatment` users are expected to see `new_page`. The raw data contains some mismatched rows (e.g., a `control` user shown the `new_page`), which are removed during data cleaning.

## Methodology

### 1. Data Cleaning
- Identified and removed rows where `group` and `landing_page` did not match the expected pairing (`control`/`old_page` and `treatment`/`new_page`).
- Checked for and confirmed there were no missing values.
- Identified and removed one duplicate `user_id`.
- Final cleaned dataset (`df2`): **290,584 unique users**.

### 2. Probability Analysis
Computed baseline conversion probabilities directly from the cleaned data:
- Overall conversion rate across all users.
- Conversion rate for the `control` group.
- Conversion rate for the `treatment` group.
- The observed difference in conversion rates between groups (`obs_diff`).
- Probability that a given user received the new page (checking that the experiment was split ~50/50).

### 3. A/B Hypothesis Testing
Formally tested whether the new page's conversion rate is significantly better than the old page's, using a **Type I error rate (α) of 5%**:

**Hypotheses**

$$H_0: p_{new} - p_{old} \leq 0 \qquad H_1: p_{new} - p_{old} > 0$$

**Approach A — Bootstrap Simulation (10,000 iterations)**
- Simulated conversions for the new and old page samples under the null hypothesis (assuming both pages share the same population conversion rate).
- Built a sampling distribution of the difference in conversion rates (`p_diffs`).
- Compared the observed difference (`obs_diff`) against this simulated distribution to compute an empirical p-value.

**Approach B — Two-Sample Z-Test**
- Used `statsmodels.stats.proportions_ztest()` as a built-in statistical check on the simulation results.
- Performed a right-tailed test (`alternative='larger'`), consistent with the alternative hypothesis.

Both approaches produced consistent results, reinforcing confidence in the conclusion.

### 4. Regression Analysis
Validated the A/B test conclusion using **logistic regression** (appropriate since the response variable, `converted`, is binary):

- **Model 1:** `converted ~ intercept + ab_page` — tests the effect of page version alone.
- **Model 2:** `converted ~ intercept + ab_page + CA + UK` — adds country as a control variable (`US` as baseline).
- **Model 3:** `converted ~ intercept + ab_page + CA + UK + CA:ab_page + UK:ab_page` — adds interaction terms between page version and country to test whether the effect of the new page differs by region.

Regression coefficients were exponentiated (`np.exp()`) to interpret them as odds ratios.

## Key Results

| Metric | Value |
|---|---|
| Overall conversion rate | 11.96% |
| Control group conversion rate | 12.04% |
| Treatment group conversion rate | 11.88% |
| Observed difference (new − old) | −0.16 pp (new page slightly lower) |
| Bootstrap simulation p-value | 0.90 |
| Two-sample z-test p-value | 0.90 |
| Logistic regression p-value (`ab_page`) | 0.190 |
| Odds ratio (`ab_page`, new vs. old) | ~0.985 |

- **All four analytical approaches — probability comparison, bootstrap simulation, z-test, and logistic regression — agree**: there is no statistically significant evidence that the new page outperforms the old page.
- Adding **country** (`CA`, `UK`) and **page–country interaction** terms did not materially change the effect of `ab_page`, and none of the added predictors were statistically significant at α = 0.05.

## Conclusion & Recommendation

Across every statistical method used, the p-values obtained (0.90 for the hypothesis tests, 0.19 for the regression models) are far greater than the significance threshold of 0.05. This means we **fail to reject the null hypothesis** in all cases — there is insufficient evidence that the new page increases conversions, and the observed conversion rate for the new page was actually marginally lower than the old page.

**Recommendation:** Keep the existing (old) page. The data does not support the cost and risk of rolling out the new page company-wide. If the business wants more certainty, running the experiment for a longer period could help account for effects like *change aversion*, where existing users initially react negatively to any change regardless of its actual quality.

## Tech Stack

- **Python 3**
- `pandas` / `numpy` — data manipulation
- `statsmodels` — hypothesis testing (`proportions_ztest`) and logistic regression (`Logit`)
- `matplotlib` — data visualization
- Jupyter Notebook


## References

- [statsmodels: `proportions_ztest`](https://www.statsmodels.org/stable/generated/statsmodels.stats.proportion.proportions_ztest.html)
- [pandas: `DataFrame.join`](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.join.html)
- [Statology: Null Hypothesis of Logistic Regression](https://www.statology.org/null-hypothesis-of-logistic-regression/)

---

