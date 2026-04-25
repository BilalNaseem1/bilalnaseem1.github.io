---
layout: page
title: ML Model Serving Platform
description: End-to-end ML deployment platform with automated retraining, A/B testing, and drift monitoring — serving 10K+ predictions/second with p99 latency under 50ms.
img: assets/img/3.jpg
importance: 2
category: machine-learning
featured: true
github: https://github.com/BilalNaseem1/ml-serving-platform
---

## Overview

A self-contained ML platform that takes a model from experiment to production — including versioning, canary deployments, online feature serving, and automated drift alerts.

## Components

- **Model Registry**: MLflow for experiment tracking, artifact storage, and model versioning
- **Serving Layer**: FastAPI endpoints containerized with Docker, deployed on AWS ECS with autoscaling
- **Feature Store**: Redis-backed online feature store with pre-computed offline features in S3
- **A/B Testing**: Traffic splitting at the API gateway layer with statistical significance tracking
- **Monitoring**: Feature drift detection with Evidently AI; automated retraining triggers on p-value threshold

## Key Outcomes

- Reduced model deployment time from **2 weeks to 4 hours**
- Caught **3 silent model degradation events** via drift monitoring before business impact
- **p99 latency under 50ms** at 10K req/s with horizontal pod autoscaling

## Tech Stack

`Python` `FastAPI` `MLflow` `Docker` `AWS ECS` `Redis` `Evidently AI` `Weights & Biases` `PyTorch` `scikit-learn`
