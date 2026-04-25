---
layout: page
title: CardioVascular Risk Prediction
description: End-to-end ML project predicting 10-year cardiovascular disease risk — 25+ derived datasets, ensemble models, Bayesian hyperparameter optimization, Explainable AI (LIME, SHAP), and Streamlit deployment.
img: assets/img/12.jpg
importance: 1
category: machine-learning
featured: true
github: https://github.com/BilalNaseem1/Machine-Learning-I-CardioVascular-Risk-Prediction-with-Deployment-on-Streamlit
---

## Overview

A full-cycle machine learning project predicting the **10-year risk of coronary heart disease (CHD)** using a dataset of 3,390 patients with 17 demographic, behavioral, and medical attributes.

The project was designed around a real clinical objective — maximizing **recall** to minimize missed diagnoses — while maintaining acceptable precision to avoid unnecessary tests.

## Data Challenges

- **Imbalanced target**: 85% no-risk / 15% at-risk — standard accuracy is meaningless here
- **Null values** in 7 of 17 columns, requiring feature-specific imputation strategies
- **Heavily right-skewed** numerical distributions with significant outliers

## Feature Engineering

Extensive experimentation across **25+ derived datasets** and **18 Python notebooks**:

- **Null imputation**: BPMeds rows dropped (<2%); `cigsPerDay` imputed from median of smokers only; `glucose` compared across Median, Iterative Imputer, and KNN (k=5) — Iterative Imputer won
- **Outlier treatment**: Winsorizing with per-column optimized thresholds (grid search via Naive Bayes) — ROC-AUC jumped from 0.718 → 0.745
- **Feature extraction**: Created `mean_BP` from `sysBP` + `diaBP` (highly correlated) — another +0.003 AUC jump
- **Feature reduction**: Gradient Boosting importance + PCA explored; all features retained (reduction hurt score)
- **Scaling**: MinMaxScaler applied before modeling

## Modeling

### Hyperparameter Optimization
- **Optuna TPE** (Tree-structured Parzen Estimator) and **CMA-ES** on LightGBM, XGBoost, Logistic Regression
- **SkOpt Bayesian Optimization** on all models including a Keras Neural Network (ReLU hidden layers, Sigmoid output, Binary Cross Entropy loss)
- RepeatedStratifiedKFold (10 splits × 3 repeats) to handle class imbalance during CV

### Ensemble Methods
- **Soft Voting** over top 5 models (NN, XGB, GB, RF, LR) with weights [1.5, 1, 1, 1, 1.5]
- **Stacking** with Logistic Regression as meta-model

| Model | AUC-ROC (test) | AUC-PR (test) |
|---|---|---|
| Logistic Regression | 0.701 | 0.315 |
| Gradient Boosting | 0.700 | 0.320 |
| Neural Network | 0.703 | 0.293 |
| Soft Voting | 0.690 | **0.322** |

## Best Model

**Logistic Regression** selected — highest AUC-PR on test set, best recall, and interpretable by design.

At probability threshold **0.1**: Recall = **0.837**, Precision = **0.223**

SMOTE was attempted to address class imbalance but did not improve AUC-PR (0.2864 vs 0.315).

## Explainable AI

- **LIME** — local explanations per prediction; shows top contributing features per instance
- **SHAP** — both local (force plots) and global (summary dot plot); confirmed `age` and `mean_BP` as dominant features; `BMI` and `diabetes` least important globally
- **Counterfactual** — minimum feature changes needed to flip the prediction, holding `age` and `sex` fixed

## Deployment

Deployed on **Streamlit** — takes continuous inputs via sliders, discrete inputs via buttons, and outputs:
- Spider plot of patient features vs. dataset mean
- CVD risk prediction (risk / no risk)
- LIME local explanation plot

## Tech Stack

`Python` `scikit-learn` `LightGBM` `XGBoost` `Keras` `PyCaret` `Optuna` `SkOpt` `SHAP` `LIME` `Streamlit` `pandas` `NumPy` `seaborn`
