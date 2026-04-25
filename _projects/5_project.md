---
layout: page
title: Large-Scale NLP Classification Pipeline
description: Distributed text classification system fine-tuning BERT on 10M+ documents using Spark NLP — processing a full corpus in under 3 hours on a 10-node cluster.
img: assets/img/1.jpg
importance: 5
category: machine-learning
github: https://github.com/BilalNaseem1/nlp-pipeline
---

## Overview

A scalable NLP pipeline that classifies large document corpora using fine-tuned BERT models, distributed with Spark NLP and served as a REST API with batching support.

## Architecture

- **Training**: HuggingFace Transformers with mixed-precision fine-tuning on GPU
- **Distributed inference**: Spark NLP on EMR cluster — 10x throughput vs. single-node
- **Serving**: FastAPI with request batching for throughput optimization
- **Active learning**: Uncertainty sampling to surface high-value labeling candidates

## Key Outcomes

- Processes **10M+ documents in under 3 hours** on a 10-node EMR cluster
- Fine-tuned BERT outperforms TF-IDF baseline by **+18% F1** on domain corpus
- Active learning loop reduced annotation cost by **40%** for custom label categories

## Tech Stack

`Python` `HuggingFace Transformers` `Spark NLP` `AWS EMR` `FastAPI` `Docker` `PyTorch` `scikit-learn`
