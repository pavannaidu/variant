# JSON Data Processing in Databricks

> **⚠️ Disclaimer**: This code was developed using Cursor and AI assistance. Please review (human) it thoroughly before using in production workloads.

Two approaches for handling JSON data: `parse_json` (VARIANT) and `from_json` with schema evolution.

## parse_json (VARIANT)

The `parse_json` function returns a `VARIANT` value, providing flexible storage for semi-structured data without strict schema requirements.

### Key Benefits
- **8x Performance Improvement** over JSON strings
- **Binary Encoding** for faster access
- **Open Source** (Spark 4.0+ and Delta 4.0+)
- **Flexible Schema** - no need to define schemas for evolving data

### Usage

```sql
-- Convert JSON to Variant
SELECT PARSE_JSON('{"id": 123, "name": "John"}') as variant_col;

-- Create table with Variant
CREATE TABLE my_table (
  id BIGINT,
  data VARIANT
);

-- Query Variant data
SELECT data.key FROM my_table;
SELECT data:field.subfield FROM my_table;  -- Nested access
SELECT data:array[0] FROM my_table;        -- Array access
```

### When to Use VARIANT
- Data doesn't need strict schematization
- Schema changes frequently
- You want maximum flexibility
- Type mismatches shouldn't cause failures

## from_json with Schema Evolution

The `from_json` function parses JSON and returns a struct value with automatic schema inference and evolution (requires Lakeflow Declarative Pipelines).

### Key Features
- **Automatic Schema Inference** - detects new fields automatically
- **Schema Evolution** - accommodates new fields without breaking
- **Type Mapping** - maps to appropriate Spark data types
- **Performance Optimization** - better storage and query performance

### Usage

```sql
-- Automatic schema inference and evolution
from_json(jsonStr, NULL, map("schemaLocationKey", "<uniqueKey>"))

-- Fixed schema (any Databricks environment)
from_json(jsonStr, schema)
```

### Schema Evolution Modes

| Mode | Behavior |
|------|----------|
| `addNewColumns` | Stream fails, adds new columns to schema |
| `rescue` | Never evolves schema, stores new columns in rescue column |
| `failOnNewColumns` | Stream fails on new columns |
| `none` | Ignores new columns |

### When to Use from_json
- You want strict schema enforcement
- Need optimized storage and query performance
- Want to fail on type mismatches
- Need to extract partial results from corrupted records

## Quick Comparison

| Function | Best For | Availability |
|----------|----------|--------------|
| `parse_json` | Flexible data, changing schemas, no type enforcement | All Databricks environments |
| `from_json` | Strict schemas, type enforcement, optimized performance | Lakeflow Declarative Pipelines only |

## When to Choose

- **Use `parse_json` (VARIANT)**: Semi-structured data, frequent schema changes, maximum flexibility
- **Use `from_json`**: Strict schemas, type enforcement, optimized storage/performance

## Execution Environment

This project uses **Databricks Connect** as the execution environment, which allows you to:
- Run Databricks code locally in your development environment
- Connect to remote Databricks clusters for execution
- Develop and test Databricks applications using your local IDE
- Leverage Databricks' distributed computing capabilities while maintaining a local development workflow

Databricks Connect bridges your local development environment with Databricks' cloud infrastructure, enabling seamless development and testing of data processing workflows.

## References

- [from_json Schema Evolution](https://docs.databricks.com/aws/en/dlt/from-json-schema-evolution)
- [Variant Data Type](https://www.databricks.com/blog/introducing-open-variant-data-type-delta-lake-and-apache-spark)

