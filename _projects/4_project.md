---
layout: page
title: Predictive Maintenance System
description: ML system predicting industrial equipment failures 48 hours in advance from sensor telemetry — combining mechanical engineering domain knowledge with gradient boosting.
img: assets/img/11.jpg
importance: 4
category: machine-learning
github: https://github.com/BilalNaseem1/predictive-maintenance
---

## Overview

A predictive maintenance system built on industrial IoT sensor data that flags equipment failure risk 48 hours before occurrence — drawing on my mechanical engineering background to engineer physically meaningful features.

## Approach

- **Domain-informed features**: Vibration FFT components, thermal gradients, duty-cycle-normalized wear metrics — derived from first-principles mechanical analysis
- **Modeling**: LightGBM ensemble with calibrated probabilities; Shapley values for interpretability
- **Pipeline**: Airflow-orchestrated daily retraining on rolling 90-day window
- **Alerting**: Threshold-based alerting via PagerDuty with confidence scores and SHAP explanations

## Key Outcomes

- **87% precision, 91% recall** on held-out test set
- Estimated **$2.4M/year** in avoided unplanned downtime at pilot customer
- Physics-informed features improved AUC by **+0.09** vs. raw sensor features alone

## Tech Stack

`Python` `LightGBM` `SHAP` `scikit-learn` `Apache Airflow` `InfluxDB` `Pandas` `NumPy` `SciPy`
