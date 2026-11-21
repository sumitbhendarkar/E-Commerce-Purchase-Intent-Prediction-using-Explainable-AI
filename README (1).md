# E-Commerce Purchase-Intent Prediction with Explainable AI

> **Project status: In progress**  
> An end-to-end machine-learning capstone focused on predicting e-commerce session conversion, explaining model predictions, and serving them through an API and analytics dashboard.

## 1. Project overview

The goal of this project is to build a system that estimates whether an e-commerce browsing session will end in a purchase. The system is intended to provide a purchase probability, a predicted outcome, and explanations of the behaviours that influenced the prediction.

The business use case is conversion-rate optimisation. A marketing or growth team could use the model's scores and explanations to decide which sessions may be worth considering for an intervention such as a discount, free-shipping offer, or remarketing campaign. The model supports that decision; it does not prove that an intervention will cause a purchase.

The project follows the requirements in the supplied Project Requirements Document (PRD) and is organised around the CRISP-DM lifecycle.

## 2. Problem definition

- **Learning type:** Supervised learning
- **ML task:** Binary classification
- **Input:** E-commerce session features describing page visits, browsing duration, engagement, visitor context, and traffic source
- **Target:** `Converted`
- **Target value `0`:** Session did not result in a purchase
- **Target value `1`:** Session resulted in a purchase
- **Intended model output:** Purchase probability and a classification decision, with explanations

The dataset is imbalanced. A model that always predicts `0` would achieve approximately 84.28% accuracy on the supplied CSV while identifying no purchasing sessions. For that reason, model evaluation will emphasise purchase-class recall and Precision-Recall AUC (PR-AUC), alongside the full metric suite required by the PRD.

## 3. Dataset

The project package includes:

- `dataset/ecommerce_sessions.csv` — session-level data
- `dataset/data_dictionary.csv` — column definitions and data types

### Dataset snapshot

The supplied CSV was inspected during initial data understanding.

| Item | Result |
|---|---:|
| Sessions | 12,000 |
| Total columns | 18 |
| Input features | 17 |
| Target column | `Converted` |
| No-purchase sessions (`0`) | 10,114 (84.28%) |
| Purchase sessions (`1`) | 1,886 (15.72%) |
| Missing values in raw CSV | 0 |
| Exact duplicate rows in raw CSV | 0 |

### Feature groups

| Group | Columns |
|---|---|
| Page counts | `Administrative`, `Informational`, `ProductRelated` |
| Page durations | `Administrative_Duration`, `Informational_Duration`, `ProductRelated_Duration` |
| Engagement / value | `BounceRates`, `ExitRates`, `PageValues`, `SpecialDay` |
| Visit context | `Month`, `Weekend`, `VisitorType` |
| Integer-coded categories | `OperatingSystems`, `Browser`, `Region`, `TrafficType` |
| Target | `Converted` |

The integer-coded columns `OperatingSystems`, `Browser`, `Region`, and `TrafficType` represent categories, not continuous measurements. They will be handled as categorical variables during preprocessing.

## 4. Initial exploratory findings

The following observations were checked against the supplied CSV:

- The purchase conversion rate is **15.72%**.
- Average `ProductRelated` page count is approximately **34.41** for purchasing sessions versus **17.11** for non-purchasing sessions.
- Average `ProductRelated_Duration` is approximately **1,250.67 seconds** for purchasing sessions versus **624.72 seconds** for non-purchasing sessions.
- Average `PageValues` is approximately **50.56** for purchasing sessions versus **5.08** for non-purchasing sessions.
- `PageValues` has the strongest positive Pearson correlation with `Converted` among the numerical features inspected (approximately **0.640**).
- Returning visitors have the highest observed conversion rate among the supplied visitor categories (approximately **16.57%**).
- November has the highest observed monthly conversion rate in this dataset (**20.00%**, 447 purchases from 2,235 sessions).

These are descriptive associations, not causal conclusions. They will inform feature investigation and modelling, but they do not by themselves establish which features will generalise best.

## 5. Prediction-timing and data-leakage considerations

The current design discussion favours **mid-session prediction**: score a session after some browsing behaviour has been observed, while there may still be time for a business intervention.

There is an important limitation in the supplied data: it contains aggregated session-level records, not timestamped browsing events or partial-session snapshots. Therefore, this dataset alone cannot prove that the proposed model works at a genuine mid-session scoring point.

Feature availability must be checked against the intended scoring time. `PageValues` is a particularly important feature to investigate because the PRD notes that it can be a strong purchase signal and may reflect pages visited close to checkout. A with-versus-without-`PageValues` ablation study is planned. The feature will not be labelled as leakage unless its availability at prediction time is shown to be invalid.

## 6. Planned workflow

1. **Business and data understanding** — define the business question, inspect the dataset, validate the target and document limitations.
2. **Exploratory data analysis** — analyse conversion, buyer/non-buyer behaviour, visitor types, seasonality and correlations; use Matplotlib/Seaborn and at least one interactive Plotly visual.
3. **Feature engineering and selection** — create a small number of explainable behavioural features and assess their usefulness with more than one importance method.
4. **Preprocessing** — build reusable scikit-learn pipelines with a `ColumnTransformer`; handle categorical variables, skewed numerical features, missing values and class imbalance without leakage.
5. **Model development** — compare Logistic Regression, Decision Tree, Random Forest, XGBoost, LightGBM and CatBoost using stratified cross-validation and a full metric suite.
6. **Explainable AI** — implement global and local SHAP explanations, LIME explanations, and a model card.
7. **Prediction API** — build a FastAPI service with `POST /predict`, `GET /health`, Pydantic validation and OpenAPI documentation.
8. **Analytics dashboard** — build a Streamlit dashboard for overview, session scoring, funnel/cohort analytics, model comparison, explainability and system status.
9. **MLOps and delivery** — add MLflow tracking, Docker/Docker Compose, GitHub Actions, tests, documentation and AWS-ready configuration.

## 7. Technology stack

| Area | Tools |
|---|---|
| Language | Python |
| Data handling | pandas, NumPy |
| Visualisation | Matplotlib, Seaborn, Plotly |
| Preprocessing and classical ML | scikit-learn, imbalanced-learn |
| Gradient boosting | XGBoost, LightGBM, CatBoost |
| Explainability | SHAP, LIME |
| API | FastAPI, Uvicorn, Pydantic |
| Dashboard | Streamlit |
| Experiment tracking | MLflow |
| Persistence | joblib |
| Testing and code quality | pytest, Black, Ruff |
| Packaging and CI/CD | Docker, Docker Compose, GitHub Actions |
| Cloud target | AWS ECR and ECS / EC2 |
| Version control | Git and GitHub |

## 8. Repository structure (target layout)

The following is the intended structure as the project develops. Directories and components that have not yet been implemented are planned, not completed deliverables.

```text
purchase-intent-xai/
├── dataset/
│   ├── ecommerce_sessions.csv
│   └── data_dictionary.csv
├── notebooks/
│   ├── 01_eda.ipynb
│   ├── 02_features.ipynb
│   ├── 03_modeling.ipynb
│   └── 04_xai.ipynb
├── src/
│   ├── config/
│   ├── data/
│   ├── preprocessing/
│   ├── features/
│   ├── models/
│   ├── explain/
│   ├── api/
│   └── utils/
├── dashboard/
├── models/
├── tests/
├── docker/
├── docs/
├── .github/workflows/
├── config.yaml
├── requirements.txt
├── pyproject.toml
├── docker-compose.yml
└── README.md
```

## 9. Current progress

See [`PROJECT_PROGRESS.md`](PROJECT_PROGRESS.md) for the detailed progress log, completed learning milestones, in-progress implementation work, and remaining deliverables.

## 10. Current limitations

- The supplied data is session-aggregated and does not contain event timestamps or mid-session snapshots.
- The availability of each feature at the intended scoring time still needs to be established.
- No final model performance, selected threshold, SHAP/LIME results, API behaviour, or deployment status is claimed at this stage.
- The project is under active development; the final results will be added after experiments and validation are completed.

## 11. Project status

This repository is a work in progress. The objective is a reproducible, explainable, and deployable machine-learning workflow—not just a high-accuracy notebook model.
