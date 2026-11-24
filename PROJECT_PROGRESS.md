# Project Progress Log

**Project:** E-Commerce Purchase-Intent Prediction with Explainable AI  
**Last updated:** 2026-11-25
**Overall status:** In progress

This file records what has been investigated, what has been designed, what has been implemented and what still needs verification. A topic is not marked complete merely because its code or instructions have been drafted.

## Progress summary

| Workstream | Status | Current position |
|---|---|---|
| Project and business understanding | Complete | PRD reviewed; business objective and binary-classification target defined. |
| Initial dataset inspection | Complete | CSV dimensions, target balance, missing values, duplicates and feature meanings inspected. |
| Exploratory data analysis | In progress | Initial findings reviewed; full notebook, Plotly requirement and chart takeaways still need final verification. |
| Feature engineering | In progress | Six candidate features defined and explained; execution and output validation in the working notebook still need confirmation. |
| Data preprocessing | In progress | Train/test split and preprocessing design prepared; local notebook execution and verification still pending. |
| Model development | Not started | Six required algorithms remain to be trained and compared. |
| Explainable AI | Not started | SHAP, LIME and model card remain to be implemented. |
| FastAPI service | Not started | Prediction and health endpoints remain to be built. |
| Streamlit dashboard | Not started | Six dashboard sections remain to be built. |
| MLOps and deployment | Not started | MLflow, tests, Docker, GitHub Actions and AWS-ready configuration remain. |
| Final documentation and demo | In progress | Initial README and progress log drafted; final documentation and demo remain. |

## 1. Project and business understanding — complete

- Reviewed the project requirements document.
- Defined the task as supervised binary classification.
- Identified `Converted` as the target: `0` means no purchase and `1` means purchase.
- Identified the intended business use: estimate session conversion likelihood and provide explanations to support marketing/CRO decisions.
- Discussed mid-session prediction as the current design preference, subject to a data-availability limitation described below.

## 2. Initial dataset inspection — complete

The supplied `ecommerce_sessions.csv` was inspected directly.

| Check | Observed result |
|---|---:|
| Rows | 12,000 |
| Columns | 18 |
| Input features | 17 |
| Target | `Converted` |
| `Converted = 0` | 10,114 (84.28%) |
| `Converted = 1` | 1,886 (15.72%) |
| Missing values in raw CSV | 0 |
| Exact duplicate rows | 0 |

The data dictionary was reviewed. `OperatingSystems`, `Browser`, `Region`, and `TrafficType` are integer-coded categories and should not be treated as continuous quantities.

The majority-class baseline is approximately 84.28% accuracy if every session is predicted as no-purchase. Such a classifier would identify none of the purchasing sessions, so accuracy alone is not an adequate success measure.

## 3. Exploratory data analysis — in progress

Initial EDA code and interpretations have been worked through for:

- Purchase versus non-purchase class distribution.
- Buyer versus non-buyer comparisons.
- Product-page engagement boxplot.
- Conversion rate by visitor type.
- Monthly conversion rate.
- Correlation analysis for numerical features.

### Initial findings checked against the CSV

| Finding | Observed value |
|---|---:|
| Mean `ProductRelated` for non-buyers | 17.11 pages |
| Mean `ProductRelated` for buyers | 34.41 pages |
| Mean `ProductRelated_Duration` for non-buyers | 624.72 seconds |
| Mean `ProductRelated_Duration` for buyers | 1,250.67 seconds |
| Mean `PageValues` for non-buyers | 5.08 |
| Mean `PageValues` for buyers | 50.56 |
| Pearson correlation: `PageValues` vs `Converted` | +0.640 |
| Returning-visitor conversion rate | 16.57% |
| New-visitor conversion rate | 10.17% |
| Other-visitor conversion rate | 15.04% |
| November conversion rate | 20.00% |

These results are descriptive associations. They do not show that a feature causes a purchase or guarantee that it will improve an unseen-data model.

### Remaining EDA work

- Verify the EDA notebook runs from top to bottom.
- Add at least one interactive Plotly visual, as required by the PRD.
- Add a one-line takeaway below every major chart.
- Ensure the written findings connect to modelling decisions.

## 4. Feature engineering — in progress

Six candidate behavioural features have been defined:

| Candidate feature | Definition |
|---|---|
| `TotalPagesViewed` | Sum of administrative, informational and product-related page counts. |
| `TotalBrowsingDuration` | Sum of the three page-duration features. |
| `AverageTimePerPage` | Total browsing duration divided by total pages viewed. |
| `ProductPageShare` | Product-related page count divided by total pages viewed. |
| `ProductDurationShare` | Product-related duration divided by total browsing duration. |
| `ReturningVisitorFlag` | 1 for `Returning_Visitor`, otherwise 0. |

The original `VisitorType` feature is retained because the binary flag combines `New_Visitor` and `Other` into one value and therefore loses category detail.

The three ratio/average features can be undefined when their denominators are zero. The planned implementation uses `NaN` for these cases so they can be handled explicitly during preprocessing, rather than silently treating an undefined ratio as a real zero.

**Status note:** The feature definitions and code have been discussed, and the concepts have been reviewed. Execution of all feature-engineering cells and validation of the resulting notebook outputs have not yet been confirmed.

## 5. Data preprocessing — in progress

The planned preprocessing workflow is:

- Separate `X` and `y`, keeping `Converted` out of the input features.
- Use an 80/20 train/test split with `stratify=y` and a fixed random seed.
- Treat `Month`, `OperatingSystems`, `Browser`, `Region`, `TrafficType`, `VisitorType`, and `Weekend` as categorical.
- Use a duration pipeline with median imputation, `log1p`, and `RobustScaler`.
- Use a numerical pipeline with median imputation and `RobustScaler` for the remaining numerical features.
- Use a categorical pipeline with most-frequent imputation and `OneHotEncoder(handle_unknown="ignore")`.
- Combine the column-specific pipelines with `ColumnTransformer`.
- Fit transformations on training data only; transform test data using the already-fitted preprocessor.
- Handle resampling only inside training folds if a resampling strategy is selected during modelling.

A reference check of this proposed configuration against the supplied CSV produced a 9,600-row training set, a 2,400-row test set, and 81 transformed columns. This is an expected reference result for the described configuration, not a claim that the user's local notebook has already been run and verified.

**Status note:** The implementation outline and code have been prepared. The user's execution of the preprocessing cells, inspection of the output shapes and feature names, and final validation are still pending.

## 6. Prediction timing and leakage risk — design issue under investigation

The current design preference is mid-session scoring, after some browsing behaviour has been observed and while an intervention may still be useful.

However, the supplied CSV contains aggregated session-level records, not timestamped event sequences or mid-session snapshots. It therefore cannot, by itself, validate a genuine mid-session prediction workflow.

`PageValues` requires particular scrutiny because the PRD describes it as a strong purchase signal and notes that it may reflect pages visited close to checkout. Its availability at the intended scoring time must be investigated. A with-versus-without-`PageValues` ablation study is planned; a strong predictive relationship alone is not proof of leakage.

## 7. Not started

The following deliverables have not yet been implemented or evaluated:

- Logistic Regression, Decision Tree, Random Forest, XGBoost, LightGBM and CatBoost comparison.
- Stratified cross-validation, hyperparameter tuning and full evaluation metrics.
- Business-value-based decision-threshold selection.
- Model and permutation/SHAP feature-importance analysis.
- SHAP global and local explanations; LIME local explanations.
- Model card.
- FastAPI prediction and health endpoints with Pydantic validation.
- Streamlit dashboard.
- MLflow tracking and model registration.
- Pytest suite, Black/Ruff checks, Docker/Docker Compose and GitHub Actions.
- AWS-ready deployment configuration and final demo.

## 8. Next actions

1. Finish and verify the EDA notebook, including Plotly and written chart takeaways.
2. Run the feature-engineering cells and verify the six candidate features and their missing-value behaviour.
3. Run the preprocessing notebook cells and confirm the split, feature groups, transformed matrix dimensions and output feature names.
4. Wrap preprocessing and the eventual estimator together in a scikit-learn `Pipeline` for cross-validation, so each fold fits its own transformations without leakage.
5. Move on to baseline model training only after the preprocessing checks pass.

---

*This log will be updated as implementation and validation are completed. Do not mark a deliverable complete until its code or artifact has been run and checked.*
