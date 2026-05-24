---
layout: page
title: I Built a Stock Price Tracker with yfinance
description: "A daily Airflow pipeline for 5 US stocks — and the scheduling concept every tutorial glosses over."
img: assets/img/projects/airflow-stock-tracker/cover.png
importance: 2
tags: [data-engineering, airflow, docker, python, postgres, yfinance]
featured: true
github: https://github.com/BilalNaseem1/airflow-projects
medium: https://medium.com/@bilalnaseem19/i-built-a-stock-price-tracker-with-yfinance-heres-what-logical-date-and-idempotent-upserts-actually-mean
---

*Airflow · Docker · Postgres · yfinance · 5 stocks · daily pipeline*

---

In Project 00 I built a weather pipeline and learned the basics: sensors, XComs, hooks, connections. Everything ran on `@hourly` with `catchup=False`. I never had to think about time.

Project 01 forced me to think about time. A lot. The stock price tracker runs on a `@daily` schedule with `catchup=True` — and the moment I turned it on, I realized I had no idea what Airflow actually means when it says "daily."

This article is about that confusion and how I resolved it. Along the way we build a full pipeline: 5 stocks, daily OHLCV data, idempotent upserts, weekend skipping, and automatic historical backfill.

---

## The Pipeline

{% include figure.html path="assets/img/projects/airflow-stock-tracker/pipeline-tasks.png" caption="The complete pipeline — create_table → check_market_open → fetch_prices → validate_prices → store_prices" class="img-fluid rounded" zoomable=true %}

---

## The Concept That Changes Everything: `logical_date`

Every Airflow tutorial I read mentioned `logical_date` in passing. Most called it `execution_date` (the old name) and moved on. None of them explained why it's different from when the task actually runs — and that difference is the entire mental model.

**The Newspaper Analogy:** A newspaper covers the events of Monday. But it can only be printed and delivered after Monday ends — so it arrives on Tuesday morning. The newspaper's content date is Monday. Its delivery date is Tuesday. In Airflow: `logical_date` = content date. Actual run time = delivery date.

{% include figure.html path="assets/img/projects/airflow-stock-tracker/logical-date-timeline.png" caption="A @daily run fires AFTER the interval ends — always one day behind its logical_date" class="img-fluid rounded" zoomable=true %}

In concrete terms: with `start_date = 2026-05-01` and `schedule="@daily"`, the first run fires at May 2, 00:00 UTC. Its `logical_date` is May 1. It processes May 1's data.

> **The Classic Mistake:** Never use `datetime.now()` inside a DAG task. If you do, backfilling breaks completely — every historical run would fetch today's data instead of its own day's data. Always use `logical_date`.

```python
# Correct — uses the data interval date
def fetch_prices(trade_date: str):
    start = trade_date  # "2026-05-01" — from logical_date via XCom
    end = str((pendulum.parse(trade_date) + timedelta(days=1)).date())
    df = yf.download(tickers=STOCKS, start=start, end=end)
```

---

## catchup vs backfill — Two Ways to Load History

Both load historical data. They work differently.

{% include figure.html path="assets/img/projects/airflow-stock-tracker/catchup-vs-backfill.png" caption="catchup fills history automatically on unpause · backfill gives you manual control over any custom range" class="img-fluid rounded" zoomable=true %}

| | catchup | backfill |
|---|---|---|
| Triggered by | Airflow automatically | You manually via CLI |
| When | On DAG unpause | Whenever you want |
| Range | `start_date` → now | Any range you specify |
| Use case | "Always keep up to date" | "Fix this specific range" |

In our pipeline, `catchup=True` meant that the moment I unpaused the DAG, Airflow immediately started firing runs for every trading day since May 1st. 23 days of history loaded automatically — I wrote zero extra code.

---

## Writing the DAG

### Task 1: `create_table` — idempotency from the start

The first task creates the `stock_prices` table. The important detail isn't that it creates a table — it's the `UNIQUE` constraint:

```sql
CREATE TABLE IF NOT EXISTS stock_prices (
    id         SERIAL PRIMARY KEY,
    symbol     TEXT NOT NULL,
    open       NUMERIC,
    high       NUMERIC,
    low        NUMERIC,
    close      NUMERIC,
    volume     BIGINT,
    price_date DATE NOT NULL,
    fetched_at TIMESTAMP NOT NULL DEFAULT now(),
    CONSTRAINT uq_symbol_date UNIQUE (symbol, price_date)
);
```

`CONSTRAINT uq_symbol_date UNIQUE (symbol, price_date)` means no two rows can have the same stock on the same date. This is the foundation of idempotency — we'll use it in `store_prices` to make re-runs safe.

### Task 2: `check_market_open` — skipping is not failing

US markets are closed on weekends. yfinance returns empty data for those days silently — no error, just nothing. Without a guard, we'd store empty rows and wonder why our data had gaps.

My first instinct was to raise a `ValueError`. That was wrong. `ValueError` marks the task as failed, triggers 3 retries, and fires `on_failure_callback`. You'd get paged at 2am because it's Saturday.

{% include figure.html path="assets/img/projects/airflow-stock-tracker/skip-vs-fail.png" caption="ValueError pages you at 2am on Saturday — AirflowSkipException silences it as intended" class="img-fluid rounded" zoomable=true %}

```python
from airflow.exceptions import AirflowSkipException

@task
def check_market_open(logical_date=None):
    weekday = logical_date.day_of_week  # 0=Monday, 6=Sunday
    if weekday >= 5:
        raise AirflowSkipException(
            f"Market closed on {logical_date.date()} — skipping."
        )
    return str(logical_date.date())  # passes date forward via XCom
```

### Task 3: `fetch_prices` — XCom in practice

`check_market_open` returns a date string. `fetch_prices` receives it as a parameter. Airflow handles the transfer automatically via XCom — the metadata database acts as the bridge between tasks running in complete isolation.

{% include figure.html path="assets/img/projects/airflow-stock-tracker/xcom-flow.png" caption="XComs make data flow look like regular Python function calls — Airflow handles the transfer" class="img-fluid rounded" zoomable=true %}

> **XCom Size Limit:** XComs are stored in the metadata database. They are designed for small values only — a string, a number, a file path. We write prices to a CSV file and pass only the path through XCom. Never push a DataFrame, a large list, or binary data through XCom.

### Task 5: `store_prices` — idempotent upsert

This is where idempotency is enforced at the database level. The SQL does the heavy lifting:

```sql
INSERT INTO stock_prices (symbol, open, high, low, close, volume, price_date)
VALUES (%s, %s, %s, %s, %s, %s, %s)
ON CONFLICT (symbol, price_date)
DO UPDATE SET
    open       = EXCLUDED.open,
    high       = EXCLUDED.high,
    low        = EXCLUDED.low,
    close      = EXCLUDED.close,
    volume     = EXCLUDED.volume,
    fetched_at = now();
```

{% include figure.html path="assets/img/projects/airflow-stock-tracker/upsert-idempotency.png" caption="Re-running the same DAG run never duplicates data — ON CONFLICT DO UPDATE handles it at the SQL level" class="img-fluid rounded" zoomable=true %}

---

## Wiring: Implicit vs Explicit Dependencies

```python
# implicit — inferred from XCom data flow
date      = check_market_open()   # must run first
file      = fetch_prices(date)    # must run after check_market_open
validated = validate_prices(file)
store_prices(validated)

# explicit — no data flows, order still matters
create_table >> date  # table must exist before we try to write
```

Prefer implicit wherever data flows between tasks — they stay in sync with the code automatically.

---

## The Mistakes I Made

**1. `ValueError` for weekends** — Every weekend run showed red, triggered 3 retries each, and fired `on_failure_callback`. The fix was `AirflowSkipException` — one import, one word change, all weekend runs went from red to pink.

**2. Wiring inside a task function** — I accidentally indented the wiring code inside `store_prices`. Python treated it as part of the function body — it never executed during DAG parsing. Only `create_table` ran, nothing else. The fix was pure indentation — move the wiring to the DAG body level.

**3. Wrong import path for Airflow 3.x** — `airflow.providers.postgres.operators.postgres` no longer exists in newer provider versions. The operator moved to `airflow.providers.common.sql.operators.sql`. The DAG showed a red import error in the UI until I fixed this.

**4. Wrong credentials in the Airflow Connection** — I set the connection login as `postgres` but my infra `.env` had `POSTGRES_USER=admin`. Every `create_table` run failed with "password authentication failed." The task log was the only place the actual error appeared — always check the task log first, not the container logs.

---

## What This Pipeline Taught Me

- **`logical_date` is not execution time.** Airflow processes the data interval after it ends. Always pass `logical_date` to external APIs — never `datetime.now()`.
- **`catchup=True` is backfilling for free.** Unpause the DAG and history loads automatically. Zero extra code. The value of getting the mental model right.
- **Skipping is not failing.** `AirflowSkipException` exists for expected conditions. `ValueError` is for unexpected errors. The distinction matters for on-call noise.
- **Idempotency is a database concern.** `ON CONFLICT DO UPDATE` makes re-runs safe at the SQL level — no application logic needed.
- **The task log is your best friend.** Container logs show infrastructure noise. Task logs show exactly what your code did. Always check task logs first.
- **Indentation is logic.** In Python, where you put code determines when it runs. Wiring inside a function body is dead code during DAG parsing.
