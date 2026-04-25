---
layout: page
title: Automated Data Quality Framework
description: Open-source-inspired data quality and observability framework with rule-based checks, anomaly detection, and Slack alerting — integrated into CI/CD for zero-trust data pipelines.
img:
importance: 6
category: data-engineering
github: https://github.com/BilalNaseem1/data-quality-framework
---

## Overview

A lightweight data quality framework that sits between raw ingestion and analytics serving — enforcing schema contracts, freshness SLAs, distribution checks, and referential integrity on every pipeline run.

## Components

- **Rules engine**: Declarative YAML-based quality rules compiled to SQL assertions
- **Anomaly detection**: Statistical z-score and IQR-based checks on numeric column distributions
- **Freshness monitoring**: Table-level SLA enforcement with configurable alert thresholds
- **Reporting**: HTML quality reports generated per run, stored in S3
- **Alerting**: Slack + PagerDuty integration with severity tiering (warning vs. critical)

## Key Outcomes

- Caught **12 silent data quality issues** in the first month of production deployment
- Reduced time-to-detection on data incidents from **hours to minutes**
- Framework adopted by 3 other internal teams as a standard dependency

## Tech Stack

`Python` `dbt tests` `Great Expectations` `SQL` `Apache Airflow` `AWS S3` `Slack API` `YAML`
