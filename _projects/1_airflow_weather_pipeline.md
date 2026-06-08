---
layout: page
title: Airflow in Practice - Docker Networking, XComs, and the Decisions Nobody Explains
description: "A real-world Karachi weather pipeline: Docker, Airflow, Postgres, and every mistake along the way."
img: assets/img/projects/airflow-weather-pipeline/cover.jpg
importance: 1
tags: [data-engineering, airflow, docker, python, postgres]
featured: true
github: https://github.com/BilalNaseem1/airflow-projects
medium: https://medium.com/@bilalnaseem19/i-built-my-first-data-pipeline-heres-what-nobody-tells-you-337bfce5ca47
---

*Airflow · Docker · Postgres · Open-Meteo · Karachi weather · 5 tasks · 1 hour*

---

I spent weeks learning Airflow concepts. DAGs, tasks, operators, sensors. I could explain them. But I hadn't built anything real yet. So I decided to build a live weather data pipeline for Karachi — and document every single thing that confused me.

All code for this pipeline is available on [GitHub](https://github.com/BilalNaseem1/airflow-projects). The full stack runs with three commands: `docker network create shared-db`, `docker compose up -d` in `infra/`, and `docker compose up -d` in `weather-pipeline/`.

This isn't a tutorial that starts from a perfect setup. This is what it actually looks like to go from zero to a working data pipeline: Docker networking errors, authentication failures, XCom confusion, and the moment everything finally clicked.

By the end of this article you'll understand how a real data pipeline is structured, what Airflow actually does under the hood, and why data engineers think the way they do.

{% include figure.html path="assets/img/projects/airflow-weather-pipeline/pipeline-tasks.png" caption="The complete pipeline — five tasks, five concepts" class="img-fluid rounded" zoomable=true %}

---

## The architecture decision nobody teaches

Before writing a single line of Python, I had to decide how to run Airflow locally. The internet is full of tutorials that hand you a single `docker-compose.yml` and say "run this." I wanted to understand the choices.

My first instinct was to run everything in one Docker Compose file. Simple, fast. But then I thought about the future: what if I want to build a second pipeline next month? Do I copy-paste the entire Postgres setup? Spin up a new database?

The answer was to separate concerns from day one: **one Compose file for infrastructure** (Postgres + pgAdmin) and **one Compose file per project** (Airflow). Both connect via a shared Docker network.

{% include figure.html path="assets/img/projects/airflow-weather-pipeline/docker-network.png" caption="Both stacks share one network — Airflow can reach Postgres by container name" class="img-fluid rounded" zoomable=true %}

The key insight: `docker network create shared-db` creates a network that lives at the Docker engine level — outside any individual Compose file. Any container that joins it can reach any other container by name. `shared-postgres` becomes a hostname.

> **The gotcha nobody warns you about:** If `external: true` is set in your Compose file and the network doesn't exist yet, Docker won't create it for you — it fails with a cryptic error. Create the network manually once before starting anything: `docker network create shared-db`

---

## What a data engineer actually does vs DevOps

Here's something that took me an embarrassingly long time to realise: everything I'd done to this point — Docker setup, networking, Airflow installation, connection configuration — is **infrastructure work**. At a real company, the platform team handles all of this.

A data engineer joining a company would get: "Here's the Airflow URL, here's your login, the connection is already configured, go write DAGs." The infrastructure exists. Your job is the pipeline logic.

Understanding the infrastructure anyway makes you a far better pipeline author. You understand why connections work the way they do, why networks matter, why credentials don't live in your code. But know the distinction.

---

## Writing the DAG — five tasks, five concepts

### Task 1: create_table — idempotency matters more than you think

The first task creates the database table. Simple enough. But there's a critical detail: `CREATE TABLE IF NOT EXISTS`, not just `CREATE TABLE`.

This is called **idempotency** — the property of an operation that produces the same result whether you run it once or a hundred times. In Airflow, tasks get retried. Pipelines get backfilled. Code that isn't idempotent causes silent data corruption when tasks run more than once.

```python
create_table = SQLExecuteQueryOperator(
    task_id="create_table",
    conn_id="postgres",   # reference, not credentials
    sql="""
        CREATE TABLE IF NOT EXISTS weather_readings (
            id           SERIAL PRIMARY KEY,
            city         TEXT NOT NULL,
            temperature  NUMERIC,
            recorded_at  TIMESTAMP
        );
    """
)
```

Notice `conn_id="postgres"` — this is just a string key. The actual hostname, username, and password live in Airflow's metadata database, stored through the UI. Your code never contains credentials. Deploy the same DAG to dev, staging, and production by changing the Connection, not the code.

Also notice: no `@task` decorator. `SQLExecuteQueryOperator` is already a task class — the "task-ness" is baked in. The `@task` decorator is only needed when you're wrapping your own Python functions.

### Task 2: is_api_available — why you always check before you work

Before pulling weather data, I check the API is actually reachable. If I skip this and the API is down, the extract task fails with a confusing connection error buried in a stack trace. A sensor fails cleanly with a clear message.

A sensor is a special task whose only job is to wait for a condition. It checks repeatedly at a configurable interval and only lets downstream tasks proceed when the condition is met.

> **Always set timeout.** Without it, a permanently unavailable API leaves the sensor in a running state forever, occupying a worker slot and potentially blocking other pipelines. I learned this from the docs — a 5-minute timeout is a safe default for an external API check.

```python
@task.sensor(poke_interval=30, timeout=300)
def is_api_available() -> PokeReturnValue:
    response = requests.get(WEATHER_URL)
    condition = response.status_code == 200
    return PokeReturnValue(
        is_done=condition,
        xcom_value=response.json()  # pass data forward
    )
```

The `xcom_value` field is elegant: the sensor already fetched the data to check if the API works. Instead of throwing that data away and fetching again in the next task, we pass it forward. No duplicate API calls.

### Tasks 3 & 4: XComs — how tasks share data

This was the concept that took me the longest to internalise. Tasks in Airflow run in complete isolation — different processes, sometimes different machines. They cannot share Python variables directly.

XCom (cross-communication) is Airflow's solution. It's a small storage table in the metadata database. When a `@task` function returns a value, Airflow saves it. When another `@task` function receives it as a parameter, Airflow fetches it automatically.

{% include figure.html path="assets/img/projects/airflow-weather-pipeline/xcom-flow.png" caption="XComs make data flow look like regular Python function calls" class="img-fluid rounded" zoomable=true %}

```python
raw          = is_api_available()    # raw is an XCom reference
weather_data = extract_weather(raw)  # Airflow injects at runtime
```

> **XCom warning:** XComs are stored in the metadata database. They are designed for small values — a dictionary, a file path, a count. Never return a DataFrame, a large list, or binary data from a `@task` function. Pass the file path instead.

### Task 5: store_weather — when to use a Hook directly

Every Airflow operator that talks to an external service uses a Hook underneath. Usually the operator wraps it so cleanly you never see it. But sometimes you need a capability the operator doesn't expose.

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

## The wiring — implicit vs explicit dependencies

This was the part that confused me most. How does Airflow know what order to run the tasks?

There are two kinds of dependencies. **Implicit dependencies** come from data flow: when one task's return value is passed as a parameter to another task, Airflow infers the order automatically. **Explicit dependencies** use the `>>` operator and are needed when no data flows between tasks.

```python
# implicit — Airflow infers these from data flow
raw          = is_api_available()       # sensor must run first
weather_data = extract_weather(raw)     # extract must run after sensor

# explicit — no data flows between these pairs
process_weather(weather_data) >> store_weather()   # file transfer, not XCom
create_table >> is_api_available()                 # operator, not @task
```

{% include figure.html path="assets/img/projects/airflow-weather-pipeline/dag-dependencies.png" caption="Prefer implicit where possible — they stay in sync with code automatically" class="img-fluid rounded" zoomable=true %}

---

## The mistakes I made (so you don't have to)

**1. Port conflict on startup**

My first `docker compose up` failed because port 5432 was already in use — I had a local Postgres instance running. The fix is obvious in hindsight: use port 5433 externally, keep 5432 internally. Airflow containers talk to each other on the Docker network using the internal port anyway.

**2. Stale volumes causing auth failures**

After changing my database credentials, I kept getting "password authentication failed" errors even on fresh `docker compose up` runs. Postgres initialises from environment variables only on first startup — if a volume already exists with data, it ignores the new credentials entirely.

`docker compose down -v` wipes the volumes. Always use it when you change Postgres credentials during development.

**3. Missing network on the scheduler container**

The most frustrating one. I'd configured the `shared-db` network in my Compose file but the scheduler container wasn't joining it. The symptom: `could not translate host name "shared-postgres" to address`.

The fix was to verify which networks each container was actually on using `docker inspect`, then connect the missing containers manually with `docker network connect shared-db container-name`.

**4. Forgetting to call task functions**

The `@task` decorator defines a task but doesn't register it. You must call the function inside the DAG. This is the same rule as the DAG function itself — Airflow's file processor imports your Python file and looks for instantiated objects.

---

## The result: live weather data, every hour

With `schedule='@hourly'` and `catchup=False`, the pipeline runs once per hour and inserts a new row into `weather_readings`. After a day, you have 24 rows. After a week, 168 rows of Karachi's weather history — queryable, visualisable, real.

```sql
SELECT city, temperature, humidity, recorded_at
FROM weather_readings
ORDER BY recorded_at DESC
LIMIT 5;
```

---

## What this pipeline actually taught me

- **Idempotency isn't optional** — every task should be safe to run multiple times
- **Connections decouple credentials from code** — the same DAG deploys to any environment
- **Sensors fail cleanly** — they're far better than cryptic errors inside the task that needed the precondition
- **XComs are for small values** — identifiers, paths, counts. Never DataFrames
- **Hooks are not workarounds** — they're the intended extension point when an operator doesn't cover your case
- **Implicit dependencies are self-documenting** — prefer them over explicit `>>` wherever data flows
- **The infrastructure is DevOps — the pipeline logic is data engineering.** Know the difference
