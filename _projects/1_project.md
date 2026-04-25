---
layout: page
title: Real-Time Analytics Pipeline
description: High-throughput streaming pipeline processing 50M+ events/day using Kafka, Spark Streaming, and Delta Lake — with end-to-end latency under 2 minutes.
img: assets/img/12.jpg
importance: 1
category: data-engineering
featured: true
github: https://github.com/BilalNaseem1/realtime-pipeline
---

## Overview

A production-grade real-time data pipeline that ingests, transforms, and serves 50M+ events per day with sub-2-minute end-to-end latency.

## Architecture

- **Ingestion**: Apache Kafka with 12 partitions per topic for parallelism
- **Processing**: Spark Structured Streaming with stateful aggregations and watermarking
- **Storage**: Delta Lake on S3 with ACID transactions and schema enforcement
- **Serving**: Materialized views in Snowflake, refreshed every 5 minutes
- **Monitoring**: Prometheus + Grafana dashboards for lag, throughput, and error rates

## Key Outcomes

- Reduced reporting latency from **4 hours to under 2 minutes**
- Handles **3x traffic spikes** without manual intervention via auto-scaling
- **Zero data loss** over 6 months of production operation
- Schema evolution handled without downtime using Delta Lake's schema merging

## Tech Stack

`Python` `PySpark` `Apache Kafka` `Delta Lake` `Snowflake` `AWS S3` `AWS EMR` `Prometheus` `Grafana`
