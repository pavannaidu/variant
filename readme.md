# JSON Data Processing 

When working with JSON data in Databricks, you have two primary approaches for handling schema evolution and data processing: `parse_json` (VARIANT) and `from_json` with schema evolution.

## parse_json (VARIANT) Approach

The `parse_json` function returns a `VARIANT` value from the JSON string, providing a flexible and efficient way to store semi-structured data. This circumvents schema inference and evolution by doing away with strict types altogether.

### What is Variant?

Variant is a new open-source data type for storing semi-structured data that provides an order of magnitude performance improvements compared with storing data as JSON strings, while maintaining flexibility for supporting highly nested and evolving schemas. 

**Key Benefits:**
- **8x Performance Improvement**: Benchmarks show 8x better performance over String columns for both nested and flat schemas
- **Binary Encoding**: Uses efficient binary encoding format for faster access and navigation compared to JSON strings
- **Open Source**: Fully open-sourced in Apache Spark and Delta Lake (available in Spark 4.0 and Delta 4.0)
- **No Vendor Lock-in**: Avoids proprietary data warehouse dependencies
- **Flexible Schema**: No need to define explicit schemas for unknown, changing, or frequently evolving data structures

### Use Cases for Variant:
- **Endpoint Detection & Response (EDR)**: Reading and combining logs with different JSON schemas
- **Ad-click Analysis**: Handling unknown and constantly changing schemas
- **IoT Telemetry**: Processing application telemetry with evolving data structures
- **Semi-structured Data**: Any scenario where JSON sources have unknown, changing, and frequently evolving schemas

### Advantages of VARIANT:
- **Flexible Schema Handling**: No strict type requirements, handles schema evolution gracefully
- **Always Succeeds**: VARIANT ingestion will always succeed for valid JSON records, even with type mismatches
- **Semi-Structured Flexibility**: Particularly well-suited for data that doesn't need strict schematization
- **Rapid Schema Changes**: Ideal when schema changes too quickly to cast into a fixed schema without frequent stream failures and restarts
- **No Type Enforcement**: Users don't need to deal with rescued data columns containing fields that don't conform to schema
- **Universal Availability**: Available with and without Lakeflow Declarative Pipelines

### How to Use Variant:

```sql
-- Convert JSON string to Variant using PARSE_JSON()
SELECT PARSE_JSON('{"id": 123, "name": "John"}') as variant_col;

-- Create table with Variant columns
CREATE TABLE my_table (
  id BIGINT,
  data VARIANT
);

-- Insert Variant data
INSERT INTO my_table VALUES (1, PARSE_JSON('{"key": "value"}'));

-- Path navigation with dot notation
SELECT data.key FROM my_table;
```

### Use Cases for VARIANT:
- Data that does not need to be schematized
- Schema changes too quickly to cast into a schema without frequent stream failures
- You don't want to fail on data with mismatched types
- Users don't want to deal with rescued data columns
- **Performance-critical applications**: When you need 8x better performance than JSON strings
- **Unknown or evolving schemas**: Perfect for data sources with unpredictable structure changes

## from_json: Schema Inference and Evolution Approach

The `from_json` SQL function parses a JSON string column and returns a struct value. When used with Lakeflow Declarative Pipelines, you can enable schema inference and evolution, which automatically manages the schema of the returned value.

### Key Features:
- **Automatic Schema Inference**: Detects new fields in incoming JSON records (including nested JSON objects)
- **Type Mapping**: Infers field types and maps them to appropriate Spark data types
- **Schema Evolution**: Automatically evolves the schema to accommodate new fields
- **Data Handling**: Automatically handles data that does not conform to the current schema
- **Schema Enforcement**: Maintains strict schema while allowing evolution
- **Performance Optimization**: Better storage optimization and lower query latency for structured data
- **Partial Data Recovery**: Can extract partial results from corrupted JSON records and store malformed records in rescue columns

### Syntax for Schema Evolution:

```sql
-- Automatic schema inference and evolution
from_json(jsonStr, NULL, map("schemaLocationKey", "<uniqueKey>" [, otherOptions]))

-- Fixed schema (available in any Databricks environment)
from_json(jsonStr, schema, [, options])
```

### Schema Evolution Modes:

| Mode | Behavior on New Column |
|------|----------------------|
| `addNewColumns` (default) | Stream fails. New columns are added to the schema. Existing columns do not evolve data types. |
| `rescue` | Schema is never evolved and stream does not fail. All new columns are recorded in the rescued data column. |
| `failOnNewColumns` | Stream fails. Stream does not restart unless schemaHints are updated or offending data is removed. |
| `none` | Does not evolve the schema, new columns are ignored, and data is not rescued unless rescuedDataColumn option is set. |

### Schema Hints:
You can provide `schemaHints` to influence how `from_json` infers column types:

```sql
-- Treat 'a' as STRING instead of inferred BIGINT
from_json(data, NULL, map('schemaLocationKey', 'x', 'schemaHints', 'a STRING'))

-- Treat nested object as MAP
from_json(data, NULL, map('schemaLocationKey', 'y', 'schemaHints', 'a MAP<STRING, BIGINT>'))
```

### Use Cases for from_json:
- You want to enforce your data schema (reviewing every schema change before persisting)
- You want to optimize storage and require low query latency and cost
- You want to fail on data with mismatched types
- You want to extract partial results from corrupted JSON records and store malformed records in the `_corrupt_record` column

## Comparison Summary

| Function | Use Cases | Availability |
|----------|-----------|--------------|
| `from_json` | Schema evolution maintains the schema. Enforce data schema, optimize storage, fail on type mismatches, extract partial results from corrupted records | Available with schema inference & evolution only in Lakeflow Declarative Pipelines |
| `parse_json` | VARIANT is well-suited for data that doesn't need schematization. Flexible data, rapidly changing schemas, no type enforcement needed | Available with and without Lakeflow Declarative Pipelines |

## Choosing Between Approaches

- **Use `parse_json` (VARIANT)** when: Data is truly semi-structured, schema changes frequently, you want maximum flexibility, or type mismatches should not cause failures
- **Use `from_json`** when: You have relatively strict schemas, need type enforcement at write time, want to optimize storage and query performance, or have legacy pipelines expecting fixed types

## Important Notes

- `from_json` schema inference and evolution syntax is only available in Lakeflow Declarative Pipelines
- Each `from_json` expression must have a unique `schemaLocationKey` per pipeline
- Schema locations are cleared and re-inferred from scratch if the table is fully refreshed
- Downstream queries referring to `from_json` fields may be skipped until the function executes successfully at least once
- **Future Enhancement**: Shredding/sub-columnarization for Variant type is planned to improve performance of querying specific paths within Variant data

## Execution Environment

This notebook was executed using **Databricks Connect**, which allows you to run Databricks notebooks locally while connecting to a remote Databricks cluster. The notebook includes a robust fallback mechanism to handle both Databricks Connect and local Spark environments:

```python
def get_spark() -> SparkSession:
    try:
        from databricks.connect import DatabricksSession
        return DatabricksSession.builder.getOrCreate()
    except Exception:
        return SparkSession.builder.getOrCreate()
```

### Databricks Connect Benefits:
- **Local Development**: Develop and test notebooks locally using your preferred IDE (VS Code, PyCharm, etc.)
- **Remote Execution**: Execute code on remote Databricks clusters without leaving your local environment
- **Seamless Integration**: Access Unity Catalog, volumes, and all Databricks features from local notebooks
- **Version Control**: Better integration with Git workflows for notebook development
- **Real-time Debugging**: Debug and iterate on code locally while leveraging cloud compute resources
- **Cost Efficiency**: Develop locally and only use cluster resources when needed

## References

- [Infer and evolve the schema using from_json in Lakeflow Declarative Pipelines](https://docs.databricks.com/aws/en/dlt/from-json-schema-evolution#what-is-the-difference-between-from_json-and-parse_json)
- [Introducing the Open Variant Data Type in Delta Lake and Apache Spark](https://www.databricks.com/blog/introducing-open-variant-data-type-delta-lake-and-apache-spark)
- [Databricks Connect Documentation](https://docs.databricks.com/dev-tools/databricks-connect.html)

