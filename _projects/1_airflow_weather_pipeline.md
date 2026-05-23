---
layout: page
title: I Built My First Data Pipeline
description: "Airflow · Docker · Postgres · Open-Meteo · Karachi weather · 5 tasks · 1 hour"
img: assets/img/projects/airflow-pipeline-cover.jpg
importance: 1
category: data-engineering
featured: true
github: https://github.com/BilalNaseem1/airflow-projects
medium: https://medium.com/@bilalnaseem19/i-built-my-first-data-pipeline-heres-what-nobody-tells-you-337bfce5ca47
---

**DATA ENGINEERING · PROJECT 00**

A real-world Karachi weather pipeline: Docker, Airflow, Postgres, and every mistake along the way.

I spent weeks learning Airflow concepts — DAGs, tasks, operators, sensors. I could explain them. But I hadn't built anything real yet. So I decided to build a live weather data pipeline for Karachi and document every single thing that confused me.

The full stack runs with three commands: `docker network create shared-db`, `docker compose up -d` in `infra/`, and `docker compose up -d` in `weather-pipeline/`.

---

## The Architecture Decision Nobody Teaches

Before writing a single line of Python, I had to decide how to run Airflow locally. My first instinct was one Docker Compose file — simple, fast. But I thought about the future: what if I want to build a second pipeline next month?

The answer: **one Compose file for infrastructure** (Postgres + pgAdmin) and **one Compose file per project** (Airflow), connected via a shared Docker network.

```
infra/docker-compose.yml        weather-pipeline/docker-compose.yml
┌─────────────────────┐         ┌──────────────────────────────────┐
│ shared-postgres      │         │ webserver   scheduler    worker  │
│ port 5433            │         │ port 8080   parses DAGs  runs    │
│ pgadmin port 5050    │         │                          tasks   │
└────────────┬────────┘         └────────────────┬─────────────────┘
             └─────────── shared-db network ──────┘
```

The key insight: `docker network create shared-db` creates a network at the Docker engine level — outside any individual Compose file. Any container that joins it can reach any other by name. `shared-postgres` becomes a hostname.

> **Gotcha nobody warns you about:** If `external: true` is set in your Compose file and the network doesn't exist yet, Docker fails with a cryptic error. Create the network manually once before starting anything.

---

## The DAG — Five Tasks, Five Concepts

```
create_table  →  is_api_available  →  extract_weather  →  process_weather  →  store_weather
SQL Operator     Sensor                @task + XCom        CSV write            Hook + COPY
```

### Task 1: create_table — Idempotency Matters More Than You Think

```python
create_table = SQLExecuteQueryOperator(
    task_id="create_table",
    conn_id="postgres",   # reference, not credentials
    sql="""
        CREATE TABLE IF NOT EXISTS weather_readings (
            id          SERIAL PRIMARY KEY,
            city        TEXT NOT NULL,
            temperature NUMERIC,
            recorded_at TIMESTAMP
        );
    """
)
```

`CREATE TABLE IF NOT EXISTS` — not `CREATE TABLE`. This is **idempotency**: the operation produces the same result whether you run it once or a hundred times. In Airflow, tasks get retried and pipelines get backfilled. Code that isn't idempotent causes silent data corruption.

`conn_id="postgres"` is just a string key. The actual hostname, username, and password live in Airflow's metadata database. Your code never contains credentials. Deploy the same DAG to dev, staging, and production by changing the Connection, not the code.

### Task 2: is_api_available — Why You Always Check Before You Work

A sensor is a special task whose only job is to wait for a condition. It checks repeatedly at a configurable interval and only lets downstream tasks proceed when the condition is met.

```python
@task.sensor(poke_interval=30, timeout=300)
def is_api_available() -> PokeReturnValue:
    response = requests.get(WEATHER_URL)
    condition = response.status_code == 200
    return PokeReturnValue(
        is_done=condition,
        xcom_value=response.json()  # pass data forward — no duplicate API calls
    )
```

> **Always set timeout.** Without it, a permanently unavailable API leaves the sensor running forever, occupying a worker slot and blocking other pipelines. 5 minutes is a safe default for an external API check.

The `xcom_value` field is elegant: the sensor already fetched the data to check if the API works. Instead of throwing that away and fetching again in the next task, we pass it forward.

### Tasks 3 & 4: XComs — How Tasks Share Data

Tasks in Airflow run in complete isolation — different processes, sometimes different machines. They cannot share Python variables directly.

**XCom** (cross-communication) is Airflow's solution: a small storage table in the metadata database. When a `@task` function returns a value, Airflow saves it. When another `@task` function receives it as a parameter, Airflow fetches it automatically.

```python
raw          = is_api_available()       # raw is an XCom reference
weather_data = extract_weather(raw)     # Airflow injects at runtime
```

> **XCom warning:** XComs are designed for small values — a dictionary, a file path, a count. Never return a DataFrame, a large list, or binary data. Pass the file path instead.

### Task 5: store_weather — When to Use a Hook Directly

Loading a CSV into Postgres with the `COPY` command is orders of magnitude faster than row-by-row `INSERT` statements. No standard operator exposes this cleanly. `PostgresHook` has a `copy_expert` method that does exactly this.

```python
@task
def store_weather():
    hook = PostgresHook(postgres_conn_id="postgres")
    hook.copy_expert(
        sql="""
            COPY weather_readings
                (city, temperature, humidity, wind_speed, weather_code, recorded_at)
            FROM STDIN WITH CSV HEADER
        """,
        filename="/tmp/weather.csv"
    )
```

---

## The Wiring — Implicit vs Explicit Dependencies

```python
# implicit — Airflow infers these from data flow
raw          = is_api_available()       # sensor must run first
weather_data = extract_weather(raw)     # extract must run after sensor

# explicit — no data flows between these pairs
process_weather(weather_data) >> store_weather()    # file transfer, not XCom
create_table >> is_api_available()                  # operator, not @task
```

Implicit dependencies stay in sync with code automatically. Prefer them wherever data flows.

---

## The Mistakes I Made (So You Don't Have To)

**1. Port conflict on startup** — Port 5432 was already in use from a local Postgres instance. Fix: use port 5433 externally, keep 5432 internally. Airflow containers talk to each other on the Docker network using the internal port anyway.

**2. Stale volumes causing auth failures** — After changing database credentials, I kept getting "password authentication failed" errors even on fresh `docker compose up` runs. Postgres initialises from environment variables only on first startup — if a volume already exists, it ignores new credentials entirely. `docker compose down -v` wipes the volumes. Always use it when you change Postgres credentials during development.

**3. Missing network on the scheduler container** — I'd configured `shared-db` in my Compose file but the scheduler container wasn't joining it. Symptom: `could not translate host name "shared-postgres" to address`. Fix: `docker inspect` to see which networks each container was on, then `docker network connect shared-db container-name`.

**4. Forgetting to call task functions** — The `@task` decorator defines a task but doesn't register it. You must call the function inside the DAG.

---

## The Result: Live Weather Data, Every Hour

With `schedule='@hourly'` and `catchup=False`, the pipeline runs once per hour and inserts a new row into `weather_readings`. After a day: 24 rows. After a week: 168 rows of Karachi's weather history — queryable, visualisable, real.

```sql
SELECT city, temperature, humidity, recorded_at
FROM weather_readings
ORDER BY recorded_at DESC
LIMIT 5;
```

---

## What This Pipeline Actually Taught Me

- **Idempotency isn't optional** — every task should be safe to run multiple times
- **Connections decouple credentials from code** — the same DAG deploys to any environment
- **Sensors fail cleanly** — far better than cryptic errors inside the task that needed the precondition
- **XComs are for small values** — identifiers, paths, counts; never DataFrames
- **Hooks are not workarounds** — they're the intended extension point when an operator doesn't cover your case
- **Implicit dependencies are self-documenting** — prefer them over explicit `>>` wherever data flows
- **The infrastructure is DevOps — the pipeline logic is data engineering**; know the difference

---

`Airflow 3.2` · `Docker` · `PostgreSQL` · `Open-Meteo API` · `5 tasks` · `@hourly`
