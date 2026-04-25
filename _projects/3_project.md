---
layout: page
title: Enterprise Data Warehouse
description: Modular dbt-based data warehouse on Snowflake with Airflow orchestration, automated data quality checks, and full lineage documentation.
img: assets/img/7.jpg
importance: 3
category: data-engineering
featured: true
github: https://github.com/BilalNaseem1/data-warehouse
---

## Overview

A production data warehouse serving as the single source of truth for business analytics — built with dbt on Snowflake, orchestrated by Airflow, and enforced by automated data quality gates.

## Architecture

- **Modeling**: dbt with a 3-layer architecture (raw → staging → mart), 200+ models
- **Orchestration**: Apache Airflow with dynamic DAG generation for environment parity
- **Quality**: Great Expectations for schema, freshness, and distribution checks on every run
- **Lineage**: Auto-generated dbt docs + DataHub integration for full upstream/downstream visibility
- **CI/CD**: dbt Cloud CI for PR-level model testing before merge

## Key Outcomes

- Consolidated **7 disparate reporting sources** into a single governed warehouse
- Reduced analyst query time by **60%** through pre-aggregated mart models
- **100% data quality coverage** on critical tables — zero silent failures in production

## Tech Stack

`dbt` `Snowflake` `Apache Airflow` `Great Expectations` `DataHub` `Python` `SQL` `GitHub Actions`
