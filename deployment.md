# Databricks Asset Bundle Deployment

Deploy and run the Spark Declarative Pipelines using Databricks Asset Bundles.

## Prerequisites

### 1. Configure Variables in `databricks.yml`

Edit the `databricks.yml` file and set your target environment variables:

```yaml
variables:
  workspace:
    description: "Workspace URL"
    default: https://e2-demo-field-eng.cloud.databricks.com  # Change to your workspace
  catalog:
    description: "Target catalog for pipeline tables"
    default: pavan_naidu  # Change to your catalog
  schema:
    description: "Target schema for pipeline tables"
    default: json  # Change to your schema
  volume:
    description: "Volume name for raw data"
    default: raw_data  # Change to your volume
```

### 2. Ensure Resources Exist in Databricks

Before deploying, verify these resources exist:
- **Catalog**: The catalog specified in your variables (e.g., `pavan_naidu`)
- **Schema**: The schema within that catalog (e.g., `json`)
- **Volume**: The volume with source data (e.g., `pavan_naidu.json.raw_data`)
- **Source Directory**: Directory in volume containing JSON files (e.g., `users_stream`)

### 3. Databricks CLI

- Databricks CLI installed and configured with authentication to your workspace

## Quick Start

```bash
# Validate configuration
databricks bundle validate

# Deploy and run
databricks bundle deploy && databricks bundle run user_schema_evolution_pipeline
```

## Project Structure

```
variant/
├── databricks.yml           # Bundle config (workspace, variables)
├── deployment.md            # Deployment guide
├── readme.md                # Project overview
├── resources/
│   └── pipelines.yml        # SDP pipeline definition
└── src/
    ├── SDP/
    │   ├── faker.ipynb      # Data generator for pipeline.ipynb
    │   └── pipeline.ipynb   # SDP pipeline notebook
    └── variant.ipynb        # Variant demo / examples
```

## Configuration

**Variables** (defined in `databricks.yml`):
- `catalog`: Target catalog (default: `pavan_naidu`)
- `schema`: Target schema (default: `json`)
- `volume`: Source volume (default: `raw_data`)

**Pipeline** (`resources/pipelines.yml`):
- Name: `[dev] User Schema Evolution Pipeline`
- Source: `/Volumes/pavan_naidu/json/raw_data/users_stream`
- Target: `pavan_naidu.json`
- Mode: Serverless, Photon enabled, triggered (batch)

## Commands

### Deploy
```bash
databricks bundle deploy
```
Uploads notebook and creates/updates the pipeline.

### Run
```bash
databricks bundle run user_schema_evolution_pipeline
```
### Override Variables
```bash
databricks bundle deploy --var catalog=my_catalog --var schema=my_schema
```

## Clean Up

```bash
databricks bundle destroy
```
⚠️ Deletes pipeline configuration (not tables)

## References

See `readme.md` for details on `parse_json` (VARIANT) vs `from_json` approaches.