# Phishing Website Classification Using Hybrid URL & HTML Features

Undergraduate thesis project — Department of Statistics, Yıldız Technical University.

## Overview

Phishing attacks remain one of the most common vectors for credential theft, and legacy defenses (blocklists, static heuristic rules) struggle to keep pace with how quickly attackers rotate domains. This project builds a classification pipeline that detects phishing websites using 23 features derived from the URL string and a small set of HTML-level attributes — designed to be fast and lightweight enough for real-time filtering, rather than relying on heavier page-rendering or deep learning pipelines.

The project compares three statistical learning algorithms, optimizes the decision threshold against real-world cost asymmetry (a missed phishing page is far costlier than a false alarm), and uses SHAP to make model decisions interpretable to a human reviewer.

## Dataset

- **Source:** [PhreshPhish](https://huggingface.co/datasets) — a large-scale, real-world phishing/legitimate URL corpus (CC BY 4.0)
- **Size:** ~666,315 labeled URLs (45% phishing, 55% legitimate)
- **Split:** 75% train / 25% test, stratified

## Features (23 total)

- **15 URL-based features:** length, depth, subdomain count, digit/special-character ratios, HTTPS usage, suspicious keyword presence, and other structural/lexical signals
- **8 HTML-based features:** external link ratio, null anchor ratio, external resource ratio, suspicious form actions, login form presence, and page-language signals

## Methodology

1. **Preprocessing:** median imputation and standard scaling, fit only on the training split to avoid leakage
2. **Modeling:** Logistic Regression, Random Forest, and XGBoost, with additional comparisons against LightGBM, CatBoost, soft voting, and stacking ensembles
3. **Hyperparameter tuning:** Bayesian optimization via Optuna
4. **Validation:** 10-fold stratified cross-validation plus 5 repeated 75/25 splits (different seeds) to check result stability
5. **Threshold optimization:** shifted the decision threshold from the default 0.50 to 0.23, a point independently agreed upon by F1 maximization, the Youden index, and G-mean on the validation set
6. **Explainability:** SHAP (TreeExplainer) applied to the final XGBoost model to identify and visualize the top predictors behind each classification

## Results

| Model | Precision | Recall | F1 | AUC |
|---|---|---|---|---|
| Logistic Regression | 0.880 | 0.915 | 0.897 | 0.909 |
| Random Forest | 0.910 | 0.912 | 0.911 | 0.984 |
| **XGBoost (tuned)** | **0.902** | **0.920** | **0.911** | **0.987** |

Shifting the operating threshold from 0.50 to 0.23 captured **2,740 additional phishing pages** at a cost of only a **1.32 percentage-point** rise in false alarms — a favorable trade-off given the asymmetric cost of missed detections in a security context.

SHAP analysis identified **path length**, **query string length**, and **page language** as the three most influential predictors of the model's classification decisions.

## Repository structure

```
.
├── README.md
├── requirements.txt
├── notebooks/        # exploratory analysis, modeling, SHAP analysis
├── src/               # reusable preprocessing / feature engineering / training code
└── data/              # not included — see Dataset section for source
```

## Tech stack

Python · pandas · NumPy · scikit-learn · XGBoost · LightGBM · CatBoost · Optuna · SHAP · matplotlib · seaborn

## Limitations & future work

Results are specific to the PhreshPhish corpus and have not yet been tested for cross-dataset generalization. The feature set does not include adversarial robustness testing, and dynamic (JavaScript-rendered) phishing pages were out of scope. See the full thesis for a detailed discussion of limitations and proposed extensions (e.g., a three-tier block/review/allow triage system, cross-dataset validation, adversarial training).

## Author

Azra Altundaş — Department of Statistics, Yıldız Technical University

