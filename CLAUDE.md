# CLAUDE.md

## Project Overview

**Variant** is an educational/demonstration project for semi-structured data processing in Databricks using the VARIANT data type and Spark Declarative Pipelines (SDP). It showcases schema evolution, medallion architecture, and flexible JSON querying patterns.

**Stack:** Python, SQL, PySpark, Databricks (Delta Lake, Unity Catalog, Asset Bundles)

## Repository Structure

```
variant/
├── databricks.yml              # Databricks Asset Bundle config (variables, targets)
├── deployment.md               # Deployment guide and CLI commands
├── readme.md                   # Main docs: VARIANT type, parse_json vs from_json
├── resources/
│   ├── images/pipeline.png     # Architecture diagram
│   └── pipelines.yml           # SDP pipeline definition (Photon, serverless)
└── src/
    ├── variant.ipynb           # Standalone VARIANT exploration notebook
    └── SDP/
        ├── faker.ipynb         # Fake data generator (3-phase schema evolution)
        ├── pipeline.ipynb      # SDP pipeline: bronze → silver → gold
        └── readme.md           # Step-by-step demo instructions
```

## Key Entry Points

- `src/variant.ipynb` — Interactive VARIANT data type exploration (SQL queries, field access, null handling)
- `src/SDP/faker.ipynb` — Generates phased test data with evolving schemas (1,000 records across 3 phases)
- `src/SDP/pipeline.ipynb` — Medallion architecture pipeline with data quality expectations

## Build & Deploy

This project uses **Databricks Asset Bundles (DAB)** — no npm/pip/make. All commands require the Databricks CLI.

```bash
databricks bundle validate          # Validate config
databricks bundle deploy            # Deploy to workspace
databricks bundle run <pipeline>    # Execute a pipeline
databricks bundle destroy           # Clean up deployment
```

Configuration variables in `databricks.yml`: `workspace_url`, `catalog`, `schema`, `volume`.

## Architecture

**Medallion pattern (Bronze → Silver → Gold):**

1. **Bronze** (`users_bronze`) — Raw JSON ingested via `from_json()` with schema evolution (`addNewColumns`, schema hints, `rescue` mode)
2. **Silver** (`users_silver`) — Flattened columns, data quality expectations enforced via `@dp.expect_or_drop`
3. **Gold** — Aggregation tables (`user_stats_by_occupation`, `user_geographic_summary`)

## Code Conventions

- **Config constants** at top of notebooks: `CATALOG`, `SCHEMA`, `VOLUME`
- **Spark session** obtained via `DatabricksSession.builder.getOrCreate()` with fallback
- **VARIANT field access** in SQL: `column:field`, `column:field.subfield`, `column:array[index]`
- **SDP tables** use decorator pattern: `@dp.table()` + `@dp.expect_or_drop()`
- **Data generation** uses `faker==23.0.0`, installed per-notebook via `%pip install`
- **No formal test suite** — validation is done through SDP expectations and manual notebook runs

## Testing & Validation

No automated test framework. Data quality is enforced by SDP expectations:

```python
@dp.expect_or_drop("valid_age", "age IS NOT NULL AND age BETWEEN 0 AND 120")
@dp.expect_or_drop("valid_email", "email IS NOT NULL AND email LIKE '%@%'")
```

To validate changes, run the notebooks in order: `faker.ipynb` → `pipeline.ipynb` → inspect gold tables.

## No Linting/Formatting

No linters or formatters are configured. Follow existing notebook style: clear docstrings, consistent indentation, SQL comments with arrow notation (→).
